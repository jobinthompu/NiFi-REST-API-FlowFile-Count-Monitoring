# NiFi REST API - FlowFile Count Monitoring


## Short Description

Sample Flow creation for Monitoring and alerting when Connection FlowFile Count exceeds a threshold using NiFi REST API.

## Introduction

Recently I was asked how to monitor and alert flowfile count in a connection queue when it exceeds a predefined threshold, I am trying to capture the steps to implement the same.

## Prerequisites

1) To test this, Make sure HDF-2.x version of NiFi is up an running.

2) You Already have a connection with data queued in it(say more than 20 flowfiles). Else you can create one like below:

![alt tag](https://github.com/jobinthompu/NiFi-Storm-Integration/blob/master/resources/images/OriginalFlow.jpg)

3) Make a note of the Connection name and uuid:

![alt tag](https://github.com/jobinthompu/NiFi-Storm-Integration/blob/master/resources/images/Original_Flow_Settings.jpg)
