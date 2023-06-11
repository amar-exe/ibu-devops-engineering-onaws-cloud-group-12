 # DevOps Engineering on AWS Project - Group 12
 
Project done in DevOps Engineering on AWS cloud course at International Burch University given by professor Dženana Dževlan.
 
## Group 12 Team members:
 
 
### Ejub Šabić  
<a href="https://github.com/https://github.com/saba8814" target="_blank">
<img src=https://img.shields.io/badge/github-%2324292e.svg?&style=for-the-badge&logo=github&logoColor=white alt=github style="margin-bottom: 5px;" />
</a>
<a href="https://linkedin.com/in/https://www.linkedin.com/in/ejub-sabic/" target="_blank">
<img src=https://img.shields.io/badge/linkedin-%231E77B5.svg?&style=for-the-badge&logo=linkedin&logoColor=white alt=linkedin style="margin-bottom: 5px;" />
</a>
<a href="https://instagram.com/ejub.sabic" target="_blank">
<img src=https://img.shields.io/badge/instagram-%23000000.svg?&style=for-the-badge&logo=instagram&logoColor=white alt=instagram style="margin-bottom: 5px;" />
</a>  
 
 
 
 
### Amar Fazlić  
<a href="https://github.com/https://github.com/amar-exe" target="_blank">
<img src=https://img.shields.io/badge/github-%2324292e.svg?&style=for-the-badge&logo=github&logoColor=white alt=github style="margin-bottom: 5px;" />
</a>
<a href="https://linkedin.com/in/https://www.linkedin.com/in/amar-fazlic-84b747184/" target="_blank">
<img src=https://img.shields.io/badge/linkedin-%231E77B5.svg?&style=for-the-badge&logo=linkedin&logoColor=white alt=linkedin style="margin-bottom: 5px;" />
</a>
<a href="https://instagram.com/amar.exe" target="_blank">
<img src=https://img.shields.io/badge/instagram-%23000000.svg?&style=for-the-badge&logo=instagram&logoColor=white alt=instagram style="margin-bottom: 5px;" />
</a>  
 
 
 
 
### Kerim Ahmedspahić  
<a href="https://github.com/Kerim357" target="_blank">
<img src=https://img.shields.io/badge/github-%2324292e.svg?&style=for-the-badge&logo=github&logoColor=white alt=github style="margin-bottom: 5px;" />
</a>
<a href="https://www.linkedin.com/in/kerim-ahmedspahic-35052b229/" target="_blank">
<img src=https://img.shields.io/badge/linkedin-%231E77B5.svg?&style=for-the-badge&logo=linkedin&logoColor=white alt=linkedin style="margin-bottom: 5px;" />
</a>
<a href="https://www.instagram.com/_kerim357/" target="_blank">
<img src=https://img.shields.io/badge/instagram-%23000000.svg?&style=for-the-badge&logo=instagram&logoColor=white alt=instagram style="margin-bottom: 5px;" />
</a>  
 
 
<br/>  
 
# Project Documentation
## Phase 1: Planning the design and estimating cost
During previous month we were in "DevOps Engineering on AWS cloud" course that required us to work on a final project titled "Building a Highly Available, Scalable Web Application". We started off with building architecture diagram proposal for the aformentioned application.
At first, we planned to have separate private subnets for both our databases and web application. Based on the example given by AWS "here", we've created following architecture diagram.
 
<img src=/docs/diagrams/initial_architecture_plan.png alt=instagram style="margin-bottom: 5px;" />

But as we worked on the project, we realized this plan wouldn't work. Instead, we used private subnets only for our databases, and our EC2 instances were publicly available.Even though we were initially worried about this change, it turned out to be the right decision. We successfully completed our project, and the public could access our EC2 instances over the Elastic Load Balancer DNS. This experience taught us the importance of being flexible and adaptable when working on a project. Therefore here's the final version of the architecture diagram that was in the end used in our project:

<img src=/docs/diagrams/actual_architecture.png alt=instagram style="margin-bottom: 5px;" />
 
When we've settled upon the architecture above, we've proceeded on to the next task, which was estimating costs of "Building a Highly Available, Scalable Web Application", we've headed to AWS Pricing Calculator. Estimation summary is given below, but for the full report click <a href=https://github.com/amar-exe/ibu-devops-engineering-onaws-cloud-group-12/blob/main/docs/CostEstimate.pdf>HERE.</a> \
<img src=https://i.ibb.co/wcN9Gkh/estimate.jpg alt=instagram style="margin-bottom: 5px;" />
## Phase 2: Creating a basic functional web application
We first set up our VPC, creating a secure environment for our Web Application. Our VPC contained 2 public and 2 private subnets. To allow public access to our application we had to set up an Internet Gateway to connect our VPC. We created a private and public route table to direct the traffic and establish a connection between the gateway and VPC.
 
We deployed one EC2 instance that we used to test the Web Application. Since we had no dedicated database we had to create a database when the instance launched. We entered the following into the user data section when creating an instance:
```
#!/bin/bash -xe
apt update -y
apt install nodejs unzip wget npm mysql-server -y
#wget https://aws-tc-largeobjects.s3.us-west-2.amazonaws.com/CUR-TF-200-ACCAP1-1-DEV/code.zip -P /home/ubuntu
wget https://aws-tc-largeobjects.s3.us-west-2.amazonaws.com/CUR-TF-200-ACCAP1-1-79581/1-lab-capstone-project-1/code.zip -P /home/ubuntu
cd /home/ubuntu
unzip code.zip -x "resources/codebase_partner/node_modules/*"
cd resources/codebase_partner
npm install aws aws-sdk
mysql -u root -e "CREATE USER 'nodeapp' IDENTIFIED WITH mysql_native_password BY 'student12'";
mysql -u root -e "GRANT all privileges on *.* to 'nodeapp'@'%';"
mysql -u root -e "CREATE DATABASE STUDENTS;"
mysql -u root -e "USE STUDENTS; CREATE TABLE students(
id INT NOT NULL AUTO_INCREMENT,
name VARCHAR(255) NOT NULL,
address VARCHAR(255) NOT NULL,
city VARCHAR(255) NOT NULL,
state VARCHAR(255) NOT NULL,
email VARCHAR(255) NOT NULL,
phone VARCHAR(100) NOT NULL,
PRIMARY KEY ( id ));"
sed -i 's/.*bind-address.*/bind-address = 0.0.0.0/' /etc/mysql/mysql.conf.d/mysqld.cnf
systemctl enable mysql
service mysql restart
export APP_DB_HOST=$(curl http://169.254.169.254/latest/meta-data/local-ipv4)
export APP_DB_USER=nodeapp
export APP_DB_PASSWORD=student12
export APP_DB_NAME=STUDENTS
export APP_PORT=80
npm start &
echo '#!/bin/bash -xe
cd /home/ubuntu/resources/codebase_partner
export APP_PORT=80
npm start' > /etc/rc.local
chmod +x /etc/rc.local
```
(This code can also be found in helper-scripts folder)
 
This ensured we have a MySQL server set up and ready for adding students. But it had it’s downsides, because each instance was isolated and the application was not scalable. Last task was to actually test this deployment with some basic CRUD tasks, which we performed manually, and below you can find the screenshots:
(insert screenshots)
## Phase 3: Decoupling the application components
As our original architecture included multiple AZs we did not need to do any updating or reconfiguration of the VPC, therefore we skipped task #1 and immediately went to task #2. In this task we've set up Amazon RDS, alongside with appropriate Security Groups to ensure that the database can only be accesed from the EC2 instances in our VPC, therefore making it more secure.
(PHOTO 5)
Next step was to setup Cloud 9 IDE, which we've done, together with security groups similiar to the one in previous tasks, making sure that Cloud 9 has access to both EC2 instances and RDS MySQL database. After we made sure Cloud 9 can "see" EC2 instance and RDS MySQL Database by using "ping" and "ssh" commands on their internal IPs, we proceeded on with task #4 which was provisioning AWS secrets manager. We've done that by running command below:
```
aws secretsmanager create-secret \
    --name Mydbsecret \
    --description "Database secret for web app" \
    --secret-string "{\"user\":\"...\",\"password\":\"...\",\"host\":\"rds-db-group12.cbnaxw1s5msc.us-east-1.rds.amazonaws.com\",\"db\":\"STUDENTS\"}"
```
(This code can also be found in helper-scripts folder)
 
Next we've made the new EC2 instance, that's been set up with following user data, so it uses RDS and does not actually run local MySQL server:
```
#!/bin/bash -xe
apt update -y
apt install nodejs unzip wget npm mysql-client -y
#wget https://aws-tc-largeobjects.s3.us-west-2.amazonaws.com/CUR-TF-200-ACCAP1-1-DEV/code.zip -P /home/ubuntu
wget https://aws-tc-largeobjects.s3.us-west-2.amazonaws.com/CUR-TF-200-ACCAP1-1-79581/1-lab-capstone-project-1/code.zip -P /home/ubuntu
cd /home/ubuntu
unzip code.zip -x "resources/codebase_partner/node_modules/*"
cd resources/codebase_partner
npm install aws aws-sdk
export APP_PORT=80
npm start &
echo '#!/bin/bash -xe
cd /home/ubuntu/resources/codebase_partner
export APP_PORT=80
npm start' > /etc/rc.local
chmod +x /etc/rc.local
```
(This code can also be found in helper-scripts folder)
 
After this we proceeded on to migrate the database from Phase 2 into to the RDS, we've done that by running following scripts from Cloud9 environment:
 
```
mysqldump -h 10.0.0.204 -u nodeapp -p --databases STUDENTS > data.sql
mysql -h rds-db-group12.cbnaxw1s5msc.us-east-1.rds.amazonaws.com -u admin -p  STUDENTS < data.sql
```
(This code can also be found in helper-scripts folder)
 
At this point we made sure that database was successfully migrated, retested all CRUD operations as per task #7, and terminated both EC2 instances, as we concluded that they'll be replaced by ELB and auto scaling EC2 instances, as per proposed architecture in Phase 4. 
## Phase 4: Implementing high availability and scalability
 First and most important thing about high availability and scalability is the Application Load Balancer. We have used 2 availability zones: ```us-east-1a``` and ```us-east-1b```. We referenced our University Course Lab: Creating a Highly Available Environment for this.

After creating a load balancer we needed to set up our EC2 instances. First we created a Launch Template and enabled auto assigning IPv4 addresses to our to enable public access to our Web Application.

Once our ALB was up and running we tested it by entering the Web App over the DNS address that was assigned to the ALB. We did a few tests on different devices to check if the database was syncing properly over all devices.

We also needed to test our application to check if it truly was highly scalable. For this we used https://github.com/alexfernandez/loadtest and this script:
```
loadtest --rps 1000 -c 500 -k http://alb-group12-1087131232.us-east-1.elb.amazonaws.com/students
```

Our ALB was configured to have a maximum of 6 instances, minimum of 1 and a desired capacity of 2 instances. With the script above we made 1000 requests every seconds, with 500 requests concurrently. The Application handled it without problem, turning on 5 instances to process the load.
