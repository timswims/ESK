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
Now that you have successfuly uploaded sample data you can now upload your own data. 

<details>
<summary><b>Deploying uploadMyLog</b></summary>

 To deploy the uploadMyLog file please follow the directions below.

1. ??Before uploading logs, enter property values to be used in uploading log in file `SCRIPT_HOME/DBLogTrial/uploadSample/config/upload.properties`.
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

2. Download and install uploadMyLog.zip file found in the DBLogTrial.zip package. 
   <pre>
   $ <b> cd ~ </b>
   $ <b> mkdir ./scratch </b>
   $ <b> cd scratch </b>
    $ <b> unzip uploadMyLog.zip </b>
   </pre>

After extracting the Zip file as above, you will see a directory named DBLogTrial with a subdirectory named uploadMyLog. This document refers to the uploadMyLog directory as SCRIPT_HOME

3. Now we need to make the executable script.
   <pre>
   $<b> cd ~/scratch/DBLogTrial </b>
   $<b> cd uploadMyLog/ </b> 
   $<b> chmod +x ./uploadMyLog.sh </b>
   $<b> chmod +x ./uploadMyLogTraditional.sh </b>
</pre>

#### Using uploadMyLog

To verify the status of the uploads, follow these steps:

1.  Log on to Oracle Management Cloud.
2.  Navigate to Log Analytics.
    1. From the Welcome to Oracle Management Cloud page, click the **navigation icon** ![alt text][hamburger] on the top-left corner to view the Management Cloud navigation pane if it is not already there. Select **Log Analytics**.
3.  Navigate to the **Log Admin** page and view status of the uploads.
    1. From the left navigation pane, select **Log Admin**.
    2. Select **Uploads**.
 1. 
    3. From the Uploads page, you should see the uploads that you performed earlier. If an upload shows 0 in Progress and 0 Failed, it has completed.
       1. If necessary, click an upload name to see the Status of the upload. For example, click `alertlog_<timestamp>`. If the upload has completed successfully, you will seen a green stick in the **Status** field.
   
   ![uploads-1] ![uploads-2] ![uploads-3]
</details>

## 5. Install a Gateway Agent


