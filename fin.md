<!-- # Getting Started : Simplify your Full-Stack Environment with Bunnyshell -->

Introduction to Bunnyshell
--------------------------

Bunnyshell is an Environment as a Service (EaaS) platform that simplifies the creation and management of full-stack environments for development, staging, and production. It provides developers with fully managed and pre-configured development environments, enabling them to focus on building great products. Let's explore the core features and benefits of Bunnyshell that set it apart from other similar platforms.

### Core Features and Functionalities of Bunnyshell EaaS Platform

-   **EaaS Model**: Bunnyshell follows the Environment as a Service model, where developers can access fully managed and pre-configured environments. EaaS expands on the conventional IaaS paradigm for application development.
-   **Unified Platform**: With Bunnyshell, you can perform all stages of your development in one platform. Say goodbye to complex setups and enjoy the convenience of managing your testing, development, and staging environments in minutes.
-   **Collaboration Made Easy**: Bunnyshell allows multiple participants to clone environments and experiment with ideas and features independently, eliminating the stress of pull requests and breaking changes.
-   **Accelerated Deployment**: Bunnyshell enables businesses to deploy all-in-one application environments systematically and rapidly, enhancing productivity and time-to-market.
-   **Reproduction and Testing**: Easily reproduce and test your current solution or explore different use cases in seconds.
-   **Cost Management**: Bunnyshell helps you optimize costs by efficiently managing your environments and resources.
-   **User-friendly Interface**: The interface was so easy to use it took me a total of 4 hours to get my app running, 3:45min of it has me updating my code to fix minor bugs and reading the docs

Tutorial: Setting Up Full-Stack Environments with Bunnyshell
------------------------------------------------------------

In this tutorial, we will cover two segments: the first segment will demonstrate using Docker to start a full-stack environment, and the second segment will focus on using Terraform.

### Pre-tutorial

Before we start, let's talk about variables in Bunnyshell:
- When working with with environment variables, you can create either a project, environment, or component variable. 

---
> Setup Project Variables
![Env_Prj](https://raw.githubusercontent.com/Bayurzx/Bunnyshell-Article-Getting_Started/article/images/prj_var1.jpg)
![Env_Prj](https://raw.githubusercontent.com/Bayurzx/Bunnyshell-Article-Getting_Started/article/images/prj_var2.jpg)

> Setup Environment Variables
![Env_Var](https://raw.githubusercontent.com/Bayurzx/Bunnyshell-Article-Getting_Started/article/images/env_var_1.jpg)
![Env_Var](https://raw.githubusercontent.com/Bayurzx/Bunnyshell-Article-Getting_Started/article/images/env_var_2.jpg)

> Setup Component Variables
![Comp_Var](https://raw.githubusercontent.com/Bayurzx/Bunnyshell-Article-Getting_Started/article/images/comp_var_.jpg)


- When you define your variables in your YAML file, they will most likely appear in the component variables or environment variables section, depending on the context.
- You should define your secrets from the UI and click on the secret toggle because it's a secret and it shouldn't be exposed in the YAML file.


---
![secrets](https://raw.githubusercontent.com/Bayurzx/Bunnyshell-Article-Getting_Started/article/images/secrets.jpg)

> Note: The hierarchy that Bunnyshell will follow to select the variable is component first, environment second, and project variable last. In case you have different values with the same key.



### Segment 1: Using Docker-Compose to Set Up Full-Stack Environment

1.  Setup your full-stack environment with a Dockerfile and a docker-compose file in your project directory.
    - To set up a full-stack environment using Docker, you need to have a docker-compose.yaml file in your GitHub directory. Bunnyshell platform will search your repo and automatically use the files it needs to create your desired environment.
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

3.  Bunnyshell automatically starts your environments as defined in the docker-compose.yaml file

![running](https://raw.githubusercontent.com/Bayurzx/Bunnyshell-Article-Getting_Started/article/images/3.jpg)

### Segment 2: Using Terraform to Set Up Full-Stack Environment

To set up a full-stack environment using Terraform, you can utilize Terraform's infrastructure provisioning capabilities. 
Here's an example of Terraform configuration created after you have setup your terraform code and bunnyshell environment for provisioning infrastructure resources:


```yaml
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

When creating your terraform template, it's important to setup the yaml format in a way that the environment houses the component as shown in the yaml template above. 
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
-   Above, I created a terraform component by simply defining `kind: Terraform`.
-   `name: staging-onlinesafety-s3-cdn` defines the name of the component.
-   `gitRepo`, `gitBranch`, `gitApplicationPath`, as the names imply, define the GitHub repository URL, git remote or GitHub branch, and the location of the Terraform directory.
-   `runnerImage: 'hashicorp/terraform:1.5.1'` is a lightweight Alpine based image with Terraform components installed to help with Terraform commands.
    -   Note that when running scripts with Terraform null_providers, only sh scripts may work. Certain Bash syntax may not work as commonly expected with Alpine images. To fix this, you may add another Bunnyshell component with an image that is suitable for your purpose.
-   In summary, `deploy` and `destroy` are basically `terraform apply --auto-approve` and `terraform destroy --auto-approve`, respectively.
    -   `aws_secret_access_key` and `aws_access_key_id` are defined as outputs in our Terraform code files, allowing us to connect to AWS.
    -   `{{ env.vars.AWS_ACCESS_KEY_ID }}` and `{{ env.vars.AWS_SECRET_ACCESS_KEY }}` are how we use our defined environment variables (secrets).
-   `exportVariables` are printed out and saved at your component level

![running](https://raw.githubusercontent.com/Bayurzx/Bunnyshell-Article-Getting_Started/article/images/4.jpg)



Additional Features and Resources
---------------------------------

Bunnyshell provides additional features and resources that can enhance your experience:

-   Templates: Bunnyshell offers a collection of templates on GitHub (<https://github.com/bunnyshell/templates/>), which can serve as starting points for setting up common environments or configurations.
-   More detailed understanding on how environment variables work: 
    -   <https://documentation.bunnyshell.com/docs/variables>
    -   <https://documentation.bunnyshell.com/docs/variables-interpolation>

Getting Support with Bunnyshell
-------------------------------

If you need any further assistance or support with Bunnyshell, refer to the official documentation (<https://documentation.bunnyshell.com/docs>) for comprehensive guides and resources. The documentation can help you explore advanced features, troubleshoot issues, and find answers to frequently asked questions.

By following this tutorial and leveraging Bunnyshell's powerful features, you can simplify the process of creating and managing full-stack environments, enabling your team to deliver software faster and focus on building great products.

Round-Up
--------

To get a clearer picture, you can always check out their [templates](https://github.com/bunnyshell/templates). You can also visit my GitHub repository, where the code mentioned above was based: [GitHub](https://github.com/Bayurzx/bunnyshell-hackathon).

Reach Out
---------
If you have any questions or simply want to connect, feel free to reach out to me through the following link.

[<img src="https://img.shields.io/badge/LinkedIn-0077B5?style=for-the-badge&logo=linkedin&logoColor=white" />](https://www.linkedin.com/in/adebayo-omolumo/) 
[<img src="https://img.shields.io/badge/@Adebayoomolumo-1DA1F2?style=for-the-badge&logo=twitter&logoColor=white" />](https://www.linkedin.com/in/adebayo-omolumo/)