# Managing zDevOps Configuration as Code

DevOps is not limited to application source code. The configuration that defines and controls the zDevOps environment should also be **versioned, tested, reviewed, and managed as code**.

<img src="../.images/zdo.png" width="250" align="right">
The IBM zDevOps team has created a sample **zDevOps Admin repository** that provides a foundation for managing zDevOps configuration files using Git. The repository demonstrates how administrators can apply the same Git-based development and governance practices used by application teams to the management of the zDevOps platform.


The sample repository can be used as a starting point for defining your organization's own structure, workflows, and automation for managing zDevOps configuration as code.




The **zDevOps Admin repository** provides administrators with a controlled environment to develop and manage this configuration using the same Git-based practices used by application teams.

## What This Repository Provides

The repository supports two primary workflows:

* **Develop and validate** — Administrators can create personal Git branches to develop and test configuration changes using sample application source code before introducing changes into the shared environment.
* **Manage enterprise configuration as code** — Approved configuration is maintained in Git and progressed through a controlled process of review, validation, and deployment.

This approach provides:

* **Version control** — Every configuration change is tracked in Git.
* **Code review** — Changes can be reviewed before they are introduced into the shared environment.
* **Validation** — Configuration can be tested before deployment.
* **Traceability** — Changes are associated with commits, branches, and pull requests.
* **Repeatability** — The same configuration can be consistently deployed across environments.
* **Governance** — Administrative changes follow the same controlled SDLC practices as application changes.

## The Admin Repository

The process starts with a central **zDevOps Git repository**.

The repository contains:

```text
zdevops/
├── conf/
│   ├── dbb/
│   ├── wazi-deploy/
│   └── scripts/
├── sample-source/
└── .github/
    └── workflows/
```

The `conf/` directory contains the configuration being developed by the administrator.

Sample application source is included so administrators can test configuration changes before making them available to development teams.

This creates an **admin sandbox** where changes can be developed and tested safely without affecting the shared environment. The original framework describes this as a development environment where administrators can use personal Git branches and sample application source to validate configuration changes. 

## Git-Based Administration

The administrator clones the repository and creates a personal branch from `main`.

Using VS Code, the administrator can:

* Edit configuration
* Test changes
* Commit changes
* Push the branch
* Review differences

This is the same basic Git workflow used by application teams.

The difference is that the administrator is managing the **DevOps platform configuration**, rather than the application itself.

## Test Before Promotion

A configuration change should be tested before it is promoted.

The sample application source provides a controlled way to exercise the configuration.

For example, a change to DBB configuration can be tested against the sample source to verify that the expected behavior still works.

The goal is simple:

> **Test the configuration before making it available to everyone.**

## Infrastructure as Code

This approach applies the principles of **Infrastructure as Code** to the zDevOps platform.

The configuration becomes:

* **Versioned** in Git
* **Testable** using sample source
* **Reviewable** through pull requests
* **Automated** through the Git workflow
* **Auditable** through Git history
* **Repeatable** across environments



## The Big Picture

The following workflow shows the complete administrator lifecycle, from developing a configuration change in Git to making the validated configuration available to application teams.

 
<img src="../.images/zDevOps-IaC.png" style="width: 850px; max-width: 100%; height: auto;">
               
The administrator works with the configuration in Git and uses the sample application source to verify changes. Once validated, the configuration is promoted to the centralized USS location:

```text
/u/etc/zdevops
```

Application teams then consume this shared configuration from their user builds and CI/CD pipelines.

This creates a clear separation between the **development of the DevOps platform** and the **development of applications**.

## The Result

The zDevOps administrator now has a controlled lifecycle for managing the DevOps platform:

* Git provides the source of truth.
* Branches provide a safe development area.
* Sample source provides a repeatable test environment.
* Automation validates changes.
* USS provides the centralized configuration used by development teams.

The result is a simple **configuration-as-code lifecycle** for z/OS:

> **Develop → Test → Validate → Promote → Consume**

