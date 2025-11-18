# AWS Redshift Cluster using Terraform

This repository contains Terraform code to provision an Amazon Redshift cluster, the required IAM role, and an IAM policy attachment that grants Redshift read-only access to Amazon S3.
It also includes the IAM permissions required for the user/role running Terraform and example SQL commands to test the Redshift cluster after deployment.

## 🚀 Features

Creates an AWS IAM Role for Redshift

Attaches AmazonS3ReadOnlyAccess to the role

Deploys a single-node Redshift cluster

Outputs a ready-to-use environment for running SQL queries

### 📁 Terraform Resources
1. AWS Provider
```
provider "aws" {
  region = "us-east-1"
}
```

3. IAM Role for Redshift
```
resource "aws_iam_role" "redshift_role" {
  name = "terraform-redshift-role"

  assume_role_policy = jsonencode({
    Version = "2012-10-17",
    Statement = [
      {
        Effect = "Allow",
        Principal = {
          Service = "redshift.amazonaws.com"
        },
        Action = "sts:AssumeRole"
      }
    ]
  })
}
```

3. Attach S3 Read-Only Access
```
resource "aws_iam_role_policy_attachment" "redshift_s3" {
  role       = aws_iam_role.redshift_role.name
  policy_arn = "arn:aws:iam::aws:policy/AmazonS3ReadOnlyAccess"
}
```

4. Redshift Cluster
```
resource "aws_redshift_cluster" "example" {
  cluster_identifier = "tf-redshift-cluster"
  database_name      = "dev"
  master_username    = "awsuser"
  master_password    = "YourPassword123!"
  node_type          = "ra3.xlplus"
  cluster_type       = "single-node"

  iam_roles = [aws_iam_role.redshift_role.arn]

  skip_final_snapshot = true
}
```

### 🔐 Required IAM Permissions for Terraform User

The IAM user or role running Terraform must have the following permissions:
```
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "redshift:CreateCluster",
        "redshift:ModifyCluster",
        "redshift:DeleteCluster",
        "redshift:DescribeClusters",
        "redshift:DescribeOrderableClusterOptions",
        "redshift:DescribeLoggingStatus",
        "redshift:EnableLogging",
        "redshift:DescribeClusterSubnetGroups",
        "redshift:CreateClusterSubnetGroup"
      ],
      "Resource": "*"
    },
    {
      "Effect": "Allow",
      "Action": [
        "ec2:DescribeSubnets",
        "ec2:DescribeVpcs",
        "ec2:DescribeSecurityGroups"
      ],
      "Resource": "*"
    },
    {
      "Effect": "Allow",
      "Action": [
        "iam:CreateRole",
        "iam:GetRole",
        "iam:DeleteRole",
        "iam:AttachRolePolicy",
        "iam:DetachRolePolicy",
        "iam:ListRolePolicies",
        "iam:ListAttachedRolePolicies",
        "iam:ListInstanceProfilesForRole",
        "iam:PutRolePolicy",
        "iam:DeleteRolePolicy"
      ],
      "Resource": "*"
    },
    {
      "Effect": "Allow",
      "Action": [
        "iam:PassRole"
      ],
      "Resource": "arn:aws:iam::<account-id>:role/terraform-redshift-role"
    }
  ]
}
```

Replace <account-id> with your AWS account ID.

### ▶️ Deploying the Infrastructure
1. Initialize Terraform
```
terraform init
```

3. Preview Changes
```
terraform plan
```

4. Apply Changes
```
terraform apply
```

### 🧪 Example SQL Commands

After connecting to your Redshift cluster using your SQL client (e.g., DBeaver, Aginity, Redshift Query Editor), you can run the following SQL commands to create and test a sample table.

Create a Table
```sql
CREATE TABLE users (
    user_id INT IDENTITY(1,1),
    first_name VARCHAR(50),
    last_name VARCHAR(50),
    email VARCHAR(100),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```
Insert Sample Data
```sql
INSERT INTO users (first_name, last_name, email)
VALUES
('John', 'Doe', 'john.doe@example.com'),
('Sarah', 'Lee', 'sarah.lee@example.com'),
('Michael', 'Brown', 'mike.brown@example.com'),
('Emily', 'Clark', 'emily.clark@example.com');
```

Query the Table
```sql
SELECT * FROM users;
```

Create table orders
```sql
CREATE TABLE orders (
    order_id       INTEGER,
    customer_name  VARCHAR(255),
    region         VARCHAR(100),
    product        VARCHAR(255),
    quantity       INTEGER,
    price          DECIMAL(10,2),
    order_date     DATE
);
```

Load data from S3 into the orders table
```sql
COPY orders
FROM 's3://sales-data-bucket-1760066349/sales_data.csv'
IAM_ROLE 'arn:aws:iam::<account-id>:role/terraform-redshift-role'
FORMAT AS CSV
IGNOREHEADER 1
DATEFORMAT 'DD-MM-YYYY';
```

Query the orders table
```sql
SELECT * from orders;
```

### 🧹 Cleanup

To delete all created resources:
```
terraform destroy
```
