# Innovate Inc. Cloud Architecture Design (AWS)

## 1. Cloud Environment Structure

### **Accounts Structure:**

- **Management Account**: For consolidated billing, AWS Control Tower, and IAM management.
- **Production Account**: Runs the live application environment (EKS, RDS, etc.).
- **Development Account**: Used for development and testing of new features.
- **Security/Logging Account (Optional)**: For centralized logging, monitoring, and security tooling (e.g., GuardDuty, Security Hub).

### **Justification:**

- **Isolation**: Clear separation between production and non-production environments to reduce risk.
- **Security**: Least privilege access enforced between accounts.
- **Billing Transparency**: Easier to track costs per environment/project.

---

## 2. Network Design

### **VPC Structure:**

- **Region**: `eu-west-1` 
- **VPC with 3 AZs** for High Availability (HA)
- **Subnets**:
  - 3× Public Subnets (for NAT gateways, bastion if needed)
  - 3× Private Subnets (for EKS nodes, RDS, application workloads)
  - 3× Isolated Subnets (for RDS if not accessed via internet)

### **Security Measures:**

- **NAT Gateway**: For secure internet access for private resources.
- **Security Groups**: Strictly allow only required traffic (e.g., frontend → backend, backend → DB).
- **Network ACLs**: Optional extra layer of security.
- **VPC Flow Logs**: Enabled for traffic analysis.
- **PrivateLink/VPC Endpoints**: For secure AWS service access without leaving the AWS network.

---

## 3. Compute Platform (Kubernetes)

### **Service**: Amazon EKS (Elastic Kubernetes Service)

### **Node Groups**:

- Managed Node Groups using EC2 instances (Graviton or Spot instances for savings in dev).
- Split between:
  - **Frontend/Backend workloads** (auto-scaling)
  - **Jobs/Cron pods** (optional for background tasks)

### **Auto Scaling**:

- **Horizontal Pod Autoscaler (HPA)**: For auto-scaling of application pods based on CPU or memory usage.
- **Cluster Autoscaler/Karpenter**: For dynamic node scaling within the cluster.

### **Containerization Strategy:**

- **Image Building**: Use GitHub Actions or AWS CodeBuild for building Docker images.
- **Image Registry**: Store images in Amazon ECR (Elastic Container Registry).
- **Deployment**: Use Helm charts and ArgoCD/GitOps for deploying services to EKS.

### **Security:**

- **IAM Roles for Service Accounts (IRSA)**: Provide necessary permissions to Kubernetes pods.
- **PodSecurityPolicies or OPA/Gatekeeper**: For enforcing security policies at the pod level.
- **Secrets Management**: Use AWS Secrets Manager or Kubernetes secrets with encryption for managing sensitive data.

---

## 4. Database (PostgreSQL)

### **Service**: Amazon RDS for PostgreSQL (Multi-AZ deployment)

### **Justification:**

- **Managed Service**: Handles backups, patching, and maintenance.
- **Multi-AZ**: Provides automatic failover for high availability.
- **Encryption**: Data at rest and in transit is encrypted.

### **Backups & DR**:

- **Automated Backups**: Configure daily backups with a retention period between 7–35 days.
- **Snapshots**: For point-in-time recovery.
- **Cross-Region Snapshots**: Optional for disaster recovery across regions.

### **High Availability**:

- **Multi-AZ**: Database is deployed across multiple availability zones for failover.
- **Subnet Groups**: Used to span multiple Availability Zones (AZs) for redundancy.
- **Security Groups**: Control database access from only trusted sources (e.g., the EKS cluster).

---

## 5. CI/CD Strategy

### **CI/CD Toolchain:**

- **GitHub Actions** or **AWS CodePipeline** for continuous integration and continuous deployment (CI/CD).

### **CI Pipeline**:

- **Lint** code, **test**, and **build Docker images**.
- Push images to **Amazon ECR**.

### **CD Pipeline**:

- **Helm Chart Deployment**: Use Helm for application deployment to Amazon EKS.
- **GitOps**: Use ArgoCD or FluxCD for continuous deployment and managing deployments through GitOps.

### **Security**:

- Use **OIDC** (OpenID Connect) for GitHub Actions to assume IAM roles.
- Apply least-privilege roles for deployments to ensure that only necessary permissions are granted.

---

## 6. High-Level Architecture Diagram

