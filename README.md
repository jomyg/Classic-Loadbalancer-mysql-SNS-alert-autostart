#  Classic-Loadbalancer-mysql-SNS-alert-autostart

[![Build](https://travis-ci.org/joemccann/dillinger.svg?branch=master)](https://travis-ci.org/joemccann/dillinger)

## Description

This script is very useful for those who are looking for getting an alert if an service is down. How can we get an alert if the MariaDB service is down. 
The script will also help to autostart using the help of crontab if the service is down

## Pre-Requests
```
Aws CLI for the SNS communication from EC2 to aws portal on both DB servers
SNS Topic with subscription (Email)
Classic load balancer with the internal load balacing enabled
```

### Lets go to the deployment

Httpd installation on both Db1 and db2 servers for the health check:
``` 
yum install httpd -y
systemctl enable httpd
systemctl start httpd
systemctl status httpd
``` 
Classic-load balancer creation
```
* FIrst create the Classic Lb and enable internal load balancer.
* Set listener port from 3306 to 3300
* Health check is TCP to 80 as we have already installed the httpd on db servers.
* Use allowed secuirty group and include the both db servers to LB and create.

After, On mysql client test server, you can test the Master master replication using the created Lb dns name:

[root@tester ~]# mysql -u project -p -h internal-master-internal-lb-76612346.ap-south-1.elb.amazonaws.com -e 'select * from company.employees;'
Enter password:
+------+-------+
| ID   | Name  |
+------+-------+
|   11 | jomy  |
|   12 | alex  |
|   13 | sabu  |
|   14 | anand |
+------+-------+
```
Now we can Create a SNS alert on AWS account with one email subscription.
Confirm the SNS email to validate the email with AWS. Copy the ARN of the SNS alert to configure the 

```
#!/bin/bash

SHELL=/bin/bash
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin

UP=$(service mariadb status | grep running | grep -v not | wc -l);
host=`hostname`
msg="Mysql service is down on the server ${host}"
if [ "$UP" -ne 1 ];
then
        aws sns publish --topic-arn arn:aws:sns:ap-south-1:154613195738:mysql-alert --message "${msg}";
        sudo service mariadb start
        aws sns publish --topic-arn arn:aws:sns:ap-south-1:154613195738:mysql-alert --message "Mysql started again now!";
else
        echo "All is well.";
fi
```
[root@db2 ~]# crontab -l
* * * * *       sh      sns.sh

You need to setup the aws cli
