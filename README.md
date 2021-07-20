# AWS Fault Tolerant Website

### Architecture
<p align="center"> <img src="https://lucid.app/publicSegments/view/f48de5e0-e4d8-44fc-aa4b-72b7696024bc/image.png" alt="Architecture" /> </p>

### Description
We are creating a web server linked with MySQL and Wordpress. We will then create a snapshot image of the web server and dependancies for autoscaling purposes, which will all be attatched to an application load balancer. We will be using free tier services with the exception of some services like RDS instances which do cost money **(IMPORTANT!)**.

### Prerequisites
1. You need to have an AWS account. They do offer a free tier account.
2. You can run the installation steps through the AWS console or the AWS Command Line.
3. You should have some knowledge of the AWS console and AWS services.

### Dependancies
1. Wordpress
2. Apache

### Installation
**1. Create RDS instance**

  <ol type="a">
    <li>Select MySQL for the DB type</li>
    <li>Use the production template for high performance and availability. They do offer a free tier template</li>
    <li>Change the DB instance identifier</li>
    <li>Create master username for database</li>
    <li>Change master password for database</li>
    <li>Change DB instance size to a burstable classes and set to a db.t2.micro, minimizing the size of our database and saving money</li>
    <li>Set Storage to General purpose SSD to keep our database cheap</li>
    <li>Deploy into default VPC</li>
    <li>Go to additional configurations and name your database, so WordPress can access it</li>
    <li>Leave everything else as is, double check the estimated monthly cost</li>
  </ol>

**2.Create security group called Web-DMZ**
  <ol type="a">
    <li>Create new security group called web-DMZ</li>
  <li>Add SSH rule with TCP on port 22 from anywhere</li>
  <li>Add HTTP rule with TCP on port 80 from anywhere</li>
  <li>Launch and create a key pair</li>
  </ol>

**IMPORTANT: You should not usually allow port access from anywhere, however since this project is for testing we will**

**3.Open up the default VPC to set up a port for MySQL traffic**
  <ol type="a">
    <li>Edit the default security group</li>
    <li>Add rule with TCP protocol and port range 3306 with our web-dmz security group as our source</li>
    <li>Configure security grou, select web-DMZ</li>
  </ol>
  
**4. Create an Application Load Balancer**
  <ol type="a">
    <li>Create a load balancer and select the application load balancer</li>
    <li>Give it a name and should be internet facing</li>
    <li>Should be listening through HTTP</li>
    <li>Leave it in default VPC</li>
    <li>Select all availability zones</li>
    <li>Configure routing target route: name it MyWebServers</li>
  </ol>
  
**5. Launch an EC2 instance**
  <ol type="a">
    <li>Choose the linux 2 ami</li>
    <li>We are gonna use the general purpose free tier t2.micro</li>
    <li>Go to advanced details, pass it the bootstrap script at the bottom of this section to install WordPress</li>
    <li>Click add storage, leave everything as default</li>
    <li>Click Add Tags</li>
    <li>Click Configure Security Group</li>
    <li>Add Web-DMZ security group</li>
    <li>Click review and launch</li>
    <li>Select key pair</li>
  </ol>
  
  ```
  #!/bin/bash
  yum install httpd php php-mysql -y
  amazon-linux-extras install -y php7.2
  cd /var/www/html
  wget https://wordpress.org/wordpress-5.4.1.tar.gz
  tar -xzf wordpress-5.4.1.tar.gz
  cp -r wordpress/* /var/www/html/
  rm -rf wordpress
  rm -rf wordpress-5.4.1.tar.gz
  chmod -R 755 wp-content
  chown -R apache:apache wp-content
  service httpd start
  chkconfig httpd on
  ```
  
**6. Configure WordPress Site**
  <ol type="a">
    <li>Paste the web server's public IP Adress into the browser</li>
    <li>Should bring you to a wordpress configuration page</li>
    <li>Enter in your database name, username, and password</li>
    <li>Change local host to your database endpoint which is under connectivity when you click on your database in the console</li>
    <li>Finish configuring your wordpress site with the site name, user, and password</li>
    <li>You should now be in the administrator dashboard</li>
    <li>Go to settings on the WordPress site</li>
    <li>Current WordPress URL is the webserver IP Address, we are going to change it to the Application Load Balancer DNS Name</li>
  </ol>

**7. Register Target for Application Load Balancer**
  <ol type="a">
    <li>Go to EC2 dashboard, under load balancers, and click on target groups</li>
    <li>Click on register a target</li>
    <li>Add our web server instance as a target</li>
  </ol>
  
**8. Create an image of our EC2 instance for auto-scaling**
  <ol type="a">
  <li>Select the EC2 instance/webserver in the console</li>
  <li>Click on actions, click image, click create image</li>
  <li>Name the image, and create image, which will be saved under AMI</li>
  </ol>
  
**9. Create launch configuration**
  <ol type="a">
  <li>Go to my-ami and select the image you created in the previous step</li>
  <li>Select a t2.micro</li>
  <li>Give launch configuration a name</li>
  <li>Add a bootstrap script to update servers when they come online, found at the bottom of the section</li>
  <li>Leave storage as default</li>
  <li>Select web-DMZ seceurity group</li>
  <li>Launch and select key pair</li>
  </ol>
  
  ```
  #!/bin/bash
  yum update -y
  ```
**10. Create Auto-Scaling Group**
  <ol type="a">
  <li>Give AS group a name</li>
  <li>Set group size to 2</li>
  <li>Select availability zones</li>
  <li>Under advanced details select to recieve traffice from one or more load balancers</li>
  <li>Then give the target group for your webservers</li>
  <li>Configure Scaling policies</li>
  <li>Use scaling policies to adjust the capacity, scale between 2 and 3</li>
  <li>Set metric type to be CPU utilization and target value to be 90%</li>
  <li>Default rest and launch</li>
  </ol>
  
**IMPORTANT:** Remember, if you dont want to be charged, make sure to delete your RDS instance, your ALB, and your autoscaling groups and servers.
