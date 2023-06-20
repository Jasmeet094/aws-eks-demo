## AWS EKS - AWS Code PipeLine with AWS EKS Demo (CI/CD)

#### In this demo we are going to create a aws pipeline which we will create simple web application running on EKS cluster and also register a Dns name `pipeline.jasmeetdevops.com` in Route 53 . 


### Prerequisites and Steps to Follow the Demo

#### 1.  An AWS EKS Cluster running
#### 2. This cluster should have running/installed following resources 
```
a. AWS Load Balancer Controller - So that Application Load Balancer gets created.
b. Ingress Class - ALB can use Default Class to create ALB
c. External DNS pod should running = To register DNS record in Route 53
```
#### 3.  Create a ECR Repository 
#### 4. Create a Code Commit Repository  and Upload Below files to Code Commit Repository
```
buildspec.yml
Dockerfile
index.html
kube-manifests/ 

```

#### 5. Create a Iam Role for service Code Build which will access AWS EKS and attach below policy to it. Trust relationships should have permission so that  any role in AWS account can assume this role.
Trust relationships
```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": {
                "AWS": "*"
            },
            "Action": "sts:AssumeRole"
        }
    ]
}
```
Attach this Policy to describe eks
```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": "eks:Describe*",
            "Resource": "*"
        }
    ]
}
```

#### 6. Update EKS Cluster aws-auth ConfigMap with new role created in previous step
    we need to attach the role in ConfigMap `aws-auth` so that Codebuild Service can authenticate to our EKS Cluster. To do so add below lines under `Map-Roles` section

    ```
    mapRoles: |
    - groups:
      - system:masters
      rolearn: arn:aws:iam::018185988195:role/EksCodeBuildKubectlRole
      username: build
      
    ``` 
#### 7. These  are the environment variables which we have use in Codebuild service while creating it
```
REPOSITORY_URI = 307436399520.dkr.ecr.us-west-2.amazonaws.com/new-repo
EKS_KUBECTL_ROLE_ARN = arn:aws:iam::307436399520:role/EksCodeBuildKubectlRole
EKS_CLUSTER_NAME = dev-ops-test
AWS_DEFAULT_REGION = us-west-2
AWS_ACCOUNT_ID = 307436399520
```

#### 8. Then Create CodePipeline 
 
 ```
a. While creating pipeline , Choose Code Commit Repo which we created earlier as source provider. 
b. Click on next  then choose AWS Codebuild for build provider.
c. Create a new project for codebuild.
d. Choose `Ubuntu` for Operating System and Choose Runtime `Standard` and For Image always use `Latest` image version.
e. For environment variables , create 3 variables as mentioned in step 7.
f. For buildspec file let it empty (means use buildspec file)
g. After filling all required places click on Continue to pipeline and it will forward you to the Codepipeline page/
h. For Deploy -> Click on Skip Deploy Stage
i. Then it will start  creating codepipeline and also it will execute it.
```

#### 9. Your first pipeline should failed because the role created by the codebuild doesn't have required permissions to Elastic Container registry. So add the Full ECR access policy to the Role created by Codebuild Project. (Policy name = AmazonEC2ContainerRegistryFullAccess)

#### 10. Now also for this same Role of Codebuild we need to attach a policy with permission to Assume the Role which we created at Step no.5 , We are doing this so that in post-build phase COdeBuld service will assume this role to get the credentials of our account to connect/login to our EKS cluster
```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "VisualEditor0",
            "Effect": "Allow",
            "Action": "sts:AssumeRole",
            "Resource": "arn:aws:iam::307436399520:role/arn:aws:iam::307436399520:role/EksCodeBuildKubectlRole"
        }
    ]
}
```

#### 11. Now Go to CodePipeline and Click on Release Change so that our Code pipeline will run again.

#### 12. Monitor the Pipeline ,it should complete successfully.
#### 13. Go to Route 53 to see if records got created their
#### 14. Also Check ALB if it is in active state.
#### 15. Now access the app URL `devops.jasmeetdevops.com`
#### 16. Now you can see that  our CiCd pipeline is creating a image ,pushing it to ECR and then also deploying the application in fully automated way.
#### 17. After all testing , delete AWS ECR , CodeCommit , Pipeline and CodeBuild project. And also delete running apps and Ingress service.
==============================================================================================

## Buildspec File Explained
#### PreBuild Phase
```
Buildspec file in its `Prebuild Phase` is creating a tag for the Docker image which will be created in `Build Phase` and updating the Image in deployment file of application with the AWS ECR repo and tag along with it is login to the ECR and exporting the kube config file.
```

#### Build Phase
```
In Build Phase it is creating Docker Image
```

#### Post Build Phase

```
In this phase this is pushing the image to the ECR repo. And getting the credentials from Iam Role which we created For Codebuild service to get authorize with EKS cluster and using these credentials it is setting the context of our eks cluster. After all it is applying the yaml files and also sending the artifacts to build.json file
```

