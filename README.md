[hamburger]: ./Tutorial_Images/hamburger.png
[agent-download]: ./Tutorial_Images/agent_download.png


# <center>Analyzing Logs Using Oracle Log Analytics</center>
**<center>OMC Log Analytics</center>**
<center>An application that allows you to condense logs to find issues faster.</center>


## Table of Contents
- [<center>Analyzing Logs Using Oracle Log Analytics</center>](#centeranalyzing-logs-using-oracle-log-analyticscenter)
  - [Table of Contents](#table-of-contents)
  - [Before You Begin](#before-you-begin)
      - [Background](#background)
      - [What Do You Need?](#what-do-you-need)
  - [1. Prepare Your Environment](#1-prepare-your-environment)
      - [Identify Your Oracle Management Cloud Instance](#identify-your-oracle-management-cloud-instance)
      - [Check for TLS 1.2 Support](#check-for-tls-12-support)
      - [Verify your host connectivity to Oracle Management Cloud](#verify-your-host-connectivity-to-oracle-management-cloud)
  - [2. Prepare to Upload Log Data](#2-prepare-to-upload-log-data)
  - [3. Upload Sample Logs and Evaluate](#3-upload-sample-logs-and-evaluate)
      - [Uploading Sample Logs to Log Analytics](#uploading-sample-logs-to-log-analytics)

## Before You Begin

This tutorial helps you get started with a kit for evaluating Oracle Log Analytics. This kit includes a set of sample log files and it guides you through uploading a set of your own log files. You can then explore Oracle Management Cloud Log Analytics functionality such as detecting anomalies in logs and correlating logs data. The kit also guides you through the installation of a Gateway Agent to help facilitate on-going data uploads.

#### Background
Oracle Log Analytics monitors, aggregates, indexes, and analyzes all log data from your applications and infrastructure enabling you to correlate this data and find problems faster. Using sample logs data, you can immediately explore the basic functionalities of Oracle Log Analytics. You can then collect logs from your own environment and explore the data in more depth. Sample log files and other scripts are provided in the form of a zip file.

#### What Do You Need?
* An Oracle Cloud account: if you do not already have one, sign up for a free trail [here](https://cloud.oracle.com/home). For more information, see [How Do I Access Oracle Management Cloud?](https://docs.oracle.com/en/cloud/paas/management-cloud/omcgs/access-oracle-management-cloud.html#GUID-838A6676-0224-4CF1-8BC8-8327887D24B7)
* An Oracle Management Cloud instance: If you don't already have an Oracle Management Cloud instance, create one. For more information, see [Creating an Oracle Management Cloud Instance](https://docs.oracle.com/en/cloud/paas/management-cloud/omcgs/access-oracle-management-cloud.html#GUID-C15E9F94-78CD-4868-A9F8-DCF50D267A2E).
* Appropriate license: Configure your license edition Oracle Management Cloud – Log Analytics Edition. For more information, see [How Do I License Oracle Management Cloud?](https://docs.oracle.com/en/cloud/paas/management-cloud/omcgs/oracle-management-cloud-license-information.html#GUID-A2F5635B-D7DE-4277-B707-035A96F19B26)
* A Unix host with connectivity to your Oracle Management Cloud instance and with a version of cURL that supports TLS 1.2 protocol.
* For the installation of the Gateway Agent, the following Operating Systems are supported: Oracle Linux 5.x, 6.x, and 7.x (64-bit), Red Hat Enterprise Linux 5.x, 6.x, and 7.x (64-bit), SUSE Linux Enteprise Server 11 (x86_64), AIX 6.1 or later (minimum level 6100-07), AIX 7.1 or later (minimum level 7100-03), Solaris 10.x and 11.x, Microsoft Windows versions 2008 Enterprise R2 (Intel 64-bit, Developer) or later, Microsoft Windows Server 2012 Standard (Intel 64-bit), Microsoft Windows 2012 Standard R2 (Intel 64-bit, Developer) or later.

## 1. Prepare Your Environment

#### Identify Your Oracle Management Cloud Instance

To meet the pre-requisites, you must have signed up for an Oracle Cloud account and created an Oracle Management Cloud instance.

Sign in to Oracle Cloud as a user with the OMC Administrator role. Your Oracle Management Cloud instance tile should be displayed on the My Services dashboard.

1. On your Management Cloud tile, click the **Action Menu** ![alt text][hamburger] then click **Open Service Console**. You are now viewing the Oracle Management Cloud instances page.
2. On the instance you want to access, click the **Manage this instance** menu and select **OMC URL**. The Oracle Management Cloud console home page is displayed.
3. Navigate to **Administration** > **Agents** > **Download** and select the Gateway Agent agent type from drop-down list. Make a note of your `TENANT_NAME` and `OMC_URL` values for your environment.

![alt text][agent-download]

#### Check for TLS 1.2 Support
Ensure your cURL version supports the TLS 1.2 protocol by running the following command from a terminal session on your Unix host:

<pre>
$ <b>curl --help | grep -i tlsv1.2</b>
--tlsv1.2       Use TLSv1.2 (SSL)
</pre>
Note the output of this command confirms that the version of the TLS protocol is TLSv1.2 (SSL).

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

* PDF files with detailed instructions for uploading and evaluating sample data and your own data, as well as instructions for installing a Gateway agent
* Alertlog.zip (Sample database alert logs)
* Messages.zip (Sample Linux syslog files)
* <span>uploadSample.sh</span> (a shell script for uploading the sample logs)
* <span>uploadMyLog.sh</span> (a shell script for uploading your own logs)
  
The next steps will guide you through these files.

## 3. Upload Sample Logs and Evaluate

You are now ready to upload some sample logs to Log Analytics and explore its functionality.

#### Uploading Sample Logs to Log Analytics

To upload the provided sample logs, follow these steps:
1. Before uploading logs, enter property values to be used in uploading log in file `SCRIPT_HOME/DBLogTrial/uploadSample/config/upload.properties`.
   * Go to the `SCRIPT_HOME/DBLogTrial/uploadSample/config` directory.
   * Use an editor of your choice to edit file `upload.properties` to set appropriate values for the following properties:
     * `UPLOAD_ROOT`: your `OMC_URL`
     * `IDENTITY_DOMAIN`: your `TENANT_NAME`
     * `USERNAME`: your OMC username
     * (Optional) `HTTPS_PROXY`

    **Mandatory Properties**
    <pre>
    # URL for uploading data to OMC
    # Examples:
    # UPLOAD_ROOT=https://inst1-acme.itom.management.us2.oraclecloud.com
    # UPLOAD_ROOT=https://inst2-xyz.itom.management.europe.oraclecloud.com
    # UPLOAD_ROOT=https://a123456.itom.management.us2.oraclecloud.com
    # This is a required parameter. The "https://" part is optional.
    UPLOAD_ROOT=
    
    # Subscription Identity Domain
    # EX:
    # IDENTITY_DOMAIN=acme
    # This is a required parameter
    IDENTITY_DOMAIN=
    
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
   
