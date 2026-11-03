# zDevOps GitHub Actions – z/OS Build Pipeline

This composite GitHub Action automates the zDevOps build, packaging, and publishing workflow on z/OS USS using IBM Dependency Based Build (DBB) 3.0.5.

## Workflow Overview

1. **Initialize:** Clean runner logs and prepare the z/OS USS workspace.
2. **Clone repositories:** Clone the application branch and optionally the shared Common repository.
3. **Version package:** Generate an artifact version using the first seven characters of the Git commit SHA.
4. **Refresh configuration (Admin only):** Copy build, deploy, and script configurations from the Admin repository.
5. **Build and publish:** Execute the DBB pipeline to build, package, and publish artifacts to Artifactory.
6. **Prepare code scanning:** Convert the DBB build list from IBM-1047 to UTF-8 for IBM zCodeScan.
7. **Collect logs:** Retrieve the build list and DBB logs to the GitHub Actions runner.

## GitHub Actions Composite Action

The following code implements the main pipeline steps.

```yaml
runs:
  using: composite
  steps:
    # Initialize runner logs and z/OS workspace
    - name: Init Runner and zDevOps test script env var
      shell: powershell
      run: |
        Get-ChildItem -Path "jobLogs_*" -Directory |
          Remove-Item -Recurse -Force -ErrorAction SilentlyContinue
        New-Item -Path "${{env.jobLogs}}" -ItemType Directory -Force |
          Out-Null

    - name: Init USS workSpace and logs
      shell: powershell
      run: ssh ${{inputs.zOS}} 'ls -t /tmp/_zBuilds | tail -n +2 | xargs -I{} rm -fr "/tmp/_zBuilds/{}"; mkdir -p ${{env.workSpace}}; df -P tmp'

    # Clone application and optional shared repositories
    - name: Git clone on zOS USS
      shell: powershell
      run: ssh ${{inputs.zOS}} '${{inputs.zProfile}}; git clone -b ${{github.ref_name}} ${{env.myRepoURL}} ${{env.workSpace}}/${{env.appDirName}}'

    - name: Git clone Common repository
      if: ${{ env.CommonURL != '' }}
      shell: powershell
      run: ssh ${{inputs.zOS}} '${{inputs.zProfile}}; git clone -b feature/v1.0.1/new-folder-structure ${{env.CommonURL}} ${{env.workSpace}}/${{env.appDirName}}/shared-copybook'

    # Create a version based on the Git commit
    - name: Get Git Commit ID to version a package
      shell: powershell
      run: |
        $short = $env:GITHUB_SHA.Substring(0, 7)
        "ART_VER=1.0.$short" >> $env:GITHUB_ENV
        Write-Host "ART_VER=1.0.$short"

    # Admin workflows only: refresh shared zDevOps configuration
    - name: Refresh zDevOps test scripts
      shell: powershell
      run: ssh ${{inputs.zOS}} 'mkdir -p ${{inputs.zDO}}; cd ${{env.workSpace}}/${{env.appDirName}}/conf; cp -r build deploy scripts ${{inputs.zDO}}; chmod +x ${{inputs.zDO}}/scripts/*.sh; chmod +x ${{inputs.zDO}}/build/*.sh; ls -lasT ${{inputs.zDO}}'

    # Build, package, and publish using IBM DBB
    - name: zBuild/Package/Publish (IBM DBB 3.0.5)
      shell: powershell
      run: ssh ${{inputs.zOS}} '${{inputs.zProfile}}; cd ${{env.workSpace}}/${{env.appDirName}}; dbb build pipeline --verbose --artf conf/build/jfrogPat --release-id ReleaseID_tag123 --build-id BuildID_${{github.run_number}}-${{env.ART_VER}} --hlq ibmuser.zbuilder'

    # Convert the build list for zCodeScan
    - name: Prep the DBB BuildList for IBM zCodeScan
      shell: powershell
      run: ssh ${{inputs.zOS}} '${{inputs.zProfile}}; cd ${{env.workSpace}}/${{env.appDirName}}; iconv -f IBM-1047 -t UTF-8 logs/buildList.txt > logs/buildList_utf8.txt'

    # Download build list and logs to the runner
    - name: Stage the DBB BuildList
      shell: powershell
      run: scp ${{inputs.zOS}}:${{env.workSpace}}/${{env.appDirName}}/logs/buildList_utf8.txt ${{env.jobLogs}}

    - name: Save all logs
      if: always()
      shell: powershell
      run: scp ${{inputs.zOS}}:${{env.workSpace}}/${{env.appDirName}}/logs/*.log ${{env.jobLogs}}
```

## Requirements

- Windows GitHub Actions runner with SSH and SCP access to z/OS.
- Git and IBM DBB 3.0.5 configured on z/OS USS.
- Valid z/OS environment profile and build configuration.
- Artifactory credentials and configuration.
- IBM zCodeScan for downstream code scanning.

## Notes

- The Common repository is optional and cloned only when `CommonURL` is configured.
- The configuration refresh step is intended for Admin workflows; normal application builds do not require it.
- The `if: always()` condition ensures log collection is attempted even if an earlier step fails.
- The build ID combines the GitHub Actions run number with the generated artifact version.
- The example uses SSH to execute commands on z/OS and SCP to retrieve build outputs.