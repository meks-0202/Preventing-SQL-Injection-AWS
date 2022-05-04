# Preventing SQL Injection, Geo Location and Query String

### In this project I tried to implement AWS WAF with ALB to block SQL Injection, Geo Location and Query string. For this I created an Application Load Balancer, the Elastic load balancer automatically distributes the incoming appliication traffic accross 2 EC2 instances. After this I created a set of rules to block access from certain geo locations, SQL Injection and block certain query string parameters.

Before going to the walkthrough, let's have a look at the basics for this project.
## AWS WAF
* AWS  WAF is a web application firewall that helps you to protect your web  applications against common web exploits that might affect availability  and compromise security.
* AWS  WAF gives you control over how traffic reaches your applications by  enabling you to create security rules that block common attack patterns  like SQL injection and cross-site scripting.
* It only allows the request to reach the server based on the rules or patterns you define.
* Users create their own rules and specify the conditions that AWS WAF searches for in incoming web requests.
* The cost of WAF is only for what you use. 
* The pricing is based on how many rules you deploy and how many web requests your application receives.
* For example, you can deploy AWS WAF on Amazon CloudFront, Load Balancer or API Gateways.

## Elastic Load Balancer
* ELB is a service that automatically distributes incoming application traffic and scales resources to meet traffic demands.
* It helps in adjusting capacity according to incoming application and network traffic.
* It  can be enabled within a single availability zone or across multiple  availability zones to maintain consistent application performance.
* ELB offers features like:
   * Detection of unhealthy EC2 instances.
   * Spreading EC2 instances across healthy channels only.
   * Centralized management of SSL certificates.
   * Optional public key authentication.
   * Support for both IPv4 and IPv6.
* ELB accepts incoming traffic from clients and routes requests to its registered targets.
* When  an unhealthy target or instance is detected, ELB stops routing traffic  to it and resumes only when the instance is healthy again.
* ELB monitors the health of its registered targets and ensures that the traffic is routed only to healthy instances.
* ELB's are configured to accept incoming traffic by specifying one or more listeners. A listener is a process that checks for connection requests.
* Listeners  are configured with a protocol and port number from the client to the  ELB and vice-versa i.e., back from ELB to the client.
* ELB supports the following :
   * Application Load Balancers.
   * Network Load Balancers.
   * Gateway Load Balancers.
   * Classic Load Balancers.
* Each load balancer is configured differently.
* For Application and Network Load Balancers, you register targets in target groups and route traffic to target groups.
* Gateway Load Balancers use Gateway Load Balancer endpoints to securely exchange traffic across VPC boundaries.
* For Classic Load Balancers, you register instances with the load balancer.
* AWS  recommends users to work with Application Load Balancer to use multiple  Availability Zones because if one availability zone fails, the load  balancer can continue to route traffic to the next available one.
* We can have our load balancer be either internal or internet-facing.
* The  nodes of an internet-facing load balancer have Public IP addresses, and  the DNS name is publicly resolvable to the Public IP addresses of the  nodes.
* Due to the point above, internet-facing load balancers can route requests from clients over the Internet.
* The  nodes of an internal load balancer have only Private IP addresses, and  the DNS name is publicly resolvable to the Private IP addresses of the  nodes.
* Due  to the point above, internal load balancers can only route requests  from clients with access to the VPC for the load balancer.
* Both internet-facing and internal load balancers route requests to your targets using Private IP addresses.
* Your targets do not need Public IP addresses to receive requests from an internal or an internet-facing load balancer.
* You  can create your own rules, depending on your requirements, whether to  block or allow the incoming and outgoing request. You can also customise  the string that appears in your web request.
*  Blocking malicious requests.
* You can also configure rules in AWS WAF to identify and block web requests threats like SQL injections and cross-site scripting.
* Tune your rules and monitor traffic                                
* AWS WAF also allows us to review our rules and customize them to prevent new attacks from reaching the server.

<img src='https://github.com/meks-0202/Preventing-SQL-Injection-AWS/blob/main/Assests/1.png' align="middle">

## Walkthrough
### 1) Launching Instances
Create 2 instances in EC2 both with 'Amazon Linux 2 AMI' (image) and instance type 't2 micro'.
<img src="https://github.com/meks-0202/Preventing-SQL-Injection-AWS/blob/main/Assests/instance%201.png"> <img src="https://github.com/meks-0202/Preventing-SQL-Injection-AWS/blob/main/Assests/instance2.png">

<b>Configuring instance details: </b>
  * Auto-assign Public IP : Enable
  * In Advanced Details, under the User data use the following script to create an HTML page served by an Apache httpd web server.
  ```shell
  #!/bin/bash
sudo su
yum update -y
yum install httpd -y
systemctl start httpd
systemctl enable httpd
echo "<html><h1> Welcome to Test Server 1 </h1><html>" >> /var/www/html/index.html
  ```
  <img src="https://github.com/meks-0202/Preventing-SQL-Injection-AWS/blob/main/Assests/instance%203.png"><img src="https://github.com/meks-0202/Preventing-SQL-Injection-AWS/blob/main/Assests/instance%204.png">
The rest of the fields are set as default.No need to change anything in next page for storage.
<p><b>Tags:</b></p> 

* Key: Name
* Value: Server 1

<img src="https://github.com/meks-0202/Preventing-SQL-Injection-AWS/blob/main/Assests/instance%205.png"> 
<p><b>Security Groups: </b>Create a new security group</p>
Name: Server-SG

Description: can be anything you want

  Add Rules:
  * Type: SSH.
  * Source: Anywhere (From ALL IP addresses accessible).
  * Type: HTTP
  * Source: Anywhere (From ALL IP addresses accessible).
  * Type: HTTPS
  * Source: Anywhere (From ALL IP addresses accessible).

<img src="https://github.com/meks-0202/Preventing-SQL-Injection-AWS/blob/main/Assests/instance%206.png">

Review and launch!
Proceed similarly to create another instance with the following script
  ```shell
  #!/bin/bash
sudo su
yum update -y
yum install httpd -y
systemctl start httpd
systemctl enable httpd
echo "<html><h1> Welcome to Test Server 2 </h1><html>" >> /var/www/html/index.html
  ```
Also while configuring security group this time chose 'Select and existing security group' and select the group created previously.

<img src="https://github.com/meks-0202/Preventing-SQL-Injection-AWS/blob/main/Assests/instance%208.png">

### 2) Creating a WAF load balancer and target group
Navigate to Load Balancers from the left side menu under Load balancing and go to, Create Load Balancer. I used Application load balancer for this project.

<img src="https://github.com/meks-0202/Preventing-SQL-Injection-AWS/blob/main/Assests/waf%201.png">

The configuration setup for load balancer is as follows: 

* Name: Enter WAF-LoadBalancer
* Scheme: internet-facing (an Internet-facing load balancer routes requests from clients over the Internet to targets).
* IP address type: IPv4
* Listeners: 
      * Load Balancer Protocol : HTTP
      * Load Balancer Port  : 80
* VPC : Select default VPC.
* Availability zones: Select all available zones using the checkbox.
   
* Tags:

  * Key : Name
  * Value : WAF-LoadBalancer
* Security Groups: Select an existing security group, Webserver-SG (the Security Group was already created during EC2 instances launch).

<img src="https://github.com/meks-0202/Preventing-SQL-Injection-AWS/blob/main/Assests/waf%202.png"><img src="https://github.com/meks-0202/Preventing-SQL-Injection-AWS/blob/main/Assests/waf%20%203.png">
#### For Target group click on create new target group
 
 * Target group name : sqliWAFTargetGroup  
 * Under Health check settings : 
     
     * Path : /index.html
 * Under Advanced health check settings:
  
    * Healthy threshold : 3   
    * Unhealthy threshold: 2 (Default)   
    * Timeout: 5 seconds (Default)
    * Interval: Enter 6 seconds
    * Success codes: 200 (Default)
<img src="https://github.com/meks-0202/Preventing-SQL-Injection-AWS/blob/main/Assests/waf%20%204.png">
<img src="https://github.com/meks-0202/Preventing-SQL-Injection-AWS/blob/main/Assests/waf%206.png">
<img src="https://github.com/meks-0202/Preventing-SQL-Injection-AWS/blob/main/Assests/waf%207.png">
<img src="https://github.com/meks-0202/Preventing-SQL-Injection-AWS/blob/main/Assests/waf%209.png">

### 3) Testing for SQL Injection
Once both the target shows a healthy status (important), navigate to load balancer and check the dns in your our browser. It should show both the servers on refreshing as the traffic is distribute to both the serves. 
Next I check for SQL Injection:- 
1) I added a directory to the url with a parameter and value, but since we do no have any folder in our web page here we find a status not found for the page

<img src="https://github.com/meks-0202/Preventing-SQL-Injection-AWS/blob/main/Assests/sqli%201.jpg">
2) next i added a parameter admin to index.html 

  * /?admin=1' OR '1=1'; -- 
  * /?admin=45

Here SQL Injection is possible as we can successfully inject a SQL query and manipulate the output.
the Query string went inside the server  and the server always passes the query string inside and it is resolved  by the code that we write. Here the query string is passed and there  is no code to resolve the this but it wont throw any error it just  becames an unused value. so you got a response back.   
<img src="https://github.com/meks-0202/Preventing-SQL-Injection-AWS/blob/main/Assests/sqli%202.jpg">
<img src="https://github.com/meks-0202/Preventing-SQL-Injection-AWS/blob/main/Assests/sqli%203.jpg">

### 4) Creating AWS WAF web ACL
Navigate to WAF under security, identity and compliance section. select Web ACL's and Create web ACL .
 Describing web ACL and associate it to AWS resources :
  
   * Name : AWS-WAF-Web-ACL
   * Description : WAF for  preventing SQL Injection, Geo location and Query String parameters
   * Resource type : Regional resources
   * Region : I selected US East (N.Virginia) from the dropdown (I'm working on N.Virginia region).
   * Associated AWS resources : Add AWS resources 
      
      * Resource type : Application Load Balancer
      * Select sqli-LoadBalancer Load balancer from the list.
   <img src="https://github.com/meks-0202/Preventing-SQL-Injection-AWS/blob/main/Assests/acl%201.png">
   <img src="https://github.com/meks-0202/Preventing-SQL-Injection-AWS/blob/main/Assests/acl%202.png">
   
   Next step is to add rules.
   1) Rule for geo location.
      
      * Rule type : Rule builder
      * Name : GeoLocationRestriction
      * Type : Regular type
      * If a request : Doesn't match the statement (NOT)
      * Inspect : Originates from a country in 
      * Country codes : <Your Country> In this I selected India-IN
      * IP address to use to determine the country of origin : Select Source IP address
      * Under Then : Action Select Block.
   <b>Add rule.</b>
  <img src="https://github.com/meks-0202/Preventing-SQL-Injection-AWS/blob/main/Assests/acl%203.png">
  <img src="https://github.com/meks-0202/Preventing-SQL-Injection-AWS/blob/main/Assests/acl%204.png">

2) Rule for query string restriction
  
   * Name : Enter QueryStringRestriction
   * Type : Select Regular type
   * If a request : matches the statement
   * Inspect : Query string
   * Match type : Contains string
   * String to match : admin
   * Text transformation : default.
   * Under Then : Action Select Block.<b>Add rules.</b>
   Now Anytime in the request URL contains a query string as admin WAF will block that request.

3) Rule for SQLI
  
   * Under Rules, click on Add rules and then select Add managed rule groups.   
   * Click on AWS managed rule groups.
   * Scroll down to SQL database and enable the corresponding Add to web ACL button. 
  <img src="https://github.com/meks-0202/Preventing-SQL-Injection-AWS/blob/main/Assests/acl%205.png">
  
  Create WAF ACL
  <img src="https://github.com/meks-0202/Preventing-SQL-Injection-AWS/blob/main/Assests/acl%206.png">
  <img src="https://github.com/meks-0202/Preventing-SQL-Injection-AWS/blob/main/Assests/acl%207.png">
  
### 5) Testing
  Let's go back again to the dns server and check for the same query set strings.
  * /items?id=98
  * ?admin=1' OR '1=1';--
  This time we get an error 403 forbidden. 
  <img src="https://github.com/meks-0202/Preventing-SQL-Injection-AWS/blob/main/Assests/test.png">
  We successfuly implemented WAF to block SQL Injection, geo location and query string.
