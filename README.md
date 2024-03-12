# Investment Analytics 
Development of investment analytics involves gathering, browsing through and cleaning a large amount of data on portfolio positions, prices, product reference, cashflow and risk metrics. This data is then joined, aggregated and transformed in multiple combinations and sequential orders to develop the final analytics. The fastest way to develop such analytics is to use sql for linear calculations and python for nonlinear calculations while maintaining a holistic view of all the data and analytics we are developing. [DBT](https://www.getdbt.com/) and [SQLMesh](https://sqlmesh.com/) are two great tools to build data products fast in a holistic way.

## Challenge

Building of any such analytics needs the following components in addition to tools such as DBT and SQLMesh.
1. A platform for data storage.
2. A platform for setting up automatic jobs for building analytics.
3. A development environment for python, DBT and SQLMesh with CI/CD pipelines to update the code that is used in the automatic jobs.

The challenge is to setup a minimum viable suite of components described above on AWS as fast as possible.
# Framework for Data and Analytics development on AWS

Here is one minimum viable setup with all the components described in the previous section. It uses the following services.
## Data Storage
**Redshift** with both native tables and **S3** data imported into redshift as external tables. 
## Automatic Jobs 
We can use **AWS StepFunctions** to orchestrate AWS Lambdas and ECS Fargate tasks. We can use these services to build complex DAG workflows either triggered periodically or in response to events such as loading excel files to S3 or a button press. To make it easy to setup the automatic jobs, we can use serverless framework which helps us to setup AWS infrastructure using code. We will start with an initial project that can deploy the following on AWS:
   1. A slim container with python and AWS CLI in which all the AWS lambda and Fargate tasks will be run.
   2. Fargate task definitions
   3. Lambda function definitions
   4. StepFunctions that will define the workflow and the order of execution of Fargate tasks and Lambdas.
### Slim Containers
To run python code in a container, the container needs to have all the necessary python modules and the python code. One way to ensure this is to build a different container for each task. The disadvantage of this approach is that the number of containers can explode and the containers can become quite bloated. To avoid this, we use a slim container with only python and AWS CLI. 
For each task, we can prepackage a virtual environment with all the necessary python modules and the associated python code as compressed tar.gz files in an S3 bucket. The paths to the virtual environment and the python code are passed as environment variables to the container. The container downloads the code and virtual environment from S3 at these paths and executes the shell script passed as another environment variable.
## Development Environment
We will use VSCode as the development environment with the following extensions:
1. dbt Power User
2. VSCode-dbt

We can use CI/CD pipelines to automatically build the compressed tar.gz files for the virtual environment and the python code, and copy to S3 as we commit changes. The CI/CD pipeline for the serverless framework project will create or update lambda, fargate task and step function definitions as we commit changes to the code. This ensures that any changes to the python code or the serverless framework code will be automatically reflected in the automatic jobs. It will be convenient to implement the pipelines if the github repositories are linked in AWS CodeCatalyst and are implemented as CodeCatalyst workflows.

## Architecture

![Architecture](./Arch.drawio.png)
