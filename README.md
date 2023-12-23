# Terraform VPC 3-Tier Architecture implementing CloudWatch + ALB + Autoscaling with Launch Templates

This Terraform project aims to create a VPC with a 3-tier architecture on AWS. The architecture consists of a public subnet containing a bastion EC2 instance, a private subnet where ec2 instances will be launch using launch templates with autoscaling, an Apache server installed in the ec2 instances launched. An index.html file along with a metadata  will be displayed for the EC2 instances.

## Prerequisites

- AWS Account: You need an active AWS account.
- AWS CLI: Install and configure AWS CLI on your local machine.
- SSH Key Pair: Generate an SSH key pair and place the private key (key.pem) in the private-key directory.
- Terraform: Install Terraform on your local machine.
- It will be required to have a registered Domain in AWS Route53 to implement this usecase.

## Domain Configuration

This project assumes the use of a registered domain in AWS Route53 for certain components. Before running Terraform, ensure to modify the following files located in the project directory to update the domain name:

1. `terraform-manifest/c6-02-datasource-route53-zone.tf`:
    - Locate and update the `domain_name` field in this file with your registered domain name.
    - Example: Change `<your-route53-domain>.com` to `*.yourdomain.com`.

2. `terraform-manifest/c11-acm-certificatemanager.tf`:
    - Locate and update the `domain_name` field in this file with your registered domain name.
    - Example: Change `*.<your-route53-domain>.com` to `*.yourdomain.com`.

3. `terraform-manifest/c12-route53-dnsregistration.tf`:
    - Update the `name` field to reflect your chosen subdomain.
    - Example: Change `"nlb.<your-route53-domain>.com"` to `"your-subdomain.yourdomain.com"`.

    **Note**: Make sure to replace placeholders with your actual domain and subdomain names before executing Terraform commands.

## SNS Configuration

This project utilizes SNS. Before running Terraform, ensure to modify the following file located in the project directory to update your SNS email:

1. `terraform-manifest/c13-05-autoscaling-notifications.tf`:
    - Locate and update the `endpoint` field and pass the desired email that will be used to send the SNS.
    - Example: Change `<your-email>@gmail.com` to `your-email@gmail.com`.

## AWS CloudWatch Synthetics Configuration

1. Create the nodejs code of your app to be used as blueprint for the canaries. This can also be done using aws mgmt console, to do this do the following:
    - Go to “cloud watch synthetics”
    - Go to “Create canary”
    - Under canary builder and on “Application or endpoint URL” write: `<your-route53-domain>`
    - Now, under “Script editor” aws will throw a nodejs code for your app, copy and paste it inside the file: `terraform-manifest/sswebsite2/nodejs/node_modules/sswebsite2.js`
    - Create a zip file of the following files:

    ```bash
    cd terraform-manifest/sswebsite2
    ```
	
    ```bash
    zip -r sswebsite2v1.zip nodejs
    ```
    
## Usage

1. Clone this repository to your local machine.

2. Ensure Terraform is installed on your system.

3. Set up your AWS credentials using the AWS CLI:

    ```bash
    aws configure
    ```

4. Modify the c2-variable.tf file to adjust any desired settings such as region, instance type, and key pair.

5. Modify variables in terraform.tfvars and other .tfvars files if needed.

6. Run the following commands in the terminal:

    ```bash
    terraform init
    ```

    ```bash
    terraform validate
    ```

    ```bash
    terraform plan -var-file="secrets.tfvars"
    ```

    ```bash
    terraform apply -var-file="secrets.tfvars"
    ```
    
## Verify the resources

1. Verify the following in AWS mgmt console:

    - Confirm SNS Subscription in your email
    - Verify EC2 Instances
    - Verify Launch Templates (High Level)
    - Verify Autoscaling Group (High Level)
    - Verify Load Balancer
    - Verify Load Balancer Target Group - Health Checks
    - Cloud Watch
    - ALB Alarm
    - ASG Alarm
    - CIS Alarms

2. Access and Test the app

    ```
    http://cloudwatch.<your-route53-domain>.com
    ```
    
    ```
    http://cloudwatch.<your-route53-domain>.com/app1/index.html
    ```

    ```
    http://cloudwatch.<your-route53-domain>.com/app1/metadata.html
    ```

## Cleanup

- Terraform Destroy

    ```bash
    terraform plan -destroy  # You can view destroy plan using this command
    ```

    ```bash    
    terraform destroy -auto-approve
    ```

- Clean-Up Files

    ```bash
    rm -rf .terraform*
    ```

    ```bash
    rm -rf terraform.tfstate*
    ```

## License

This project is licensed under MIT License.