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

Edit a route in public route table, and associate additional route for anywhere ips with the Internet Gateway. (This is what allows a public subnet to be accisble from the Internet)
Click VPC > route table , select the Public subnet. select `actions` > `edit routes`
![vpc](/images/11.png)

Create 3 Elastic IPs
`VPC` > `Elastic IP addresses` > `Allocate Elastic IP address` - add a name tag and click on `allocate`
![vpc](/images/12.png)

Create a Nat Gateway and assign the Elastic IPs
Click on `VPC` > `NAT gateways` > `Create NAT gateway`

- Select a Public Subnet
- Connection Type: Public
- Allocate Elastic IP 

![vpc](/images/13.png)

Update the Private route table - add anywhere ips and associate it the NAT gateway.
![vpc](/images/14.png)

Create a Security Group