# 🚀 Jenkins Distributed Build Architecture with Linux Agent

### Building GitHub Projects on a Dedicated Ubuntu Jenkins Agent

![AWS](https://img.shields.io/badge/AWS-EC2-orange?logo=amazonaws)
![Jenkins](https://img.shields.io/badge/Jenkins-CI%2FCD-red?logo=jenkins)
![Ubuntu](https://img.shields.io/badge/Agent-Ubuntu-E95420?logo=ubuntu)
![Java](https://img.shields.io/badge/Java-21-orange?logo=openjdk)
![Maven](https://img.shields.io/badge/Maven-3.9.9-C71A36?logo=apachemaven)
![GitHub](https://img.shields.io/badge/GitHub-Source%20Control-181717?logo=github)
![SSH](https://img.shields.io/badge/Connection-SSH-blue?logo=openssh)
![DevOps](https://img.shields.io/badge/Domain-DevOps-purple)

> **A practical Jenkins distributed-build project demonstrating how to connect an Ubuntu AWS EC2 server as a Jenkins SSH agent, assign builds using labels, pull a Java/Maven application from GitHub, execute a Maven build, and archive the resulting WAR artifact.**

<img width="1536" height="1024" alt="4d6da0c5-850b-424c-87a0-909f502e0512" src="https://github.com/user-attachments/assets/61993b7b-3662-4e36-a20e-0b65ddcccc2a" />

---

# 📌 Project Overview

As CI/CD environments mature, Jenkins builds should not necessarily run directly on the Jenkins controller.

A more scalable architecture separates:

```text
Jenkins Controller
        |
        | Coordinates jobs
        |
        v
Jenkins Agent
        |
        | Builds application
        |
        v
Maven / Java / Git
```

In this project, the Jenkins controller runs on one EC2 server while a second **Ubuntu EC2 instance acts as a dedicated Jenkins build agent**.

The agent is connected to Jenkins using **SSH**, assigned the label:

```text
MavenBuilder
```

and configured to execute only jobs intended for that label.

The resulting workflow is:

```text
Developer
    |
    v
GitHub Repository
    |
    v
Jenkins Controller
    |
    | SSH
    v
Ubuntu Jenkins Agent
    |
    +---- Git
    |
    +---- JDK 21
    |
    +---- Maven
    |
    v
Maven Build
    |
    v
WAR Artifact
    |
    v
Jenkins Archive
```

This project extends a basic Jenkins installation into a more realistic **distributed CI/CD architecture**.

---

# 🎯 Project Objectives

The objectives of this project are to:

- Provision a dedicated Ubuntu EC2 instance for Jenkins builds.
- Configure secure SSH connectivity from the Jenkins controller to the agent.
- Install Java 21 on the agent.
- Create a dedicated Jenkins workspace directory.
- Connect the agent to Jenkins using the SSH launch method.
- Configure an agent label.
- Restrict specific jobs to the dedicated agent.
- Validate agent execution through a test build.
- Connect Jenkins to a GitHub repository.
- Configure Maven 3.9.9.
- Build a Java application from GitHub.
- Generate a WAR artifact.
- Archive the build artifact in Jenkins.

---

# 📚 Table of Contents

1. [Project Overview](#-project-overview)
2. [Project Objectives](#-project-objectives)
3. [Why Use Jenkins Agents?](#-why-use-jenkins-agents)
4. [Real-World Scenario](#-real-world-scenario)
5. [Solution Architecture](#-solution-architecture)
6. [Architecture Components](#-architecture-components)
7. [Technology Stack](#-technology-stack)
8. [Construction Rules](#-construction-rules)
9. [Implementation Flow](#-implementation-flow)
10. [Prerequisites](#-prerequisites)
11. [Step 1 — Create the Ubuntu EC2 Agent](#1--create-the-ubuntu-ec2-agent)
12. [Step 2 — Create the EC2 Key Pair](#2--create-the-ec2-key-pair)
13. [Step 3 — Configure the Agent Security Group](#3--configure-the-agent-security-group)
14. [Step 4 — SSH to the Agent Server](#4--ssh-to-the-agent-server)
15. [Step 5 — Install Java 21](#5--install-java-21)
16. [Step 6 — Create the Jenkins Agent Directory](#6--create-the-jenkins-agent-directory)
17. [Step 7 — Configure Directory Ownership](#7--configure-directory-ownership)
18. [Step 8 — Add the Agent to Jenkins](#8--add-the-agent-to-jenkins)
19. [Step 9 — Configure the SSH Launch Method](#9--configure-the-ssh-launch-method)
20. [Step 10 — Verify the Agent Connection](#10--verify-the-agent-connection)
21. [Step 11 — Run a Build on the Agent](#11--run-a-build-on-the-agent)
22. [Step 12 — Connect Jenkins to GitHub](#12--connect-jenkins-to-github)
23. [Step 13 — Configure Maven Build](#13--configure-maven-build)
24. [Step 14 — Archive the WAR Artifact](#14--archive-the-war-artifact)
25. [Step 15 — Execute the GitHub Build](#15--execute-the-github-build)
26. [Build Execution Flow](#-build-execution-flow)
27. [Network and SSH Architecture](#-network-and-ssh-architecture)
28. [Security Architecture](#-security-architecture)
29. [Agent Configuration Reference](#-agent-configuration-reference)
30. [Verification Checklist](#-verification-checklist)
31. [Troubleshooting](#-troubleshooting)
32. [Cost Considerations](#-cost-considerations)
33. [Production Improvements](#-production-improvements)
34. [Skills Demonstrated](#-skills-demonstrated)
35. [Project Structure](#-project-structure)
36. [Lessons Learned](#-lessons-learned)
37. [Future CI/CD Evolution](#-future-cicd-evolution)
38. [Official Documentation](#-official-documentation)
39. [Project Author](#-project-author)

---

# 🧠 Why Use Jenkins Agents?

A Jenkins controller is primarily responsible for:

- managing jobs,
- scheduling work,
- storing configuration,
- coordinating builds,
- communicating with agents.

The actual build workloads can be delegated to agents.

Jenkins documentation recommends configuring executors on agents rather than using the controller itself for builds because this keeps the controller focused on orchestration and improves stability. :contentReference[oaicite:1]{index=1}

The architecture therefore becomes:

```text
                Jenkins Controller
                       |
                       |
                  SSH Connection
                       |
                       v
                Jenkins Agent
                       |
           +-----------+-----------+
           |           |           |
           v           v           v
          Git        Java        Maven
                       |
                       v
                   Build
```

---

# 💼 Real-World Scenario

Imagine a development team managing multiple Java applications.

The Jenkins controller receives builds from:

```text
GitHub
   |
   +---- Application A
   |
   +---- Application B
   |
   +---- Application C
```

Rather than running all builds directly on the controller, dedicated agents can be created:

```text
                  Jenkins Controller
                         |
          +--------------+--------------+
          |                             |
          v                             v
    MavenBuilder                  DockerBuilder
      Ubuntu                        Linux
          |                             |
       Maven                         Docker
       Java                          Build
```

This provides an architecture where build environments can be tailored to workload requirements.

---

# 🏗️ Solution Architecture

```text
                         GITHUB
                           |
                           |
                    Source Repository
                           |
                           v
                +----------------------+
                | Jenkins Controller   |
                |                      |
                | Job Scheduling       |
                | Credentials          |
                | Build Coordination   |
                +----------+-----------+
                           |
                           | SSH
                           |
                           v
                +----------------------+
                | Ubuntu EC2 Agent     |
                |                      |
                | Label: MavenBuilder  |
                | Executor: 1          |
                |                      |
                | Git                  |
                | JDK 21               |
                | Maven 3.9.9          |
                +----------+-----------+
                           |
                           v
                      Maven Build
                           |
                           v
                      WAR Artifact
                           |
                           v
                    Jenkins Archive
```

---

# 🧩 Architecture Components

| Component | Responsibility |
|---|---|
| Jenkins Controller | Job scheduling and orchestration |
| Ubuntu EC2 Agent | Executes builds |
| SSH | Controller-to-agent connection |
| GitHub | Source-code repository |
| Git | Source-code checkout |
| JDK 21 | Java build/runtime environment |
| Maven 3.9.9 | Java build automation |
| Jenkins Label | Selects appropriate build agent |
| Jenkins Workspace | Build working directory |
| Artifact Archive | Stores generated build artifacts |

---

# 🛠️ Technology Stack

### Cloud

- Amazon EC2
- AWS Security Groups
- EC2 Key Pair
- Private IP networking

### CI/CD

- Jenkins
- Jenkins SSH Build Agent
- Jenkins Freestyle Jobs

### Build Tools

- OpenJDK 21
- Maven 3.9.9
- Git

### Source Control

- GitHub

### Communication

- SSH
- TCP/IP

---

# 📐 Construction Rules

## Rule 1 — Separate Controller and Build Workloads

The Jenkins controller coordinates the pipeline.

The Ubuntu agent performs the application build.

```text
Controller
    |
    | Orchestration
    v
Agent
    |
    | Build
    v
Application
```

---

## Rule 2 — Use Labels to Control Job Placement

The agent is assigned:

```text
MavenBuilder
```

Jobs intended to execute on this machine must use the matching label.

Jenkins uses labels to group agents and control where jobs execute. :contentReference[oaicite:2]{index=2}

---

## Rule 3 — Restrict Agent Access

The agent security group should allow SSH:

```text
From:
Jenkins Controller Private IP
```

and optionally:

```text
From:
Trusted Administrator IP
```

Do not unnecessarily expose the agent SSH port to the entire Internet.

---

## Rule 4 — Use Private Networking Between Jenkins and Agent

When both EC2 instances are in the same VPC, the Jenkins controller should connect to the agent using the agent's **private IP address**.

Example:

```text
Jenkins Controller
10.0.1.10
      |
      | SSH
      v
Agent
10.0.2.10
```

This avoids unnecessary exposure of the agent's SSH service through its public address.

---

## Rule 5 — Use SSH Credentials Securely

The agent's private SSH key should be stored in the Jenkins Credentials store.

Do not publish it in GitHub.

Do not place it inside a public README.

Do not paste the private key into chat or screenshots.

---

## Rule 6 — Host-Key Verification Should Be Properly Configured

For a learning lab, users may choose a relaxed verification strategy, but for production the Jenkins SSH connection should use a proper host-key verification strategy.

Avoid permanently using:

```text
Non-verifying strategy
```

because it weakens protection against man-in-the-middle attacks.

The Jenkins SSH Build Agents plugin specifically has a history of host-key verification security concerns, making proper host-key management an important production consideration. :contentReference[oaicite:3]{index=3}

---

# 🔄 Implementation Flow

```text
Create Ubuntu EC2
        |
        v
Create Key Pair
        |
        v
Configure Security Group
        |
        v
Allow SSH from Administrator
        |
        v
Allow SSH from Jenkins Controller
        |
        v
SSH into Agent
        |
        v
Install OpenJDK 21
        |
        v
Verify Java
        |
        v
Create /opt/jenkins
        |
        v
Set Ownership to ubuntu
        |
        v
Configure Jenkins Node
        |
        v
Configure SSH Credentials
        |
        v
Connect Agent
        |
        v
Verify Agent Logs
        |
        v
Restrict Freestyle Job to MavenBuilder
        |
        v
Build
        |
        v
Clone GitHub Repository
        |
        v
Run Maven Install
        |
        v
Generate WAR
        |
        v
Archive WAR
```

---

# ✅ Prerequisites

You should already have:

- Jenkins Controller installed and running.
- AWS account.
- Existing Jenkins server.
- AWS VPC.
- EC2 key pair.
- Basic SSH knowledge.
- GitHub repository containing a Java/Maven project.
- Maven configured in Jenkins.
- Access to Jenkins Credentials.
- An Ubuntu AMI.

---

# 1 — Create the Ubuntu EC2 Agent

Create another EC2 instance.

This instance will become:

```text
Jenkins Build Agent
```

Recommended:

```text
AMI:
Ubuntu Server

Architecture:
64-bit

Storage:
20 GB+ EBS

Public IP:
Optional for production
```
<img width="1418" height="821" alt="Screenshot 2026-09-14 112024" src="https://github.com/user-attachments/assets/627ecaf1-d0f1-456f-9864-bf3cb3902dd9" />

The agent does not need to expose Jenkins itself.

Its main role is to execute build commands.

---

# 2 — Create the EC2 Key Pair

Create an EC2 key pair.

Example:

```text
Name:
jenkins-agent-key
```

Download the private key and store it securely.

Example:

```text
jenkins-agent-key.pem
```
<img width="1384" height="717" alt="Screenshot 2026-09-14 112036" src="https://github.com/user-attachments/assets/256835dc-0ddb-4d7c-bd44-dcd32214007d" />

Do not commit this file to GitHub.

---

# 3 — Configure the Agent Security Group

The agent requires SSH connectivity.

## SSH from Administrator

```text
Protocol:
TCP

Port:
22

Source:
Your trusted IP
```

## SSH from Jenkins Controller

Add another SSH rule.

Recommended source:

```text
Jenkins Controller Security Group
```

or, depending on your AWS design:

```text
Jenkins Controller Private IP/32
```

For example:

```text
TCP 22
Source: Jenkins Controller Private IP/32
```

This allows:
<img width="862" height="466" alt="Screenshot 2026-09-14 112203" src="https://github.com/user-attachments/assets/94810931-cb83-4377-a99b-4df3db9fc608" />

```text
Jenkins Controller
       |
       | SSH :22
       v
Ubuntu Agent
```

---

# 4 — SSH to the Agent Server

From your local terminal:

```bash
ssh -i "jenkins-agent-key.pem" ubuntu@<AGENT-PUBLIC-IP>
```

Verify:

```bash
whoami
```

Expected:
<img width="1080" height="489" alt="Screenshot 2026-09-14 112834" src="https://github.com/user-attachments/assets/a1577c6b-2cb3-4292-8f93-d0391b62d4cb" />

```text
ubuntu
```

---

# 5 — Install Java 21

On the Ubuntu agent:

```bash
sudo apt update && sudo apt install openjdk-21-jdk -y
```

Verify:

```bash
java -version
```

Expected output will identify OpenJDK 21.

Jenkins agents use Java to run the Jenkins agent process, so the agent machine needs a compatible Java runtime.
<img width="1096" height="230" alt="Screenshot 2026-09-14 113112" src="https://github.com/user-attachments/assets/01d2760b-d6ac-4f27-9bc3-47f3c47e49eb" />

---

# 6 — Create the Jenkins Agent Directory

Create:

```bash
sudo mkdir /opt/jenkins
```

This directory will become the agent's remote root/workspace area.

---
<img width="1096" height="230" alt="Screenshot 2026-09-14 113112" src="https://github.com/user-attachments/assets/6fcc4597-af76-4cc1-8aba-54ea07a9dfa6" />

# 7 — Configure Directory Ownership

Check the Ubuntu user:

```bash
id ubuntu
```

Then assign ownership:

```bash
sudo chown ubuntu:ubuntu /opt/jenkins
```

Verify:

```bash
ls -ld /opt/jenkins
```
<img width="1096" height="230" alt="Screenshot 2026-09-14 113112" src="https://github.com/user-attachments/assets/171b2d95-9741-4896-bf09-77268c7d4b11" />

Expected ownership should resemble:

```text
ubuntu ubuntu
```

This ensures the SSH user can work within the Jenkins agent directory.

---

# 8 — Add the Agent to Jenkins

Return to the Jenkins web interface.

Navigate to:

```text
Jenkins Dashboard
        |
        v
Manage Jenkins
        |
        v
Nodes
        |
        v
New Node
```

Create:

```text
Name:
MavenBuilder
```

Select:

```text
Permanent Agent
```

---

# Agent Configuration

Configure:

```text
Number of executors:
1
```
<img width="1859" height="850" alt="Screenshot 2026-09-14 113604" src="https://github.com/user-attachments/assets/14484d8a-e0e1-480e-bff7-0c763b4fcb79" />

Using one executor means this agent will execute one build at a time.

Jenkins uses the executor count to determine how many concurrent tasks an agent can perform. :contentReference[oaicite:4]{index=4}

---

## Remote Root Directory

Enter:

```text
/opt/jenkins
```

---

## Labels

Enter:

```text
MavenBuilder
```

This label will be used to send specific jobs to this agent.

---

## Usage

Select:

```text
Only build jobs with label expressions matching this node
```

This prevents unrelated jobs from automatically being assigned to this agent.

---

# 9 — Configure SSH Launch Method

For the launch method choose:

```text
Launch agents via SSH
```

The Jenkins SSH Build Agents plugin provides the mechanism for launching agents over SSH. :contentReference[oaicite:5]{index=5}

---

## Host

Use the **private IP address** of the Ubuntu agent.

Example:

```text
172.31.71.39
```
<img width="569" height="197" alt="Screenshot 2026-09-14 113758" src="https://github.com/user-attachments/assets/a889b1d9-89b0-492f-bd1e-dc5a65edfa0d" />

Architecture:

```text
Jenkins Controller
172.31.71.39
        |
        | SSH
        v
Agent
172.31.71.39
```

---
<img width="1678" height="457" alt="Screenshot 2026-09-14 113820" src="https://github.com/user-attachments/assets/2518fd54-054a-42cd-ba05-a52ed9347e48" />

## Credentials

Create a new SSH credential.

Example:

```text
ID:
jenkins-agent-ssh

Description:
SSH credential for MavenBuilder Ubuntu Agent

Username:
ubuntu
```
<img width="1661" height="651" alt="Screenshot 2026-09-14 113832" src="https://github.com/user-attachments/assets/57aca251-88e8-4c8d-b091-a7a2cf2a9669" />

For the private key, use the private key associated with the agent EC2 key pair.

### Important

Do not publish the private key.

Do not add it to your GitHub repository.

Do not place the private key in this README.

---

# Host Key Verification

For a learning environment, your existing implementation may use:

```text
Host Key Verification Strategy:
Non-verifying
```
<img width="550" height="605" alt="Screenshot 2026-09-14 113841" src="https://github.com/user-attachments/assets/c2ff5c47-1d78-4103-b5af-9ed670e2d39f" />

However, this is not recommended for production.

A production implementation should verify the agent's SSH host key.

This protects the Jenkins controller from connecting to an unexpected host.
<img width="1669" height="694" alt="Screenshot 2026-09-14 114259" src="https://github.com/user-attachments/assets/ae6c354f-ec31-43cd-bbb4-042d98c2e316" />

---

# 10 — Verify the Agent Connection

Save the node configuration.

Open:

```text
Jenkins
   |
   v
Manage Jenkins
   |
   v
Nodes
   |
   v
MavenBuilder
```

Open the node log.

A successful connection should indicate that Jenkins has established an SSH connection and launched the agent process.

The expected architecture is:

```text
Jenkins Controller
      |
      | SSH
      v
Ubuntu Agent
      |
      v
Jenkins Agent Process
```
<img width="1808" height="490" alt="Screenshot 2026-09-14 115124" src="https://github.com/user-attachments/assets/7d9f1bf8-3ad7-4fe1-b93e-33c673032319" />

<img width="1620" height="577" alt="Screenshot 2026-09-14 115131" src="https://github.com/user-attachments/assets/c7abcfa4-63da-409c-bf12-771659f2dde2" />

---

# 11 — Run a Build on the Agent

Create or use an existing Freestyle project.

Go to:

```text
Job
   |
   v
Configure
```

Find:

```text
Restrict where this project can be run
```

Enable it.

Enter:

```text
MavenBuilder
```

Save.

Now run:

```text
Build Now
```
<img width="1876" height="835" alt="Screenshot 2026-09-14 115538" src="https://github.com/user-attachments/assets/6d5e8ae7-462a-49f1-8601-a973b7d2e346" />

Open:

```text
Console Output
```
<img width="1907" height="641" alt="Screenshot 2026-09-14 115632" src="https://github.com/user-attachments/assets/8a45e84d-dd76-4f4b-865f-e17d8f4ea481" />

You should see that the job was scheduled and executed on the designated agent.

The important thing to identify is the workspace location:

```text
/opt/jenkins/workspace/<JOB-NAME>
```

This confirms that the build executed on the Ubuntu agent rather than the controller.

---

# 12 — Connect Jenkins to GitHub

Now that the agent is operational, the next stage is to build a real project from GitHub.

Create a new Jenkins job:

```text
New Item
   |
   v
GitHub-Maven-Build
```

Select:

```text
Freestyle project
```
<img width="1330" height="767" alt="Screenshot 2026-09-14 120805" src="https://github.com/user-attachments/assets/444443d9-3b5f-4b94-bba2-7a0768a969eb" />

---

# Source Code Management

Select:

```text
Git
```

Enter the repository URL.

HTTPS example:

```text
https://github.com/<USERNAME>/<REPOSITORY>.git
```
<img width="1381" height="769" alt="Screenshot 2026-09-14 120815" src="https://github.com/user-attachments/assets/46107aeb-55e3-4597-9b19-564fd2528ce2" />

SSH example:

```text
git@github.com:<USERNAME>/<REPOSITORY>.git
```

For a private repository, configure an appropriate Jenkins Git credential.

---

# Branch

Specify the branch to build.

For example:

```text
*/electron
```

or:

```text
*/electron
```

depending on your repository.

---

# 13 — Configure Maven Build

Scroll to:

```text
Build Steps
```
<img width="1088" height="756" alt="Screenshot 2026-09-14 120824" src="https://github.com/user-attachments/assets/529d7c3b-1851-4579-bd8c-35d74982cf20" />

Add:

```text
Invoke top-level Maven targets
```

Select the Maven installation previously configured in Jenkins:

```text
MAVEN3.9
```
<img width="1281" height="388" alt="Screenshot 2026-09-14 121416" src="https://github.com/user-attachments/assets/ac6e35cd-922c-4ab7-b01e-6e0c3527298f" />

Version:

```text
3.9.9
```

For the goals:

```text
install
```

A more complete lifecycle command could also be:

```text
clean install
```

For this demonstration, the requested goal is:

```text
install
```

The Maven build will compile, test and package the project according to its `pom.xml`.

---

# 14 — Archive the WAR Artifact

After the Maven build step, add:

```text
Post-build Actions
        |
        v
Archive the artifacts
```

Use:

```text
**/*.war
```

This instructs Jenkins to archive WAR files produced anywhere within the workspace.

Jenkins' artifact archiving supports wildcard patterns and stores matching artifacts so they can be accessed from the Jenkins build page. :contentReference[oaicite:6]{index=6}

For example:

```text
target/vprofile-v2.war
```

would match:

```text
**/*.war
```
<img width="1338" height="393" alt="Screenshot 2026-09-14 120847" src="https://github.com/user-attachments/assets/37623529-e149-4687-b98a-5d97c7bc0635" />

---

# 15 — Execute the GitHub Build

Click:

```text
Save
```

Then:

```text
Build Now
```
<img width="1136" height="632" alt="Screenshot 2026-09-14 121525" src="https://github.com/user-attachments/assets/5069b2c1-1710-4f68-ba63-ec6a1b629ae8" />

Jenkins should perform:

```text
GitHub Repository
       |
       v
Jenkins Controller
       |
       v
MavenBuilder Agent
       |
       +---- Git checkout
       |
       +---- Maven install
       |
       +---- Tests
       |
       +---- Package
       |
       v
WAR Artifact
       |
       v
Jenkins Archive
```
<img width="1659" height="445" alt="Screenshot 2026-09-14 121543" src="https://github.com/user-attachments/assets/17867e1e-0ec4-4393-8422-95c0fdf1cd6c" />

---

# 🔄 Build Execution Flow

The complete workflow is:

```text
                    DEVELOPER
                        |
                        v
                      GitHub
                        |
                        v
               Jenkins Controller
                        |
                        | SSH
                        v
               +------------------+
               | Ubuntu Agent     |
               | MavenBuilder     |
               +--------+---------+
                        |
              +---------+---------+
              |         |         |
              v         v         v
             Git       Java      Maven
                        |
                        v
                   Maven Install
                        |
                        v
                    Unit Tests
                        |
                        v
                    WAR Package
                        |
                        v
               Jenkins Artifact
```

---

# 🌐 Network and SSH Architecture

The agent communication is intentionally separated from the public Jenkins web interface.

```text
                   INTERNET
                       |
                       |
                       v
              Jenkins Controller
              Public / Private IP
                       |
                       |
                  SSH TCP 22
                       |
                       v
              Ubuntu EC2 Agent
                  Private IP
                       |
             +---------+---------+
             |                   |
             v                   v
            Git                Maven
                                 |
                                 v
                              Java 21
```

### Recommended Security Flow

```text
Administrator
      |
      | SSH
      v
Controller

Controller
      |
      | Private SSH
      v
Agent
```

The agent should not require a publicly accessible Jenkins port.

---

# 🔐 Security Architecture

Security is one of the most important aspects of distributed Jenkins.

## 1. EC2 Security Group

Allow:

```text
TCP 22
Source:
Trusted administrator IP
```

and:

```text
TCP 22
Source:
Jenkins Controller private IP
```

---

## 2. Private IP Communication

Whenever controller and agent share a VPC:

```text
Controller
10.x.x.x
   |
   v
Agent
10.x.x.x
```

Use the private IP.

This avoids unnecessarily exposing the agent SSH endpoint to the public Internet.

---

## 3. SSH Authentication

Jenkins connects using an SSH credential.

The architecture is:

```text
Jenkins Credential
       |
       v
Private SSH Key
       |
       v
Ubuntu Agent
       |
       v
Public Key / EC2 Authorized Access
```

---

## 4. Host Key Verification

Avoid:

```text
Non-verifying
```

in production.

Use a proper host-key verification strategy so that Jenkins can verify that it is connecting to the expected SSH server.

---

## 5. Least Privilege

The Jenkins agent should not receive unnecessary privileges.

The build user should have only the permissions required to:

- read/write its workspace,
- run Maven,
- use Git,
- create build artifacts,
- perform necessary build commands.

Avoid giving the Jenkins user unrestricted root access simply to make builds work.

---

# 🧾 Agent Configuration Reference

| Configuration | Value |
|---|---|
| Node name | `MavenBuilder` |
| Node type | Permanent Agent |
| Executors | `1` |
| Remote root | `/opt/jenkins` |
| Label | `MavenBuilder` |
| Usage | Only jobs matching label |
| Launch method | SSH |
| Host | Agent private IP |
| Username | `ubuntu` |
| Java | OpenJDK 21 |
| Build tool | Maven 3.9.9 |
| SCM | GitHub |

---

# 🧪 Verification Checklist

```text
[✓] Ubuntu EC2 agent created

[✓] EC2 key pair created

[✓] SSH from administrator allowed

[✓] SSH from Jenkins controller allowed

[✓] Agent accessible by SSH

[✓] OpenJDK 21 installed

[✓] Java version verified

[✓] /opt/jenkins created

[✓] Directory ownership configured

[✓] MavenBuilder node created

[✓] Remote root configured

[✓] Label configured

[✓] SSH credentials configured

[✓] Agent connected successfully

[✓] Node log verified

[✓] Freestyle job restricted to MavenBuilder

[✓] GitHub repository configured

[✓] Maven 3.9.9 configured

[✓] Maven install executed

[✓] WAR generated

[✓] WAR archived
```

---

# 🚨 Troubleshooting

## Agent Is Offline

Check:

```text
Jenkins
  ↓
Nodes
  ↓
MavenBuilder
  ↓
Log
```

Also confirm from the Jenkins controller:

```bash
ssh ubuntu@<AGENT-PRIVATE-IP>
```

---

# Agent SSH Connection Fails

Check the AWS Security Group.

Verify:

```text
TCP 22
Source: Jenkins Controller
```

Confirm the Jenkins controller can route to the private IP.

---

# Java Not Found

On the agent:

```bash
java -version
```

If missing:

```bash
sudo apt update
sudo apt install openjdk-21-jdk -y
```

Then:

```bash
java -version
```

---

# Permission Denied for `/opt/jenkins`

Check:

```bash
ls -ld /opt/jenkins
```

Expected ownership:

```text
ubuntu ubuntu
```

Fix:

```bash
sudo chown ubuntu:ubuntu /opt/jenkins
```

---

# Job Runs on the Controller

Go to:

```text
Job
   |
   v
Configure
```

Make sure:

```text
Restrict where this project can be run
```

is enabled.

Then specify:

```text
MavenBuilder
```

---

# Maven Command Not Found

Check Jenkins:

```text
Manage Jenkins
    ↓
Tools
    ↓
Maven installations
```

Confirm:

```text
MAVEN3.9
```

is correctly configured.

---

# WAR Artifact Is Not Archived

Confirm the Maven build actually produced a WAR.

On the agent workspace:

```bash
find . -name "*.war"
```

If the output contains:

```text
target/application.war
```

then:

```text
**/*.war
```

will normally match it.

Jenkins supports wildcard-based artifact archiving from the build workspace. :contentReference[oaicite:7]{index=7}

---

# 💰 Cost Considerations

This architecture adds a second EC2 instance.

Therefore, the AWS cost model becomes:

```text
Jenkins Controller EC2
        +
Jenkins Agent EC2
        +
EBS
        +
Public IPv4 where used
        +
Data Transfer
```

For a learning environment, a small agent can be sufficient.

However, build-heavy workloads may require more CPU and memory.

A useful future optimization is to make agents **ephemeral or dynamically provisioned**, allowing build infrastructure to scale based on demand rather than maintaining permanently running instances.

---

# 📈 Why This Architecture Is Better Than Building Everything on the Controller

A basic Jenkins installation looks like:

```text
Developer
   |
   v
Jenkins Controller
   |
   v
Build
```

The distributed architecture becomes:

```text
Developer
   |
   v
Jenkins Controller
   |
   +----------+
              |
              v
        Build Agent
              |
              v
             Build
```

This provides several benefits:

- Controller resources remain available for orchestration.
- Build environments can be specialized.
- Different agents can support different toolchains.
- Workloads can be scaled horizontally.
- Failed agents can be replaced without rebuilding the controller.
- Security boundaries can be clearer.
- Docker, Maven, Node.js or other workloads can be separated.

Jenkins explicitly documents labels as a mechanism for assigning jobs to particular agents and recommends agents for build execution. :contentReference[oaicite:8]{index=8}

---

# 🏭 Production Improvements

## 1. Dedicated Build Agents

Instead of one agent:

```text
Jenkins Controller
       |
       +---- MavenBuilder
       |
       +---- DockerBuilder
       |
       +---- NodeBuilder
       |
       +---- TestAgent
```

---

## 2. Ephemeral Agents

A stronger architecture can create agents only when builds are needed:

```text
Jenkins
   |
   v
Build Request
   |
   v
Provision Agent
   |
   v
Run Build
   |
   v
Destroy Agent
```

This improves cost efficiency and reduces persistent attack surface.

---

## 3. Infrastructure as Code

Terraform can provision:

```text
VPC
EC2
Security Groups
IAM
Elastic IP
```

---

## 4. Configuration Management

Ansible can automate:

```text
Ubuntu
Java
Maven
Git
Jenkins Agent
```

---

## 5. Pipeline as Code

The Freestyle project can eventually become:

```text
Jenkinsfile
```

A Pipeline can express build stages as code and version them alongside the application.

Jenkins' current documentation demonstrates building Maven applications through Pipeline stages and executing the Maven command on the agent. :contentReference[oaicite:9]{index=9}

---

# 🔮 Future CI/CD Evolution

This project is the bridge between a basic Jenkins server and a distributed CI/CD platform.

## Stage 1

```text
GitHub
   ↓
Jenkins Controller
   ↓
MavenBuilder
   ↓
Maven
   ↓
WAR
```

## Stage 2

```text
GitHub
   ↓
Jenkins
   ↓
Maven
   ↓
Unit Tests
   ↓
SonarQube
```

## Stage 3

```text
GitHub
   ↓
Jenkins
   ↓
SonarQube
   ↓
Quality Gate
   ↓
Docker
```

## Stage 4

```text
Docker
   ↓
Amazon ECR
```

## Stage 5

```text
Amazon ECR
   ↓
Amazon ECS
   ↓
Production
```

---

# 🧠 Skills Demonstrated

## AWS

- Amazon EC2
- Security Groups
- Private IP networking
- Key Pairs
- VPC networking
- Cloud cost awareness

## Jenkins

- Controller/agent architecture
- SSH build agents
- Nodes
- Labels
- Executors
- Remote root configuration
- Job assignment
- Freestyle projects
- Git integration
- Artifact archiving

## Linux

- Ubuntu administration
- SSH
- APT
- Java installation
- Linux permissions
- Directory ownership
- Network troubleshooting

## Java

- OpenJDK 21
- Jenkins agent runtime

## Maven

- Maven 3.9.9
- Java application builds
- Maven lifecycle
- WAR packaging

## Git

- GitHub integration
- Branch selection
- Source checkout

## Security

- SSH key authentication
- Security Group restrictions
- Private IP communication
- Host-key verification
- Credential management
- Least privilege

## DevOps

- Distributed build architecture
- CI/CD
- Build automation
- Artifact management
- Infrastructure design
- Troubleshooting
- Scalability planning

---

# 📂 Project Structure

```text
jenkins-agent-cicd/
│
├── README.md
│
├── architecture/
│   └── jenkins-controller-agent.png
│
├── docs/
│   ├── agent-setup.md
│   ├── github-build.md
│   ├── security.md
│   └── troubleshooting.md
│
└── screenshots/
    ├── agent-ec2.png
    ├── security-group.png
    ├── jenkins-node.png
    ├── node-log.png
    ├── job-configuration.png
    ├── build-console.png
    └── archived-artifact.png
```

---

# 🔐 GitHub Security Rules

Never commit:

```text
*.pem
*.key
id_rsa
id_ed25519
Jenkins credentials
AWS credentials
agent private keys
passwords
```

Example `.gitignore`:

```gitignore
# SSH / AWS private keys
*.pem
*.key
id_rsa
id_ed25519

# Environment files
.env
.env.*

# Credentials
credentials/
secrets/

# Logs
*.log
```

---

# 🧠 Lessons Learned

## 1. Jenkins Controller and Agent Have Different Responsibilities

The controller orchestrates.

The agent executes.

```text
Controller
   ↓
Scheduling / Management

Agent
   ↓
Build / Test / Package
```

This makes the architecture easier to scale.

---

## 2. Labels Are a Powerful Scheduling Mechanism

The label:

```text
MavenBuilder
```

allows a job to express:

> "I need an agent capable of performing my Maven build."

This becomes increasingly useful when a Jenkins environment contains multiple specialized agents. :contentReference[oaicite:10]{index=10}

---

## 3. Private Networking Matters

Using the agent's private IP allows:

```text
Controller
   ↓
VPC Private Network
   ↓
Agent
```

instead of:

```text
Controller
   ↓
Internet
   ↓
Agent
```

The first design has a smaller network exposure surface.

---

## 4. Build Artifacts Should Be Managed Deliberately

The generated WAR is not merely an output file.

It is a build artifact that can later become an input to:

```text
Artifact Repository
       ↓
Docker Image
       ↓
ECR
       ↓
Deployment
```

Jenkins supports archiving artifacts so they can be retrieved from the Jenkins build page. :contentReference[oaicite:11]{index=11}

---

# 🏆 Portfolio Value

This project demonstrates a meaningful evolution from a single Jenkins server into a **distributed CI/CD architecture**.

It demonstrates:

```text
AWS Infrastructure
       ↓
Linux Administration
       ↓
Jenkins Controller
       ↓
SSH Build Agent
       ↓
GitHub Integration
       ↓
Maven Build
       ↓
WAR Artifact
       ↓
Artifact Management
```

More importantly, it demonstrates an understanding of **why** Jenkins agents exist rather than simply how to create one.

---

# 📌 Project Summary

### Project

**Jenkins Distributed Build Architecture with Linux Ubuntu Agent**

### Controller

```text
Jenkins
```

### Agent

```text
Ubuntu EC2
```

### Agent Label

```text
MavenBuilder
```

### Runtime

```text
OpenJDK 21
```

### Build Tool

```text
Maven 3.9.9
```

### Source Control

```text
GitHub
```

### Connection

```text
SSH
```

### Artifact

```text
WAR
```

---

# 📚 Official Documentation

### Jenkins Agents

https://www.jenkins.io/doc/book/using/using-agents/

### Jenkins SSH Build Agents

https://plugins.jenkins.io/ssh-slaves/

### Jenkins Projects

https://www.jenkins.io/doc/book/using/working-with-projects/

### Jenkins Artifact Archiving

https://www.jenkins.io/doc/pipeline/steps/core/

### Jenkins Maven

https://www.jenkins.io/doc/tutorials/build-a-java-app-with-maven/

### Maven

https://maven.apache.org/

### Git

https://git-scm.com/

### AWS EC2

https://docs.aws.amazon.com/ec2/

---

# 👨‍💻 Project Author

## Zhibrad

**Cloud & DevOps Portfolio**

Areas of focus:

```text
AWS
Linux
Jenkins
CI/CD
Docker
Maven
Git
Cloud Security
Infrastructure Automation
DevOps
```

---

# ⭐ Final Architecture

```text
                         DEVELOPER
                             |
                             v
                          GitHub
                             |
                             v
                    +------------------+
                    | Jenkins          |
                    | Controller       |
                    +--------+---------+
                             |
                         SSH :22
                             |
                             v
                    +------------------+
                    | Ubuntu EC2 Agent  |
                    |                  |
                    | Label:           |
                    | MavenBuilder     |
                    |                  |
                    | OpenJDK 21       |
                    | Maven 3.9.9      |
                    | Git              |
                    +--------+---------+
                             |
                             v
                       Maven Install
                             |
                             v
                       Unit Tests
                             |
                             v
                        WAR Build
                             |
                             v
                    Jenkins Artifact
```

> **Built to demonstrate real-world CI/CD engineering: distributed Jenkins architecture, secure SSH-based build agents, GitHub integration, Maven automation, artifact management, AWS infrastructure and production-oriented DevOps thinking.**
