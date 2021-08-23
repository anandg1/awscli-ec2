# Launching an AWS EC2 instance using Command Line Interface(CLI)

[![Build Status](https://travis-ci.org/joemccann/dillinger.svg?branch=master)]()

Let us have a look into the process of launching an AWS EC2 instance that is through AWS CLI and hosting a sample website in it.   
  
> AWS CLI is a unified tool for running and managing your various AWS services. Just download and install the tool and you will be able to control multiple AWS services from the command line, which makes the process of managing the AWS more easier.

Creating an instance with AWS CLI is the same as launching one with AWS console.


## Pre-requisite: 

Before starting to use the CLI, please remember to [configure](https://github.com/anandg1/aws-cli-configuration) AWS CLI access.


# Steps:


## 1) Creating a new Key Pair:

The most important step is to create a key pair. This key pair must be kept safe and secure with the user so that the person can access the EC2 instance created using this key pair.

The key-pair can be created using the below command:

```sh
aws ec2 create-key-pair --key-name cli-keypair --query 'KeyMaterial' --output text > cli-keypair.pem
```
where,
- 'cli-keypair' is the name given to the key pair.
- 'cli-keypair.pem' is the file to which the key pair is saved.


## 2)  Creating a security group:

For security group use the below command:
```sh
aws ec2 create-security-group --group-name cli-securitygroup --description "Security group with ports 22,80,443 open from all ips"
```

Now, the ports mentioned in the security group description are opened using the following commands for all IP ranges.

```sh
aws ec2 authorize-security-group-ingress --group-name cli-securitygroup --protocol tcp --port 22 --cidr 0.0.0.0/0

aws ec2 authorize-security-group-ingress --group-name cli-securitygroup --protocol tcp --port 80 --cidr 0.0.0.0/0

aws ec2 authorize-security-group-ingress --group-name cli-securitygroup --protocol tcp --port 443 --cidr 0.0.0.0/0
```

where,
- 'cli-securitygroup' is the custom name given to the security group.
- All that is given after --description "..." is the description of the security group which we have just created.

>Now, we need the group id of the security group 'cli-securitygroup' and the amazon machine image(AMI) ID of the instance to be launched. Here I'm oing to use the Amazon Linux 2 AMI (HVM), SSD Volume Type.


## 3)  Obtaining the IDs required to launch the instance:

To obtain the group id of the security group 'cli-securitygroup', run the following command:
```sh
aws ec2 describe-security-groups --group-name cli-securitygroup
```
To obtain the AMI id of Amazon Linux AMI (HVM), SSD Volume Type, run the following command:
```sh
aws ec2 describe-images --owners amazon --filters 'Name=name,Values=amzn2-ami-hvm-2.0.????????-x86_64-gp2' 'Name=state,Values=available' --output json
```


## 4) Launching an EC2 instance:

Launch the EC2 using the following command:
```sh
aws ec2 run-instances --image-id ami-04db49c0fb2215364 --security-group-ids sg-0a64705e709eaccc0 cli-securitygroup --count 1 --instance-type t2.micro --key-name cli-keypair
```
where,
- The instance is of 't2.mico' tier.
- 'Count' is the number of instances launched is 1.

We can obtain the id of the newly created EC2 instance using the following command:
```sh
aws ec2 describe-instances
```
Here is how we add a key tag 'cli-EC2' to the EC2 instance:
```sh
 aws ec2 create-tags --resources i-05bf4e7bab8fc4638  --tags Key=Name,Value=cli-EC2
```
where, 
- the ID which comes after '--resources' is the ID of our EC2.

Here is how we could see the public IP and private IP.

```sh
aws ec2 describe-instances --instance-ids i-05bf4e7bab8fc4638 --query 'Reservations[0].Instances[0].PublicIpAddress'  
```
```sh
aws ec2 describe-instances --instance-ids i-05bf4e7bab8fc4638 --query 'Reservations[0].Instances[0].PrivateIpAddress'
```


## 5) Accessing the EC2 using SSH:

We could access the EC2 using the keypair and public IP.
```sh
ssh -i cli-keypair.pem ec2-user@65.0.205.5
```
> Please make sure that the keypir file has a permission '400'.


## 6) Setting up the website in the EC2:

- Install apache webserver and PHP using the following commands:
```sh
sudo yum install php -y
sudo yum install httpd -y
sudo systemctl enable httpd.service
sudo chkconfig httpd on
sudo systemctl restart httpd.service
```
- Move to the document root:
```sh
cd /var/www/html
```
and put the website files there.

- Change the ownership and permission to apache

```sh
sudo chown -R apache:apache /var/www/html/*
```

- Restart the webserver
```sh
sudo systemctl restart httpd.service
```

Now, we could browse the website in the web-browser either using the public IP or Public DNS name
```sh
http://public_ip
http://PUBLIC_DNS
```


# Extras:


> We can stop the EC2 from CLI using the following command
```sh
aws ec2 stop-instances --instance-ids <instance_id...>
```
> > We can even terminate the EC2 from CLI using the following command
```sh
aws ec2 terminate-instances --instance-ids <instance_id...>
````
