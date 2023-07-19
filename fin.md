# Getting Started : Simplify your Full-Stack Environment with Bunnyshell

Introduction to Bunnyshell
--------------------------

Bunnyshell is an Environment as a Service (EaaS) platform that simplifies the creation and management of full-stack environments for development, staging, and production. It provides developers with fully managed and pre-configured development environments, enabling them to focus on building great products. Let's explore the core features and benefits of Bunnyshell that set it apart from other similar platforms.

Summary:
> It's basically provides containers (environments for your work). i.e your frontend, DB, and backend can be in the container or different containers for your desired use case

### Core Features and Functionalities of Bunnyshell EaaS Platform

-   EaaS Model: Bunnyshell follows the Environment as a Service model, where developers can access fully managed and pre-configured environments. EaaS expands on the conventional IaaS paradigm for application development.
-   Streamlined Development: Bunnyshell allows you to perform all stages of your development in one platform. You can deploy your testing, development, and staging environments in minutes.
-   Allows several participants/users to clone environments and experiment with ideas and features independently without the stress of pull requests and breaking changes
-   Enables businesses to deploy an all-in-one application environment systematically and fast
-   Reproduce and test current solution, issues for a different use case in seconds.
-   Cost Management 


Tutorial: Setting Up Full-Stack Environments with Bunnyshell
------------------------------------------------------------

In this tutorial, we will cover two segments: the first segment will demonstrate using Docker to start a full-stack environment, and the second segment will focus on using Terraform.
### Pre-tutorial

Before we start let talk about environments in bunnyshell
    - When working with with environment variables, you can create either a project, environment, or component variable. 
    - When you define your environment variables in your yaml file, they will most likely appear in the the component variables or environment variables depending on what section.
    - You should define your secrets from the UI and click on the secret well because it's a secret and it shouldn't be exposed in the yaml file
---
![secrets](https://)

> Note: The hierarchy at which bunnyshell will select the variable is component 1st, environment 2nd, project variable. Incase you have different values with the same key.



### Segment 1: Using Docker-Compose to Set Up Full-Stack Environment

1.  Setup your full-stack environment with a Dockerfile and a docker-compose file in your project directory.
    - To set up a full-stack environment using Docker, You need to have a docker-compose.yaml file in your GitHub directory. Bunnyshell platform will search your repo and instinctively use the files it needs to create your desired environment. 
Here's an example docker-compose file for a development environment:
<br>
<br>
    ```yaml
    version: '3'
    services:
      backend:
        build:
          context: ./backend
          dockerfile: Dockerfile
        # env_file: ./backend/.env
        ports:
          - "1337:1337"
      frontend:
        build:
          context: ./frontend
          dockerfile: Dockerfile
        ports:
          - "3000:3000"
        depends_on:
          - backend

    ```

2.  Define the services and configurations required for your environment in the docker-compose file.

1.  Bunnyshell automatically starts your environments as defined in the docker-compose.yaml file



### Segment 2: Using Terraform to Set Up Full-Stack Environment

To set up a full-stack environment using Terraform, you can utilize Terraform's infrastructure provisioning capabilities. 
Here's an example of Terraform configuration created after you have setup your terraform code and bunnyshell environment for provisioning infrastructure resources:


```yaml (bunnyshell)
kind: Environment
name: Staging
type: primary
urlHandle: kostzj
environmentVariables:
    AWS_ACCESS_KEY_ID: '<<BNS_SECRET>>'
    AWS_REGION: us-east-1
    AWS_SECRET_ACCESS_KEY: '<<BNS_SECRET>>'

components:

    -
        kind: Terraform
        name: staging-onlinesafety-s3-cdn
        gitRepo: 'https://github.com/Bayurzx/bunnyshell-hackathon.git'
        gitBranch: tf
        gitApplicationPath: /bunnyshell/terraform
        runnerImage: 'hashicorp/terraform:1.5.1'
        deploy:
            - 'cd bunnyshell/terraform'
            - '/bns/helpers/terraform/get_managed_backend > zz_backend_override.tf'
            - 'terraform init -input=false -no-color'
            - 'terraform apply -var "aws_access_key_id={{ env.vars.AWS_ACCESS_KEY_ID }}" -var "aws_secret_access_key={{ env.vars.AWS_SECRET_ACCESS_KEY }}" -var "ID=bunnyshell" -input=false -auto-approve -no-color'
            - 'BNS_TF_STATE_LIST=`terraform show -json`'
            - 'CLOUD_FRONT_DOMAIN=`terraform output --raw CloudFrontDomain`'
            - 'CLOUD_FRONT_URL=`terraform output --raw CloudFrontURL`'
            - 'TRUNCATED_UNIQUE_ID=`terraform output --raw truncated_unique_id`'
            - 'ACCEPTANCE_TEST_STATUS=`terraform output --raw acceptance_test_status`'
        destroy:
            - 'cd bunnyshell/terraform'
            - '/bns/helpers/terraform/get_managed_backend > zz_backend_override.tf'
            - 'terraform init -input=false -no-color'
            - 'terraform destroy -var "aws_access_key_id={{ env.vars.AWS_ACCESS_KEY_ID }}" -var "aws_secret_access_key={{ env.vars.AWS_SECRET_ACCESS_KEY }}" -var "ID=bunnyshell" -input=false -auto-approve -no-color'

        exportVariables:
            - CLOUD_FRONT_DOMAIN
            - CLOUD_FRONT_URL
            - TRUNCATED_UNIQUE_ID
            - ACCEPTANCE_TEST_STATUS
volumes:
    -
        name: db-data
        size: 1Gi
        type: disk


```

When creating your terraform template, it's important to setup the yaml format in a way that the environment houses the component as shouwn in the yaml template above. 
- `Kind`, `name`, `type`,  `environmentVariables` are automatically generated
  - `environmentVariables` is only generated only if you had created Environment variables beforehand.
`urlHandle`: is created after you have created your terraform environment
```yaml
kind: Environment
name: Staging
type: primary
urlHandle: kostzj
environmentVariables:
    AWS_ACCESS_KEY_ID: '<<BNS_SECRET>>'
    AWS_REGION: us-east-1
    AWS_SECRET_ACCESS_KEY: '<<BNS_SECRET>>'

```

You can now define your components to serve whatever functions you want.   

```yaml
components:

    -
        kind: Terraform 
        name: staging-onlinesafety-s3-cdn
        gitRepo: 'https://github.com/Bayurzx/bunnyshell-hackathon.git'
        gitBranch: tf
        gitApplicationPath: /bunnyshell/terraform
        runnerImage: 'hashicorp/terraform:1.5.1'
        deploy:
            - 'cd bunnyshell/terraform'
            - '/bns/helpers/terraform/get_managed_backend > zz_backend_override.tf'
            - 'terraform init -input=false -no-color'
            - 'terraform apply -var "aws_access_key_id={{ env.vars.AWS_ACCESS_KEY_ID }}" -var "aws_secret_access_key={{ env.vars.AWS_SECRET_ACCESS_KEY }}" -var "ID=bunnyshell" -input=false -auto-approve -no-color'
            - 'BNS_TF_STATE_LIST=`terraform show -json`'
            - 'CLOUD_FRONT_DOMAIN=`terraform output --raw CloudFrontDomain`'
            - 'CLOUD_FRONT_URL=`terraform output --raw CloudFrontURL`'
            - 'TRUNCATED_UNIQUE_ID=`terraform output --raw truncated_unique_id`'
            - 'ACCEPTANCE_TEST_STATUS=`terraform output --raw acceptance_test_status`'
        destroy:
            - 'cd bunnyshell/terraform'
            - '/bns/helpers/terraform/get_managed_backend > zz_backend_override.tf'
            - 'terraform init -input=false -no-color'
            - 'terraform destroy -var "aws_access_key_id={{ env.vars.AWS_ACCESS_KEY_ID }}" -var "aws_secret_access_key={{ env.vars.AWS_SECRET_ACCESS_KEY }}" -var "ID=bunnyshell" -input=false -auto-approve -no-color'

        exportVariables:
            - CLOUD_FRONT_DOMAIN
            - CLOUD_FRONT_URL
            - TRUNCATED_UNIQUE_ID
            - ACCEPTANCE_TEST_STATUS

```
- Above, I created a terraform component by simply defining `kind: Terraform`.
- `name: staging-onlinesafety-s3-cdn` defines the name of the component
- `gitRepo`, `gitBranch`, `gitApplicationPath`, as the names implies defines the GitHub repo url, git remote or github branch and the location to the terraform directory
- `runnerImage: 'hashicorp/terraform:1.5.1'` which is basically a lightweight alpine image with terraform components installed to help make with terraform commands.
  - Note that when running scripts with terraform null_providers, only sh scripts may work certain bash syntax may not work as common with alpine images. To fix this you may add another bunnyshell component with an image that might work with your purpose



Additional Features and Resources
---------------------------------

Bunnyshell provides additional features and resources that can enhance your experience:

-   Templates: Bunnyshell offers a collection of templates on GitHub (<https://github.com/bunnyshell/templates/>), which can serve as starting points for setting up common environments or configurations.

Getting Support with Bunnyshell
-------------------------------

If you need any further assistance or support with Bunnyshell, refer to the official documentation (<https://documentation.bunnyshell.com/docs>) for comprehensive guides and resources. The documentation can help you explore advanced features, troubleshoot issues, and find answers to frequently asked questions.

By following this tutorial and leveraging Bunnyshell's powerful features, you can simplify the process of creating and managing full-stack environments, enabling your team to deliver software faster and focus on building great products.