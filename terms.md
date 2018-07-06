# General Terms
**Agent**
An agent is typically a machine, or container, which connects to a Jenkins master and executes tasks when directed by the master.
**Artifact**
An immutable file generated during a Build or Pipeline run which is **archived** onto the Jenkins Master for later retrieval by users.
**Build**
Result of a single execution of a Project
**Cloud**
A System Configuration which provides dynamic Agent provisioning and allocation, such as that provided by the  [Azure VM Agents](https://plugins.jenkins.io/azure-vm-agents)  or  [Amazon EC2](https://plugins.jenkins.io/ec2)  plugins.
**Core**
The primary Jenkins application (jenkins.war) which provides the basic web UI, configuration, and foundation upon which Plugins can be built.
**Downstream**
A configured Pipeline or Project which is triggered as part of the execution of a separate Pipeline or Project.
**Executor**
A slot for execution of work defined by a Pipeline or Project on a Node. A Node may have zero or more Executors configured which corresponds to how many concurrent Projects or Pipelines are able to execute on that Node.
**Fingerprint**
A hash considered globally unique to track the usage of an Artifact or other entity across multiple Pipelines or Projects.
**Folder**
An organizational container for Pipelines and/or Projects, similar to folders on a file system.
**Item**
An entity in the web UI corresponding to either a: Folder, Pipeline, or Project.
**Job**
A deprecated term, synonymous with Project.
**Label**
User-defined text for grouping Agents, typically by similar functionality or capability. For example linux for Linux-based agents or docker for Docker-capable agents.
**Master**
The central, coordinating process which stores configuration, loads plugins, and renders the various user interfaces for Jenkins.
**Node**
A machine which is part of the Jenkins environment and capable of executing Pipelines or Projects. Both the Master and Agents are considered to be Nodes.
**Project**
A user-configured description of work which Jenkins should perform, such as building a piece of software, etc.
**Pipeline**
A user-defined model of a continuous delivery pipeline, for more read the Pipeline chapter in this handbook.
**Plugin**
An extension to Jenkins functionality provided separately from Jenkins Core.
**Publisher**
Part of a Build after the completion of all configured Steps which publishes reports, sends notifications, etc.
**Stage**
stage is part of Pipeline, and used for defining a conceptually distinct subset of the entire Pipeline, for example: "Build", "Test", and "Deploy", which is used by many plugins to visualize or present Jenkins Pipeline status/progress.
**Step**
A single task; fundamentally steps tell Jenkins _what_ to do inside of a Pipeline or Project.
**Trigger**
A criteria for triggering a new Pipeline run or Build.
**Update Center**
Hosted inventory of plugins and plugin metadata to enable plugin installation from within Jenkins.
**Upstream**
A configured Pipeline or Project which triggers a separate Pipeline or Project as part of its execution.
**Workspace**
A disposable directory on the file system of a Node where work can be done by a Pipeline or Project. Workspaces are typically left in place after a Build or Pipeline run completes unless specific Workspace cleanup policies have been put in place on the Jenkins Master.
