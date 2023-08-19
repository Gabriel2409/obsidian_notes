#azure #todo

https://www.youtube.com/watch?v=aonA7Kb7WGE

## Devops philosophy

- What:
  - Development + Operations
  - Philosophy: operation should work like one unit
- PETULA
  - Planning
  - Execution
  - Testing
  - UAT
  - go Live
  - Assistance and support
- Devops is not a position, it is not a department, it is not about automation

## Azure Devops Overview

Dev part

- Planning phase is done in the Boards
- Execution phase is done in Repos
- Testing is done in Test Plans

ops part

- Pipelines will create the build
- Output of the pipeline is an Artifact that goes in the UAT

Then there is go live and support
![[DevOps_azure.excalidraw]]

## Connecting to azure portal

- Create Azure devops Organisation from Azure portal (Use default directory, not local account)
- Set up billing to point to the active subscription / management group in Organisation settings
- In the devops project, go to Pipelines then Service Connections. Create a new Service connection: Azure Resource Manager and grant access to all pipelines: this will allow the pipeline to deploy resources on azure

## Pipelines

- To do the build, we need a machine. This is executed on an agent = software which runs inside a machine and executes the pipeline task. Final output is an artifact

### Local agent

Note that if you try to use a microsoft hosted agent, you can have a parallelism error. There is a link to request more agents but in the mean time it is possible to create your own agent and launch it on your local machine

- Go to the organisation settings, then Agen pools > Add pool > Self hosted and give a name such as `desktopagent`
- In pipeline permission, tick Grant access permission to all pipelines and Auto-provision this agent pool in all projects
- Click on the created agent, download it and extract it
- Then in powershell, run `.\config.cmd` to configure it and `.\run.cmd` to launch it in interactive mode (you will need to create a personal access token with read/write access on agent pools)

In the pipeline yaml file, replace

```yaml
pool:
  vmImage: '<vmtouse>'
```

with

```yaml
pool:desktopagent
```

Now your pipeline will use the agent on your computer to build
