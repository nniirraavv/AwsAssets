# MEAN STACK Version-2


- A MEAN Stack is a free, open-source, popular JavaScript software stack used for developing dynamic websites and web applications. It is made from four components: MongoDB, Express, Angular, and Node.js. Angular is used for frontend development, while Node.js, Express, and MongoDB are used for backend development. A MEAN stack is based on JavaScript language so it can handle all aspects of an application.

- In this CloudFormation tutorial Version-2 we will install all four software with the latest version as well as its required dependency. And will separate the Frontend as application server and backend as database server and communicate/secure them with the appropriate security group.

  **(Frontend application server):** includes Angular/Node.JS/Nginx   
**(Backend database server):** includes MongoDB database

------------------------------------
## **Prerequisites:**   

Visual Studio Code – to make a yaml file of CloudFormation
1) Basic understanding of yaml language  
2) Basic knowledge of AWS management console/services 
3) You have to make 2 security groups through the AWS management console  
4) priorly for the Ref!! in CF template as Highlighted in template.  
5) In Web Server security group, you have to open port: 80=HTTP/443=HTTPS/22=SSH and for database server you have to open port: 27017 – (in database security group/inbound rules mention source as a SecurityGroup of web server to connect securely with web server.)
##

## **Software Versions:**  

With the help of this CloudFormation template you will get below versions of software and its dependency. You can change it according to your requirements. But in this tutorial, I have marked all the latest stable versions of software and its compatible dependencies.  
- Angular = 16.x  
- Node = 18.x  
- Yarn = 1.22  
- Pm2 = 5.0  
- MongoDB = 5.x  
- Ubuntu = 20.04   

**So, Let’s get started!!!**  

**Step-1:** In this step we will launch an ec2 instance with appropriate region and SSHKey for the web server. You can change the region as per your requirement.


```shel
AWSTemplateFormatVersion: "2010-09-09"
Parameters:
  SSHKey:
    Type: AWS::EC2::KeyPair::KeyName
    Description: Name of the key pair to SSH into the instances
Resources:
  WebServerInstance:
    Type: "AWS::EC2::Instance"
    Properties:
      AvailabilityZone: us-east-1a
      ImageId: ami-0261755bbcb8c4a84  # Replace with the desired Amazon Machine Image (AMI) ID
      InstanceType: t2.micro  # Replace with the desired EC2 instance type
      KeyName: !Ref SSHKey  # Replace with your EC2 key pair name
      SecurityGroupIds:
        - !Ref WebServerSecurityGroup
```

## **Command Explanation:**  
### Application Server Side

- **AWSTemplateFormatVersion:**   
This specifies the version of the CloudFormation template format being used. In this case, it is set to **"2010-09-09",** which is the earliest version of the format that is widely supported.  

- **Parameters:**   
This section defines the input parameters for the template. In this case, there is one parameter named "SSHKey" of type **"AWS::EC2::KeyPair::KeyName".** This parameter is used to specify the name of the key pair that will be used to SSH into the EC2 instance.

- **Resources:**   
This section defines the AWS resources that will be created when the CloudFormation stack is launched. In this case, there is one resource defined named **"webserverinstance"** of type **"AWS::EC2::Instance"**. This resource represents an EC2 instance that will be provisioned.  

- **AvailabilityZone:**   
Specifies the availability zone in which the EC2 instance will be launched. In this case, it is set to **"us-east-1a"** ; you can replace it as per your choice.  

- **ImageId:**   
Specifies the Amazon Machine Image (AMI) ID that will be used to launch the EC2 instance. It is set to **"ami-0261755bbcb8c4a84"**. You would need to replace this with the desired AMI ID.  

- **InstanceType:**  
Specifies the EC2 instance type that will be used for the instance. In this case, it is set to "t2.micro", which is a general-purpose instance type with a small resource footprint. You can replace this with the desired instance type.  

- **KeyName:**  
Specifies the name of the key pair that will be associated with the EC2 instance for SSH access. The value is obtained from the **"SSHKey"** parameter using the **!Ref** function.  

- **SecurityGroupIds:**  
Specifies the security group(s) that will be associated with the EC2 instance. The value is obtained from the **" - !Ref WebServerSecurityGroup "** resource using the **!Ref** function. 


**Step-2:** 
In this step we will write basic userdata script which is basically one kind of command which is used to install required software and its related dependency.

```bash
UserData:
        Fn::Base64: |
          #!/bin/bash
          sudo apt-get update -y
          sudo apt-get install -y curl gnupg2 unzip git gcc g++ make


          # Install Node.js
          curl -sL https://deb.nodesource.com/setup_18.x | sudo bash -
          sudo apt-get install -y nodejs


          # Install Yarn and Gulp
          sudo npm install -g yarn gulp


          # Install Angular CLI
          sudo npm install -g @angular/cli


          # Install PM2
          sudo npm install -g pm2


          # Install and configure Nginx
          sudo apt-get install -y nginx
          sudo systemctl restart nginx
```
## **Command Explanation:**  

- **sudo apt-get update -y:**  
Updates the package lists on the instance by fetching the latest package information from the configured repositories.

- **sudo apt-get install -y curl gnupg2 unzip git gcc g++ make:**  
Installs several packages required for development and building software, including curl, gnupg2, unzip, git, gcc, g++, and make.

- **curl -sL https://deb.nodesource.com/setup_18.x | sudo bash -:**   
Downloads and runs the NodeSource script, which sets up the Node.js repository on the instance.

- **sudo apt-get install -y nodejs:**  Installs Node.js on the instance.

- **sudo npm install -g yarn gulp:**  
  Installs the Yarn package manager and the Gulp task runner globally using npm.  

- **sudo npm install -g @angular/cli:**  
Installs the Angular CLI globally using npm.

- **sudo npm install -g pm2:**   Installs the PM2 process manager globally using npm.

- **sudo apt-get install -y nginx:**  Installs the Nginx web server on the instance.

- **sudo systemctl restart nginx:** 
 Restarts the Nginx service to apply any changes made to the configuration.

------------------------------------------------------
## **Database Server Side:**

**Step-3:**  In this step we will launch ec2 instance with appropriate region and same SSHKey for database server

```bash
DatabaseServerInstance:
    Type: "AWS::EC2::Instance"
    Properties:
      AvailabilityZone: us-east-1a
      ImageId: ami-0261755bbcb8c4a84  # Replace with the desired Amazon Machine Image (AMI) ID
      InstanceType: t2.micro  # Replace with the desired EC2 instance type
      KeyName: !Ref SSHKey  # Replace with your EC2 key pair name
      SecurityGroupIds:
        - !Ref DatabaseServerSecurityGroup
```

## **Command Explanation:**

- **AvailabilityZone:**   
Specifies the availability zone in which the EC2 instance will be launched. In this case, it is set to "us-east-1a" you can replace it as per your choice.

- **ImageId:** 
Specifies the Amazon Machine Image (AMI) ID that will be used to launch the EC2 instance. It is set to **"ami-0261755bbcb8c4a84".** You would need to replace this with the desired **AMI ID.**

- **InstanceType:** 
Specifies the EC2 instance type that will be used for the instance. In this case, it is set to "t2.micro", which is a general-purpose instance type with a small resource footprint. You can replace this with the desired instance type. 

- **KeyName:**  
Specifies the name of the key pair that will be associated with the EC2 instance for SSH access. The value is obtained from the **"SSHKey"** parameter using the **!Ref** function.

- **SecurityGroupIds:**  
Specifies the security group(s) that will be associated with the EC2 instance. The value is obtained from the **" - !Ref DatabaseServerSecurityGroup "** resource using the **!Ref** function.

**Step-4:** In this step we will write basic userdata script which is basically one kind of command which is used to install MongoDB Database.

```bash
  UserData:
Fn::Base64: |
          #!/bin/bash
          sudo apt-get update -y
          sudo wget -qO - https://www.mongodb.org/static/pgp/server-5.0.asc | sudo apt-key add -
          echo "deb [ arch=amd64,arm64 ] https://repo.mongodb.org/apt/ubuntu focal/mongodb-org/5.0 multiverse" |
 sudo tee /etc/apt/sources.list.d/mongodb-org-5.0.list
          sudo apt-get update
          sudo apt-get install -y mongodb-org
          sudo systemctl start mongod
          sudo systemctl enable mongod
```

## **Command Explanation:**
- **Fn::Base64:**   
function is a CloudFormation intrinsic function in AWS that encodes a string value using Base64 encoding. Base64 encoding is a way to represent binary or non-textual data in a format that can be safely transmitted or stored as text.

- **sudo apt-get update -y:**  
This command updates the package lists on the instance by fetching the latest package information from the configured repositories.

- **sudo wget -qO - https://www.mongodb.org/static/pgp/server-5.0.asc | sudo apt-key add -:**   
This command downloads the MongoDB public key from the specified URL and adds it to the system's package manager keyring.

- **echo "deb [ arch=amd64,arm64 ] https://repo.mongodb.org/apt/ubuntu focal/mongodb-org/5.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-5.0.list:**  
This command adds the MongoDB repository to the package manager's sources list, specifically the mongodb-org package.

- **sudo apt-get update:**   
This command updates the package lists again, including the newly added MongoDB repository.

- **sudo apt-get install -y mongodb-org:** 
This command installs the mongodb-org package, which installs MongoDB on the instance.

- **sudo "sed -i 's/bindIp: 127.0.0.1/bindIp: 0.0.0.0/' /etc/mongod.conf\n":**   
This command modifies the MongoDB configuration file located at /etc/mongod.conf to bind MongoDB to listen on all network interfaces (0.0.0.0) instead of just the loopback address (127.0.0.1).

- **sudo systemctl start mongodb:** This command starts the MongoDB service.

- **sudo systemctl enable mongodb:** 
This command configures the MongoDB service to start automatically on system boot.

**Step-5:** Configure Web Server Security group: In this step we will make a security group that is used to allow ports to communicate with. In this security group we will allow SSH=port=22 / Node=port=3000 /HTTP=port=80 / HTTPS=port=443. In this security group CIDR is kept 0.0.0.0/0 open for any ware you can replace it through AWS-Management console/EC2/SecurityGroup/edit inbound rules

```bash
webServerSecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: Security group for the web server
      SecurityGroupIngress:
        - IpProtocol: tcp
          FromPort: '22'
          ToPort: '22'
          CidrIp: 0.0.0.0/0
        - IpProtocol: tcp
          FromPort: '80'
          ToPort: '80'
          CidrIp: 0.0.0.0/0
        - IpProtocol: tcp
          FromPort: '443'
          ToPort: '443'
          CidrIp: 0.0.0.0/0


Outputs:
  WebServerPublicIP:
    Value: !GetAtt WebServerInstance.PublicIp
```

## **Command Explanation:**

- **WebserverSecurityGroup:**  
 This resource represents the EC2 security group used by the instance.

- **GroupDescription:**  
 A description for the security group, which in this case is "Enable MongoDB & Node.js server ports".

- **SecurityGroupIngress:**  
 Specifies the inbound rules for the security group. It allows incoming traffic on ports 22 (SSH), 443 (HTTPS) and 80 (HTTP) from any IP address (0.0.0.0/0).

- **Outputs:**  
This section defines the outputs of the CloudFormation stack and public IP of launched instances.

**Step-6:** Configure Database Server Security group: In this security group with the specified properties It allows inbound TCP traffic on port 27017 only from the EC2 instances associated with the referenced source security group ID.

```bash
databaseServerSecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: Security group for the database server
      SecurityGroupIngress:
        - IpProtocol: tcp
          FromPort: '27017'
          ToPort: '27017'
          SourceSecurityGroupId: !GetAtt WebServerSecurityGroup.GroupId


DatabaseServerPrivateIP:
    Value: !GetAtt DatabaseServerInstance.PrivateIp
```

## **Command Explanation:**

- **Database server SecurityGroup:**  
 This resource represents the EC2 security group used by the instance.

- **GroupDescription:**  
 A description for the security group, which in this case is "Enable MongoDB server ports".

- **SecurityGroupIngress:**  
 Specifies the inbound rules for the security group. It allows incoming traffic on ports 27017 (MongoDB).

- **Outputs:**  
This section defines the outputs of the CloudFormation stack and Private IP of launched instances.
