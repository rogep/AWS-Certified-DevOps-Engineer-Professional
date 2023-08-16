# ECS & EKS

## EC2 vs ECS (EC2) vs Fargate

- ECS

  - if you are already using containers

- EC2 mode

  - Large workload
  - Price conscious

- Fargate
  - Large workload
  - Overhead conscious
  - Small / Burst workloads
  - Batch / Periodic workloads

## EKS (Elastic Kubernetes Service)

- AWS Managed Kubernetes - open source and cloud agnostic
- Can be run
  - On AWS
  - Outposts
  - EKS anywhere
  - EKS distro
- Control plane scales and runs on multiple AZs
- Integrates with AWS services
  - ECR, ELB, IAM, VPC
- EKS Cluster = EKS Control Plane & EKS Nodes
- etcd distributed across multiple AZs
- Nodes
  - Self managed
    - WINDOWS, GPU, inferentia, bottlerocket, outputs, local zones... Check node types
  - Managed node groups
  - Fargate pods
- Storage:
  - EBS, EFS, FSx Lustre, FSx for NetApp ONTAP

Deployments you have two VPCs - 1st: AWS MANAGED (control plane managed & multi AZ) - 2nd: Worker nodes (EC2) - Control plane ENIs injected into customer VPC - EKS admin via public endpoint

# CODE\* SUITE - SDLC Automation

Development pipeline

- Code (Repos)
  - CodeCommit
- Build (Jenkins)
  - CodeBuild
- Test (Jenkinks/Unit testing)
  - CodeBuild
- Deploy (Jenkins or other tooling)
  - CodeDeploy

AWS CodePipeline Orchestrates the entire development pipeline.

- A pipeline can only be tied to ONE branch
- Buildspec.yml (codebuild) / appspec.yml (CodeDeploy)
- CodeDeploy can be used to deploy directly to EC2
- Code deploy also can build to
  - AWS Elastic beanstalk
  - AWS OpsWorks
  - AWS CloudFormation
  - Amazon ECS or ECS (blue/green)
  - AWS Service catalog or Alexa Skills Kit
  - Built version onto S3

## AWS CodePipeline

An overview of the CICD system in AWS

- A continuous delivery tool
- Controls the flow from source, through build towards deployment
- Pipelines are built from stages
- Stages cn have sequential or parallel ACTIONS
- Movement between stages can require manual approval
- Artifacts can be loaded into an action, and generated from an action
- State changes => Event bridge (success/failed/cancelled)
- CloudTrail/Console can be used to view/interact

## CodeBuild
