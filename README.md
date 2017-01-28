# NiFi REST API - FlowFile Count Monitoring


## Short Description

Sample Flow creation for Monitoring and alerting when Connection FlowFile Count exceeds a threshold using NiFi REST API.

## Introduction

Recently I was asked how to monitor and alert flowfile count in a connection queue when it exceeds a predefined threshold, I am trying to capture the steps to implement the same.

## Prerequisites

1) To test this, Make sure HDF-2.x version of NiFi is up an running.

2) You Already have a connection with data queued in it(say more than 20 flowfiles). Else you can create one like below:

![alt tag](https://github.com/jobinthompu/NiFi-REST-API-FlowFile-Count-Monitoring/blob/master/images/OriginalFlow.jpg)

3) Make a note of the Connection name and uuid to be monitored:

![alt tag](https://github.com/jobinthompu/NiFi-REST-API-FlowFile-Count-Monitoring/blob/master/images/Original_Flow_Settings.jpg)

## Creating a Flow to Monitor Connection Queue.

1) Drop a GenerateFlowFile processor to the canvas and make "Run Schedule" 60 sec so we dont execute the flow to often.

2) Drop an UpdateAttribute processor, connect GenerateFlowFile's success relation to it and add below properties to it( the connection uuid noted above, threshold say 20, your NiFi host and port):

```
CONNECTION_UUID : dcbee9dd-0159-1000-45a7-8306c28f2786
COUNT			: 20
NIFI_HOST		: localhost
NIFI_PORT		: 8080
```
![alt tag](https://github.com/jobinthompu/NiFi-REST-API-FlowFile-Count-Monitoring/blob/master/images/UpdateAttribute.jpg)

3) Drop a InvokeHTTP processor to the canvas, connect UpdateAttribute's success relation to it, auto terminate all other relations and update its 2 properties as below:

```
HTTP Method	: GET
Remote URL	: http://${NIFI_HOST}:${NIFI_PORT}/nifi-api/connections/${CONNECTION_UUID}
```
![alt tag](https://github.com/jobinthompu/NiFi-REST-API-FlowFile-Count-Monitoring/blob/master/images/InvokeHTTP.jpg)

4) Drop an EvaluateJsonPath processor to extract values from json with below properties, connect Response relation of InvokeHTTP to it, and auto terminate its failure and unmatched relations.

```
QUEUE_NAME : $.status.name
QUEUE_SIZE : $.status.aggregateSnapshot.flowFilesQueued
```
![alt tag](https://github.com/jobinthompu/NiFi-REST-API-FlowFile-Count-Monitoring/blob/master/images/EvaluateJsonPath.jpg)