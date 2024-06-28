# 🔨 Hands-on: Staged deployments

In this hands-on lab your will a staged deployment pipelines with approvals.

## Creating and protecting environments

Go to the Environments section :

![Environments](assets/environments-section.png)

Create 1 environment called `Production` and select the **None** option:

![Create environments](assets/new-environment.png)

You should see the all the created environments. Click on the `Production` environment and go to the `Approvals and checks` tab. On the right side, click on the `+` button. Search for **Approvals** and click **Next**.

Add yourself as the `Required approvals` for this environment:

![Approvals](assets/approvals.png)

Click on **Create** 

Click again on the `+` button and search for **Branch control** and set the `Branch name` to `main`:

![Branch control](assets/branch-control.png)

This will ensure that only deployments from the `main` branch will be allowed to the `Production` environment.

You now have a protected environment that requires your approval before deploying:

![Protected environment](assets/prod-environment.png)

## Adding an input for picking environments to manual pipeline trigger

Modify the pipeline YAML file you created in Hands-On Lab 1 ('My First Pipeline'). Add a `boolean` input to the `parameters` section, just before the `job` section you previously created. 
This attribute when set to true will prevent the pipeline from executing the production deployment. 

All the parameters types are available in the [documentation](https://learn.microsoft.com/en-us/azure/devops/pipelines/process/runtime-parameters?view=azure-devops&tabs=script#parameter-data-types)

<details>
<summary>Solution</summary>

```YAML
parameters:
- name: skipProductionDeployment
  displayName: 'Skip the production deployment'
  type: boolean
  default: true
```

</details>

## Chaining pipelines steps and conditional execution

Update the pipeline file to include three stages:  
  
1. **Tests**: Runs on `ubuntu-latest` in parallel with the `Build` stage. This stage includes two jobs:  
    - One for simulating `Unit tests`  
    - Another for `UI tests`  
     
2. **Build**: Wrap the existing job inside a stage and ensure it runs in parallel with the `Tests` stage.  
     
3. **Production**: Runs on `ubuntu-latest` after the `Tests` and `Build` stages. Deploys to the `Production` environment only if selected as the input parameter. To simulate deployment, the job will execute three steps, each logging `Step x deploying...` to the pipeline log and pausing for 10 seconds. 

<details>
<summary>Solution</summary>

```YAML
stages:
- stage: Tests
  jobs:
  - job: 
    displayName: Unit tests
    pool:
      vmImage: 'ubuntu-latest'
    steps:
      - script: echo simulate running your unit tests!
        displayName: 'Run unit tests'
  - job: 
    displayName: UI tests
    pool:
      vmImage: 'ubuntu-latest'
    steps:
      - script: echo "🧪 Testing..."
        displayName: 'UI Test'

- stage: Build
  dependsOn: [] # This will remove the implicit dependency and run in parallel with the stage: Tests above 
  jobs:
  - job: Build
    displayName: 'Build job'
    pool:
      vmImage: 'ubuntu-latest'
    steps:
    - script: |
        echo "🎉 The job started by: $(Build.Reason)"
        echo "🔎 The name of your branch is $(Build.SourceBranchName)."
      displayName: 'Run a multi-line script'

- stage: Production
  condition: ${{ ne(parameters.skipProductionDeployment, true) }}
  dependsOn:
  - Tests
  - Build
  jobs:
  - deployment:
    displayName: Production deploy
    pool:
      vmImage: 'ubuntu-latest'
    environment: Production
    strategy:
     runOnce:
       deploy:
         steps:
          - script: |
              echo "🚀 Step 1..."
              sleep 10
            displayName: 'Step 1'
          - script: |
              echo "🚀 Step 2..."
              sleep 10
            displayName: 'Step 2'
          - script: |
              echo "🚀 Step 3..."
              sleep 10
            displayName: 'Step 3'
```

</details>

Let's run the pipeline manually:

![Manual run](assets/manual-trigger.png)

You can decide to skip the production deployment by checking the checkbox.

Upon first launch, you will see a section to review the permissions for accessing the `Production` environment. Approve this environment. When you reach the production stage, if you haven't skipped it, you will receive a notification to approve or reject it:  

![Approval review](assets/approval-review.png)

Open it and click **Approve**

The pipeline will pause until you approve the deployment or it times out, based on the time limit set in the environment. If this time limit is reached, the entire pipeline will be canceled.

![Permission review](assets/approve-production-deployment.png)

If you accept the deployment you should see something like this:

![Multi Stage deployment result](assets/multi-stage-deployment-result.png)

## Summary

In this lab, you learned how to create and secure environments in Azure DevOps and incorporate them into a pipeline. You also learned how to conditionally execute jobs in parallel and chain jobs using the `dependsOn` keyword. 
