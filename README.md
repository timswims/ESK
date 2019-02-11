[hamburger]: ./Tutorial_Images/general/hamburger.png
[agent-download]: ./Tutorial_Images/general/agent_download.png
[cluster-icon]: ./Tutorial_Images/general/cluster_icon.png
[gear]: ./Tutorial_Images/general/gear.png
[global-context]: ./Tutorial_Images/general/global_context.png

[uploads-1]: ./Tutorial_Images/log_analytics/uploads_1.png
[uploads-2]: ./Tutorial_Images/log_analytics/uploads_2.png
[uploads-3]: ./Tutorial_Images/log_analytics/uploads_3.png
[uploads-4]: ./Tutorial_Images/log_analytics/uploads_4.png
[clustering]: ./Tutorial_Images/log_analytics/clustering.gif
[drill-down]: ./Tutorial_Images/log_analytics/drill_down.gif
[correlating-logs]: ./Tutorial_Images/log_analytics/correlating_logs.gif
[page-view]: ./Tutorial_Images/general/page_view.png
[log-alerts]: ./Tutorial_Images/log_analytics/logalerts.gif



<h1> Analyzing Logs Using Oracle Log Analytics </h1>

**OMC Log Analytics**

An application that allows you to condense logs to find issues faster.

## Table of Contents

  - [Before You Begin](#before-you-begin)
      - [Background](#background)
      - [What Do You Need?](#what-do-you-need)
  - [1. Prepare Your Environment](#1-prepare-your-environment)
      - [Identify Your Oracle Management Cloud Instance](#identify-your-oracle-management-cloud-instance)
      - [Check for TLS 1.2 Support](#check-for-tls-12-support)
      - [Verify your host connectivity to Oracle Management Cloud](#verify-your-host-connectivity-to-oracle-management-cloud)
  - [2. Prepare to Upload Log Data](#2-prepare-to-upload-log-data)
  - [3. Upload Sample Logs and Evaluate](#3-upload-sample-logs-and-evaluate)
      - [Verifying the Status of the Uploads](#verifying-the-status-of-the-uploads)
      - [Checking the Outcome of the Cluster Operation](#checking-the-outcome-of-the-cluster-operation)
      - [Examining Potential Issues](#examining-potential-issues)
      - [Correlating Logs](#correlating-logs)
  - [4. Upload Your Own Logs and Evaluate](#4-upload-your-own-logs-and-evaluate)
    - [Deploying uploadMyLog](#deploying-uploadMyLog)
    - [Using uploadMyLog](#Using-uploadMyLog)
    - [Viewing Uploaded Log Records](#Viewing-Uploaded-Log-Records)
  - [5. Install a Gateway Agent](#5-install-a-gateway-agent)

## Before You Begin

This tutorial helps you get started with a kit for evaluating Oracle Log Analytics. This kit includes a set of sample log files and it guides you through uploading a set of your own log files. You can then explore Oracle Management Cloud Log Analytics functionality such as detecting anomalies in logs and correlating logs data. The kit also guides you through the installation of a Gateway Agent to help facilitate on-going data uploads.

#### Background

Oracle Log Analytics monitors, aggregates, indexes, and analyzes all log data from your applications and infrastructure enabling you to correlate this data and find problems faster. Using sample logs data, you can immediately explore the basic functionalities of Oracle Log Analytics. You can then collect logs from your own environment and explore the data in more depth. Sample log files and other scripts are provided in the form of a zip file.

#### What Do You Need?

- An Oracle Cloud account: if you do not already have one, sign up for a free triall [here](https://cloud.oracle.com/home). For more information, see [How Do I Access Oracle Management Cloud?](https://docs.oracle.com/en/cloud/paas/management-cloud/omcgs/access-oracle-management-cloud.html#GUID-838A6676-0224-4CF1-8BC8-8327887D24B7)
- An Oracle Management Cloud instance: If you don't already have an Oracle Management Cloud instance, create one. For more information, see [Creating an Oracle Management Cloud Instance](https://docs.oracle.com/en/cloud/paas/management-cloud/omcgs/access-oracle-management-cloud.html#GUID-C15E9F94-78CD-4868-A9F8-DCF50D267A2E).
- Appropriate license: Configure your license edition Oracle Management Cloud – Log Analytics Edition. For more information, see [How Do I License Oracle Management Cloud?](https://docs.oracle.com/en/cloud/paas/management-cloud/omcgs/oracle-management-cloud-license-information.html#GUID-A2F5635B-D7DE-4277-B707-035A96F19B26)
- A Unix host with connectivity to your Oracle Management Cloud instance and with a version of cURL that supports TLS 1.2 protocol.
- For the installation of the Gateway Agent, the following Operating Systems are supported: Oracle Linux 5.x, 6.x, and 7.x (64-bit), Red Hat Enterprise Linux 5.x, 6.x, and 7.x (64-bit), SUSE Linux Enteprise Server 11 (x86_64), AIX 6.1 or later (minimum level 6100-07), AIX 7.1 or later (minimum level 7100-03), Solaris 10.x and 11.x, Microsoft Windows versions 2008 Enterprise R2 (Intel 64-bit, Developer) or later, Microsoft Windows Server 2012 Standard (Intel 64-bit), Microsoft Windows 2012 Standard R2 (Intel 64-bit, Developer) or later.

## 1. Prepare Your Environment

#### Identify Your Oracle Management Cloud Instance

To meet the pre-requisites, you must have signed up for an Oracle Cloud account and created an Oracle Management Cloud instance.

Sign in to Oracle Cloud as a user with the OMC Administrator role. Your Oracle Management Cloud instance tile should be displayed on the My Services dashboard.

1. On your Management Cloud tile, click the **Action Menu** ![hamburger] then click **Open Service Console**. You are now viewing the Oracle Management Cloud instances page.
2. On the instance you want to access, click the **Manage this instance** menu and select **OMC URL**. The Oracle Management Cloud console home page is displayed.
3. Navigate to **Administration** > **Agents** > **Download** and select the Gateway Agent agent type from drop-down list. Make a note of your `TENANT_NAME` and `OMC_URL` values for your environment.

![agent-download]

#### Check for TLS 1.2 Support

Ensure your cURL version supports the TLS 1.2 protocol by running the following command from a terminal session on your Unix host:

<pre>
$ <b>curl --help | grep -i tlsv1.2</b>
--tlsv1.2       Use TLSv1.2 (SSL)
</pre>

Note the output of this command confirms that the version of the TLS protocol is TLSv1.2 (SSL).
If your compute does not support TLSv1.2 you may need to update `yum`.

#### Verify your host connectivity to Oracle Management Cloud

Your Unix host must have HTTPS access to Oracle Management Cloud. To check this, run the following commands from your Unix host:

1. If you are using a proxy server, set a pointer to it. If you are not using a proxy server, skip this step.
   <pre>
    $ <b>export HTTPS_PROXY=www-proxy.example.com:80</b>
   </pre>
2. The `OMC_URL` parameter value you noted for your environment represents the URL where the sample logs will be uploaded. Check the connectivity to your specific URL. The format of the command and sample output is shown below:

   <pre>
   $ <b>curl -I --tlsv1.2  Your OMC_URL</b>
   HTTP/1.0 200 Connection established
   HTTP/1.1 200 OK
   Date: Fri, 12 Jan 2018 00:56:42 GMT
   Server: Oracle-Application-Server-11g X-Frame-Options: SAMEORIGIN
   Last-Modified: Wed, 07 Dec 2017 23:27:01 GMT 
   ETag: "2b14-5267f6d5bfb40"
   Accept-Ranges: bytes
   Content-Length: 11028
   Vary: Accept-Encoding
   Cache-Control: no-cache,no-store
   </pre>

    The command output indicates that the connection was established.

## 2. Prepare to Upload Log Data

1. Download the sample data zip file [here](https://apexapps.oracle.com/pls/apex/f?p=44785:112:0::::P112_CONTENT_ID:23996).
2. On your host, make a directory called `SCRIPT_HOME` and save the `DBLogTrial.zip` file to it. Move into the `SCRIPT_HOME`diretory and unzip the file.

    <pre>
    $ <b>cd SCRIPT_HOME</b>
    $ <b>unzip DBLogTrial.zip</b>
    </pre>

Some of the files part of this set are:

- PDF files with detailed instructions for uploading and evaluating sample data and your own data, as well as instructions for installing a Gateway agent
- Alertlog.zip (Sample database alert logs)
- Messages.zip (Sample Linux syslog files)
- <span>uploadSample.sh</span> (a shell script for uploading the sample logs)
- <span>uploadMyLog.sh</span> (a shell script for uploading your own logs)

The next steps will guide you through these files.

## 3. Upload Sample Logs and Evaluate

You are now ready to upload some sample logs to Log Analytics and explore its functionality.

<details>
<summary><b>Uploading Sample Logs to Log Analytics</b></summary>

To upload the provided sample logs, follow these steps:

1. Before uploading logs, enter property values to be used in uploading log in file `SCRIPT_HOME/DBLogTrial/uploadSample/config/upload.properties`.
   - Go to the `SCRIPT_HOME/DBLogTrial/uploadSample/config` directory.
   - Use an editor of your choice to edit file `upload.properties` to set appropriate values for the following properties:
   - `UPLOAD_ROOT`: your `OMC_URL`
   - `IDENTITY_DOMAIN`: your `TENANT_NAME`
   - `USERNAME`: your OMC username
   - (Optional) `HTTPS_PROXY`


    **Mandatory Properties**
    <pre>
    # URL for uploading data to OMC
    # Examples:
    # UPLOAD_ROOT=https://inst1-acme.itom.management.us2.oraclecloud.com
    # UPLOAD_ROOT=https://inst2-xyz.itom.management.europe.oraclecloud.com
    # UPLOAD_ROOT=https://a123456.itom.management.us2.oraclecloud.com
    # This is a required parameter. The "https://" part is optional.
    UPLOAD_ROOT= <br/>
    # Subscription Identity Domain
    # EX:
    # IDENTITY_DOMAIN=acme
    # This is a required parameter
    IDENTITY_DOMAIN= <br/>
    # OMC user name
    # EX:
    # USERNAME=john.doe@xyz.com
    # This is a required parameter
    USERNAME=
    </pre>

    **Optional Property**
    <pre>
    # If you need to access OMC (Oracle Management Cloud) through a proxy server,
    # set "HTTPS_PROXY=proxy_host:port
    # E.g., HTTPS_PROXY=www-proxy.xyz.com:80
    HTTPS_PROXY=
    </pre>

2. Go to the `SCRIPT_HOME/DBLogTrial/uploadSample` directory, and run the <span>uploadSample.sh</span> script to upload the sample alert logs and syslog, respectively, as shown below. Enter your OMC password when prompted.
   <pre>
   $ <b> cd .. </b>
   $ <b> ./uploadSample.sh alertlog </b>
   $ <b> ./uploadSample.sh syslog </b>
   </pre>

Take note of the name of the upload at the bottom of each script output. An upload is identified by its name in Log Analytics UI.

Ex:
<pre>   
Upload name: alertlog.2018-01-07_19:43:25
Upload name: syslog.2018-01-07_19:43:32
</pre>

#### Verifying the Status of the Uploads

To verify the status of the uploads, follow these steps:

1.  Log on to Oracle Management Cloud.
2.  Navigate to Log Analytics.
    1. From the Welcome to Oracle Management Cloud page, click the **navigation icon** ![alt text][hamburger] on the top-left corner to view the Management Cloud navigation pane if it is not already there. Select **Log Analytics**.
3.  Navigate to the **Log Admin** page and view status of the uploads.
    1. From the left navigation pane, select **Log Admin**.
    2. Select **Uploads**.
 3. 
    4. From the Uploads page, you should see the uploads that you performed earlier. If an upload shows 0 in Progress and 0 Failed, it has completed.
       1. If necessary, click an upload name to see the Status of the upload. For example, click `alertlog_<timestamp>`. If the upload has completed successfully, you will seen a green stick in the **Status** field.
   
   ![uploads-1] ![uploads-2] ![uploads-3]
</details>



<details>
<summary><b>Viewing Uploaded Log Records</b></summary>
To view the records from an upload, follow these steps:

1. Navigate to the **Uploads** page.
2. From the **Uploads** page, select an upload, click the menu icon ![alt text][hamburger] on the right and click **View in Log Explorer** to view the records from that upload.
   ![uploads-4]
3. From the log explorer page, you can view the alert log records from the upload that you selected.



Some of the information shown on the page includes:

- The uploaded alert log entries are for the period from August 9 to August 24, 2017.
- The log entries came from the upload whose name is in the Query bar.
- The histogram shows the daily volumes of log records. This helps identify any abnormality in record volumes at a glance. You can drill down by clicking a bar on the chart.
- The first 25 of the 1920 records that came with the upload. The records are in date order from newest to oldest. You can reverse the order by clicking the arrowhead in the Time (`<time zone>`) field.
- You can browse the rest of log records by using the pagination at the bottom of the page.
</details>

<details>
<summary><b>Detecting Anomalies with the Cluster Command</b></summary>

To detect anomalies based on log records, you can use Log Analytics cluster command, which automatically groups log records based on severity, such as error, fault, fatal and warning, and dynamically identified patterns, potential issues, outliers, and trends.
 - To perform clustering on the log records, from the **Visualize** panel click the currently selected visualization (e.g. **Records with Histogram**), and click **Cluster** ![cluster-icon] icon.

![clustering]

#### Checking the Outcome of the Cluster Operation
The cluster operation reduced 1920 log records to 123 clusters, identified 25 potential issues, 37 outliers, and 26 trends.

Examine the log clusters, and then click **Potential Issues**.

#### Examining Potential Issues

From the **Potential Issues** tab, you can look at the log clusters that Log Analytics identifies as potential issues, if you see a cluster with a sample message that may be pointing to an issue of significance or of interest, click the value in the **Count** column to drill down see the records of the cluster.

For example, the following sample message indicates that the Oracle database instance had problems writing to a control file due to a file I/O error. This kind of problem is critical; it tends to result in an abnormal shutdown of the instance.

![drill-down]

Let’s drill down to the log record by clicking the count value of 1 on the left of the sample message.

Drilling down on a log cluster allows you to see the log record(s) including the original log entry (or entries) in that cluster. In this case, you will see the record with the timestamp of Aug 9, 2017, 5:23:58PM (UTC-8:00 or PST) showing a file I/O error affecting the writing to a database control file.

#### Correlating Logs

Log Analytics allows you to quickly correlate logs from different sources (e.g. database logs and syslog) based on time to determine whether there is a correlation between events captured in log records. Let’s query the log records for entities demo_db_instance and demo_host 30 seconds before 5:23:28 PM (UTC-8:00) and 30 seconds after that by following these steps:

1. Click ![gear] at the bottom of the **Original Log Content** field, and then select **Advanced Log Fitler Options...**.
2. From the Advanced Log Filter Options pop-up window, enter 30 (seconds) for **Time Range - Before**, 30 (seconds) for **Time Range – After**, and click **OK**.
3. In the **Global Context** ![global-context] bar near the top, enter `demo_host` next to `demo_db_instance`, click in the Query bar to clear any existing filter, and click **Run**.
4. The above query retrieves the log records uploaded for entities `demo_db_instance` and `demo_host` for the period of 5:23:28 PM to 5:24:28 PM on August 9. Examine the 31 records in the two-page output to see the sequence of the events that were captured in the logs in the one-minute period, and which of the events may have had an effect on other events.

![correlating-logs]

You may have noticed that at 5:23:58PM, system logs (syslog) recorded that some I/O errors occurred on disk device sdd1 (see page 2), and database alert logs recorded that the database encountered I/O errors (see page 1); then at 5:24:00PM the database was terminated.
</details>

## 4. Upload Your Own Logs and Evaluate
Now that you were able to upload some sample logs lets look at what your own logs look like on your enviornment.

<details>
<summary><b>Deploying uploadMyLog</b></summary>

 To deploy the uploadMyLog file please follow the directions below.

1. Download and install uploadMyLog.zip file found in the DBLogTrial.zip package. 
   <pre>
   $ <b> cd ~ </b>
   $ <b> mkdir ./scratch </b>
   $ <b> cd scratch </b>
   $ <b> unzip uploadMyLog.zip </b> 
   </pre>

After extracting the Zip file as above, you will see a directory named DBLogTrial with a subdirectory named uploadMyLog. This document refers to the uploadMyLog directory as SCRIPT_HOME

2. Now we need to make the executable script.
   <pre>
   $<b> cd ~/scratch/DBLogTrial </b>
   $<b> cd uploadMyLog/ </b> 
   $<b> chmod +x ./uploadMyLog.sh </b>
   $<b> chmod +x ./uploadMyLogTraditional.sh </b>
</pre>

</details>
<details>
<summary><b>Using uploadMyLog</b></summary>

This section provides the steps for using the uploadMyLog package to upload sample logs to explore Log Analytics features.

1. Before uploading your logs, enter property values to be used in uploading the log in file `SCRIPT_HOME/DBLogTrial/uploadMyLog/config/upload.properties`.
   - Go to the `SCRIPT_HOME/DBLogTrial/uploadMyLog/config` directory.
   - Use an editor of your choice to edit file `upload.properties` to set appropriate values for the following properties:
   - `UPLOAD_ROOT`: your `OMC_URL`
   - `IDENTITY_DOMAIN`: your `TENANT_NAME`
   - `USERNAME`: your OMC username
   - (Optional) `HTTPS_PROXY`


    **Mandatory Properties**
    <pre>
    # URL for uploading data to OMC
    # Examples:
    # UPLOAD_ROOT=https://inst1-acme.itom.management.us2.oraclecloud.com
    # UPLOAD_ROOT=https://inst2-xyz.itom.management.europe.oraclecloud.com
    # UPLOAD_ROOT=https://a123456.itom.management.us2.oraclecloud.com
    # This is a required parameter. The "https://" part is optional.
    UPLOAD_ROOT= <br/>
    # Subscription Identity Domain
    # EX:
    # IDENTITY_DOMAIN=acme
    # This is a required parameter
    IDENTITY_DOMAIN= <br/>
    # OMC user name
    # EX:
    # USERNAME=john.doe@xyz.com
    # This is a required parameter
    USERNAME=
    </pre>

    **Optional Property**
    <pre>
    # If you need to access OMC (Oracle Management Cloud) through a proxy server,
    # set "HTTPS_PROXY=proxy_host:port
    # E.g., HTTPS_PROXY=www-proxy.xyz.com:80
    HTTPS_PROXY=
    </pre>
2. To upload your own Oracle Database alertlog, take the log and zip it into an alertlog.zip file. Move the alertlog.zip file into <SCRIPT_HOME>/logs. Please ensure you name the file exactly alertlog.zip, as the uploader will be looking for a file of that name.
   <pre>
   $ <b>zip alertlog.zip ./<your alertlog filename></b>
   </pre> 
3. To upload your own system logfile (typically the file /var/log/messages), take the log and zip it into a messages.zip file. Move messages.zip file into <SCRIPT_HOME>/logs. Please ensure you name the file exactly messages.zip, as the uploader will be looking for a file of that name.
    <pre>
   $ <b>zip messages.zip ./messages></b>
   </pre> 
4. Go to the SCRIPT_HOME directory, and run the uploadMyLog.sh script to upload the sample alert logs and syslog, respectively, as shown below. Enter your OMC password when prompted.
   <pre>
   $ <b>./uploadMyLog.sh alertlog </b>
   $ <b>./uploadMyLog.sh syslog </b>
   </pre> 

   ##### Take note of the name of the upload at the bottom of each script output. An upload is identified by its name in Log Analytics UI.Examples of output lines containing upload names are:
   <pre>
    <b>Upload name: alertlog.2018-01-07_19:43:25</b>
    <b>Upload name: syslog.2018-01-07_19:43:32</b>
   </pre> 

To verify the status of the uploads, follow these steps:

1.  Log on to Oracle Management Cloud.
2.  Navigate to Log Analytics.
    1. From the Welcome to Oracle Management Cloud page, click the **navigation icon** ![alt text][hamburger] on the top-left corner to view the Management Cloud navigation pane if it is not already there. Select **Log Analytics**.
3.  Navigate to the **Log Admin** page and view status of the uploads.
    1. From the left navigation pane, select **Log Admin**.
    2. Select **Uploads**.
  
    3. From the Uploads page, you should see the uploads that you performed earlier. If an upload shows 0 in Progress and 0 Failed, it has completed.
       1. If necessary, click an upload name to see the Status of the upload. For example, click `alertlog_<timestamp>`. If the upload has completed successfully, you will seen a green stick in the **Status** field.
   
   ![uploads-1] ![uploads-2] ![uploads-3]
</details>

</details>
<details>
<summary><b>Viewing Uploaded Log Records</b></summary>

To view the records from an upload, follow these steps.
1. Navigate to the Uploads page. If necessary see, [Verifying the Status of the Uploads](#verifying-the-status-of-the-uploads)
2. From the Uploads page, click the **navigation icon** ![alt text][hamburger], and click **View in Log Explorer** to view the records from that upload. Let's perform the steps to view the alert log records in log explorer. 
3. From the Log Explorer page, you can view the alert log records from the upload that you selected.

![log-alerts]

   
Some of the information shown on the page includes

-  The period of the uploaded alert log entries.
-  The log entries came from the upload whose name is in the Query bar.
-  The histogram shows the daily volumes of log records. This helps identify any abnormality in record volumes at a glance. You can drill down by clicking a bar on the chart.
-  The first 25 of the records that came with the upload. The records are in date order from newest to oldest. You can reverse the order by clicking the arrowhead in the Time (<time zone>) field. You can browse the rest of log records by using the pagination at the bottom of the page.

<p align="center">
    <img src="./Tutorial_Images/general/page_view.png" />
</p>

##### Note - the log entries you see will vary depending on the record in the alertlog and messages logs that you upload. 

</details>

## 5. Install a Gateway Agent

<details>
<summary><b>Deploy Oracle Management Cloud Gateway</b></summary>

### Before You Begin:

### Background

Oracle Management Cloud (OMC) Gateway (highlighted in red in the diagram below) is an optional yet vital component of an Oracle Management Cloud deployment. It serves as a channel between Oracle Management Cloud agents and Oracle Management Cloud.

While it is possible for the OMC agent that resides on each host to communicate directly with OMC’s backend, for security reasons, an organization may want to limit the number of hosts that can connect to the Internet directly. In this case, it is best to set up a small number of OMC Gateway hosts, and enable Internet access only for those hosts.
For a trial, since the number of hosts may be small, it is possible to do a trial without the gateway. However, if there is a desire to limit security exposure even in a trial environment, then it is a good idea to set up the OMC Gateway.

--insert picture--

There are 5 steps for deploying OMC Gateway.

1. Download the Oracle Management Cloud Gateway Software
   
2. Create Registration Key
   
3. Edit the Response File
   
4. Install the Gateway
   
5. Verify Gateway Installation

What Do You Need?

* A valid Oracle Cloud account, an Oracle Management Cloud instance and "OMC Administrator" role credentials. See How do I Access Oracle Management Cloud? in Getting Started with Oracle Management Cloud. You should already have these if you followed a prior tutorial.

* A host: the OMC Gateway needs to be installed on a host where it will run. For production deployment, one or more dedicated physical or VM hosts is recommended. For trial, it is possible to use a host where the entities you want to monitor are installed. Linux, Windows, Solaris SPARC and AIX based hosts are supported. See the “Supported Operating Systems” section in Common Prerequisites.
  
* A staging location: an empty directory on your host where you download or copy the agent files.
  
* An installation directory: an empty directory on your host where the agent will be installed. Ensure the directory is created with the required permissions. See the "Permissions Required on the Agent Base Directory" section in Common Prerequisites.

### 1. Download the Oracle Management Cloud Gateway Software

The Oracle Management Cloud gateway software, including a gateway installation script and an editable response file, is provided in a zip file that you can download from your Oracle Management Cloud console. The zip file is platform specific.

#### Registering For a Cloud Account

If you don't have an Oracle Cloud Account, sign up for one using the Registering For a Cloud Account section below. If you already have a cloud account, then skip the Registering For a Cloud Account section section

1.	Go to the Oracle Cloud Infrastructure Page: https://cloud.oracle.com/en_US/cloud-infrastructure.

2.	At the top of the page click the try for free button. 
	--Insert Picture Here--

3.	Here is where you will put in the information for your trial account

4.	Fill out the information for the Account Details section (Section 1)

	* Account Type
  
	* Cloud Account Name
  
	* Default Data Region
  
	* Email Address
  
	* First Name
  
	* Last Name
  
	* Country/Territory
  
	* Address
  
	* City
  
	* State
  
	* Zip/Postal Code
  
--Insert Picture Here--

5.	For the Verification Code Section (Section 2), fill out the information for this as well.

	* Country/Region Calling Code
  
	* Mobile
  
--Insert Picture Here--

6.	Click on the Request Code button to receive the verification code via a text message to the mobile number that you provided when you filled out the Mobile Number Field.

7.	Once you receive the code, type that code into Verification code field and click the verify button. It may take a minute or so for the verification button to work.

8.	In the Credit Card Details Section (Section 3) click on the Add Payment Button and provide information from a credit card. Debit cards can be used as well. You will be asked to verify your address and provide the card information in separate windows

9.	In the Terms & Conditions section (Section 4), check the complete box and click the complete box.

10.	Following this completion your cloud account will start to be provisioned and will take a few minutes to be completed.

#### Access the Oracle Management Cloud Console

Sign in to Oracle Cloud as a user with the OMC Administrator role. Your Oracle Management Cloud instance tile should be displayed on the My Services dashboard.

--Add Picture Here--

1. Go to cloud.oracle.com and click Sign In.

--Insert Picture Here--

2. Your Sign In procedure varies depending on the type of account that your tenant is configured.

    In most cases, if your tenant is on “Cloud Account with Identity Cloud Service”, select “Cloud Account with Identity Cloud Service” as your account type, enter the name of your account, and click My Services.

    Enter your user id and password.

--Insert Picture Here--

    On the other hand, if you have a Traditional Cloud Account (most likely because it was provisioned prior to April 2018), select “Traditional Cloud Account” as account type. Select “US Commercial 2 (us2)” for data center if your account was provisioned in the United States.
  
  * Supply the name of your identity domain.
  
  * Enter your user id and password.

3. On the Oracle Cloud Dashboard, click the menu link next to the Oracle logo (highlighted in red) toward the top of the page to open up the navigation menu to the left.

--Insert Picture Here--

4. On the navigation menu, click on Services to expand the list of services, and click Management Cloud.

--Insert Picture Here--
  
5. This takes you to the Oracle Management Cloud home page, which looks like the following. You will be using the **Global Navigation Menu** to the left to carry out the remaining setup. If the menu is hidden, click the link next to Oracle logo (highlighted in red) to bring up the menu.

--Insert Picture Here--

#### Save and Extract the Gateway Files

1. On the Oracle Management Cloud home page, click the **Global Navigation Menu** on the top-left corner and navigate to **Administration > Agents.**
   
2. On the **Agents – Oracle Management Cloud** page, click the **Download** tab. The Agent Software Download page is displayed.
   
3. Select Gateway from the **Agent Type** drop-down list, and select one of the choices (such as Linux (64-bit)) that matches the type of O/S on the host where you will be installing the gateway from the **Operating System** drop-down list. The gateway software link for the gateway you’ve selected is displayed.
   
4. A list of link would show up under Download. Click the link on the gateway file that you wish to download.
   
5. If you download the Gateway file to your PC instead of the host that you plan to run the Gateway, move the downloaded file to your Gateway host and unzip the file into a staging directory of your choice. To do this for linux, use the following steps:
	
	*	From Local Machine Terminal Session - SSH into your OCI Instance by running the      below command (You will use your OCI Public IP Address instead of 129.***.***.**):
		
        **`ssh opc@129.***.***.**`**

	*	From OPC Terminal Session type pwd to see where you are currently at in your         directory.  You should see the following:

         **`/home/opc`**

	*	From OPC Terminal Session  Make a directory called agent.  Run the following command in your terminal:
		
        **`mkdir agent`**

	*	From OPC Terminal Session Make a directory called omc.  Run the following command in your terminal:
		
        **`mkdir omc`**

	*	From OPC Terminal Session type the following command:
		
        **`exit`**

	*	From Local Machine Session we will now copy our cloud agent file over to our OPC Session:
  
		a.	Locate the file path of the 'gateway_linux.x64_1.32.0.zip
        

		b.	run the following command (File path will depend on where you saved the agent and IP address will be different)

        **`ssh opc@scp Downloads/gateway_linux.x64_1.32.0.zip opc@129.***.***.**:/home/opc/omc`**

	*	From Local Machine Session ssh back into your OCI    account (Your IP Address will be different than shown below:

        **`ssh opc@129.***.***.**`**
         

	*	From your OPC terminal inside the OMC directory session type the following command to see the contents of your directory.  You should now see a .zip called gateway_linux.x64_1.32.0.zip:
		
         **`ls`**

	*	From your OPC terminal inside the omc directory unzip the cloudagent file by running the below command
		
         **`unzip gateway_linux.x64_1.32.0.zip`**

6. Please also make a note of the values of TENANT_ID and UPLOAD_ROOT, which are shown as the bottom of the page. You will need these information for a later step when you set up the agent.rsp file.
   
### 2. Create a Registration Key

A registration key is issued for your identity domain, and it’s used when you deploy gateways and agents.

1. On the Oracle Management Cloud home page, click the **Global Navigation Menu** on the top-left
corner and navigate to **Administration > Agents.**

2. On the **Agents – Oracle Management Cloud** page, click the **Registration Keys** tab. The Registration Keys page opens.
   
3. Enter the required details to create a new Registration Key:
   
	* In the **Name** field, specify a name to identify the registration key.
 
	* In the **Registration Limit** field, enter a number that indicates the maximum number of gateways, data collectors, and agents that can be associated with the registration key. If you are not sure, just put 10,000, which should be enough for a trial.
  
	* Click **Create New Key.** A new registration key will then be created. Make a note of this registration “Key Value”, it will be used by the AgentInstall.sh script at the time of installation.

### 3. Edit the Response File

The *AgentInstall.sh* script is used to carry out the actual install of the OMC Gateway. The script requires a set of parameters that are specific to your environment. These parameters are specified in the response file *agent.rsp.*

1. On your Linux / UNIX or Microsoft Windows host, logged in as the owner of the Oracle software (here, oracle) navigate to your staging directory. Edit the *agent.rsp* file using any standard text editor. Add your values for the **mandatory parameters** in the *agent.rsp* file. Here are example values (make sure to replace these with the correct values for your environment):
   
	* *TENANT_ID=example-tenant*
  
	Note: this value must be exactly as shown in the UI, the format is instance name-domain name

	* *UPLOAD_ROOT=https://example-tenant.management.us2.oraclecloud.com/*  
	* *AGENT_REGISTRATION_KEY=xxxxxxx-yyyyyyyyyyyyyyyyy*
  
	* *AGENT_BASE_DIRECTORY=/oracle/GatewayAgent*
  
	(or for example when installing on Microsoft Windows)

	*AGENT_BASE_DIRECTORY=C:\Oracle\GatewayAgent*

	The AGENT_BASE_DIRECTORY is where your agent executables and other run-time files will reside. Create a directory that is owned by the user id of the Oracle software (here, *oracle*). It is best to have a standard location that is common across all your hosts, so that the agent.rsp file is standardized and you just need to set up the file once.

2. For security reasons, if you are using a **proxy server** (a dedicated host or system that acts as an intermediary between your host and Oracle Management Cloud) you must define its parameters next. In this case, the proxy server was configured to use authentication, so it requires a user name and password. Edit the following parameters in the *agent.rsp* file:
   
	* *OMC_PROXYHOST=www-proxy.example.com* 
  
	* *OMC_PROXYPORT=80*
  
	* *OMC_PROXYUSER=oracle*
  
Save and close the response file.

### 4a. Install the Gateway on Linux / UNIX
You install a gateway by running the AgentInstall.sh script from the command line. By default, the gateway install script picks up all its required parameters from the response file you just edited in the same directory.

1. On your Linux / UNIX host, logged in as the owner of the Oracle software (here, oracle) navigate to your staging directory. Run the installer script using this command:
   
    **`./AgentInstall.sh`** 
 
 --insert image here--
 
2. To enable the gateway to start automatically when the host is booted up, add a startup script. Switch to the root user.

    **`$ su - root`**
    **`password`**
    **`#`**
   
3. Create a shell script named startomcagent.sh under the /etc/init.d directory using any standard text editor with the following:


    **`#!/bin/sh`**
    **`su - <Agent_Install_User> -c`**
    **`<Agent_Base_Directory>/agent_inst/bin/omcli start agent`**
 
    For example, if the gateway is installed under the /oracle/omc directory and gateway installation owner is oracle, then the content of the shell script should be as follows:
 
    *#!/bin/sh*
    *su - “oracle” -c “/oracle/omc/agent_inst/bin/omcli start agent”*

4. Save the script file as startomcagent.sh.
   
5. Change the permission of the file to 755. Ensure that the owner of the script file and all the other files in the */etc/init.d* directory is *root.*
   
6. For Linux, create symbolic links under 
    */etc/rc.d/rc2.d, /etc/rc.d/rc3.d, /etc/rc.d/rc5.d,*
    and */etc/rc.d/rc6.d*
   directories to make the newly created shell script file accessible in the host startup process. Prefix the symbolic link with S and the priority level. For example, to create the symbolic links with priority level 81, run the following commands:

   *ln -s /etc/init.d/startomcagent.sh /etc/rc2.d/S81startomcagent.sh*

   *ln -s /etc/init.d/startomcagent.sh /etc/rc3.d/S81startomcagent.sh* 

   *ln -s /etc/init.d/startomcagent.sh /etc/rc5.d/S81startomcagent.sh*

   *ln -s /etc/init.d/startomcagent.sh /etc/rc6.d/S81startomcagent.sh*
   
7. For Solaris, create symbolic links under /etc/rc.d/rc2.d, /etc/rc.d/rc3.d directories c. Prefix the symbolic link with S and the priority level. To create the symbolic links with priority level 81, run the following commands:

    *ln -s /etc/init.d/startomcagent.sh /etc/rc2.d/S81startomcagent.sh* 

    *ln -s /etc/init.d/startomcagent.sh /etc/rc3.d/S81startomcagent.sh*
   
8.  For AIX, create symbolic links under
*/etc/rc.d/rc2.d, /etc/rc.d/rc3.d,* and */etc/rc.d/rc5.d*
directories with priority level 81. Prefix the symbolic link with S and the priority level. To create the symbolic links with priority level 81, run the following commands:

    *ln -s /etc/init.d/startomcagent.sh /etc/rc2.d/S81startomcagent.sh* 

    *ln -s /etc/init.d/startomcagent.sh /etc/rc3.d/S81startomcagent.sh* 

    *ln -s /etc/init.d/startomcagent.sh /etc/rc5.d/S81startomcagent.sh*

### 4b. Install the Gateway on Windows

You install a gateway by running the AgentInstall.bat script from the command line. By default, the gateway install script picks up all its required parameters from the response file you just edited in the same directory.

1. On Windows hosts, launch the command line interface as administrator.
   
2. Go to the directory where you unzip the package for gateway software, and run AgentInstall.bat. Make sure you run it in a command line window as administrator. The script will install Gateway
software and set it to start automatically as a Windows service.

--insert image here--

### 5. Verify the Gateway Installation
After installing the gateway, you must verify the installation.

1. From the Oracle Management Cloud home page, click the **Global Navigation Menu** on the top-left corner and navigate to **Administration > Agents.**

2. Click the **Gateways** tab.
   
3. Check if the host name of your deployed gateway exists in the list of available gateways. You can click the gateway entry and match the specified registration key value with the registration key that you had used when deploying the gateway.

--insert image here--
   
 4. On your Linux / UNIX host, run the following *omcli* commands to verify that the gateway was successfully deployed:

    *<AGENT_BASE_DIRECTORY>/agent_inst/bin/omcli status agent* - Run this command to display a list of properties for the newly installed gateway. Check if the last successful upload and last attempted upload values (date and time) are the same.

    *<AGENT_BASE_DIRECTORY>/agent_inst/bin/omcli status agent connectivity* - Run this command to verify that there are no significant connectivity issues with connections associated with the gateway and Oracle Management Cloud.

5. If installing on a Microsoft Windows host, run the following *omcli* commands to verify that the gateway was successfully deployed:
   
    *<AGENT_BASE_DIRECTORY>\agent_inst\bin\omcli.bat status agent* - Run this command to display a list of properties for the newly installed gateway. Check if the last successful upload and last attempted upload values (date and time) are the same.
  
    *<AGENT_BASE_DIRECTORY>\agent_inst\bin\omcli.bat status agent connectivity* - Run this command to verify that there are no significant connectivity issues with connections associated with the gateway and Oracle Management Cloud.

The following screenshot shows one of the verification steps as it would appear on a Linux host:

--insert image here--

You are now ready to perform additional installation of agents and the optional Enterprise Manager Data Collector for Oracle Management Cloud.

### Want to Learn More?
* Getting Started with Oracle Management Cloud
* Installing a Gateway
</details>

