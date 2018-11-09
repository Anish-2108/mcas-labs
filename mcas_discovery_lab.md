# Cloud Discovery

[:arrow_left: Home](./README.md)

## Estimated time to complete

:clock10: 30 min

Continuous reports in Cloud Discovery analyze all logs that are forwarded from your network using Cloud App Security. They provide improved visibility over all data, and automatically identify anomalous use using either the Machine Learning anomaly detection engine or by using custom policies that you define.
To use this capability, you will perform in this lab the configuration and troubleshooting of the Cloud Discovery feature.

---

## Configure and test continuous reports

[:arrow_up: Top](#Cloud-Discovery)

> NOTE: The Docker engine has been pre-installed on LinuxVM in your lab environment, using the commands provided in the [deployment guide](https://docs.microsoft.com/en-us/cloud-app-security/discovery-docker-ubuntu).

Those commands download a script installing the Docker engine on your host computer (Ubuntu in this case) and pull the latest Cloud App Security collector image from the Docker library.

``` bash
curl -o /tmp/MCASInstallDocker.sh https://adaprodconsole.blob.core.windows.net/public-files/MCASInstallDocker.sh && chmod +x /tmp/MCASInstallDocker.sh; /tmp/MCASInstallDocker.sh
```

### Create a data source and a log collector in the Cloud App Security portal

1. Switch to **Client01**.

2. Create a new tab in the InPrivate window and browse to [**https://portal.cloudappsecurity.com**](https://portal.cloudappsecurity.com).

   >INFO: If necessary, log in using the credentials below:
   >
   >**Global Admin Username**
   >
   >**Global Admin Password**

3. In the Cloud App Security dashboard, click on the **Settings** icon and click **Log collectors**.

   ![Settings](media/dis-settings.png "Settings")

4. On the **Data sources tab**, click the **Add data source...** button.

    ![New data source](media/dis-newsource.png "New data source")

5. In the Add data source window, use the settings below (do not close the window yet):

    >|||
    >|---------|---------|
    >|Name| **SquiDLogs**|
    >|Source| **SQUID (Common)**|
    >|Receiver type| **FTP**|
    >|Anonymize private information |**Check the box**|
    >
    ![Squid source](media/dis-squidsource.png)

    >:memo: **NOTE:** In this lab we use FTP as the receiver type but usually companies will use Syslog.

6. While still in the Add data source dialog, click **View sample of expected log file**.

    >:memo: **NOTE:** Using this information, you can verify with your network team that the provided logs match the format expect by Cloud App Security. If it doesn't, you can should a custom parser.

    ![Verify log format](media/dis-verifylog.png "Verify log format")

7. In the Verify your log format dialog, click **Download sample log** and save to your desktop. Those logs will be used to simulate an appliance sending traffic logs to the log collector.

    ![Download sample](media/dis-downloadsample.png "Download sample log")

8. Close the *Verify your log format* window, then click **Add** in the **Add** data source dialog.

    ![Add source](media/dis-addsource.png "Add source")

    >**INFO:** We just created a data source which is the logical representation of the network appliance data source type the log collector will receive.

9. Next, click on the **Log collectors tab** and click the **Add log collector...** button.

    ![Add log collector](media/dis-addlogcollector.png "Add log collector")

10. In the Create log collector dialog, provide the settings below and click the **Update** button.

    |||
    |-----|-----|
    |Name|**LogCollector**
    |Host IP address|**192.168.141.125**
    |Data source(s)|**SquidLogs**

    ![Create log collector](media/dis-addlogcollector.png "Create log collector")

11. After clicking on the **Update** button, you have now the required steps to create your log collector instance on **LinuxVM**.
    >:warning: Do not close this window!

    ![Create log collector command](media/dis-addlogcollectortoken.png "Create log collector command")

    ``` bash
    (echo 1f5b5fb2a0d778e3d57f26ca5ab11574db0751166477940528ccf19a7c4) | docker run --name LogCollector -p 21:21 -p 20000-20099:20000-20099 -e "PUBLICIP='192.168.141.125'" -e "PROXY=" -e "SYSLOG=false" -e "CONSOLE=xyztenant.eu.portal.cloudappsecurity.com" -e "COLLECTOR=LogCollector" --security-opt apparmor:unconfined --cap-add=SYS_ADMIN --restart unless-stopped -a stdin -i microsoft/caslogcollector starter
    ```

    >**INFO:** This command line contains the different parameters to instanciate a new log collector on the Linux host:
    >- An API token to connect to Cloud App Security for uploading the logs: *1f5b5fb2a0d778e3d57f26ca5ab11574db0751166477940528ccf19a7c4*
    >- The docker parameters to configure the log collector container: *docker run ...*

12. Copy the command line provided at the end of the previous step and **minimize** the browser. Open **Putty (64-bit)**. You should have the icon on your desktop.
    ![Putty](media/dis-putty.png "Putty")

13. In the PuTTY Configuration window, enter **192.168.141.125** and click **Open**.

    ![Putty config](media/dis-puttyconfig.png "Putty config")

14. At the Putty warning message, click **Yes**.
    >**INFO:** This warning is due to the ssh certificate. You can safely ignore this warning in this lab.

    ![Putty warning](media/dis-puttywarning.png "Putty warning")

15. Log in using the credentials below.
    >|Username|Password|
    >|---|---|
    >|user01|Passw0rd1|
    >
    >:warning:The password doesn't appear in the command prompt, you can safely press enter to validate the credentials.

    ![Putty prompt](media/dis-puttylogin.png)

16. Type the command below and press **Enter**. Provide the user password when prompted.
    ``` bash
    sudo -i
    ```
    ![sudo](media/dis-sudo.png)
    >**INFO**: The previous command elevates your permissions in the Linux environment like the UAC prompt would do on a Windows machine.

17. Return to the *Create log collector* dialog, copy the **collector configuration** command from step 2 and run it in the PuTTY window.

    ![Copy token](media/dis-addlogcollectorcopy.png "Copy token")
    ![New container](media/dis-newcontainer.png "New container")
    >**INFO:** The output of this command is the id of the newly created container/log collector.

18. Now, launch **WinSCP** from the start-menu.

    ![WinSCP](media/dis-winscp.png "WinSCP")

19. Use the details below in the WinSCP window to connect to the log collector FTP service.

    |File Protocol|Host name|User name|Password|
    |-----|-----|-----|-----|
    |FTP|192.168.141.125|discovery|BP98Jw4Ns*zpTFrH|

    ![WinSCP connection](media/dis-winscpconnection.png "WinSCP connection")

    >**INFO**: this information was provided during the log collector creation.
    >
    >:memo: **NOTE:** the password is common to every new log collector. To change it, follow [this guide](https://docs.microsoft.com/en-us/cloud-app-security/troubleshoot-docker#docker-deployment) in the documentation.

    You should then be able to see a folder with your data source name.

    ![WinSCP connection](media/dis-winscpfolder.png "WinSCP connection")

    >:warning: If you are **not** able to connect to the log collector FTP service, verify that you successfully created the new log collector instance within Putty in previous steps.

20. On the left pane, move to the **Desktop** folder and drag your demo Squid log into the folder named for your data source (**SquidLogs**). After some minutes, the log collector will upload your logs.

    ![Log upload](media/dis-winscplogupload.png "Log upload")
    ![Log upload](media/dis-winscplogupload2.png "Log upload")
    ![Log upload](media/dis-winscplogupload3.png "Log upload")

21. Return to the Cloud App Security portal and click on **Settings** > **Governance log**.

    ![Settings Governance log](media/dis-governancelog.png "Settings Governance log")

22. Verify the status of the uploaded logs.

    >**INFO:** The status you see is the parsing status of the logs. Parsing status can be successful, pending or failed.

    ![Log uploaded](media/dis-loguploaded.png "Log uploaded")

23. You can also verify the **last data received** status on the *Data sources* tab under **Automatic log upload** settings.

    ![Last data received](media/dis-lastreceived.png "Last data received")

24. Go to the **Cloud Discovery dashboard** to verify the discovered apps.

    ![Discovery dashboard](media/dis-discoverydashboard.png "Discovery dashboard")

    ![Discovery data](media/dis-discoverydata.png "Discovery data")

    >:memo: **NOTE:**  After validating that your logs have been successfully uploaded and processed by MCAS, you will not usually see directly the analysis of your data. Why?
    >
    >**ANSWER:** Cloud Discovery logs are only parsed **twice a day**.

---

## How to troubleshoot the Docker log collector

[:arrow_up: Top](#Cloud-Discovery)

In this task, you will review possible troubleshooting steps to identify
issues in automatic logs upload from the log collector.

There are several things to test at different locations: in the log
collector, in MCAS, at the network level.

### Useful commands
	
- To navigate in the directories, use the "cd" command.
	Examples: 

	**cd /var/adallom** to go to the specified directory

	**cd /** to go to the root directory

	**cd ..** to go to the parent directory
	
- To display the content of the logs, use the **more file_name** command
- To display the content of the directory, use the **ll** command
- To clear the screen, use the **clear** command

- For saving typing, use the **Tab** key and perform autocompletion.

### Verify the log collector (container) status

1. On Client01, open a session on PuTTY to **192.168.141.125** and use the credentials below.

	**user01**

	**Passw0rd1**

1. Run the following commands:

	```
	sudo -i
	```
	```
	docker stats
	```
	This command will show you the status of the log collector instance:

	![CONTAINER ID 2d7cadgfS4a1 13.14\* CPU 1.22\* MEM USAGE / LIMIT 187
	.1MiB / 1.39GiB NET I/o 10.gMB / 3 .23MB](vl5158cy.jpg)
1. Press **Ctrl-C** to end the command. 
1. Next, run the command below:

	```
	docker logs --details LogCollector
	```

	This command will show you the logs from the log collector to verify if
	it encountered errors when initiating:



	![rootaubuntu-srt: \'hame,\'seb 5 \--dzt.ei15 Setting ftp configuration
	Enter again: Setting syslog Reading configuration.. . Installing
	collector successfully! zenzitive Starting 2018-06-28 2018-06-28
	2018-06-28 2010-06-28 2018-06-28 08 2018-06-28 2018-06-28 seo 2018-06-28
	53B 2018-06-28 €67 2018-06-28 2018-06-28 667 08:28: is, CRIT WARN I NEO
	CR IT I NEO 1 NEO INFO I NEO INSO I NEO INFO I NEO INFO as uzez in file)
	during parsing RBC interface \' supervisor • initialized http without
	HTTP checking Started With pid 1059 spawned: spawned : success : • with
	1062 •rsyslog• with pid 1063 with pid 2064 • Columbus\' with 1065
	rsyslog RUNNING stace, ftpd entered RUNNING state, pza RUNNING scat\* ,
	Stayed up for](4bfomeag.jpg)

	 

#### To go further in the troubleshooting, you can connect to the log collector container to investigates the different logs.

1. Type the following command:

	```
	docker exec -it LogCollector bash
	```

1. You can then explore the container filesystem and inspect the
	**/var/adallom** directory. This directory is where you will investigate
	issues with the syslog or ftp logs being sent to the collector

	![ovjlyn26.jpg](media/ovjlyn26.jpg)

-   **/adallom/ftp/discovery**: this folder contains the data source
    folders where you send the log files for automated upload. This is
    also the default folder when logging into the collector with FTP
    credentials.

-   **/adallom/syslog/discovery**: if you setup the log collector to
    receive syslog messages, this is where the flat file of aggregated
    messages will reside until it is uploaded.

-   **/adallom/discoverylogsbackup**: this folder contains the last file
    that was sent to MCAS. This is useful for looking at the raw log in
    case there are parsing issues.

1. To validate that logs are correctly received from the network appliance,
you can also verify the **/var/log/pure-ftpd** directory and check the
transfer log:

	![erx39v7i.jpg](media/erx39v7i.jpg)

1. Now, move to the **/var/log/adallom** directory.

	![0h029uih.jpg](media/0h029uih.jpg)

-   **/var/log/adallom/columbus**: this folder is where you will find
    log files useful for troubleshooting issues with the collector
    sending files. In the log-archive folder you can copy previous logs
    compressed as .tar.gz files off the collector to send to support.

-   **/var/log/adallom/columbusInstaller**: this is where you will
    investigate issues with the collector itself. You will find here
    logs related to the configuration and bootstrapping of the
    collector. For example, trace.log will show you the bootstrapping
    process:

    ![ks4ttuuq.jpg](media/ks4ttuuq.jpg)

 


### Verify the connectivity between the log collector and MCAS

An easy way to test this is to download a sample of your appliance logs
from MCAS and use WinSCP to connect to the log collector to upload that
log and see if it gets uploaded to MCAS.

 

1. Upload the logs in the folder named by your source:

![bqhxmpns.jpg](media/bqhxmpns.jpg)

 

1. Then, check in MCAS the status:

	![21pseval.jpg](media/21pseval.jpg)

 

	![mt0o095m.jpg](media/mt0o095m.jpg)

 

	> NOTE: If the log stays in the source folder, then you know you probably have a
connection issue between the log collector and MCAS.

Another way to validate the connection is to log into the container like
in the previous task and then run *netstat -a* to check if we see
connections to MCAS:

1. In the PuTTY window, type the command below:

	```
	netstat -a
	```
	![rxvauw6e.jpg](media/rxvauw6e.jpg)