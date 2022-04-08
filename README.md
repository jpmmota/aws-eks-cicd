# aws-eks-cicd
Example of a complete DevOps CI/CD pipeline deploying an application on AWS EKS.

# Objective
- Package the [vue3 realworld](https://github.com/mutoe/vue3-realworld-example-app) frontend application into a Docker container image
- Push it to AWS Elastic Container Registry (ECR)
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
docker build -t jpmmota/aws-eks-cicd:latest .
docker run --rm -it -d -p 3000:80 jpmmota/aws-eks-cicd:latest
curl localhost:3000
```
4. Create AWS Elastic Container Registry (ECR)
```bash
aws ecr create-repository --repository-name aws-eks-cicd
```

5. Push docker image to AWS (ECR)
```bash
aws ecr get-login-password --region ca-central-1 | docker login --username AWS --password-stdin 577620766037.dkr.ecr.ca-central-1.amazonaws.com
docker build -t aws-eks-cicd:1.0 .
docker tag aws-eks-cicd:1.0 577620766037.dkr.ecr.ca-central-1.amazonaws.com/aws-eks-cicd:1.0
docker push 577620766037.dkr.ecr.ca-central-1.amazonaws.com/aws-eks-cicd:1.0
```

6. Create AWS Elastic Kubernetes Service (EKS) cluster
```bash
eksctl create cluster --name eks-cicd --node-type t2.micro --nodes 2
```

7. Create create-role.json file
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
      "Service": "codebuild.amazonaws.com"
    },
      "Action": "sts:AssumeRole"
    }
  ]
}
```

8. Create put-role-policy.json file:
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "CloudWatchLogsPolicy",
      "Effect": "Allow",
      "Action": [
        "logs:CreateLogGroup",
        "logs:CreateLogStream",
        "logs:PutLogEvents"
      ],
      "Resource": "*"
    },
    {
      "Sid": "CodeCommitPolicy",
      "Effect": "Allow",
      "Action": [
        "codecommit:GitPull"
      ],
      "Resource": "*"
    },
    {
      "Sid": "S3GetObjectPolicy",
      "Effect": "Allow",
      "Action": [
        "s3:GetObject",
        "s3:GetObjectVersion"
      ],
      "Resource": "*"
    },
    {
      "Sid": "S3PutObjectPolicy",
      "Effect": "Allow",
      "Action": [
        "s3:PutObject"
      ],
      "Resource": "*"
    },
    {
      "Sid": "S3BucketIdentity",
      "Effect": "Allow",
      "Action": [
        "s3:GetBucketAcl",
        "s3:GetBucketLocation"
      ],
      "Resource": "*"
    },
    {
      "Sid": "ElasticContainerRegistryPolicy",
      "Effect": "Allow",
      "Action": [
        "ecr:*"
      ],
      "Resource": "*"
    },
    {
      "Sid": "CodeBuildAssumeRolePolicy",
      "Effect": "Allow",
      "Action": [
        "sts:AssumeRole"
      ],
      "Resource": "*"
    },
    {
      "Sid": "EKSPolicy",
      "Effect": "Allow",
      "Action": [
        "eks:*"
      ],
      "Resource": "*"
    }
  ]
}
```

9. Apply policies
```bash
aws iam create-role --role-name CodeBuildServiceRole --assume-role-policy-document file://create-role.json
aws iam put-role-policy --role-name CodeBuildServiceRole --policy-name CodeBuildServiceRolePolicy --policy-document file://put-role-policy.json
```

10. Add user to eks rbac configuration
```bash
eksctl create iamidentitymapping --cluster eks-cicd --arn <EKS_ROLE_ARN> --group system:masters --username CodeBuildServiceRole
```

11. Create deployment.yaml and service.yaml files

12. Create buildspec.yaml file that will be used by AWS CodeBuild

