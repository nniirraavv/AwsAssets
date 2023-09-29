## AWSTemplateFormatVersion:
Specifies the version of the CloudFormation template format being used. In this case, it's the 2010-09-09 version.

```bash
AWSTemplateFormatVersion: '2010-09-09'

```
## Parameters:
Defines the parameters for the CloudFormation stack. In addition to InstanceType and SSHKey parameters, it also defines parameters related to the WordPress database, including DatabaseName, DatabaseUsername, DatabasePassword, DatabaseHost, and TablePrefix.


```bash
Parameters:
  InstanceType:
    Type: String
    Default: t2.medium
    Description: Enter the EC2 instance type for the WordPress server.
  SSHKey:
    Type: AWS::EC2::KeyPair::KeyName
    Description: Name of an existing EC2 KeyPair to enable SSH access to the instance.
  DatabaseName:
    Type: String
    Description: The name of the WordPress database
  DatabaseUsername:
    Type: String
    Description: The username for the WordPress database
  DatabasePassword:
    Type: String
    Description: The password for the WordPress database
  DatabaseHost:
    Type: String
    Description: The host of the WordPress database
  TablePrefix:
    Type: String
    Description: The table prefix for the WordPress database (optional)

```
## Resources
Defines the resources to be created in the CloudFormation stack. It creates an EC2 instance (AWS::EC2::Instance) named WordPressInstance. The instance properties include the instance type (InstanceType parameter), the Amazon Machine Image (AMI) ID (ImageId), the EC2 Key Pair (SSHKey parameter), the security group ID (wordpressinstanceSecurityGroup), and tags for the instance. The UserData property contains a bash script that is base64-encoded and executed during instance launch. The script updates packages, installs Apache, PHP, MySQL, and additional PHP extensions required by WordPress.

Defines the resources to be created in the CloudFormation stack. In this case, it creates an EC2 instance (AWS::EC2::Instance) named WordPressInstance. The properties of the instance are specified, including the instance type (InstanceType parameter), the Amazon Machine Image (AMI) ID (ImageId), the EC2 Key Pair (SSHKey parameter), the security group ID (SecurityGroupIds), and tags for the instance (Tags). The UserData property contains a bash script that is base64-encoded and executed during instance launch. This script installs Apache, PHP, MySQL, and downloads and configures WordPress.

```bash
Resources:
  WordPressInstance:
    Type: AWS::EC2::Instance
    Properties:
      InstanceType: !Ref InstanceType
      ImageId: ami-053b0d53c279acc90
      KeyName: !Ref SSHKey
      SecurityGroupIds:
        - !Ref wordpressinstanceSecurityGroup
      Tags:
        - Key: Name
          Value: Wordpress-DBserver 

 ```
## Userdata
Explanation of each line in the UserData script from the beginning:
This line specifies that the script should be run using the Bash shell.

```bash
UserData:
        Fn::Base64: !Sub |
          #!/bin/bash
```
This command updates the package lists on the system, ensuring that the latest package information is available.

```bash
sudo apt-get update

```
### Install apache2, mysql and PHP

This command installs a series of packages required for running WordPress and its dependencies. It installs Apache web server (apache2), PHP, MySQL server and client (mysql-server, mysql-client), and several PHP extensions (php-bcmath, php-imagick, php-mysql, php-xml, php-zip, php-curl).

```bash
sudo apt-get install -y apache2 php mysql-server mysql-client php-bcmath php-imagick php-mysql php-xml php-zip php-curl

```
"sudo systemctl enable apache2" enables the Apache service to start automatically on system boot.

"sudo systemctl start apache2" command starts the Apache service immediately.

"sudo systemctl enable mysql" command enables the MySQL service to start automatically on system boot.

"sudo systemctl start mysql" This command starts the MySQL service immediately.

```bash
sudo systemctl enable apache2
sudo systemctl start apache2
sudo systemctl enable mysql
sudo systemctl start mysql

```

These commands navigate to the /tmp directory, download the latest version of WordPress as a tar.gz archive, extract the contents of the archive, and move the extracted files and directories to the /var/www/html/ directory, which is the default document root for Apache.

```bash
cd /tmp
wget https://wordpress.org/latest.tar.gz
sudo tar -xvf latest.tar.gz
sudo mv wordpress/* /var/www/html/

```
These commands navigate to the user's home directory (cd) and remove the default index.html file that comes with Apache. This step is necessary to ensure that WordPress can be accessed as the main website instead of the default Apache page.

```bash
cd
sudo rm -vf /var/www/html/index.html

```
These commands change the ownership (chown) of the /var/www/html/ directory and its contents to the www-data user and group. The www-data user is the default user that Apache runs as. The chmod command sets the permissions of the directory and its contents to 755, which gives read, write, and execute permissions to the owner (www-data) and read and execute permissions to others.

### Change ownership and permissions of folder /var/wwwhtml

```bash
sudo chown -R www-data:www-data /var/www/html/
sudo chmod -R 755 /var/www/html/

```
The script then continues with the configuration of the WordPress database.

"sudo mysql -e "CREATE DATABASE ${DatabaseName};" command uses the mysql command-line tool to create a new database with the name specified by the ${DatabaseName} parameter. This database will be used by WordPress.

"sudo mysql -e "CREATE USER '${DatabaseUsername}'@'localhost' IDENTIFIED BY '${DatabasePassword}';" command creates a new user with the provided ${DatabaseUsername} and ${DatabasePassword} values. This user will be associated with the WordPress database and will be used for authentication.

"sudo mysql -e "GRANT ALL PRIVILEGES ON ${DatabaseName}.* TO '${DatabaseUsername}'@'localhost';" command grants all privileges to the previously created user (${DatabaseUsername}) on the WordPress database (${DatabaseName}). This ensures that the user has the necessary permissions to perform operations on the database.

"sudo mysql -e "FLUSH PRIVILEGES;"  command flushes the privileges to ensure that the changes made in the previous steps take effect.

### Create wordpress database
```bash
# Create the WordPress database
sudo mysql -e "CREATE DATABASE ${DatabaseName};"

# Create a database user and grant privileges
sudo mysql -e "CREATE USER '${DatabaseUsername}'@'localhost' IDENTIFIED BY '${DatabasePassword}';"
          sudo mysql -e "GRANT ALL PRIVILEGES ON ${DatabaseName}.* TO '${DatabaseUsername}'@'localhost';"
          sudo mysql -e "FLUSH PRIVILEGES;"
          sudo mysql -e "exit" 

```
configuring the wp-config.php file 
"sudo touch /var/www/html/wp-config.php" command creates a new wp-config.php file in the /var/www/html/ directory. This file will store the configuration settings for WordPress.

"sudo mv /var/www/html/wp-config-sample.php /var/www/html/wp-config.php" command renames the wp-config-sample.php file to wp-config.php, which is the actual configuration file used by WordPress.

"sudo sed -i "s/database_name_here/${DatabaseName}/" /var/www/html/wp-config.php" command replaces the placeholder value 'database_name_here' in the wp-config.php file with the actual ${DatabaseName} value, which is the name of the WordPress database.

"sudo sed -i "s/username_here/${DatabaseUsername}/" /var/www/html/wp-config.php" command replaces the placeholder value 'username_here' in the wp-config.php file with the actual ${DatabaseUsername} value, which is the username of the WordPress database user.

"sudo sed -i "s/password_here/${DatabasePassword}/" /var/www/html/wp-config.php" command replaces the placeholder value 'password_here' in the wp-config.php file with the actual ${DatabasePassword} value, which is the password of the WordPress database user.

"sudo sed -i "s/localhost/${DatabaseHost}/" /var/www/html/wp-config.php" command replaces the default value 'localhost' in the wp-config.php file with the actual ${DatabaseHost} value, which is the host of the WordPress database.


"sudo systemctl restart apache2" command restarts the Apache service to apply the configuration changes made during the installation of WordPress.

### Configure wp-config.php file in wordpress

```bash
# Configure wp-config.php
sudo touch /var/www/html/wp-config.php
sudo mv /var/www/html/wp-config-sample.php /var/www/html/wp-config.php
sudo sed -i "s/database_name_here/${DatabaseName}/" /var/www/html/wp-config.php
sudo sed -i "s/username_here/${DatabaseUsername}/" /var/www/html/wp-config.php
sudo sed -i "s/password_here/${DatabasePassword}/" /var/www/html/wp-config.php
sudo sed -i "s/localhost/${DatabaseHost}/" /var/www/html/wp-config.php
sudo sed -i "s/wp_/${TablePrefix}/" /var/www/html/wp-config.php

# Restart services
sudo systemctl restart apache2
sudo systemctl restart mysql

```

### Install phpmyadmin
After that, the script moves on to installing phpMyAdmin, which is a web-based tool for managing MySQL databases.These commands navigate to the /tmp directory, download the specified version of phpMyAdmin as a .zip archive, install the unzip package, and extract the contents of the archive to the /var/www/html/ directory.. It is then moved to /var/www/html/phpmyadmin for better accessibility. The configuration file config.inc.php is copied from config.sample.inc.php, and a sed command is used to update the host value in the configuration file with the value from the DatabaseHost parameter.

Restarts the Apache service to apply the changes made during the installation and configuration of WordPress and phpMyAdmin.

```bash
cd /tmp
wget https://files.phpmyadmin.net/phpMyAdmin/5.1.1/phpMyAdmin-5.1.1-all-languages.zip
sudo apt-get install -y unzip
sudo unzip phpMyAdmin-5.1.1-all-languages.zip -d /var/www/html/

sudo mv /var/www/html/phpMyAdmin-5.1.1-all-languages /var/www/html/phpmyadmin
sudo cp /var/www/html/phpmyadmin/config.sample.inc.php /var/www/html/phpmyadmin/config.inc.php
sudo sed -i "s/\['host'\] = 'localhost'/\['host'\] = '${DatabaseHost}'/" /var/www/html/phpmyadmin/config.inc.php

# Restart Apache
sudo systemctl restart apache2

```
### Configure Security group
Creates an EC2 security group (AWS::EC2::SecurityGroup) named wordpressinstanceSecurityGroup that allows incoming traffic on ports 80, 443, and 22. This allows access to the web server and SSH.

```bash

wordpressinstanceSecurityGroup:  
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: Enable ssh http https and mysql ports
      SecurityGroupIngress:
        - IpProtocol: tcp
          FromPort: '80'
          ToPort: '80'
          CidrIp: 0.0.0.0/0
        - IpProtocol: tcp
          FromPort: '443'
          ToPort: '443'
          CidrIp: 0.0.0.0/0       
        - IpProtocol: tcp
          FromPort: '22'
          ToPort: '22'
          CidrIp: 0.0.0.0/0
        - IpProtocol: tcp
          FromPort: '80'
          ToPort: '80'
          CidrIp: 0.0.0.0/0        

```

## Outputs
There is an output named WordPressInstancePublicIP which retrieves the public IP address of the WordPress instance.
WordPressInstancePublicIP is the logical name given to the output. You can use this name to reference the output value in other parts of your infrastructure or templates.

Value: !GetAtt WordPressInstance.PublicIp specifies the value of the output. The !GetAtt function is used to retrieve the value of an attribute from a resource. In this case, it retrieves the value of the PublicIp attribute from the WordPressInstance resource.
So, when you access the value of WordPressInstancePublicIP, you will get the public IP address of the WordPress instance, allowing you to access your WordPress website using that IP address.

```bash
Outputs:
  WordPressInstancePublicIP:
    Value: !GetAtt WordPressInstance.PublicIp
    Description: The public IP address of the WordPress instance.
```
