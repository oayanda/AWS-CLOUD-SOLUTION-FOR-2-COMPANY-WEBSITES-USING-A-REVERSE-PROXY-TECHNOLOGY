#

## `Requirements for this project.`

- Create a root/master account
- Create a sub-account.
- Create a new Organizational Unit (OU) called `Dev` and add the sub-account `DevOps`

![Organizational Unit/sub-account](/images/1.png)

## `Set Up a Virtual Private Network (VPC)`

In the aws dashboard, search vpc and Create a VPC and ensure `DNS hostnames` is enabled.

![vpc](/images/2.png)

Create subnets as shown in the architecture. On the left panel menu of the VPC UI, click on Subnet > `Create Subnet`.

- oayanda-Pub-Sub-1 - 10.0.1.0/24 in Zone A
- oayanda-Pub-Sub-2 - 10.0.3.0/24 in Zone B
- oayanda-Pri-Sub-1 - 10.0.2.0/24 in Zone A
- oayanda-Pri-Sub-2 - 10.0.4.0/24 in Zone B
- oayanda-Pri-Sub-3 - 10.0.5.0/24 in Zone A
- oayanda-Pri-Sub-4 - 10.0.6.0/24 in Zone B

![vpc](/images/3.png)

Create a route table and associate it with Public subnets
Click `VPC` > `Route tables` > `Create route table`
![vpc](/images/4.png)

Associate Public subnet to the Public route table.
Click the tab `Actions` >  `edit subnet associations` - select the subnets and click `save associations`
![vpc](/images/7.png)

Create a route table and associate it with Private subnets
Click `VPC` > `Route tables` > `Create route table`
![vpc](/images/5.png)

Associate Public subnet to the Public route table.
Click the tab `Actions` >  `edit subnet associations` - select the subnets and click `save associations`
![vpc](/images/6.png)

Route Table
![vpc](/images/8.png)

Create an Internet Gateway 
Click on `VPC` > `Internet gateways` > `Create internet gateway`, enter the name and click on `create internet gateway`.
![vpc](/images/9.png)

Next, attach the internet gateway to the VPC we created earlier.
On the same page click `Attach to VPC`, select the vpc and click `attach internet gateway`
![vpc](/images/10.png)

Edit the route in the public route table, and associate additional route from other ips.(This is what allows a public subnet to be accisble from the Internet)
Click VPC > route table , select the Public subnet. select `actions` > `edit routes`
![vpc](/images/11.png)

Create Elastic IP to configured with the NAT gateway. The NAT gateway enables connection from the public subnet to private subnet and it needs a static ip to make this happen.
`VPC` > `Elastic IP addresses` > `Allocate Elastic IP address` - add a name tag and click on `allocate`
![vpc](/images/12.png)

Create a Nat Gateway and assign the Elastic IPs
Click on `VPC` > `NAT gateways` > `Create NAT gateway`

- Select a Public Subnet
- Connection Type: Public
- Allocate Elastic IP 

![vpc](/images/13.png)

Update the Private route table - add allow anywhere ip and associate it the NAT gateway.
![vpc](/images/14.png)

Create a Security Group for the following

- `Nginx Servers`: Access to Nginx should only be allowed from a Application Load balancer (ALB). At this point, we have not created a load balancer, therefore we will update the rules later. For now, just create it and put some dummy records as a place holder.
- `Bastion Servers`: Access to the Bastion servers should be allowed only from workstations that need to SSH into the bastion servers. Hence, you can use your workstation public IP address. To get this information, simply go to your terminal and type curl www.canhazip.com
- `Application Load Balancer`: ALB will be available from the Internet
- `Webservers`: Access to Webservers should only be allowed from the `Nginx` servers. Since we do not have the servers created yet, just put some dummy records as a place holder, we will update it later.
- `Data Layer`: Access to the Data layer, which is comprised of `Amazon Relational Database Service (RDS)` and `Amazon Elastic File System (EFS)` must be carefully desinged – only `webservers` should be able to connect to `RDS`, while `Nginx` and `Webservers` will have access to `EFS` Mountpoint.
![vpc](/images/SG.png)

Configure additional Compute Resources

We will need to set up and configure compute resources inside the VPC. The recources related to compute are the following

- EC2 Instances
- Launch Templates
- Target Groups
- Autoscaling Groups
- TLS Certificates
- Application Load Balancers (ALB)

- EC2 Instances

Lunch 3 Ec2 instances running a redhat server namely,`Bastion` - as the Jump server, `nginx` - for external load balancer and `webserver` - for the 2 applications.
At this point, we need to install and configure some basic requirements on the instances.

`Bastion`, Ngiinx and Webserver 

```bash
yum install -y https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm

yum install -y dnf-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm

yum install wget vim python3 telnet htop git mysql net-tools chrony -y

systemctl start chronyd
systemctl enable chronyd

# configure selinux policies for the webservers and nginx servers

setsebool -P httpd_can_network_connect=1
setsebool -P httpd_can_network_connect_db=1
setsebool -P httpd_execmem=1
setsebool -P httpd_use_nfs 1

## This section will install amazon efs utils for mounting the target on the Elastic file system

git clone https://github.com/aws/efs-utils

cd efs-utils

yum install -y make

yum install -y rpm-build

make rpm 

yum install -y  ./build/amazon-efs-utils*rpm

## seting up self-signed certificate for the nginx instance

sudo mkdir /etc/ssl/private

sudo chmod 700 /etc/ssl/private

openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/ssl/private/oayanda.key -out /etc/ssl/certs/oayanda.crt

sudo openssl dhparam -out /etc/ssl/certs/dhparam.pem 2048

## seting up self-signed certificate for the apache  webserver instance

yum install -y mod_ssl

openssl req -newkey rsa:2048 -nodes -keyout /etc/pki/tls/private/oayanda.key -x509 -days 365 -out /etc/pki/tls/certs/oayanda.crt

vi /etc/httpd/conf.d/ssl.conf
```

The `lunch templates` requires `AMI`s (Amazon Machine Images) - Create AMIs from the instances and terminate them.
![vpc](/images/ami.png)
![vpc](/images/amis.png)

Create `KMS`a Key Management Service to encrypt the Database
![vpc](/images/kms.png)

Create a Subnet group for the RDS Database in private subnet 5 and 6.
![vpc](/images/rds%20subnet.png)

Create the `RDS` Database required for the application
![vpc](/images/rds%20%20database.png)

### `Setup EFS`
Amazon Elastic File System (Amazon EFS) provides a simple, scalable, fully managed elastic Network File System (NFS) for use with AWS Cloud services and on-premises resources. In this project, we will utulize EFS service and mount filesystems on both Nginx and Webservers to store data.

- Create an EFS filesystem
- Create an EFS mount target per AZ in the VPC, associate it with both subnets dedicated for data layer
- Associate the Security groups created earlier for data layer.
- Create an EFS access point. (Give it a name and leave all other settings as default)
![vpc](/images/EFS.png)

Create `Access Points` mount the applications
`wordpress` and `tooling`
![vpc](/images/access%20point.png)

Create the TLS Certificate for `Amazon Certificate manager` for the registered domain name. Make sure of a wildcard * to accept sub-domain names like this - `*.oayanda.com`. So we can have - `wordpress.oayanda.com`, `tooling.oayanda.com` and so on. This would need to validated - normally, <= 30mins. This certificate is a requirement  for the external Load balancer and since we are only allow on secure connection in our infrasture.
![vpc](/images/AWSCertificate%20Manager.png)

Target Groups.

Now let's create three target groups that would be attached to the load balancers to route right request to the intend application namely Nginx, Wordpress and Tooling..

- Select Instances as the target type
- Ensure the protocol is TCP on port 22
- Ensure that health check passes for the target group
![vpc](/images/nginx-target.png)



### `Configure External Application Load Balancer To Route Traffic To NGINX`

Nginx EC2 Instances will have configurations that accepts incoming traffic only from Load Balancers. No request should go directly to Nginx servers. With this kind of setup, we will benefit from intelligent routing of requests from the ALB to Nginx servers across the 2 Availability Zones. We will also be able to offload SSL/TLS certificates on the ALB instead of Nginx. Therefore, Nginx will be able to perform faster since it will not require extra compute resources to valifate certificates for every request.

- Create an Internet facing ALB
- Ensure that it listens on HTTPS protocol (TCP port 443)
- Ensure the ALB is created within the appropriate VPC | AZ | Subnets
Choose the Certificate from ACM
- Select the external ALB Security Group
- Select Nginx Instances as the target group


### `Configure Internal Application Load Balancer To Route Traffic To Web Servers Wordpress & Tooling`

Since the webservers are configured for auto-scaling, there is going to be a problem if servers get dynamically scalled out or in. Nginx will not know about the new IP addresses, or the ones that get removed. Hence, Nginx will not know where to direct the traffic.

To solve this problem, we must use a load balancer. But this time, it will be an internal load balancer. Not Internet facing since the webservers are within a private subnet, and we do not want direct access to them.

Create an Internal ALB
Ensure that it listens on HTTPS protocol (TCP port 443)
Ensure the ALB is created within the appropriate VPC | AZ | Subnets
Choose the Certificate from ACM
Select Security Group
Select webserver Instances as the target group
Ensure that health check passes for the target group

![vpc](/images/loadbalancer.png)

Register or Purchase a domain name for the project. 

Create a Hosted Zone with Aws Route 53.
You need to ensure that the main domain for the WordPress website can be reached, and the subdomain for Tooling website can also be reached using a browser.
- Create an alias record for the root domain and direct its traffic to the ALB DNS name.

- Create an alias record for tooling.tooling.oayanda.com and wordpress.oayanda.com and direct its traffic to the ALB DNS name.
![vpc](/images/rout53.png)

### `Setup Lunch Template`
Prepare Launch Template For Webservers (wordpress and tooling), ngins, bastion (One per subnet)
- Make use of the right AMI is set up a launch template.
- Ensure the Instances for bastion and nginx are launched into a public subnet and the wordpress and tooling are lunched into the private subnet.
- Assign appropriate security group
- Configure Userdata as require per each instance.
For Wordpress and Tooling templates, make sure the mount access points are included.

![vpc](/images/mount%20tooling.png)
![vpc](/images/mount.png)

![vpc](/images/lunch%20templates.png)

### `Configure Autoscaling For Nginx, bastion, tooling and wordpress`

- Select the right launch template
- Select the VPC
- Select both public subnets for `bastion` and `nginx` and private subnets 1 and 2 for `tooling` and `wordpress`
- Enable Application Load Balancer for the AutoScalingGroup (ASG)
- Select the target group you created before
- Ensure that you have health checks for both EC2 and ALB
- The desired capacity is 2
- Minimum capacity is 2
- Maximum capacity is 4
- Set scale out if CPU utilization reaches 90%
- Ensure there is an SNS topic to send scaling notifications.
![vpc](/images/autoscaling%20group.png)
