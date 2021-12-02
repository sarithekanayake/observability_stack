# Welcome to Observability Stack Repo

Hi! This repository is to deploy Observability stack consists of Node Exporter, Prometheus and Grafana
Mainly it has two parts,

 1. Infrastructure provisioning
 2. Application deployment
 
# Infrastructure provisioning

Deploy 4 servers in AWS cloud using AWS CloudFormation templates

Setup as follows,
**Master**  : Act as the management server - Ansible playbooks will run on this server
**Monitoring** : Grafana and Prometheus and Node exporter running on this 
**Workers**	: Runs only Node Exporter

## Pre-requisites
In host machine,
 1. Install Git
 2. Install [AWS Cli](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-getting-started.html)


On AWS Cloud,

 1. Create user with Programmatic access (Access Key ID and Secret Access Key) using IAM service
 2. Attach necessary policies to run the CloudFromation templates (AdministratorAccess)
 3. Create a SSH Key pair to use with EC2 instances 
 4. Download generated SSH Key pair to local machine

## Steps to provision

 1. Setup credentials of user with `aws configure` command
 2. Clone the repository `git clone https://github.com/sarithekanayake/observability_stack.git`
 3. Navigate to infrastructure directory inside observability_stack repo `cd infrastructure `
 4. Run aws deploy command to deploy the infrastructure on AWS `aws cloudformation deploy --template-file infra.yml --stack-name observability-stack --parameter-overrides SSHKey=<SSH-Key Name> s3bucketname=<Unique S3 bucket name> --capabilities CAPABILITY_NAMED_IAM`

In **step 4** command, we are passing two parameters,
 

 1. SSHKey - Specify SSH Key Name we created in AWS Cloud Pre-requisites section
 2. S3 Bucket name - We are creating a S3 bucket to store miscellaneous items, provide an unique name 

**Example**

    aws cloudformation deploy --template-file infra.yml --stack-name observability-stack --parameter-overrides SSHKey=ec2sshkey s3bucketname=testbucketforpipedirvetest --capabilities CAPABILITY_NAMED_IAM

 **--capabilities CAPABILITY_NAMED_IAM** - We are creating IAM role in CF template. This is a must to pass with command to acknowledge IAM resource creation 
**NB** We can use a different stack name instead using observability-stack 

# Application Deployment

Applications will be deployed using Anisble playbook

 1. Navigate to AWS EC2 section on AWS Console
 2. Get the Public IP address of Master Node
 3. SSH into Master Node using SSH key `ssh -i <sshkey> ec2-user@<Public-IP>`
 4. Check cloud-init-output.log file to find instance configuration complete `tail -f /var/log/cloud-init-output.log`
 5. Wait until all the configurations are completed
 6. Navigate to ec2-user home directory `cd /home/ec2-user`
 7. Clone the repository `git clone https://github.com/sarithekanayake/observability_stack.git`
 8. Navigate to applications directory under observability_stack repo `cd applications`
 9. Edit inventory file and update the private IPs of servers `vim inventory`. Add Monitoring server IP to monitoring block. Add one of Worker Node IP to server1 and remaining one to server2 block.
 10. Save the changes and exit `Press [ESC] and type :wq!`
 11. Run the Ansible playbook `ansible-playbook apps.yml`

# Access Monitoring Stack

Wait until deployment completes, using Public IP address of Monitoring Node,
 

 1. Grafana access: `http://<Public-IP>:3000`
 2. Prometheus access: `http://<Public-IP>:9090`

Use `admin` and `test123` as the credentials to log into Grafana Frontend.

**To view dashboard**,
Menu -> Dashboards -> Browse -> Select "Node Exporter Full" 

We are using [Grafana Dashboards](https://github.com/rfrail3/grafana-dashboards) as the dashboard template

# Destory Infrastructure

We can simply use `aws cloudformation delete-stack --stack-name observability-stack` to destroy the setup