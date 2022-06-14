# AUTOMATE INFRASTRUCTURE WITH IAC USING TERRAFORM PART 1

## Prerequisites
- Create an IAM user named **terraform**, ensure the user has only programmatic access and grant the AdmninistratorAccess permissions. 
<img width="983" alt="Screenshot 2022-06-14 at 17 29 21" src="https://user-images.githubusercontent.com/33035619/173629225-f85bf8b0-f909-4298-91ce-f78bc4550d3d.png">

- Download the secret access key and access key ID CSV file.
- Install boto3 using pip
  ```
  pip install boto3
  ```
- Install AWS CLI and configure programmatic access using
  ```
  aws configure
  ```
  Paste in the access key ID and secret access key when prompted. Leave everything else as default.
- Create an S3 bucket to store Terraform state files.
 <img width="983" alt="Screenshot 2022-06-11 at 17 08 23" src="https://user-images.githubusercontent.com/33035619/173629603-d85dcb0c-a0e8-4f7d-8db1-c9bb4c5eb69c.png">
<img width="983" alt="Screenshot 2022-06-14 at 17 29 59" src="https://user-images.githubusercontent.com/33035619/173629854-ae189227-06d1-42db-aa1c-927c8828b618.png">

- Paste the following python code in a file and run to ensure programmatic access was successfully configured.
  ```py
  import boto3
  s3 = boto3.resource('s3')
  for bucket in s3.buckets.all():
    print(bucket.name)
  ```

## Step 1: VPC, Subnets and Security Groups
### Step 1.1: Create a directory structure and set up Terraform CLI
- Create a folder named PBL and a file called main.tf
- Set up Terraform CLI
  ```
  $ sudo apt-get update && sudo apt-get install -y gnupg software-properties-common curl
  $ curl -fsSL https://apt.releases.hashicorp.com/gpg | sudo apt-key add -
  $ sudo apt-add-repository "deb [arch=amd64] https://apt.releases.hashicorp.com $(lsb_release -cs) main"
  $ sudo apt-get update && sudo apt-get install terraform
  $ terraform -help
  ```
### Step 1.2: Create first VPC
- Add AWS as provider and a resource to create a VPC in the main.tf file
  ```
  provider "aws" {
  region = "eu-central-1"
  }

  # Create VPC
  resource "aws_vpc" "main" {
    cidr_block                     = "172.16.0.0/16"
    enable_dns_support             = "true"
    enable_dns_hostnames           = "true"
    enable_classiclink             = "false"
    enable_classiclink_dns_support = "false"
  }
  ```
- Download all necessary plugins for Terraform to run
  ```
  terraform init
  ```
 <img width="983" alt="Screenshot 2022-06-14 at 17 37 29" src="https://user-images.githubusercontent.com/33035619/173630415-9f08cd7b-c9fc-4822-9f4c-ee797be545d9.png">

- Run 
  ```
  terraform plan
  ```
  to see the changes that will be effected. If you are satisfied with everything, run
  ```
  terraform apply
  ```
 <img width="983" alt="Screenshot 2022-06-14 at 17 38 59" src="https://user-images.githubusercontent.com/33035619/173630666-b12165c1-278e-4372-af21-c59cda30729d.png">

## Step 1.3: Create subnet resource
- Add the following block of code to create two VPC subnets
  ```
    # Create public subnets1
    resource "aws_subnet" "public1" {
      vpc_id                     = aws_vpc.main.id
      cidr_block                 = "172.16.0.0/24"
      map_public_ip_on_launch    = true
      availability_zone          = "eu-central-1a"
    }

    # Create public subnet2
    resource "aws_subnet" "public2" {
      vpc_id                     = aws_vpc.main.id
      cidr_block                 = "172.16.1.0/24"
      map_public_ip_on_launch    = true
      availability_zone          = "eu-central-1b"
    }
- Run
  ```
  terraform plan
  terraform apply
  ```
- Destroy the current infrastructure with
  ```
  terraform destroy
  ```
  Type yes when prompted
 <img width="979" alt="Screenshot 2022-06-14 at 17 44 30" src="https://user-images.githubusercontent.com/33035619/173632058-3a3bcb1a-526d-4dae-83ab-bd538168f60f.png">
## Step 2: Fixing limitations by code refactoring
### Step 2.1: Fixing hard coded values
- Adding the following block to the main.tf file
  ```
    variable "region" {
        default = "eu-central-1"
    }

    variable "vpc_cidr" {
        default = "172.16.0.0/16"
    }

    variable "enable_dns_support" {
        default = "true"
    }

    variable "enable_dns_hostnames" {
        default ="true" 
    }

    variable "enable_classiclink" {
        default = "false"
    }

    variable "enable_classiclink_dns_support" {
        default = "false"
    }

    provider "aws" {
    region = var.region
    }

    # Create VPC
    resource "aws_vpc" "main" {
    cidr_block                     = var.vpc_cidr
    enable_dns_support             = var.enable_dns_support 
    enable_dns_hostnames           = var.enable_dns_support
    enable_classiclink             = var.enable_classiclink
    enable_classiclink_dns_support = var.enable_classiclink

    }
##3 Step 2.2: Fixing multiple resource blocks
- Use Terraform **data sources** to fetch list of availability zones from AWS
  ```
  # Get list of availability zones
  data "aws_availability_zones" "available" {
        state = "available"
  }
  ```
- To make use of the new data resource, add **count** to the subnet block
  ```
  # Create public subnet1
  resource "aws_subnet" "public" { 
        count                   = 2
        vpc_id                  = aws_vpc.main.id
        cidr_block              = "172.16.1.0/24"
        map_public_ip_on_launch = true
        availability_zone       = data.aws_availability_zones.available.names[count.index]
        }
- Make cidr_block dynamic
  ```
    # Create public subnet1
    resource "aws_subnet" "public" { 
        count                   = 2
        vpc_id                  = aws_vpc.main.id
        cidr_block              = cidrsubnet(var.vpc_cidr, 4 , count.index)
        map_public_ip_on_launch = true
        availability_zone       = data.aws_availability_zones.available.names[count.index]
    }
  ```
  * You can experiment with the cidrsubnet function in the terraform console by typing **terraform console** in the terminal
## Step 2.3: Remove hard coded count value
- Introduce the **length()** function which returns the length of a given list, map or string. Update the subnet resource block like so
  ```
  # Create public subnet1
    resource "aws_subnet" "public" { 
        count                   = length(data.aws_availability_zones.available.names)
        vpc_id                  = aws_vpc.main.id
        cidr_block              = cidrsubnet(var.vpc_cidr, 4 , count.index)
        map_public_ip_on_launch = true
        availability_zone       = data.aws_availability_zones.available.names[count.index]
    }
- Update the count argument with a condition
  ```
  # Create public subnets
  resource "aws_subnet" "public" {
    count  = var.preferred_number_of_public_subnets == null ? length(data.aws_availability_zones.available.names) : var.preferred_number_of_public_subnets   
    vpc_id = aws_vpc.main.id
    cidr_block              = cidrsubnet(var.vpc_cidr, 4 , count.index)
    map_public_ip_on_launch = true
    availability_zone       = data.aws_availability_zones.available.names[count.index]
    }
    ```
  <img width="694" alt="Screenshot 2022-06-14 at 17 09 56" src="https://user-images.githubusercontent.com/33035619/173632725-eb01908f-bf38-4d36-9696-1d08f06f4b9a.png">
    * Try changing the *preferred_number_of_public_subnets* to null to see how the condition works

<img width="813" alt="Screenshot 2022-06-14 at 17 54 58" src="https://user-images.githubusercontent.com/33035619/173633540-dd0fadf6-0d91-4d74-af91-1fca8dd00367.png">

# Step 3: Introducing variables.tf and terraform.tfvars
## Step 3.1: Restructure directory tree
- Create two files, one named variables.tf and the other terraform.tfvars
- Paste in the following block in the variables.tf file
  ```
  variable "region" {
      default = "eu-central-1"
  }

  variable "vpc_cidr" {
    default = "172.16.0.0/16"
  }

  variable "enable_dns_support" {
    default = "true"
  }

  variable "enable_dns_hostnames" {
    default ="true" 
  }

  variable "enable_classiclink" {
    default = "false"
  }

  variable "enable_classiclink_dns_support" {
    default = "false"
  }

  variable "preferred_number_of_public_subnets" {
      default = null
  }
  ```
  and the following in the terraform.tfvars file
  ```
  region = "eu-central-1"

  vpc_cidr = "172.16.0.0/16" 

  enable_dns_support = "true" 

  enable_dns_hostnames = "true"  

  enable_classiclink = "false" 

  enable_classiclink_dns_support = "false" 

  preferred_number_of_public_subnets = 2
  ```
  Your directory structure should look like this:
  ```
  .
  ├── main.tf
  ├── terraform.tfstate
  ├── terraform.tfstate.backup
  ├── terraform.tfvars
  └── variables.tf

  0 directories, 5 files
  ```
  Run **terraform plan** and ensure everything runs successfully.
  <img width="1766" alt="Screenshot 2022-06-14 at 17 44 00" src="https://user-images.githubusercontent.com/33035619/173633815-cddf7092-e4de-4aff-9649-42a887d91698.png">
  
  Plan Sucessfully ran
  
  ### check resourese which where created on aws 
  <img width="1722" alt="Screenshot 2022-06-14 at 17 40 57" src="https://user-images.githubusercontent.com/33035619/173633982-924eb727-a739-40d5-bddf-af302b4a7fb8.png">
<img width="1720" alt="Screenshot 2022-06-14 at 17 41 16" src="https://user-images.githubusercontent.com/33035619/173633987-8d39cc54-520b-4458-995d-48efbe74986b.png">
<img width="1732" alt="Screenshot 2022-06-14 at 17 41 53" src="https://user-images.githubusercontent.com/33035619/173633991-540118ab-9147-42c2-b3d2-2a11c26f1303.png">

### Dont forget to distroy your infastruture after use to avoid uncharges by running **terraform destroy to clean up**
  <img width="1800" alt="Screenshot 2022-06-14 at 17 45 15" src="https://user-images.githubusercontent.com/33035619/173634787-9c690cfa-1cc5-4513-9105-0ed9ecf02778.png">

