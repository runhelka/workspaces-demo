# workspaces-demo

This project allows you to start with AWS Workspaces.

## Getting Started

These instructions will get you a copy of the project up and running on your AWS Account

### Prerequisites

You will need:

- AWS Account
- `aws-cli` and `curl` installed
- Amazon Workspaces client (<https://clients.amazonworkspaces.com/>)

![alt text](img/diagram.png "Infrastructure diagram")

### Deploying infrastructure stack

`infrastructure.template` will create following resources:

- VPC
- DHCP Option with DNS adresses of SimpeAD nodes
- Two public subnets for management instance and NAT Gateway
- Two private subnets for SimpeAD nodes
- Two private subnets for Worksapces machines
- Launch template and Auto Scaling Group for management instance
- SSM document association for Domain Join automation
- Secrets Manager secret for domain Administrator password

#### Prameters

You can use default parameters provided in the template, or specify:

- `projectName` - Default: `workspaces` - project name used in resources names and tags
- `stage` - Default: `dev` - deployment stage (dev, qa, prod, test)
- `cidrPrefix` - Default: `10.0` - VPC CIDR Prefix
- `availabilityZones` - Default: `a,b` - Availability Zones to use
- `domainName` - Default: `workspaces.local` - Simple AD domain name
- `simpleAdManagementAmiId` - Default: `/aws/service/ami-windows-latest/Windows_Server-2019-English-Full-Base` - AMI ID for management instance
- `simpleAdManagementInstanceType` - Default: `t2.micro` - management instance type
- `simpleAdManagementRdpAccessSourceIp` - No default value - Public adress of management inscante operator host.

#### Deployment

```bash
$ aws --region eu-west-1 cloudformation create-stack --stack-name workspaces-infrastructure \
--template-body file://$(pwd)/infrastructure.template \
--parameters ParameterKey=simpleAdManagementRdpAccessSourceIp,ParameterValue=$(curl ifconfig.co)/32 \
--capabilities CAPABILITY_NAMED_IAM
```

### Deploying workspace stack

#### BundleId

To obtain current available Bundles invoke this command:

``` bash
$ aws workspaces describe-workspace-bundles --region eu-west-1 --owner AMAZON \
--query 'Bundles[*].{BundleId:BundleId,Name:Name,RootStorage:RootStorage.Capacity,UserStorage:UserStorage.Capacity,ComputeType:ComputeType.Name} | reverse(sort_by(@, &ComputeType))' \
--output table
```

#### Prameters

`workspace.template` requires two parameters:

- `username` - username of user already created in SimpleAD
- `workspaceBundleId` - ID of Workspace Bundle - check BundleId section

#### Deployment

``` bash
aws --region eu-west-1 cloudformation create-stack --stack-name workspaces-tbres --template-body file://$(pwd)/workspace.template \
--parameters ParameterKey=username,ParameterValue=tbres ParameterKey=workspaceBundleId,ParameterValue=wsb-df76rqys9
```

## Authors

- **Tomasz Bres** - *Initial work*
