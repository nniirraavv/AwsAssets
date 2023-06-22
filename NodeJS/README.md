# Deploy a NodeJs Application using AWS CloudFormation  [![GitHub license](https://img.shields.io/badge/license-MIT-blue.svg?style=flat-square)](https://github.com/your/your-project/blob/master/LICENSE) ![AWS](https://img.shields.io/badge/AWS-%23FF9900.svg?style=for-the-badge&logo=amazon-aws&logoColor=white) ![NodeJS](https://img.shields.io/badge/node.js-6DA55F?style=for-the-badge&logo=node.js&logoColor=white) ![Ubuntu](https://img.shields.io/badge/Ubuntu-E95420?style=for-the-badge&logo=ubuntu&logoColor=white) ![Yarn](https://img.shields.io/badge/yarn-%232C8EBB.svg?style=for-the-badge&logo=yarn&logoColor=white)
You can create a CloudFormation Template using either a YAML or JSON file. We're going to use a YAML file in this tutorial. 
In this template, we'll be creating an EC2 instance, we'll configure a Security Group for EC2, and adding a script to deploy a simple NodeJS app. 

## Blog

Read the Whole blog on [Create a CloudFormation Template to Deploy a NodeJS Application](https://www.eternalsoftsolutions.com/blog/how-to-create-a-cloudformation-template-to-deploy-a-nodejs-application/)

## Installing 

A standard configuration of installing software and related dependencies.

 * Ubuntu version: 20.04
 * NodeJS version: 19.20
 * EC2: t2. Micro
 * Yarn version: 1.22
 * Pm2 version: 1.22

 ## Note
 
 1. If you want to install a specific version of NodeJS and related dependencies then kindly follow this link (https://deb.nodesource.com/setup_16.x)  and you can change your desired version of nodejs in the template accordingly. You have to change ```setup 16.x ``` to your required version in that ```.yml file``` in VS code and then upload it to stack.
 2. To Change version of yarn and pm2 you have to change in ```user data``` section like: ```sudo apt install yarn=1.x.y:``` Replace ```1.x.y```with the specific version of Yarn you want to install. For Pm2 ```sudo npm install -g pm2@2.x.y:```Replace ```2.x.y``` with the desired version of PM2 you want to install.
 3. By modifying these lines, you can ensure that the desired versions of Yarn and PM2 are installed alongside the specified version of Node.js. Remember to replace 1.x.y and 2.x.y with the actual versions you want to use.
 4. If you want to add your project then just replace the git clone URL from  

```shel
https://github.com/5minslearn/node_with_docker.git
cd node_with_docker TO https://your-git-repo/folder  
cd your-project-repo. 
yarn pakage – command: yarn install

 ````
## CloudFormation Template to Create an EC2 Instance

There are over 224 types of resources in AWS, but we need to create an EC2 resource. Resources represent the different AWS Components that will be created and configured. We'll define the Resource type identifiers in the below format:

```AWS::aws-product-name::data-type-name```

The resource format for the EC2 instance is ```Aws::EC2::Instance```. To learn more about AWS resources and syntax, checkout the ```AWS official documentation``` and play with it. Look at the EC2 documentation to understand the declaration of EC2 instance. Both JSON and YAML syntax is available but we'll stick with YAML for this tutorial. 

There are a lot of properties available to customize the creation of our EC2 instances. To make things simple, we'll be configuring Availability Zone, Imageid, and Instance Type which are basic properties needed to create an EC2 instance. 

```shell
Resources:
  NodejsDeploy:
    Type: AWS::EC2::Instance
    Properties:
      AvailabilityZone: us-east-1a
      ImageId: Your-instance AMI-id
      InstanceType: t2.micro

```

AWS CloudFormation - EC2 Instance configuration

Here ``NodejsDeploy`` refers to the name of the resource we'll be creating. You can name your resource as your wish. 
Let's see the process to deploy the NodeJS app. 

## Deploy a NodeJS Application

We're going to deploy the NodeJS app using the User Data property in the EC2 resource.

If you don't know about EC2 user data, it is a feature of AWS EC2 which allows us to pass information during the launch of the EC2 instance. You can use it to perform custom actions, such as installing software and executing the script. 

Let's write the bash script to deploy the NodeJS app and attach it to the user data.

Here is the simple script to deploy the NodeJS application:
```shell
#!/bin/bash 
set -e
curl -sL https://deb.nodesource.com/setup_19.x | bash -
sudo apt install nodejs
node -v
npm -v
curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | sudo apt-key add -
echo "deb https://dl.yarnpkg.com/debian/ stable main" | sudo tee /etc/apt/sources.list.d/yarn.list
sudo apt update && sudo apt install yarn
yarn --version
sudo -i -u ubuntu bash << EOF
set -e
cd /home/ubuntu
sudo npm install -g pm2
git clone https://github.com/5minslearn/node_with_docker.git
cd node_with_docker
yarn install 
pm2 start yarn --time --interpreter bash --name sample_node -- start -p 8000
EOF
```
The above script installs NodeJS, Yarn, and PM2. It clones a NodeJS project from Git, installs the dependencies, and starts the app with PM2. 
Our next step is to attach this script to the CloudFormation template.

## How to Attach User Data to the CloudFormation Template

```shell
Resources:
  SampleNodejsDeploy:
    Type: AWS::EC2::Instance
    Properties:
      InstanceType: t2.micro
      ImageId: ami-08e5424edfe926b43 
      UserData: 
        Fn::Base64:
          |
          #!/bin/bash 
          set -e
          curl -sL https://deb.nodesource.com/setup_16.x | bash -
          sudo apt install nodejs
          node -v
          npm -v
          curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | sudo apt-key add -
          echo "deb https://dl.yarnpkg.com/debian/ stable main" | sudo tee /etc/apt/sources.list.d/yarn.list
          sudo apt update && sudo apt install yarn
          yarn --version
          sudo -i -u ubuntu bash << EOF
          set -e

          cd /home/ubuntu
          sudo npm install -g pm2
          git clone https://github.com/5minslearn/node_with_docker.git
          cd node_with_docker
          yarn install 
          pm2 start yarn --time --interpreter bash --name sample_node -- start -p 8000
          EOF

```
You'll notice that the ``User Data`` property is added to the EC2 block. ``Fn::Base64`` is a function in AWS CloudFormation that allows users to encode a string to base64 format. This function can be used to pass sensitive information, such as credentials, to AWS resources in a secure manner. Since EC2 user data is not encrypted its always best practice to encode it. 

Right below that line, you can see a small ``vertical bar (|)``. It is used for multi-line string support as our script is more than 1 line.

Alright. Now we have a script to deploy the NodeJS app. But, we have to remember one super important item. By default, NodeJS applications run on ``port 8000``. We should expose port 8000 from EC2. Now we need to create a security group configuration for our EC2 instance. 

## Create a Security Group

This process is similar to creating an EC2 instance, except we'll replace the type from Instance to SecurityGroup.

```shell
    NodejsDeploySG:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: for the app nodes that allow ssh, http, 8000
      SecurityGroupIngress:
      - IpProtocol: tcp
        FromPort: '80'
        ToPort: '80'
        CidrIp: 0.0.0.0/0
      - IpProtocol: tcp
        FromPort: '22'
        ToPort: '22'
        CidrIp: 0.0.0.0/0
      - IpProtocol: tcp
        FromPort: '8000'
        ToPort: '8000'
        CidrIp: 0.0.0.0/0

```
AWS CloudFormation - Security Group configuration

The above code should be pretty self-explanatory – we defined a Security group, allowing ports 22 (SSH port), 80 (HTTP port), and 8000 (NodeJS). We named the Resource as ``NodejsDeploySG``. 

## Attach the Security Group to EC2

You may be wondering – "We've created a template for creating a Security group but how will this be linked to the EC2 instance?"

The solution is simple. CloudFormation provides an intrinsic function called ``! Ref`` that allows us to reference a resource or parameter within a CloudFormation template.

```shell
Resources:
  SampleNodejsDeploy:
    Type: AWS::EC2::Instance
    Properties:
      InstanceType: t2.micro
      ImageId: Your-instance AMI-id
      SecurityGroups:
        - !Ref NodejsDeploySG
      UserData: 
        Fn::Base64:
          |
          #!/bin/bash 
          set -e
          curl -sL https://deb.nodesource.com/setup_16.x | bash -
          sudo apt install nodejs
          node -v
          npm -v
          curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | sudo apt-key add -
          echo "deb https://dl.yarnpkg.com/debian/ stable main" | sudo tee /etc/apt/sources.list.d/yarn.list
          sudo apt update && sudo apt install yarn
          yarn --version
          sudo -i -u ubuntu bash << EOF
          set -e
          cd /home/ubuntu
          sudo npm install -g pm2
          git clone https://github.com/5minslearn/node_with_docker.git
          cd node_with_docker
          yarn install 
          pm2 start yarn --time --interpreter bash --name sample_node -- start -p 8000
          EOF
            
  SampleNodejsDeploySG:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: for the app nodes that allow ssh, http 
      SecurityGroupIngress:
      - IpProtocol: tcp
        FromPort: '80'
        ToPort: '80'
        CidrIp: 0.0.0.0/0
      - IpProtocol: tcp
        FromPort: '22'
        ToPort: '22'
        CidrIp: 0.0.0.0/0
      - IpProtocol: tcp
        FromPort: '8000'
        ToPort: '8000'
        CidrIp: 0.0.0.0/0

```
Add Security Groups configuration to EC2 Instance

You can see that the ``SecurityGroups`` property is added to the EC2 instance and the created Security Group configuration is linked to the EC2 instance by using the ``!Ref`` parameter.

Now we have the CloudFormation template. But we're not yet finished. We're still missing one more thing. Can you figure it out? We created an EC2 instance, and we allowed an SSH port...but to log in using SSH we need to attach a key-value pair, right? Let's do that. 

We can attach the key-value pair name directly to the template. For example, let's assume your key-value pair name is ``CFNodejs`` you can attach the property ``KeyName`` directly to the EC2 resource block like what's shown below or we can pass it in via parameters. 

```shell
Resources:
  SampleNodejsDeploy:
    Type: AWS::EC2::Instance
    Properties:
      InstanceType: t2.micro
      ImageId: Your-instance-AMI-id
      KeyName: CFNodejs
      SecurityGroups:
        - !Ref NodejsDeploySG

```
AWS CloudFormation - Key Name for EC2


## Use parameters in the CloudFormation template

We can use parameters to get the name of the key-value pair from the user while creating the stack. Basically, parameters allow us to pass input values into CloudFormation templates at runtime. Let's see how to do that. 

```shell
Parameters:
  SSHKey:
    Type: AWS::EC2::KeyPair::KeyName
    Description: name of the key pair to ssh into the instance
Resources:
  SampleNodejsDeploy:
    Type: AWS::EC2::Instance
    Properties:
      InstanceType: t2.micro
      ImageId: Your-instance AMI-id
      KeyName: !Ref SSHKey
      SecurityGroups:
        - !Ref NodejsDeploySG
      UserData: 
        Fn::Base64:
          |
          #!/bin/bash 
          set -e
          curl -sL https://deb.nodesource.com/setup_16.x | bash -
          sudo apt install nodejs
          node -v
          npm -v
          curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | sudo apt-key add -
          echo "deb https://dl.yarnpkg.com/debian/ stable main" | sudo tee /etc/apt/sources.list.d/yarn.list

          sudo apt update && sudo apt install yarn
          yarn --version
          sudo -i -u ubuntu bash << EOF
          set -e
          cd /home/ubuntu
          sudo npm install -g pm2
          git clone https://github.com/5minslearn/node_with_docker.git
          cd node_with_docker
          yarn install 
          pm2 start yarn --time --interpreter bash --name sample_node -- start -p 8000
          EOF
            
  SampleNodejsDeploySG:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: for the app nodes that allow ssh, http 
      SecurityGroupIngress:
      - IpProtocol: tcp
        FromPort: '80'
        ToPort: '80'
        CidrIp: 0.0.0.0/0
      - IpProtocol: tcp
        FromPort: '22'
        ToPort: '22'
        CidrIp: 0.0.0.0/0
      - IpProtocol: tcp
        FromPort: '8000'
        ToPort: '8000'
        CidrIp: 0.0.0.0/0

```
AWS CloudFormation - Attach ``KeyName`` for EC2 by reading from Parameter

In the above template, we added a parameter to get the key pair name and referenced it to ``KeyName`` property. 

Great! We successfully created a CloudFormation template to create an EC2 instance and security group. In addition to that, we also added a script to deploy the NodeJS app. Now it's time to create a CloudFormation stack.

## Create a CloudFormation Stack

For more details visit our blog : 


## CloudFormation Full Template

available in the repository.

## CF-Template Command Explanations:

1. ``set -e``: This command ensures that if any command fails (exits with a non-zero status code), the script will exit immediately instead of continuing execution.
2. ``curl -sL https://deb.nodesource.com/setup_16.x | bash -``: This command uses cURL to download a script from the NodeSource repository. The script sets up the package repository for Node.js version 16.x. The downloaded script is then piped to bash - to execute it.
3. ``sudo apt install nodejs``: This command uses the apt package manager to install Node.js.
4. ``node -v``: This command prints the installed version of Node.js to the console.
5. ``npm -v``: This command prints the installed version of npm (Node Package Manager) to the console.
6. ``curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | sudo apt-key add -``: This command uses cURL to download the GPG public key for the Yarn package repository. The downloaded key is then added to the system's keyring using sudo apt-key add -.
7. ``echo "deb https://dl.yarnpkg.com/debian/ stable main" | sudo tee /etc/apt/sources.list.d/yarn.list``: This command adds the Yarn package repository to the system's package manager sources. It appends the specified repository URL to the /etc/apt/sources.list.d/yarn.list file using tee.
8. ``sudo apt update && sudo apt install yarn``: This command updates the package manager's cache with the new repository information using apt update. It then installs Yarn using apt install yarn.
9. ``yarn --version``: This command prints the installed version of Yarn to the console. 
10. ``sudo -i -u ubuntu bash << EOF``: This command starts a new shell as the ubuntu user with root privileges. The subsequent commands until EOF will be executed within this subshell.
11. ``set -e``: This command sets the script to exit immediately if any command fails, just like the earlier set -e command.
12. ``cd /home/ubuntu``: This command changes the current directory to /home/ubuntu.
13. ``sudo npm install -g pm2``: This command uses npm to globally install the pm2 process manager. The -g flag indicates a global installation.
14. ``git clone <repository URL>``: This command clones the specified Git repository. Replace <repository_url> with the actual URL of the repository you want to clone.
15. ``cd node_with_docker``: This command changes the current directory to the cloned repository's directory (node_with_docker).
16. ``yarn install``: This command installs the project dependencies defined in the package.json file using Yarn.
17. ``pm2 start yarn --time --interpreter bash --name sample_node -- start -p 8000``: This command uses pm2 to start the application. It specifies the command to execute (yarn start -p 8000), sets the interpreter to bash, assigns the application a name (sample_node), and enables logging with timestamps (--time).

These commands collectively set up the EC2 instance, install Node.js, Yarn, and pm2, clone a Git repository, install dependencies, and start the application using pm2.

## Contact
You can reach me at: [Contact Us](https://www.eternalsoftsolutions.com/contact.php)
