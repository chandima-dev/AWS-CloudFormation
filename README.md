Here's a sample **README.md** file for your GitHub repository, describing the steps to deploy the Laravel application on an EC2 instance with an RDS database using the provided CloudFormation template.

---

# Laravel Application Deployment using CloudFormation

This repository contains a CloudFormation template to deploy a Laravel web application on an EC2 instance with an RDS MySQL database. The application is deployed within a secure and optimized AWS infrastructure, leveraging best practices for networking, security, and resource provisioning.

## Requirements

- **AWS Account**: You need an AWS account to deploy resources.
- **AWS CLI**: Ensure that you have the AWS CLI installed and configured on your local machine for managing AWS resources.
- **Key Pair**: Create an EC2 key pair (if not already available) to SSH into the EC2 instance.

## CloudFormation Template Overview

The CloudFormation template (`cf-deployer.yml`) deploys the following resources:

1. **VPC**: A custom VPC for networking.
2. **Subnets**:
   - **Public Subnet**: For hosting the EC2 instance.
   - **Private Subnet**: For hosting the RDS database.
3. **Security Group**: Allows SSH (port 22) and HTTP (port 80) traffic for the EC2 instance.
4. **EC2 Instance**: An Amazon EC2 instance running Amazon Linux 2 (Free Tier eligible), which will host the Laravel application.
5. **RDS MySQL Database**: A MySQL database in a private subnet, used by the Laravel application to store data.

## Steps to Deploy

### 1. Clone the Repository
Clone this repository to your local machine or directly into your AWS Cloud9 instance.

```bash
git clone https://github.com/yourusername/your-repository-name.git
cd your-repository-name
```

### 2. Create a CloudFormation Stack

You can create the CloudFormation stack via the AWS Management Console or AWS CLI.

#### Using AWS Management Console:

1. Navigate to the **CloudFormation** console.
2. Click on **Create Stack** and select **Upload a template file**.
3. Upload the `cf-deployer.yml` file from this repository.
4. Provide a name for your stack (e.g., `LaravelStack`).
5. For the **DBPassword** parameter, enter a password for your MySQL database (it will be kept hidden).
6. Click **Next** and follow the steps to create the stack.

#### Using AWS CLI:

Alternatively, you can use the AWS CLI to create the stack:

```bash
aws cloudformation create-stack \
    --stack-name LaravelStack \
    --template-body file://cf-deployer.yml \
    --parameters ParameterKey=DBPassword,ParameterValue=your_db_password \
    --capabilities CAPABILITY_NAMED_IAM
```

### 3. Access the EC2 Instance

After the stack creation is complete, you'll have an EC2 instance running in the public subnet. You can access it using SSH.

1. Locate your EC2 instance’s **Public IP** in the CloudFormation stack outputs or EC2 dashboard.
2. SSH into the EC2 instance:

```bash
ssh -i /path/to/your/keypair.pem ec2-user@your-ec2-public-ip
```

### 4. Install Docker and Set Up Laravel

Once logged into your EC2 instance, follow these steps to install Docker and deploy the Laravel application:

#### Install Docker:
```bash
sudo yum update -y
sudo amazon-linux-extras install docker
sudo service docker start
sudo usermod -a -G docker ec2-user
exit
```

Log back in to ensure the changes take effect:
```bash
ssh -i /path/to/your/keypair.pem ec2-user@your-ec2-public-ip
```

#### Pull Laravel Docker Image and Set Up the Application:
You can either create a custom `Dockerfile` or use an official PHP Docker image to deploy the Laravel application. For this example, we’ll use an official PHP Docker image to pull and deploy Laravel.

1. **Clone the Laravel repository**:
```bash
git clone https://github.com/laravel/laravel.git /home/ec2-user/laravel
cd /home/ec2-user/laravel
```

2. **Create a Dockerfile** in the project directory (optional, if you want a custom Docker setup):
```dockerfile
# Use the official PHP image with Apache
FROM php:8.0-apache

# Enable Apache mod_rewrite
RUN a2enmod rewrite

# Set the working directory
WORKDIR /var/www/html

# Copy Laravel files into the container
COPY . .

# Install necessary dependencies
RUN docker-php-ext-install pdo pdo_mysql

# Expose port 80
EXPOSE 80
```

3. **Build and Run the Docker Container**:
```bash
docker build -t laravel-app .
docker run -d -p 80:80 laravel-app
```

4. **Configure Laravel to Connect to the RDS Database**:
   - Open the `.env` file in your Laravel project.
   - Update the database configuration with the RDS endpoint and credentials:
     ```env
     DB_CONNECTION=mysql
     DB_HOST=your-rds-endpoint
     DB_PORT=3306
     DB_DATABASE=LaravelDB
     DB_USERNAME=admin
     DB_PASSWORD=your_db_password
     ```

### 5. Access the Laravel Application

Once the Docker container is running, your Laravel application should be accessible via the **Public IP** of the EC2 instance. Open your browser and go to `http://your-ec2-public-ip`.

### 6. Clean Up

Once you are done, you can delete the CloudFormation stack to remove the resources:

```bash
aws cloudformation delete-stack --stack-name LaravelStack
```

---

## Repository Structure

```
/your-repository-name
  ├── cf-deployer.yml         # CloudFormation template
  └── README.md               # This file
```

---

## Notes

- Ensure that your **AWS credentials** are properly configured when using the AWS CLI.
- This deployment uses **Free Tier** eligible resources to minimize costs.
- Replace any placeholders (e.g., `your_db_password`, `your-ec2-public-ip`) with actual values during deployment.
- Always follow **AWS best practices** for security, especially when handling database credentials and sensitive information.

---

This **README.md** provides a step-by-step guide for deploying your Laravel application using the CloudFormation template, including setting up Docker and configuring your Laravel application to connect to the RDS database.

