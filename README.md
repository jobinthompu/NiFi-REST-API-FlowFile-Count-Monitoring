# NiFi REST API - FlowFile Count Monitoring


## Short Description

Sample Flow creation for Monitoring and alerting when Connection FlowFile Count exceeds a threshold using NiFi REST API.

## Introduction

Recently I was asked how to monitor and alert flowfile count in a connection queue when it exceeds a predefined threshold, I am trying to capture the steps to implement the same.

## Prerequisites

1) To test this, Make sure HDF-2.x version of NiFi is up an running.

2) You Already have a connection with data queued in it(say more than 20 flowfiles). Else you can create one like below:

![alt tag](https://github.com/jobinthompu/NiFi-REST-API-FlowFile-Count-Monitoring/blob/master/resources/images/OriginalFlow.jpg)

3) Make a note of the Connection name and uuid to be monitored:

![alt tag](https://github.com/jobinthompu/NiFi-REST-API-FlowFile-Count-Monitoring/blob/master/resources/images/Original_Flow_Settings.jpg)

## Creating a Flow to Monitor Connection Queue Count.

1) Drop a GenerateFlowFile processor to the canvas and make "Run Schedule" 60 sec so we dont execute the flow to often.

2) Drop an UpdateAttribute processor, connect GenerateFlowFile's success relation to it and add below properties to it( the connection uuid noted above, threshold say 20, your NiFi host and port):

```
CONNECTION_UUID : dcbee9dd-0159-1000-45a7-8306c28f2786
COUNT			: 20
NIFI_HOST		: localhost
NIFI_PORT		: 8080
```
![alt tag](https://github.com/jobinthompu/NiFi-REST-API-FlowFile-Count-Monitoring/blob/master/resources/images/UpdateAttribute.jpg)

3) Drop a InvokeHTTP processor to the canvas, connect UpdateAttribute's success relation to it, auto terminate all other relations and update its 2 properties as below:

```
HTTP Method	: GET
Remote URL	: http://${NIFI_HOST}:${NIFI_PORT}/nifi-api/connections/${CONNECTION_UUID}
```
![alt tag](https://github.com/jobinthompu/NiFi-REST-API-FlowFile-Count-Monitoring/blob/master/resources/images/InvokeHTTP.jpg)

4) Drop an EvaluateJsonPath processor to extract values from json with below properties, connect Response relation of InvokeHTTP to it, and auto terminate its failure and unmatched relations.

```
QUEUE_NAME : $.status.name
QUEUE_SIZE : $.status.aggregateSnapshot.flowFilesQueued
```
![alt tag](https://github.com/jobinthompu/NiFi-REST-API-FlowFile-Count-Monitoring/blob/master/resources/images/EvaluateJsonPath.jpg)

5) Drop a RouteOnAttribute processor to the canvas with below configs, connect EvaluateJsonPath's matched relation to it and auto terminate its unmatched relation.

```
Queue_Size_Exceeded : ${QUEUE_SIZE:gt(${COUNT})}
```
![alt tag](https://github.com/jobinthompu/NiFi-REST-API-FlowFile-Count-Monitoring/blob/master/resources/images/RouteOnAttribute.jpg)

6) Lastly add a PutEmail processor, connect RouteOnAttribute's matched relation to it and auto terminate all its relations. below are my properties set, you have to update it with your SMTP details:

```
SMTP Hostname		:	west.xxxx.yourServer.net
SMTP Port			:	587
SMTP Username		:	jgeorge@hortonworks.com
SMTP Password		: 	Its_myPassw0rd_updateY0urs
SMTP TLS			:	true
From				:	jgeorge@hortonworks.com
To					:	jgeorge@hortonworks.com
Subject				:	Queue Size Exceeded Threshold
```
![alt tag](https://github.com/jobinthompu/NiFi-REST-API-FlowFile-Count-Monitoring/blob/master/resources/images/PutEmail.jpg)

and message content should look something like below to grab all the values:

```
Message				:	Queue Size Exceeded Threshold for ${CONNECTION_UUID}

						Connection Name				:	${QUEUE_NAME}
						Threshold Set				:	${COUNT}
						Current FlowFile Count 		:	${QUEUE_SIZE}
```
![alt tag](https://github.com/jobinthompu/NiFi-REST-API-FlowFile-Count-Monitoring/blob/master/resources/images/message_content.jpg)

7) Now the flow is completed and li should look similar to below:

![alt tag](https://github.com/jobinthompu/NiFi-REST-API-FlowFile-Count-Monitoring/blob/master/resources/images/FinalFlow.jpg)

## Staring the flow and testing it

1) Lets make sure at least 21 flow files are pending in the connection named 'DataToFileSystem' which was created in the Prerequisites

2) Now lets start the flow and you should receive mail alert from NiFi stating the count exceeded Threshold set which is 20 in our case.

   My sample alret looks like below:

![alt tag](https://github.com/jobinthompu/NiFi-REST-API-FlowFile-Count-Monitoring/blob/master/resources/images/Alert_email.jpg)

3) This concludes the tutorial for monitoring your connection queue count with NiFi.

4) Too lazy to create the flow???.. Download my template [here](https://github.com/jobinthompu/NiFi-REST-API-FlowFile-Count-Monitoring/blob/master/resources/Monitor_Connection_Queue_Count.xml)

## References
[NiFi REST API](https://nifi.apache.org/docs/nifi-docs/rest-api/index.html)

[NiFI Expression Language](https://nifi.apache.org/docs/nifi-docs/html/expression-language-guide.html)

Thanks,

Jobin George


