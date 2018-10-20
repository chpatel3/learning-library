# Big Data on Oracle Cloud Infrastructure

## Table of Contents

[Overview](#overview)

[Pre-Requisites](#pre-requisites)

[Practice 1: Setup Windows 10 VM](#practice-1-setup-windows-10-vm)

[Practice 2: Log in OCI Web Console ](#practice-1-log-in-oci-web-console)

[Practice 3:  Cloudera Enterprise Data Hub on OCI ](#practice-3-cloudera-enterprise-data-hub-on-oci)

[Practice 4: Log in to Cloudera Manager](#practice-4-log-in-to-cloudera-manager)

[Practice 5: Benchmarking with Teragen/Terasort](#practice-5-benchmarking-with-teragen/terasort)

[Try Oracle Cloud Infrastructure for FREE](#try-oracle-cloud-infrastructure-for-free)

## Overview

Cloudera Enterprise Data Hub brings together the best big data technologies from the Apache Hadoop ecosystem, including HDFS, HBase, Hive, Spark, Impala, Solr, and Kudu, and adds consistent security, granular governance, and full support. It is the fastest, most secure, and easiest to use big data software available.

Cloudera on Oracle Cloud Infrastructure is a joint solution that combines the flexibility and performance of Oracle Cloud Infrastructure with the scalable data management of Cloudera Enterprise Data Hub. Our solution enables customers to realize their data strategies, from operational to analytics, with amazing performance, an unmatched data ecosystem, and the inherent benefits of moving from on-premises fixed infrastructure to elastic cloud infrastructure.

In this lab, attendees will manage a Cloudera Enterpise Data Hub on OCI and use TeraSort, a popular benchmark used to measure MapReduce performance of an Apache™ Hadoop® cluster.  The data contained in this presentation details performance of YARN scheduled TeraSort jobs on Cloudera Enterprise Data Hub running on OCI.


## Pre-Requisites 

Each attendee would be provided with a set of credentials for Oracle Cloud Infrastructure. The credentials will include 

- Tenancy Information
- Username
- Password

## Practice 1: Setup Windows 10 VM 

1) Each attendee is provided with a laptop with a Virtual Box. In the Virtual Box, Please start the Windows 10 VM (name of the VM is same as the ID of this Hands on Lab - HOL6379) 

![](media/image1.png)

## Practice 2: Log in OCI Web Console 

1) Open Chrome Browser and Log into Oracle Cloud Infrastructure (OCI) Console. 
https://console.us-phoenix-1.oraclecloud.com/
Add the Cloud Tenant name provided to you. 

![](media/image2.png)

2) Sign in by providing your username and temporary password. 

![](media/image3.png)

3) Change the temporary password provided to you. 

```
Following password will suffice password requirements 
Oracleoow2018! 
```

![](media/image4.png)

After log in, you are at the home page of OCI web console. 

<img width="1240" alt="ocihomepage" src="https://user-images.githubusercontent.com/32341971/47040466-2c38dd80-d13b-11e8-8ae1-79f8d48a3214.png">

4) Navigate to the **MENU** on the top left, and Click on **Networking** --> **Virtual Cloud Networks** and select the compartment associated with your userID. 

For instance: For **Username** *OOW10*  use **Compartment** *OOW-HOL10*

![](media/image6.png)

5) Open Gitbash in your windows 10 VM. Its already installed. You will use Gitbash for *ssh*. 

![](media/gitbash1.png)

## Practice 3: Cloudera Enterprise Data Hub on OCI 

Each attendee group is provided with a pre-installed Cloudera EDH on OCI. The setup architecture is provided below. 

![](media/arch1.png)

The setup consists of three worker nodes, one utility node and a bastion host. The utility node also hosts the Cloudera Manager, which we will use to manage the cluster. 

In this exercise we will quickly take a look at the infrastructure components that were setup on OCI for Cloudera EDH installation. 

1) Navigate to the **MENU** on the top left, and Click on **Networking** --> **Virtual Cloud Networks**. It provides details of a virtual cloud network that is setup to host the virtual instances. 

		NOTE: Make sure to select compartment compartment associated with your userID. 

![](media/image7.png)

2) Click on **"cloudera_vcn"** and you will see details of the virtual cloud network. 
	- Subnets 
	- Route tables
	- Security Lists
	- Internet Gateway
	
Worker hosts are in private subnets with no access to the internet. Bastion host is in bastion subnet with public access. Utility host is also in public subnet with internet access. 

You can navigate to security lists and route tables to get more details. 

![](media/image8.png)

3) Navigate to the **MENU** on the top left, and Click on **Compute** --> **Instances**. It provides details of a virtual cloud network that is setup to host the virtual instances. 

The shapes of the Hosts are: 
- Bastion (VM.Standard1.2) - 2 OCPUs 
- Utility (VM.Standard2.16) - 16 OCPUs 
- Worker (VM.Standard2.24) - 24 OCPUs 

[\[More on OCI Compute Shapes\]](https://docs.cloud.oracle.com/iaas/Content/Compute/References/computeshapes.htm?tocpath=Services%7CCompute%7C_____5#ComputeShapes) 

![](media/image9.png)

4) For HDFS, each worker host has block volumes attached. It is recommended, that 
- Minimum 700GB Volume size for maximum throughput
	- 25K IOPS per Volume
	- 320mb/s per Volume
- Minimum 4 Block Volumes per Worker
- Aggregate HDFS bandwidth is important for high density workloads
- Scale HDFS capacity with more Block Volumes rather than fewer of larger size

[More on OCI storage](https://cloud.oracle.com/en_US/storage)

![](media/image10.png)


## Practice 4: Log in to Cloudera Manager

1) Navigate to the **MENU** on the top left, and Click on **Compute** --> **Instances**. And click on utility instance **"CDH Utility 1" to get Public IP of the instance. 

![](media/image11.png)

In the details section, copy the Public IP of the Utility Instance

![](media/image12.png)

2) Go the following URL and replace it with the Public IP of your Utility instance, and log in! 

		http://<utility_public_ip>:7180/cmf/home
		
		Username: cdhadmin
		Password: somepassword

![](media/image13.png)

3) Start the cluster (if it is not already started). *Only one person per team needs to perform this step*. 
Cluster startup takes ~3m.  Proceed to SSH login while this runs.

![](media/image14.png)

4) Open GitBash in your Windows and SSH to the utility Node. 
Download an ssh private key from [here](https://bit.ly/2NDfXc4) and use this id_rsa key to log in to the Utility node. Make sure to provide the complete path for the key below.  

```
$ ssh -i id-rsa opc@<Utility_Node_PublicIP> 
```
**NOTE**
```
Any persmission issues, 
$chmod 0600 id_rsa 
Try log in again. 
```

5) Confirm is the cluster is started in the Cloud Manager Console. 

![](media/image15.png)

## Practice 5: Benchmarking with Teragen/Terasort

1) Open GitBash in your Windows and SSH to the utility Node. 
```
$ ssh -i id-rsa opc@<Utility_Node_PublicIP> 
```

2) Submit a 100GB Teragen. 
**NOTE**: **<UNIQUE_ID>** is above should be a unique identifier for each participant sharing the cluster.  This will be used in next steps, so ensure this is captured.  A single digit number will suffice

	The count above is 1,000,000,000 for 100GB.  We are running a small scale test due to time constraints.
```
$ hadoop jar hadoop-mapreduce-examples.jar teragen -Ddfs.replication=1 1000000000 /user/opc/<UNIQUE_ID>/terasort_in/
```

![](media/image16.png)

While the job is running, Navigate to **yarn** --> **Resource Pools** 

![](media/image17.png)

![](media/image18.png)

The job is running slow as we can see only 14GiB memory is allocated to the job with 4 vcores, because we didn’t specify we wanted more resources. Lets kill the running job first and specify more resources for this job. 

```
Ctrl+c to break out the output. 
$ mapred job -list
```
Copy the jobID and kill this job 
![](media/image19.png)
```
$ mapred job -kill <jobID>
$ hdfs dfs -rm -r -skipTrash /user/opc/<UNIQUE_ID>/
```
![](media/image20.png)


3)  Submit a 100GB Teragen again but this request more mappers which will result in more cluster resources used for faster job execution.  Lets see how fast this job executes! 

```
$ hadoop jar hadoop-mapreduce-examples.jar teragen -Ddfs.replication=1 -Dmapreduce.job.maps=120 1000000000 /user/opc/<UNIQUE_ID>/terasort_in/
```
![](media/image21.png)

![](media/image22.png)

4) Let’s run a Terasort from our 100GB Teragen and observe what happens, trigger the job using “time” so we get a runtime without having to look at job logs to calculate it.

```
$ time hadoop jar hadoop-mapreduce-examples.jar terasort -Ddfs.replication=1 -Dmapreduce.job.maps=120 -Dmapreduce.job.reduces=120 /user/opc/<UNIQUE_ID>/terasort_in/ /user/opc/<UNIQUE_ID>/terasort_out/
```
![](media/image23.png)

![](media/image26.png)

Here we can observe the memory and containers allocated. 

![](media/image24.png)

This represents the total cluster resources available to YARN. These clusters are pre-tuned for VCORE & Memory allocation.

![](media/image25.png)

Navigate back to the Home page on Cloudera Manager, observe the utilization graph while the workload is running. Teragen is a WRITE process, notice the Green graph to the far left on HDFS IO, followed by Terasort which is a READ (blue) followed by WRITE.

![](media/image27.png)

5) Let’s check the pre-tuned settings for Map-Reduce on this cluster.
- Select YARN > Configuration
- In the Search Box type **“mapreduce java”** and hit enter
	- Look at the Map and Reduce Java Opts Base

![](media/image28.png)

How did we arrive at these Xmx/Xms numbers?

- In the Search Box type **“cpu-vcores”** and hit enter

![](media/image29.png)

How did we get 80Vcores when we only have 24 OCPU (48 cores) available per worker? 

- In the Search Box type **“resource.memory-mb”** and hit enter

![](media/image30.png)

While memory in the cluster is static, YARN can be customized with a vcores value which doesn’t map exactly to the physical core ratio.  This tuning directly affects performance.

More on YARN tunning : https://www.cloudera.com/documentation/enterprise/5-15-x/topics/cdh_ig_yarn_tuning.html

Let’s try adjusting this from 1.6x to 2.5x. In the search box type **“cpu-vcores”** and adjust the Container Virtual CPU Cores from 80 to 120.

![](media/image31.png)

Since we adjusted the Vcores in YARN - we should set the Java Xmx/Xms Defaults as well.  Let’s calculate using this formula:
```
We’re using the memory per node divided by our vcore count

(yarn.nodemanager.resource.memory-mb)/vcore)
( 308224 / 120) = 2568
```

Let’s set JMX/JMS using this number.

- In the Search Box type **“mapreduce java”** and hit enter
	- Look at the Map and Reduce Java Opts Base

![](media/image32.png)

Save the changes! 
![](media/image33.png)

And Click on Restart Stale Services button at the top, which presents us with the changes! 

![](media/image34.png)

Confirm that we want to re-deploy client configuration and click the blue button at bottom right.  This will restart affected YARN services. 

![](media/image36.png)

![](media/image37.png)

Once completed, Click Finish. 

![](media/image38.png)

6) Let’s test our new configuration by submitting a Terasort again, adjusting the output path.

```
$ time hadoop jar hadoop-mapreduce-examples.jar terasort -Ddfs.replication=1 -Dmapreduce.job.maps=120 -Dmapreduce.job.reduces=120 /user/opc/<UNIQUE_ID>/terasort_in/ /user/opc/<UNIQUE_ID>/terasort_out_2/
```
Observe the differences in this second terasort run. 

While this is running, go back and browse previous views of Cloudera Manager to observe resource utilization with the new tuning.

![](media/image41.png)

Notice the resources consumption in YARN. 

![](media/image39.png)

![](media/image40.png)

Time taken to complete the second terasort. 

![](media/image42.png)

7) Now let’s revert back to the original cluster configuration.  Select YARN > Configuration > History and Rollback.

![](media/image43.png)

Click on Details next to each configuration change that was made, and choose Revert Configuration changes.

![](media/image44.png)

![](media/image45.png)

And Click on Restart Stale Services button at the top, which presents us with the changes! 

![](media/image46.png)

![](media/image47.png)

Confirm that we want to re-deploy client configuration and click the blue button at bottom right.  This will restart affected YARN services. 

![](media/image48.png)

8) If time permits, scale the teragen to 1TB and we can watch Cloudera Manager during execution. *Note that a new path is used, we want to put the 1TB Teragen in a separate location.*
```
$ hadoop jar hadoop-mapreduce-examples.jar teragen -Ddfs.replication=1 -Dmapreduce.job.maps=120 10000000000 /user/opc/<UNIQUE_ID>/1T_terasort_in/
```
![](media/image49.png)

![](media/image50.png)

![](media/image51.png)

## Try Oracle Cloud Infrastructure for FREE 

Sign up for the free Oracle Cloud Infrastructure trial account. 
https://cloud.oracle.com/tryit

![](media/image52.png) 
