IAM Complete Notes for DevOps Engineers
What is IAM?
IAM = Identity and Access Management
IAM is used to:
Create users
Create groups
Assign permissions
Control access to AWS resources
Think:
 
Plain Text
 
 
IAM = Security Gatekeeper of AWS
AWS Account Structure
 
Plain Text
 
 
AWS Account
|
├── IAM Users
├── IAM Groups
├── IAM Policies
└── IAM Roles
1. IAM User
Definition
IAM User is an individual human identity.
Examples:
 
Plain Text
 
 
reddemma-user
john-user
devops-user
Real-Time Example
When you join a company:
 
Plain Text
 
 
UST AWS Account
      |
      |
AWS Admin creates
      |
reddemma-user
You receive:
 
Plain Text
 
 
AWS Login URL
Username
Password
and login to AWS Console.
Interview Answer
IAM User represents an individual person who needs access to AWS resources.
2. IAM Group
Definition
IAM Group is a collection of IAM Users.
Example:
 
Plain Text
 
 
DevOps-Team
Developer-Team
QA-Team
Real-Time Example
 
Plain Text
 
 
DevOps Team
|
├── Reddemma
├── Rahul
├── Kumar
└── Kiran
Instead of assigning permissions individually:
 
Plain Text
 
 
Reddemma
Rahul
Kumar
Create one group:
 
Plain Text
 
 
DevOps-Team
and add users.
Interview Answer
IAM Group is used to manage permissions for multiple users collectively.
3. IAM Policy
Definition
Policy defines:
 
Plain Text
 
 
What actions are allowed?
``
Example:
 
JSON
 
 
{
  "Effect": "Allow",
  "Action": [
    "ec2:*"
  ],
  "Resource": "*"
}
Meaning:
 
Plain Text
 
 
Can perform EC2 operations
Types of Policies
AWS Managed Policies
Created by AWS.
Examples:
 
Plain Text
 
 
AmazonS3ReadOnlyAccess
AmazonEC2FullAccess
AdministratorAccess
Customer Managed Policies
Created by your organization.
Example:
 
Plain Text
 
 
Only S3 Read Access
Only EC2 Start/Stop
Interview Answer
IAM Policy is a JSON document that defines permissions for users, groups, or roles.
4. IAM Role
Most Important Topic
What is IAM Role?
IAM Role is a temporary identity used by AWS services.
Think:
 
Plain Text
 
 
User = Human
Role = AWS Service/Application
Real-Time Scenario 1
EC2 access to S3
Without Role:
 
Plain Text
 
 
EC2
|
Access Key
Secret Key
|
S3
Bad Practice ❌
With Role:
 
Plain Text
 
 
EC2
|
IAM Role
|
S3
Best Practice ✅
No credentials stored.
Why Role?
AWS follows:
 
Plain Text
 
 
Zero Trust Security
Even if:
 
Plain Text
 
 
EC2
S3
exist inside same account,
EC2 cannot access S3 automatically.
Permission required.
Real-Time Scenario 2
Lambda writes logs
 
Plain Text
 
 
Lambda
|
IAM Role
|
CloudWatch
``
Real-Time Scenario 3
EKS Pod accesses Secrets Manager
 
Plain Text
 
 
Kubernetes Pod
|
IAM Role
|
Secrets Manager
Easy Memory Trick
IAM User
 
Plain Text
 
 
Who are you?
Answer:
 
Plain Text
 
 
Human
IAM Group
 
Plain Text
 
 
Which Team?
Answer:
 
Plain Text
 
 
DevOps Team
IAM Policy
 
Plain Text
 
 
What permissions?
Answer:
 
Plain Text
 
 
EC2
S3
CloudWatch
IAM Role
 
Plain Text
 
 
What service can access?
``
Answer:
 
Plain Text
 
 
EC2 -> S3
Lambda -> CloudWatch
EKS -> Secrets Manager
Real Company Flow
 
Plain Text
 
 
AWS Account
|
└── DevOps Group
      |
      ├── EC2 Permission
      ├── S3 Permission
      └── CloudWatch Permission
                |
                |
          reddemma-user
Login:
 
Plain Text
 
 
IAM Username
IAM Password
Terraform IAM
Repository Structure
 
Plain Text
 
 
01-iam
|
├── provider.tf
├── main.tf
├── variables.tf
├── outputs.tf
└── terraform.tfvars
``
provider.tf
 
Terraform
 
 
provider "aws" {
  region = "ap-south-1"
}
Create IAM User
main.tf
 
Terraform
 
 
resource "aws_iam_user" "devops_user" {
  name = "reddemma-user"
  tags = {
    Team = "DevOps"
  }
}
Initialize Terraform
 
Shell
 
 
terraform init
Validate
 
Shell
 
 
terraform validate
Plan
 
Shell
 
 
terraform plan
Expected Output
 
Plain Text
 
 
+ aws_iam_user.devops_user
Apply
 
Shell
 
 
terraform apply
Type:
 
Plain Text
 
 
yes
Create IAM Group
 
Terraform
 
 
resource "aws_iam_group" "devops_group" {
  name = "DevOps-Team"
}
Add User to Group
 
Terraform
 
 
resource "aws_iam_user_group_membership" "membership" {
  user = aws_iam_user.devops_user.name
  groups = [
    aws_iam_group.devops_group.name
  ]
}
Create Custom Policy
 
Terraform
 
 
resource "aws_iam_policy" "s3_read_policy" {
  name = "S3ReadOnly"
  policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Effect = "Allow"
        Action = [
          "s3:GetObject",
          "s3:ListBucket"
        ]
        Resource = "*"
      }
    ]
  })
}
``
Attach Policy to Group
 
Terraform
 
 
resource "aws_iam_group_policy_attachment" "attach_policy" {
  group = aws_iam_group.devops_group.name
  policy_arn = aws_iam_policy.s3_read_policy.arn
}
Create IAM Role for EC2
 
Terraform
 
 
resource "aws_iam_role" "ec2_role" {
  name = "ec2-s3-role"
  assume_role_policy = jsonencode({
    Version = "2012-10-17"
    Statement = [{
      Action = "sts:AssumeRole"
      Effect = "Allow"
      Principal = {
        Service = "ec2.amazonaws.com"
      }
    }]
  })
}
Attach AWS Managed Policy
 
Terraform
 
 
resource "aws_iam_role_policy_attachment" "role_attach" {
  role = aws_iam_role.ec2_role.name
  policy_arn = "arn:aws:iam::aws:policy/AmazonS3ReadOnlyAccess"
}
Create EC2 Instance Profile
 
Terraform
 
 
resource "aws_iam_instance_profile" "profile" {
  name = "ec2-profile"
  role = aws_iam_role.ec2_role.name
}
Real Interview Questions
Q1. Difference between User and Role?
Answer:
IAM User is used by humans to access AWS. IAM Role is used by AWS services such as EC2, Lambda, and EKS to access AWS resources without storing credentials.
Q2. Why use IAM Roles instead of Access Keys?
Answer:
IAM Roles provide temporary credentials and eliminate hardcoded access keys, improving security.
Q3. What is Least Privilege?
Answer:
Users and services should receive only the minimum permissions required to perform their tasks.
Q4. Can EC2 access S3 without Role?
Answer:
No. EC2 requires permissions through an IAM Role or access keys. Best practice is IAM Role.
Q5. What IAM resources have you managed?
Answer:
I worked with IAM Users, Groups, Policies, Roles, Instance Profiles, and Role-Based Access Control. Using Terraform, I automated IAM user creation, group membership, policy attachment, and EC2 role provisioning.
One-Line Summary
 
Plain Text
 
 
IAM User = Human
IAM Group = Team
IAM Policy = Permission
IAM Role = AWS Service Access
Terraform = Automates all of the above
If you understand this document completely, you have enough IAM knowledge for most DevOps, Cloud Engineer, and Terraform interviews.
Provide your feedback on BizChat
 
 
You said:
what to do now help me
Copilot said:
You have already done enough IAM theory for now. Stop studying IAM further today. Start Terraform hands-on.
Today's Goal (Don't learn new AWS services)
Complete this:
 
Plain Text
 
 
AWS IAM Understanding ✅
Terraform Installation ✅
AWS CLI Configuration ✅
Create IAM User using Terraform ✅
Step 1: Create Folder Structure in Your GitHub Repo
Inside your repository:
 
Plain Text
 
 
terraform-aws-devops
|
└── aws
    └── 01-iam
        ├── provider.tf
        ├── main.tf
        ├── README.md
Step 2: Create provider.tf
 
Terraform
 
 
provider "aws" {
  region = "ap-south-1"
}
Step 3: Create main.tf
Create one IAM User:
 
Terraform
 
 
resource "aws_iam_user" "reddemma" {
  name = "reddemma-user"
}
Step 4: Configure AWS
Check:
 
Shell
 
 
aws --version
Then:
 
Shell
 
 
aws configure
Enter:
 
Plain Text
 
 
Access Key
Secret Key
ap-south-1
json
Verify:
 
Shell
 
 
aws sts get-caller-identity
``
If output comes, AWS connection is working.
Step 5: Run Terraform
Go to:
 
Shell
 
 
cd aws/01-iam
Run:
 
Shell
 
 
terraform init
Then:
 
Shell
 
 
terraform validate
Then:
 
Shell
 
 
terraform plan
You should see:
 
Plain Text
 
 
+ aws_iam_user.reddemma
If yes, proceed:
 
Shell
 
 
terraform apply
Type:
 
Plain Text
 
 
yes
Step 6: Verify
AWS Console:
 
Plain Text
 
 
IAM
→ Users
You should see:
 
Plain Text
 
 
reddemma-user
Step 7: Push to GitHub
 
Shell
 
 
git add .
git commit -m "Added IAM Terraform Lab"
git push origin main
After Completing This
Tomorrow do:
 
Plain Text
 
 
02-vpc
Then:
 
Plain Text
 
 
Create VPC manually
↓
Create same VPC using Terraform
↓
Push code to GitHub
