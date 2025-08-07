# 🧱 AWS EKS Terraform Module

This Terraform module provisions an **Amazon EKS (Elastic Kubernetes Service)** cluster along with managed node groups in an existing VPC using specified subnets.

> ⚠️ This module is intended for learning, sandbox, and controlled AWS environments due to the use of **hardcoded IAM roles**. See [Notes](#notes) for details.

---

## 📌 Module Overview

This module creates the following AWS resources:

- ✅ EKS Cluster (`aws_eks_cluster`)
- ✅ EKS Managed Node Groups (`aws_eks_node_group`)
- ❌ IAM Roles (commented out, hardcoded due to sandbox limitations)

---

## 📂 Usage Example (Root `main.tf`)

```hcl
module "eks" {
  source = "./modules/eks"

  cluster_name    = "my-prod-cluster"
  cluster_version = "1.30"
  vpc_id          = "vpc-0abcd1234efgh5678"
  subnet_ids      = ["subnet-abc1", "subnet-def2"]

  node_groups = {
    "default-ng" = {
      scaling_config = {
        desired_size = 2
        max_size     = 4
        min_size     = 1
      }
    }
  }
}
```

> 📁 Save the module in `modules/eks` and call it from your root configuration.

---

## 📥 Input Variables

| Name             | Type           | Default                | Description                                                                 |
|------------------|----------------|------------------------|-----------------------------------------------------------------------------|
| `cluster_name`   | `string`       | `"ahmed-eks- module"`  | The name of the EKS cluster.                                                |
| `cluster_version`| `string`       | `"1.30"`               | The version of the EKS control plane.                                       |
| `vpc_id`         | `string`       | `""`                   | The ID of the VPC (currently unused but reserved for future improvements).  |
| `subnet_ids`     | `list(string)` | `[]`                   | List of subnet IDs for the EKS cluster and node groups.                     |
| `node_groups`    | `map(object)`  | _Required_             | Configuration for each managed node group (see structure below).            |

### ✅ `node_groups` Structure

```hcl
node_groups = {
  "group-name" = {
    scaling_config = {
      desired_size = number
      max_size     = number
      min_size     = number
    }
  }
}
```

---

## 📤 Output Variables

| Name              | Description                                   |
|-------------------|-----------------------------------------------|
| `cluster_name`    | Name of the EKS cluster                       |
| `cluster_version` | EKS Kubernetes version                        |
| `subnet_ids`      | List of subnet IDs used in the cluster        |
| `node_groups`     | Node groups definition passed to the module   |

---

## ⚙️ Architecture Diagram

```text
                              +---------------------+
                              |     IAM Role        |
                              |  (Hardcoded Role)   |
                              +----------+----------+
                                         |
                                         v
                        +-------------------------------+
                        |         EKS Cluster           |
                        |         (Control Plane)       |
                        +-------------------------------+
                                   |        |
          +------------------------+        +------------------------+
          |                                                    |
          v                                                    v
+---------------------+                         +---------------------+
|   Managed NodeGroup |                         |   Managed NodeGroup |
|     (EC2 Workers)   |                         |     (Optional)      |
+---------------------+                         +---------------------+
          |                                                    |
          +----------------------------+-----------------------+
                                       |
                               VPC & Subnets (Public/Private)
```

---

## 📝 Notes

- **IAM Roles are hardcoded**:
  - `arn:aws:iam::877773779009:role/LabRole` is used for both the EKS cluster and the node group roles due to AWS sandbox limitations.
  - The module comments out IAM role creation (`aws_iam_role` & `aws_iam_role_policy_attachment`) for flexibility in restricted environments.
  - In production, **replace hardcoded IAM roles with proper dynamic IAM resource definitions**.

- **Access Mode**:
  - `access_config` is set to `API_AND_CONFIG_MAP`, which means the cluster can be accessed by both API and Kubernetes RBAC via the `aws-auth` ConfigMap.
  - The creator will have full admin permissions on the cluster.

- **Terraform Version Compatibility**:
  - Recommended Terraform version: `>= 1.3`
  - Required providers:
    - AWS Provider: `>= 5.0`

---

## ✅ Recommendations for Production Use

1. **Replace hardcoded IAM roles**:
   - Use dynamic role creation with appropriate trust policies and permissions.

2. **Enable logging**:
   - Add `enabled_cluster_log_types = [...]` to `aws_eks_cluster`.

3. **Set up security groups explicitly**:
   - Control access to cluster and nodes using custom SGs.

4. **Use Terraform workspaces or separate state files** for different environments (dev/staging/prod).

5. **Add resource tagging**:
   - Add meaningful tags to your cluster and node groups for cost tracking and management.

---

## 👤 Author

**Ahmed Osama Taha**  
Cloud & DevOps Engineer  
📧 ahmed.osama.taha2@gmail.com  
🔗 [LinkedIn](https://linkedin.com/in/ahmedosamataha)  
🔗 [GitHub](https://github.com/Ahmed-Osama-Taha)

---

## 📄 License

MIT License – Use at your own risk. Contributions welcome!
