# WindowsTools

A Windows setup guide for DevOps practice. Each tool has a short explanation and its own installation command.

> **Copying commands:** Open this file on GitHub or in a Markdown viewer that provides a **Copy** button for fenced code blocks. Plain `.md` files cannot create working copy buttons themselves; the viewer supplies them.

## Before you start

1. Install Chocolatey using the official instructions below.
2. Open **PowerShell as Administrator** and run the installation commands one at a time.
3. Reopen PowerShell after installation so it picks up updated environment variables. Restart Windows if an installer requests it.

The commands below preserve your supplied package names and versions; package availability and VirtualBox–Vagrant version compatibility have not been verified. `-y` automatically accepts Chocolatey's confirmation prompts. `--version` requests a specific package version for that installation.

## 1. Chocolatey — install software from the command line

Chocolatey is a package manager for Windows that installs software using commands.
In DevOps, it helps automate workstation setup so teammates can reproduce the same tool installation steps.

**Install:** Follow the [official Chocolatey installation instructions](https://docs.chocolatey.org/en-us/choco/setup/).

**Check installation:**

```powershell
choco --version
```

## 2. VirtualBox — run practice servers on your laptop

VirtualBox runs virtual machines, such as Linux servers, inside your Windows computer.
In DevOps, it provides local environments for practicing server administration, networking, and deployments without renting cloud servers.

```powershell
choco install virtualbox --version=7.2.6 -y
```

## 3. Vagrant — create repeatable virtual machine environments

Vagrant describes and manages development environments using a configuration file called a `Vagrantfile`.
In DevOps, it automates creating and configuring lab machines; with the VirtualBox provider, VirtualBox runs the VMs and Vagrant manages their setup.

```powershell
choco install vagrant --version=2.4.9 -y
```

## 4. Git — track changes and collaborate

Git records the history of source code, scripts, and infrastructure configuration files.
In DevOps, it lets teams review changes, restore earlier versions, and supply versioned code to CI/CD pipelines.

```powershell
choco install git -y
```

## 5. Amazon Corretto 17 JDK — build and run Java applications

Amazon Corretto 17 is a distribution of the Java Development Kit, including tools to compile and run Java code.
In DevOps, it supports building and testing Java applications and running Java-based tools that require Java 17.

```powershell
choco install corretto17jdk -y
```

## 6. Apache Maven — automate Java builds

Maven manages Java project dependencies and builds using a configuration file called `pom.xml`.
In DevOps, CI/CD pipelines use it to compile code, run tests, and package applications into artifacts such as JAR files.

```powershell
choco install maven -y
```

## 7. AWS CLI — manage AWS using commands

The AWS Command Line Interface lets you interact with AWS services from your terminal or scripts.
In DevOps, it helps automate tasks such as uploading deployment files to S3 and inspecting cloud resources; AWS credentials and permissions are needed to access your account.

```powershell
choco install awscli -y
```

## 8. IntelliJ IDEA Community — develop and debug Java projects

IntelliJ IDEA Community is a development environment for writing, navigating, and debugging Java code.
In DevOps, it helps you inspect application code, work with Maven projects, and reproduce build or test failures locally before changing a pipeline.

```powershell
choco install intellijidea-community -y
```

## 9. Visual Studio Code — edit automation and configuration

Visual Studio Code is a code editor for scripts, application code, and configuration files, with extensions for different languages and tools.
In DevOps, it is useful for editing Python and PowerShell scripts, YAML pipelines, Dockerfiles, and Terraform configurations.

```powershell
choco install vscode -y
```

## 10. Sublime Text 3 — make quick text and configuration edits

Sublime Text 3 is a lightweight editor for source code, scripts, logs, and configuration files.
In DevOps, it is useful for quick inspections and edits; it is an optional editor choice if VS Code already meets your needs.

```powershell
choco install sublimetext3 -y
```

## How the tools fit together

| Task | Tools |
| --- | --- |
| Install workstation software | Chocolatey |
| Create and run local server labs | Vagrant + VirtualBox |
| Version application and infrastructure files | Git |
| Build, test, and package Java applications | Corretto JDK + Maven |
| Run commands against AWS | AWS CLI |
| Edit and debug projects | IntelliJ IDEA Community, VS Code, or Sublime Text |

You do not need every tool for every DevOps project. Java projects benefit from the JDK and Maven, local VM labs use VirtualBox and Vagrant, and editor selection depends on your workflow.

