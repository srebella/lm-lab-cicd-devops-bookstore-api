# Pushing your image to container registry

In this stage we'll update your CircleCI pipeline to push to your [AWS Elastic Container Registry (ECR)](https://aws.amazon.com/ecr/)

### Pre-requisites

You'll need to have a container registry ready to push your docker images to. The terraform configuration covered in a [previous session](https://github.com/techreturners/lm-lab-eks-terraform-devopsupskill) provisioned your own Elastic Container Registry. You can use this to re-create your container registry.

Alternatively you can create a container registry manually using the AWS console.

### Step 1 - Update config.yml

Remember back on the initial setup we utilised a Python CircleCI orb for building the Python application and running the tests. We'll now take that same approach and use the [aws-ecr orb](https://circleci.com/developer/orbs/orb/circleci/aws-ecr) for building the image and pushing to AWS ECR.

Update your `config.yml` to have the following contents:

**NOTE:** You can also see an [example of this configuration](../.circleci/aws.example.config.yml)

```yaml
version: 2.1

orbs:
  python: circleci/python@1.5.0
  aws-ecr: circleci/aws-ecr@7.3.0

workflows:
  test-build-push-image:
    jobs:
      - python/test:
          args: '--dev'
          pkg-manager: pipenv
          test-tool: unittest 
      - aws-ecr/build-and-push-image:
          repo: devops-bookstore-api
          tag: '1.$CIRCLE_BUILD_NUM'
          requires:
            - "python/test"
```

If we explore the changes - you can see that we continue to utilise the Python orb and introduce the new `aws-ecr` orb.

We also introduce a new `job` called `aws-ecr/build-and-push-image`. Let's explore the setup of that job

Firstly we define the repository we'll be pushing to `repo: devops-bookstore-api` - this name will need to match the exact name of your container registry.

Then we define how the Docker image will be tagged. CircleCI provides us with some [built in environment variables](https://circleci.com/docs/2.0/env-vars/#built-in-environment-variables) such as the build number. We make use of that to specify the tag as `1.$CIRCLE_BUILD_NUM`. For example if it was build number 7 then the tag of the image would be **1.7**.

Finally we have the stage `requires` stage. By default CircleCI will run jobs in parallel. By specifying `requires` it means this stage of the build will NOT run until the `python/test` stage has completed.

**NOTE:** Don't commit and push your changes just yet until we've configured Circle with access to your AWS account. If you do accidentally commit and push, don't worry the build will just fail at this point. 

### Step 2 - Configure Circle with AWS access

Currently the [aws-ecr orb](https://circleci.com/developer/orbs/orb/circleci/aws-ecr) doesn't know how to access your AWS account.

If you browse the orb documentation you can see it mentions different configuration parameters such as the `account-url`, `aws-access-key-id` and the `aws-secret-access-key`. Those types of *secrets* shouldn't be kept within the code so we'll make use of [storing variables in the CircleCI project](https://circleci.com/docs/2.0/env-vars/#setting-an-environment-variable-in-a-project).


