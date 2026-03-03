# Amazon EKS

!!! note "The following instructions are community-authored and may not be up to date."
    It is advised to consult the official [AWS EKS documentation](https://docs.aws.amazon.com/eks/) for the latest best practices and configurations as well as the latest version of the [Assemblyline Helm chart](https://github.com/CybercentreCanada/assemblyline-helm-chart/tree/main/charts/assemblyline).

## Prerequisites

| Tool | Version | Purpose |
|------|---------|---------|
| `aws` CLI | v2+ | AWS resource management |
| `kubectl` | 1.30+ | Kubernetes management |
| `helm` | 3.x | Chart deployment |
| `eksctl` | latest | (Optional) EKS helpers |

Ensure your AWS credentials are configured with permissions for EKS, EC2, S3, ElastiCache, IAM, and ELBv2.

```bash
# Verify AWS identity
aws sts get-caller-identity

# Install kubectl EKS plugin
aws eks update-kubeconfig --name assemblyline-cluster --region us-east-1
```

## AWS Infrastructure

### VPC & Networking

The EKS cluster runs in an existing VPC with 3 subnets across 3 AZs:

| Resource | Value |
|----------|-------|
| VPC | `<YOUR_VPC_ID>` |
| Subnet AZ-a | `<SUBNET_AZ_A>` |
| Subnet AZ-b | `<SUBNET_AZ_B>` |
| Subnet AZ-c | `<SUBNET_AZ_C>` |

Requirements:

-   Subnets must be **private** (with NAT gateway) or **public** with auto-assign public IP
-   Subnets must be tagged for ALB discovery:

    ```bash
    kubernetes.io/cluster/assemblyline-cluster = shared
    kubernetes.io/role/elb = 1              # for internet-facing ALB
    kubernetes.io/role/internal-elb = 1     # for internal ALB (if needed)
    ```

### EKS Cluster

```bash
# Create the EKS cluster
aws eks create-cluster \
  --name assemblyline-cluster \
  --region us-east-1 \
  --kubernetes-version 1.30 \
  --role-arn arn:aws:iam::<AWS_ACCOUNT_ID>:role/<YOUR_CLUSTER_ROLE> \
  --resources-vpc-config \
    subnetIds=<SUBNET_AZ_A>,<SUBNET_AZ_B>,<SUBNET_AZ_C>,\
    securityGroupIds=<YOUR_ADDITIONAL_SG>,\
    endpointPublicAccess=true,\
    endpointPrivateAccess=true

# Wait for cluster to become ACTIVE (~10 minutes)
aws eks wait cluster-active --name assemblyline-cluster --region us-east-1

# Update kubeconfig
aws eks update-kubeconfig --name assemblyline-cluster --region us-east-1
```

### Node Groups

We use a **two-tier architecture** with workload separation via taints and labels:

| Node Group | Instance | Purpose | Count | Taint | Label |
|------------|----------|---------|-------|-------|-------|
| `r6a-infra` | r6a.large (2 vCPU, 16GB) | Core infrastructure | 4 ON_DEMAND | `workload=infra:NoSchedule` | `workload=infra` |
| `m6a-services` | m6a.large (2 vCPU, 8GB) | Service pods | 4-5 ON_DEMAND (autoscales) | *(none)* | `workload=services` |

**Create the infra node group:**

```bash
aws eks create-nodegroup \
  --cluster-name assemblyline-cluster \
  --nodegroup-name r6a-infra \
  --node-role arn:aws:iam::<AWS_ACCOUNT_ID>:role/<YOUR_NODE_GROUP_ROLE> \
  --subnets <SUBNET_AZ_A> <SUBNET_AZ_B> <SUBNET_AZ_C> \
  --instance-types r6a.large \
  --scaling-config minSize=3,maxSize=6,desiredSize=4 \
  --capacity-type ON_DEMAND \
  --ami-type AL2023_x86_64_STANDARD \
  --disk-size 20 \
  --labels workload=infra \
  --taints "key=workload,value=infra,effect=NO_SCHEDULE" \
  --region us-east-1 \
  --tags Project=AssemblyLine4,Environment=prod
```

**Create the services node group:**

```bash
aws eks create-nodegroup \
  --cluster-name assemblyline-cluster \
  --nodegroup-name m6a-services \
  --node-role arn:aws:iam::<AWS_ACCOUNT_ID>:role/<YOUR_NODE_GROUP_ROLE> \
  --subnets <SUBNET_AZ_A> <SUBNET_AZ_B> <SUBNET_AZ_C> \
  --instance-types m6a.large \
  --scaling-config minSize=2,maxSize=10,desiredSize=4 \
  --capacity-type ON_DEMAND \
  --ami-type AL2023_x86_64_STANDARD \
  --disk-size 20 \
  --labels workload=services \
  --region us-east-1 \
  --tags Project=AssemblyLine4,Environment=prod
```

**Wait for both node groups:**

```bash
aws eks wait nodegroup-active --cluster-name assemblyline-cluster --nodegroup-name r6a-infra --region us-east-1
aws eks wait nodegroup-active --cluster-name assemblyline-cluster --nodegroup-name m6a-services --region us-east-1
kubectl get nodes -L workload
```

### Security Groups (Cross-SG Rules)

EKS managed node groups may receive different security groups. Elasticsearch requires port 9300 connectivity between all nodes for cluster formation. Add cross-SG rules if node groups get different SGs:

```bash
# Identify the 3 security groups
# SG1: your custom data-plane SG (if exists)
# SG2: eks-cluster-sg-assemblyline-cluster-* (auto-created by EKS)
# SG3: assemblyline-cluster-node-* (if exists)

# Add cross-SG ingress rules (all traffic between the SGs)
SG1=<YOUR_DATA_PLANE_SG>
SG2=<YOUR_EKS_CLUSTER_SG>
SG3=<YOUR_NODE_SG>

aws ec2 authorize-security-group-ingress --group-id $SG1 --protocol -1 --source-group $SG2 --region us-east-1
aws ec2 authorize-security-group-ingress --group-id $SG2 --protocol -1 --source-group $SG1 --region us-east-1
aws ec2 authorize-security-group-ingress --group-id $SG2 --protocol -1 --source-group $SG3 --region us-east-1
aws ec2 authorize-security-group-ingress --group-id $SG3 --protocol -1 --source-group $SG2 --region us-east-1
```

??? info "Checking node SGs"
     ```bash
     aws ec2 describe-instances --filters Name=tag:eks:nodegroup-name,Values=r6a-infra --query 'Reservations[*].Instances[*].SecurityGroups' --region us-east-1
     ```

### IMDS Hop Limit

!!! note
    New nodes from autoscaling will also need this fix. Consider using a launch template with `MetadataOptions.HttpPutResponseHopLimit: 2` for permanent fix.

EKS managed node groups default to IMDSv2 with hop limit 1. Pods (like the ALB controller) that need instance metadata require hop limit 2. Fix this on all node instances:

```bash
# Fix hop limit on all cluster nodes
for INSTANCE_ID in $(aws ec2 describe-instances \
  --filters "Name=tag:eks:cluster-name,Values=assemblyline-cluster" "Name=instance-state-name,Values=running" \
  --query 'Reservations[*].Instances[*].InstanceId' --output text --region us-east-1); do
  aws ec2 modify-instance-metadata-options \
    --instance-id "$INSTANCE_ID" \
    --http-put-response-hop-limit 2 \
    --region us-east-1
done
```

## AWS Managed Services

### S3 Filestore Bucket

```bash
aws s3api create-bucket \
  --bucket <YOUR_S3_BUCKET> \
  --region us-east-1

# Block public access
aws s3api put-public-access-block \
  --bucket <YOUR_S3_BUCKET> \
  --public-access-block-configuration \
    BlockPublicAcls=true,IgnorePublicAcls=true,BlockPublicPolicy=true,RestrictPublicBuckets=true
```

### ElastiCache Redis

!!! tip "Redis must be accessible from the EKS VPC. Ensure the ElastiCache subnet group uses the same subnets as EKS, and the security group allows port 6379 from the EKS node security groups."

Create a Redis cluster (replication group with 2 nodes):

```bash
aws elasticache create-replication-group \
  --replication-group-id al-redis \
  --replication-group-description "AssemblyLine Redis" \
  --engine redis \
  --cache-node-type cache.t3.medium \
  --num-cache-clusters 2 \
  --cache-subnet-group-name <your-subnet-group> \
  --security-group-ids <your-redis-sg> \
  --region us-east-1
```

### ACM Certificate

Request a certificate for your domain:

```bash
aws acm request-certificate \
  --domain-name assemblyline.example.com \
  --validation-method DNS \
  --region us-east-1

# Note the certificate ARN output — you'll need it for the Helm values
# Complete DNS validation as prompted
```

## IAM Roles & Policies

### Cluster Role

The EKS cluster service role needs:

- `AmazonEKSClusterPolicy`

```
arn:aws:iam::<AWS_ACCOUNT_ID>:role/<YOUR_CLUSTER_ROLE>
```

### Node Group Role

The EC2 node group role needs:

- `AmazonEKSWorkerNodePolicy`
- `AmazonEKS_CNI_Policy`
- `AmazonEC2ContainerRegistryReadOnly`
- `AmazonSSMManagedInstanceCore` (optional, for SSM access)

```
arn:aws:iam::<AWS_ACCOUNT_ID>:role/<YOUR_NODE_GROUP_ROLE>
```

### S3 IRSA Role

Create an IAM role for Service Account (IRSA) to allow pods to access S3 without static credentials:

```bash
# 1. Create OIDC provider for the cluster (one-time)
eksctl utils associate-iam-oidc-provider --cluster assemblyline-cluster --approve --region us-east-1

# 2. Create the IAM role with S3 permissions
# Trust policy must reference the EKS OIDC provider and the specific service account
```

| Parameter | Value |
|-----------|-------|
| Role ARN | `arn:aws:iam::<AWS_ACCOUNT_ID>:role/<YOUR_S3_IRSA_ROLE>` |
| Service Account | `assemblyline` in namespace `al` |
| Bucket | `<YOUR_S3_BUCKET>` |

The role needs this S3 policy:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": ["s3:GetObject", "s3:PutObject", "s3:DeleteObject", "s3:ListBucket"],
      "Resource": [
        "arn:aws:s3:::<YOUR_S3_BUCKET>",
        "arn:aws:s3:::<YOUR_S3_BUCKET>/*"
      ]
    }
  ]
}
```

### ALB Controller IRSA

The AWS Load Balancer Controller needs its own IRSA role with the standard ALB controller IAM policy. See [AWS docs](https://docs.aws.amazon.com/eks/latest/userguide/aws-load-balancer-controller.html).

## Kubernetes Add-ons

### AWS Load Balancer Controller

!!! warning "Pass `--set vpcId=<YOUR_VPC_ID>` (or add `--aws-vpc-id` arg) to bypass IMDS VPC lookup. Without this, the controller will crash if IMDS hop limit is 1."

```bash
# Add the EKS Helm repo
helm repo add eks https://aws.github.io/eks-charts
helm repo update

# Install the controller
helm install aws-load-balancer-controller eks/aws-load-balancer-controller \
  -n kube-system \
  --set clusterName=assemblyline-cluster \
  --set serviceAccount.create=true \
  --set serviceAccount.annotations."eks\.amazonaws\.com/role-arn"=<ALB_CONTROLLER_ROLE_ARN> \
  --set replicaCount=2 \
  --set vpcId=<YOUR_VPC_ID>
```

??? success "Verification"
      ```bash
      # Should show 2/2 READY
      kubectl get deployment -n kube-system aws-load-balancer-controller
      ```

### EBS CSI Driver

Required for gp2 PersistentVolumes (Elasticsearch data):

```bash
# Install as EKS managed add-on
aws eks create-addon \
  --cluster-name assemblyline-cluster \
  --addon-name aws-ebs-csi-driver \
  --service-account-role-arn <EBS_CSI_ROLE_ARN> \
  --region us-east-1
```

??? success "Verification"
      ```bash
      # Should show provisioner: kubernetes.io/aws-ebs
      kubectl get storageclass gp2
      ```

### Cluster Autoscaler (Optional)

If using cluster autoscaler, install it and ensure node group ASG tags include:

```
k8s.io/cluster-autoscaler/enabled = true
k8s.io/cluster-autoscaler/assemblyline-cluster = owned
```

## AssemblyLine Helm Deployment

### Add Helm Repository

```bash
helm repo add assemblyline https://cybercentrecanada.github.io/assemblyline-helm-chart/
helm repo update

# Verify chart availability
helm search repo assemblyline/assemblyline --versions
```

### Create Namespace & Secrets

```bash
# Create namespace
kubectl create namespace al

# The Helm chart auto-generates most secrets, but you may want to pre-create:
# - assemblyline-system-passwords (contains datastore-password)
# These are typically auto-generated on first install.
```

### Helm Values File

Save the following as `deployment/k8s/values.yaml`:

```yaml

# 1. Ingress Configuration (AWS Load Balancer Controller)
ingressAnnotations:
  kubernetes.io/ingress.class: "alb"
  alb.ingress.kubernetes.io/ip-address-type: dualstack
  alb.ingress.kubernetes.io/scheme: internet-facing
  alb.ingress.kubernetes.io/inbound-cidrs: "<YOUR_ALLOWED_CIDRS>"
  alb.ingress.kubernetes.io/target-type: ip
  alb.ingress.kubernetes.io/listen-ports: '[{"HTTP": 80}, {"HTTPS": 443}]'
  alb.ingress.kubernetes.io/target-group-attributes: stickiness.enabled=true,stickiness.lb_cookie.duration_seconds=3600
  alb.ingress.kubernetes.io/certificate-arn: arn:aws:acm:us-east-1:<AWS_ACCOUNT_ID>:certificate/<YOUR_CERT_ID>

tlsSecretName: assemblyline-tls

# 2. Storage Classes (EBS gp2)
persistantStorageClass: gp2

# 3. Assemblyline Configuration
useInternalRedis: false
configuration:
  # 3.1 Redis (ElastiCache - plain TCP, no SSL)
  core:
    metrics:
      redis:
        host: "<YOUR_REDIS_ENDPOINT>"
        port: 6379"
    redis:
      nonpersistent:
        host: "<YOUR_REDIS_ENDPOINT>"
        port: 6379
      persistent:
        host: "<YOUR_REDIS_ENDPOINT>"
        port: 6379

  # 3.2 Filestore (S3 via IRSA)
  filestore:
    storage: ["s3://s3.amazonaws.com?s3_bucket=<YOUR_S3_BUCKET>&use_ssl=True&aws_region=us-east-1"]
    cache: ["s3://s3.amazonaws.com?s3_bucket=<YOUR_S3_BUCKET>&use_ssl=True&aws_region=us-east-1"]

  services:
    default_auto_update: true
  ui:
    fqdn: "assemblyline.example.com"

# 4. Service Accounts (IRSA for S3 Access)

# 4.1 Add annotations for the Scaler service account to use the S3 IRSA role
serviceAccountAnnotations:
   eks.amazonaws.com/role-arn: "arn:aws:iam::<AWS_ACCOUNT_ID>:role/<YOUR_S3_IRSA_ROLE>"

# 4.2 Set the service account for the other core components to use the default IRSA role (if needed)
coreServiceAccountName: "<IRSA_SERVICE_ACCOUNT_NAME>"  # e.g., "assemblyline-core-irsa"

# 5. Internal components
internalFilestore: false
internalELKStack: true
seperateInternalELKStack: true
internalDatastore: true
enableInternalEncryption: false

metricbeat:
  deployment:
    resources:
      requests:
        cpu: "100m"
        memory: "256Mi"
      limits:
        cpu: "500m"
        memory: "1Gi"

# 6. Resource Limits for the AL components
defaultReqCPU: "100m"
dispatcherReqCPU: "100m"
esMetricsReqCPU: "100m"
ingesterReqCPU: "100m"
metricsReqCPU: "100m"
scalerReqCPU: "100m"
uiReqCPU: "100m"
redisPersistentReqCPU: "100m"
redisVolatileReqCPU: "100m"

# 7. Service Server
useAutoScaler: true
serviceServerInstances: 1
serviceServerInstancesMax: 1
serviceServerReqRam: "1Gi"
serviceServerLimRam: "12Gi"
serviceServerReqCPU: "250m"
serviceServerLimCPU: "4000m"

# 8. Node affinity: schedule core pods ONLY on infra-tainted nodes
tolerations:
  - key: "workload"
    operator: "Equal"
    value: "infra"
    effect: "NoSchedule"

nodeAffinity:
  requiredDuringSchedulingIgnoredDuringExecution:
    nodeSelectorTerms:
      - matchExpressions:
          - key: workload
            operator: In
            values:
              - infra

# 9. Elastic configurations for Kibana and Elasticsearch
kibana:
  resources:
    requests:
      cpu: "250m"

datastore:
  volumeClaimTemplate:
    storageClassName: gp2
  resources:
    requests:
      cpu: "250m"
    tolerations:
     - key: "workload"
       operator: "Equal"
       value: "infra"
       effect: "NoSchedule"
    affinity:
      nodeAffinity:
        requiredDuringSchedulingIgnoredDuringExecution:
          nodeSelectorTerms:
            - matchExpressions:
              - key: workload
                operator: In
                values:
                  - infra

log-storage:
  replicas: 2
  volumeClaimTemplate:
    storageClassName: gp2
  resources:
    requests:
      cpu: "250m"
  tolerations:
     - key: "workload"
       operator: "Equal"
       value: "infra"
       effect: "NoSchedule"
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
          - matchExpressions:
              - key: workload
                operator: In
                values:
                  - infra
```

### Install the Chart

```bash
helm install assemblyline assemblyline/assemblyline \
  -n al \
  -f deployment/k8s/values.yaml \
  --version 7.1.6 \
  --wait --timeout 15m

# Wait for pods to start (this will take several minutes as Elasticsearch and other stateful services initialize)
watch kubectl get pods -n al
```

??? success "Verification"
    ```bash
    # 1. All pods running
    echo "=== Pod Status ==="
    kubectl get pods -n al | grep -v Running | grep -v Completed | grep -v NAME

    # 2. Elasticsearch cluster health
    kubectl exec -n al datastore-master-0 -c datastore -- \
      curl -s -u "elastic:$(kubectl get secret -n al assemblyline-system-passwords -o jsonpath='{.data.datastore-password}' | base64 -d)" \
      'http://localhost:9200/_cluster/health?pretty'

    # 3. Node placement verification
    echo "=== Infra pods ==="
    for node in $(kubectl get nodes -l workload=infra -o name); do
      n=$(echo $node | sed 's|node/||')
      echo "  $n: $(kubectl get pods -n al --field-selector spec.nodeName=$n --no-headers | wc -l) pods"
    done
    echo "=== Service pods ==="
    for node in $(kubectl get nodes -l workload=services -o name); do
      n=$(echo $node | sed 's|node/||')
      echo "  $n: $(kubectl get pods -n al --field-selector spec.nodeName=$n --no-headers | wc -l) pods"
    done

    # 4. ALB target group health
    aws elbv2 describe-target-groups --region us-east-1 \
      --query 'TargetGroups[?starts_with(TargetGroupName,`k8s-al-`)].TargetGroupArn' --output text | \
      tr '\t' '\n' | while read tg; do
        echo "=== $(echo $tg | grep -oP 'k8s-al-[^/]+') ==="
        aws elbv2 describe-target-health --target-group-arn "$tg" --region us-east-1 \
          --query 'TargetHealthDescriptions[*].{IP:Target.Id,State:TargetHealth.State}' --output table
      done

    # 5. Test the API
    curl -sk -o /dev/null -w "HTTP %{http_code} in %{time_total}s\n" https://assemblyline.example.com/api/v4/user/whoami/
    # Expected: HTTP 401 (unauthenticated is normal)

    # 6. Check for broken service pods
    for pod in $(kubectl get pods -n al -o name | grep alsvc- | grep -v updates); do
      kubectl logs -n al $(echo $pod | sed 's|pod/||') --tail=3 2>&1 | \
        grep -q "Waiting for receive task" && echo "BROKEN: $pod"
    done
    ```

## Known Issues & Workarounds

| Issue | Impact | Workaround | Permanent Fix |
|-------|--------|------------|---------------|
| IMDS hop limit 1 on new nodes | ALB controller can't get VPC ID | Set `--aws-vpc-id` + fix hop limit on instances | Launch template with `HttpPutResponseHopLimit: 2` |
| Cross-SG connectivity | ES cluster formation fails | Manual cross-SG ingress rules | Ensure all node groups share the same SG |
| Terraform drift | IaC not in sync | None (CLI-managed) | Import all resources into Terraform state |
| ALB rule priority | WebSocket misrouted | Correct priority: socketio before frontend | Stable after initial fix |
