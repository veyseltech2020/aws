## Part 2 - Creating Bastion Host and CONNECT to the private subnet from internet

- discuss about how to connect to the "my-private-subnet-AZ1a" instance

- Explain logic of why we need Bastion Host?

- Launch two Instance. One is in Private Subnet-1a, the other one is in Public Subnet 1a

- Configure Public instance (Bastion Host).

```text
AMI             : Amazon Linux 2
Instance Type   : t2.micro
Network         : my_first_VPC
Subnet          : my-public-subnet-AZ1a
Security Group  : 
    Sec.Group Name : Public_Sec_Grp
    Rules          : TCP --- > 22 ---> Anywhere
                   : All ICMP IPv4  ---> Anywhere
Tag             :
    Key         : Name
    Value       : Public-(Bastion Host)-EC2
```

- Configure Private instance.

```text
AMI             : Amazon Linux 2
Instance Type   : t2.micro
Network         : my_first_VPC
Subnet          : my-private-subnet-AZ1a
Security Group  : 
    Sec.Group Name : Private_Sec_Grp
    Rules          : TCP --- > 22 ---> Anywhere
Tag             :
    Key         : Name
    Value       : Private EC2
```

- This configuration adds an extra layer of security and can also be used.
```text
Rules        : TCP --- > 22 ---> Anywhere
                |         |         |
                |         |         |
                V         V         V
               TCP --- > 22 ---> Public_Sec_Grp
``` 
- Try to connect private instance via ssh
  (As you see in the dashboard there is no public IP for instance)

- Since there is no public IP of private instance, we need to connect ssh via Bastion Host instance  
- go to your local terminal
- add your private key to the ssh agent on your localhost
```bash
ssh-add ./[your pem file name]
```
- run the ssh agent if it is returning error like "Could not open a connection to your authentication agent"
```bash
eval "$(ssh-agent)"
```
-  try again to add your private key to the ssh agent
```bash
ssh-add ./[your pem file name]
```
- connect to the ec2-in-az1a-public-sn instance in public subnet
```bash
ssh -A ec2-user@ec2-3-88-199-43.compute-1.amazonaws.com
```
- once logged into the ec2-in-az1a-public-sn (bastion host/jump box), connect to 
the ec2-in-az1a-private-sn instance in the private subnet 
```bash
ssh ec2-user@[Your private EC2 private IP]
```
- Show connection of the private EC2 over the Bastion Host

### Part 3 - Creating NAT Gateway

-  discuss about how to connect to internet from the Private EC2 in private subnet 

Step 1 : Create Elastic IP

- Go to VPC console on left hand menu and select Elastic IP tab

- Tab Allocate Elastic IP address

Elastic IP address settings

```text
Network border Group : Keep it as is (us-east-1)

Amazon's pool of IPv4 addresses
```
- Click Allocate button and name it as "my_first_ElasticIP"

- create a NAT Gateway in the public subnet

STEP 2: Create Nat Gateway

- Go to VPC console on left hand menu and select Nat Gateway tab

- Click Create Nat Gateway button 
```bash
Name                      : my-NAT-gateway

Subnet                    : my-public-subnet-AZ1a

Elastic IP allocation ID  : my_first_ElasticIP
```
- click "create Nat Gateway" button

STEP 3 : Modify Route Table of Private Instance's Subnet

- Go to VPC console on left hand menu and select Route Table tab

- Select "my-private-RT" ---> Routes ----> Edit Rule ---> Add Route
```
Destination     : 0.0.0.0/0
Target ----> Nat Gateway ----> my-NAT-gateway
```
- click save routes

- go to private instance via terminal using bastion host

- try to ping www.google.com and show response.

- Go to VPC console on left hand menu and select Nat Gateway tab

- Select my-NAT-gateway --- > Actions ---> delete Nat Gateway

- Go to VPC console on left hand menu and select Elastic IP tab

- Select "my_first_ElasticIP" ----> Actions ----> Release Elastic IP Address ----> Release 

### Part 4 - Creating NAT Instance

STEP 1: Create NAT Instance

- Go to EC2 Menu Using AWS Console

```text
AMI             : ami-00a9d4a05375b2763 (Nat Instance)
Instance Type   : t2.micro
Network         : my_first_VPC
Subnet          : my-public-subnet-AZ1a
Security Group  : 
    Sec.Group Name : Public_Sec_Grp
    Rules          : TCP ---> 22 ---> Anywhere
                   : All ICMP IPv4  ---> Anywhere

Tag             :
    Key         : Name
    Value       : my-NAT-Instance
```

- Select created Nat Instance on EC2 list

- Tab Actions Menu ----> Networking ----> Enable Source/Destination Check ---> Yes, Disable

- Explain why we need to implement this process mentioned above

STEP 2: Configuring the Route Table

- Go to Route Table and select "my-private-RT"

- Select Routes sub-menu ----> Edit Rules ----> Delete blackhole for Nat Gateway

- Add Route
```
Destination     : 0.0.0.0/0
Target ----> Instance ----> Nat Instance
```

- Connect to private Instance via bastion host and ping www.google.com to show response.
