# AWS Architecture Diagram

## Application Stack (`EcsFargateStack`)

```mermaid
flowchart TD
    User(["👤 User"])
    R53["Route 53\nexample.com / www.example.com"]
    CF["CloudFront Distribution\nHTTPS-only, TLS 1.2+\nHTTP/2 & HTTP/3"]
    CFLogs["S3 Bucket\nCloudFront Access Logs"]
    ACMcf["ACM Certificate\nus-east-1\nexample.com + www"]

    subgraph VPC ["VPC (2 AZs)"]
        subgraph PublicSubnets ["Public Subnets"]
            ALB["Application Load Balancer\nHTTP→HTTPS redirect\nTLS termination"]
        end
        subgraph PrivateSubnets ["Private Subnets"]
            subgraph ECSCluster ["ECS Cluster (Container Insights)"]
                Task1["Fargate Task\nnginx:latest\n256 CPU / 512 MB"]
                Task2["Fargate Task\nnginx:latest\n256 CPU / 512 MB"]
                TaskN["... up to 6 tasks\n(Auto Scaling)"]
            end
        end
    end

    ALBLogs["S3 Bucket\nALB Access Logs (90d)"]
    ACMalb["ACM Certificate\nexample.com + www"]
    ECR["ECR Repository\nnginx-app\nScan on push"]
    CWLogs["CloudWatch Logs\n/ecs/nginx (30d)"]
    ASG["Application Auto Scaling\nCPU >70% → scale out\nCPU <30% → scale in\nmin 2 / max 6"]

    User -->|"DNS lookup"| R53
    R53 -->|"Alias"| CF
    CF -->|"HTTPS origin"| ALB
    CF --- ACMcf
    CF -->|"access logs"| CFLogs
    ALB -->|"HTTP:80 → HTTPS:443"| ALB
    ALB --- ACMalb
    ALB -->|"access logs"| ALBLogs
    ALB -->|"forward :80"| Task1
    ALB -->|"forward :80"| Task2
    ALB -->|"forward :80"| TaskN
    Task1 & Task2 & TaskN -->|"pull image"| ECR
    Task1 & Task2 & TaskN -->|"stream logs"| CWLogs
    ASG -.->|"scales"| ECSCluster

    style VPC fill:#e8f4fd,stroke:#2196F3
    style PublicSubnets fill:#fff9c4,stroke:#FFC107
    style PrivateSubnets fill:#e8f5e9,stroke:#4CAF50
    style ECSCluster fill:#f3e5f5,stroke:#9C27B0
```

---

## CI/CD Pipeline Stack (`PipelineStack`)

```mermaid
flowchart LR
    GH["GitHub\nrepo / branch"]
    SM["Secrets Manager\ngithub-token"]

    subgraph Pipeline ["CodePipeline: ecs-fargate-cdk-pipeline"]
        S1["Stage 1\nSource\nGitHub webhook"]
        S2["Stage 2\nBuild\nCodeBuild\nbuildspec.yml\n(docker build + cdk synth)"]
        S3["Stage 3\nApprove\nManual approval"]
        S4["Stage 4\nDeploy\nCodeBuild\ncdk deploy EcsFargateStack"]
    end

    ArtBucket["S3 Artifact Bucket\nversioned / encrypted\n30d lifecycle"]
    BuildLogs["CloudWatch Logs\n/codebuild/ecs-fargate-cdk"]
    ECR2["ECR\nnginx-app"]
    CF2["CloudFormation\nEcsFargateStack"]

    GH -->|"webhook push"| S1
    SM -.->|"OAuth token"| S1
    S1 -->|"SourceOutput"| S2
    S2 -->|"BuildOutput"| S3
    S3 -->|"approved"| S4
    S2 -->|"docker push"| ECR2
    S4 -->|"cdk deploy"| CF2
    S2 & S4 -->|"logs"| BuildLogs
    Pipeline --- ArtBucket
```

---

## Security & Networking Summary

| Layer | Control |
|---|---|
| DNS | Route 53 public hosted zone → CloudFront alias |
| Edge TLS | ACM cert (us-east-1), TLS 1.2+, HTTP/2 & 3 |
| ALB ingress | CloudFront managed prefix list only (pl-3b927c52) |
| ALB → ECS | Security group: ALB SG → ECS SG on :80 |
| ECS tasks | Private subnets, no public IP |
| Image supply | ECR with scan-on-push, 10-image retention |
| Secrets | GitHub token in Secrets Manager |
| Logs | ALB logs (S3, 90d), CloudFront logs (S3), ECS logs (CW, 30d), Build logs (CW, 30d) |
