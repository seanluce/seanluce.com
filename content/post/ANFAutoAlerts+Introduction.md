---
title: "Automatic Alerts for Azure NetApp Files"
date: 2020-07-23T16:35:43-04:00
categories:
- azure netapp files
tags:
- monitoring
- alerting
- azure
- cost
- cloud
- netapp
keywords:
- azure
- monitoring
- alerting
- azure
- netapp
- files
thumbnailImage: /img/logicapp.svg
---
<img src="/img/anf.svg" height="80" align="left" style="margin: 0px 20px 0px 0px;" /><img src="/img/logicapp.svg" height="80" align="left" style="margin: 0px 20px 0px 0px;" />

Controlling cloud spend is a massive challenge for cloud consumers of all sizes. Having good systems in place to monitor resources and provide proper alerting mechanisms when that consumption goes beyond expected levels is critical to any successful cloud deployment.

While Azure NetApp Files is a great enterprise grade file service, the Azure metrics that we have to work with today can be a bit limiting when it comes to capacity consumption. One of the current shortcomings is the absence of a metric that allows you to alert on a percentage of space consumed at a capacity pool or volume level. Currently, the alert threshold needs to be specified in bytes.. bytes?! Yes, BYTES. Furthermore, when a capacity pool or volume is resized, the corresponding alert threshold needs to be manually increased or decreased accordingly.

After a few conversations with customers and colleagues, I decided to see if I could automate the creating and updating of Azure monitor alert rules for Azure NetApp Files resources. Inspired by my good friend, [Kirk Ryan](https://kirkryan.co.uk/)'s [anfScheduler](https://github.com/ANFTechTeam/anfScheduler) Logic App, I thought that may be a good place to start.

### And that is how [ANFAutoAlerts](https://github.com/ANFTechTeam/ANFAutoAlerts) was born...

> [ANFAutoAlerts](https://github.com/ANFTechTeam/ANFAutoAlerts) is an Azure Logic App that automates the creating, updating, and deleting of capacity based alerts for Azure NetApp Files.

<!--more-->
ANFAutoAlerts monitors the Azure activity log for new, modified, or deleted Azure NetApp Files resources and then takes the appropriate action depending on the activity.

For more details and to deploy ANFAutoAlerts in to your environment head on over to the GitHub [repository](https://github.com/ANFTechTeam/ANFAutoAlerts).

[![ANFAutoAlerts](/img/autoalertsthumb.png)](https://github.com/ANFTechTeam/ANFAutoAlerts)

### Happy alerting!

