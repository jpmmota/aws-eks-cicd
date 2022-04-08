# aws-eks-cicd
Example of a complete DevOps CI/CD pipeline deploying an application on AWS EKS.

# Objective
- Package the [vue3 realworld](https://github.com/mutoe/vue3-realworld-example-app) frontend application into a Docker container image
- Push it to AWS Elastic Container Repository (ECR)
- Create an AWS Elastic Kubernetes Service (EKS) cluster
- Use AWS CodeBuild to automate the CI/CD pipeline by:
  - Check for changes in the code repository
  - Create new docker image, tag it and push it to AWS ECR
  - Update deployment file with new image version and reload k8s cluster configuration

# Steps
1. Fork [vue3 realworld](https://github.com/mutoe/vue3-realworld-example-app), rename the repository to aws-eks-cicd and clone it to your local machine
```bash
git clone https://github.com/jpmmota/aws-eks-cicd.git
cd aws-eks-cicd
```

2. Create Dockerfile in the project root folder

3. Build the container image locally and test it
```bash
docker build -t jpmmota/aws-eks-cicd .
docker run --rm -it -d -p 3000:80 jpmmota/aws-eks-cicd
curl localhost:3000
```
