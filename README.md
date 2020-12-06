# MLOPs - supplementary material

This template contains code and pipeline definition for a machine learning project demonstrating how to automate an end to end ML/AI workflow. The build pipelines include DevOps tasks for data sanity test, unit test, model training on different compute targets, model version management, model evaluation/model selection, model deployment as realtime web service, staged deployment to QA/prod and integration testing.

## Prerequisite
- Active Azure subscription
- At least contributor access to Azure subscription

## Getting Started:

[https://ml-devops-tutorial.readthedocs.io/en/latest/endtoend.html](https://ml-devops-tutorial.readthedocs.io/en/latest/endtoend.html)

## Architecture Diagram

This reference architecture shows how to implement continuous integration (CI), continuous delivery (CD), and retraining pipeline for an AI application using Azure DevOps and Azure Machine Learning. The solution is built on the scikit-learn diabetes dataset but can be easily adapted for any AI scenario and other popular build systems such as Jenkins and Travis. 

![Architecture](/docs/images/Architecture_DevOps_AI.png)


## Architecture Flow

### Train Model
1. Data Scientist writes/updates the code and push it to git repo. This triggers the Azure DevOps build pipeline (continuous integration).
2. Once the Azure DevOps build pipeline is triggered, it runs following types of tasks:
    - Run for new code: Every time new code is committed to the repo, the build pipeline performs data sanity tests and unit tests on the new code.
    - One-time run: These tasks runs only for the first time the build pipeline runs. It will programatically create an [Azure ML Service Workspace](https://docs.microsoft.com/azure/machine-learning/service/concept-azure-machine-learning-architecture?WT.mc_id=academic-0000-taallard#workspace), provision [Azure ML Compute](https://docs.microsoft.com/azure/machine-learning/service/how-to-set-up-training-targets?WT.mc_id=academic-0000-taallard#amlcompute) (used for model training compute), and publish an [Azure ML Pipeline](https://docs.microsoft.com/azure/machine-learning/service/concept-ml-pipelines?WT.mc_id=academic-0000-taallard). This published Azure ML pipeline is the model training/retraining pipeline.

    > Note: The Publish Azure ML pipeline task currently runs for every code change

3. The Azure ML Retraining pipeline is triggered once the Azure DevOps build pipeline completes. All the tasks in this pipeline runs on Azure ML Compute created earlier. Following are the tasks in this pipeline:

    - **Train Model** task executes model training script on Azure ML Compute. It outputs a [model](https://docs.microsoft.com/azure/machine-learning/service/concept-azure-machine-learning-architecture?WT.mc_id=academic-0000-taallard#model) file which is stored in the [run history](https://docs.microsoft.com/azure/machine-learning/service/concept-azure-machine-learning-architecture?WT.mc_id=academic-0000-taallard#run).

    - **Evaluate Model** task evaluates the performance of newly trained model with the model in production. If the new model performs better than the production model, the following steps are executed. If not, they will be skipped.

    - **Register Model** task takes the improved model and registers it with the [Azure ML Model registry](https://docs.microsoft.com/azure/machine-learning/service/concept-azure-machine-learning-architecture?WT.mc_id=academic-0000-taallard#model-registry). This allows us to version control it.

### Deploy Model

Once you have registered your ML model, you can use Azure ML + Azure DevOps to deploy it.

The **Package Model** task packages the new model along with the scoring file and its python dependencies into a [docker image](https://docs.microsoft.com/azure/machine-learning/service/concept-azure-machine-learning-architecture?WT.mc_id=academic-0000-taallard#image) and pushes it to [Azure Container Registry](https://docs.microsoft.com/azure/container-registry/container-registry-intro?WT.mc_id=academic-0000-taallard). This image is used to deploy the model as [web service](https://docs.microsoft.com/azure/machine-learning/service/concept-azure-machine-learning-architecture?WT.mc_id=academic-0000-taallard#web-service).
    
The **Deploy Model** task handles deploying your Azure ML model to the cloud (ACI or AKS).
This pipeline deploys the model scoring image into Staging/QA and PROD environments.

 In the Staging/QA environment, one task creates an [Azure Container Instance](https://docs.microsoft.com/azure/container-instances/container-instances-overview?WT.mc_id=academic-0000-taallard) and deploys the scoring image as a [web service](https://docs.microsoft.com/azure/machine-learning/service/concept-azure-machine-learning-architecture?WT.mc_id=academic-0000-taallard#web-service) on it. 
    
The second task invokes the web service by calling its REST endpoint with dummy data.
    
5. The deployment in production is a [gated release](https://docs.microsoft.com/azure/devops/pipelines/release/approvals/gates?view=azure-devops&WT.mc_id=academic-0000-taallard). This means that once the model web service deployment in the Staging/QA environment is successful, a notification is sent to approvers to manually review and approve the release. Once the release is approved, the model scoring web service is deployed to [Azure Kubernetes Service(AKS)](https://docs.microsoft.com/azure/aks/intro-kubernetes?WT.mc_id=academic-0000-taallard) and the deployment is tested.

### Repo Details

You can find the details of the code and scripts in the repository [here](/docs/code_description.md)

### References
- [Azure Machine Learning(Azure ML) Service Workspace](https://docs.microsoft.com/azure/machine-learning/service/overview-what-is-azure-ml?WT.mc_id=academic-0000-taallard)
- [Azure ML CLI](https://docs.microsoft.com/azure/machine-learning/service/reference-azure-machine-learning-cli?WT.mc_id=academic-0000-taallard)
- [Azure ML Samples](https://docs.microsoft.com/azure/machine-learning/service/samples-notebooks?WT.mc_id=academic-0000-taallard)
- [Azure ML Python SDK Quickstart](https://docs.microsoft.com/azure/machine-learning/service/quickstart-create-workspace-with-python?WT.mc_id=academic-0000-taallard)
- [Azure DevOps](https://docs.microsoft.com/azure/devops/?view=vsts&WT.mc_id=academic-0000-taallard)
