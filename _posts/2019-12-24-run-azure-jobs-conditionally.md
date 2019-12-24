---
  title: Conditional Workflow in Azure Release Pipeline
  header:
    subheading: Run jobs conditionally with manual intervention
  categories:
    - azure-devops
  tags:
    - azure-devops
---

Imagine you have some pre-release checks you want to run in an Azure DevOps Release Pipeline. Depending on the result of the Pre-Release check you may wish to run a `Manual Intervention` task to approve or reject the release.

It can be a little tricky to get this working out the box, but this post will walk you through one way of doing it with classic pipelines.

## Pre-Requisites

1. We'll need some way of persisting variables between jobs in a Stage and this is not supported out of the box in a classic pipeline. 
   
   Before starting, head over to [SetReleaseVariable on GitHub](https://github.com/paxdev/SetReleaseVariable) and follow the instructions to import the `Set Release Variable` task.

## Run Pre-Deployment Checks

1. We're going to add 2 Pipeline Variables. `IsManualInterventionRequired` will be a Boolean flag to tell us whether or not we need to run the `Manual Intervention`. `PreDeploymentResults` is a container for values, results, messages, etc. to display feedback to the user in the `Manual Intervention` window.

   ![Add Pipeline Variables](/assets/posts/Conditional/Conditional-Pipeline-Variables.jpg)


1. Add an `Agent Job` to your stage.

   ![Add Agent Job](/assets/posts/Conditional/Conditional-Add-Agent-Job.jpg)

1. Make sure "Allow scripts to access the OAuth token" is checked.

   ![Add Agent Job](/assets/posts/Conditional/Conditional-Allow-OAuth-Token.jpg)   

1. I've named the Job "Pre-Deployment Checks". Now add a PowerShell task.

   ![Add PowerShell Task](/assets/posts/Conditional/Conditional-Add-PowerShell-Task.jpg)

1. For this example we will simply set the variables manually. Obviously in practice you would set the variables conditionally as an output from your own script

   ![Set Variables](/assets/posts/Conditional/Conditional-Set-Variables.jpg)

1. If your `PreDeploymentResults` outputs a multine string, you will need to follow [these instructions](/azure-devops/multiline-variables/).

1. We need to persist the variable values to the next job. Add the `Set Release Variable` task you imported earlier.

   ![Add Persist Variables](/assets/posts/Conditional/Conditional-Add-Persist-Variables.jpg)

1. Although we seem to be setting the variable to it's own value, it's necessary to do it this way otherwise the current value will be lost at the end of the current stage. Repeat for the `IsManualInterventionRequired` and `PreDeploymentResults` variables. 

   ![Persist Variables](/assets/posts/Conditional/Conditional-Persist-Variables.jpg)


## Conditionally Run Manual Intervention 

1. Now add an `Agentless Job` to host our `Manual Intervention`

   ![Add Agentless Job](/assets/posts/Conditional/Conditional-Add-Agentless-Job.jpg)

1. I've called the task "Confirm Go-Ahead if Pre-Deployment Checks Fail".

   Under `Additional Options` \ `Run this job` make sure that `Custom condition using variable expressions` is selected.

   Set the ``Variable expression to `eq(variables['IsManualInterventionRequired'],'true')`.

   ![Custom Conditions](/assets/posts/Conditional/Conditional-Custom-Conditions.jpg)

1. Now we can add a `Manual Intervention` task.

   ![Add Manual Intervention](/assets/posts/Conditional/Conditional-Add-Manual-Intervention.jpg)

1. You can set the "Display name", which is what will be shown in the Pipeline runs.

   In the instructions, note that we are displaying the `PreDeploymentResults` variable we persisted from the last job.

   You will probably want to choose a User(s) or Group(s) to receive an email when a confirmation is pending. It contains a deep-link to the form to Resume/Reject the deployment.

   ![Manual Intervention](/assets/posts/Conditional/Conditional-Manual-Intervention.jpg)

## The actual deployment

1. For the purposes of this article I am going to add an `Agentless job` called "Do Deployment". In practice you would add a `Deployment group job` or an `Agent job` to run your actual deployment.

   Note that we need to make sure that, under `Additional options`, `Run this job` is set to `Only when all previous jobs have succeeded`.

   ![Do Deployment](/assets/posts/Conditional/Conditional-Do-Deployment.jpg)

## Testing the pipeline

1. If we create a Release, it will now run the Conditional Resume/Reject stage as expected:

   ![Failed Check](/assets/posts/Conditional/Conditional-Failed-Check.jpg)

1. Clicking "Resume" under the stage (or following the link in the notification email) gives us the option to Resume/Reject the deployment and leave a comment as to why.

   ![Resume Reject](/assets/posts/Conditional/Conditional-Resume-Reject.jpg)

1. If we Reject the release, then we can see that it will skip any further jobs in the stage.

   ![Reject](/assets/posts/Conditional/Conditional-Reject.jpg)  

1. If we Resume the release we can see that it will successfully run the deployment to the end.

   ![Resume](/assets/posts/Conditional/Conditional-Resume.jpg)  

1. Finally if we go back to the `Powershell Script` task in the `Pre-Deployment Checks` job and set `$IsManualInterventionRequired = $false`, we can see that the `Manual Intervention` step is now skipped completely and the deployment runs to the end.

   ![Skipped](/assets/posts/Conditional/Conditional-Skipped.jpg)  
 