---
title: Visualizing Windows VM access in the Microsoft Sentinel map
date: 2022-05-11 22:20:00 -0500
categories: [Azure, security]
tags: [azure,sentinel,siem]     # TAG names should always be lowercase
---

In the [previous post](https://jean0828.github.io/blog/posts/Deployment-Microsoft-Sentinel-(SIEM)) we deployed Microsoft Sentinel with the connector to gather data from the Windows VM. Now, we can explore some workbooks to see pre-defined dashboards


![sentinel workbooks](https://i.imgur.com/N33kRjX.png "sentinel workbooks"){: width="700" height="400" }

for example, we can use the workbook called "Windows Firewall". There, it can be seen Windows Security events by account and IP.


![IP connections](https://i.imgur.com/vPmRqAz.png "Ip connections"){: width="700" height="400" }


Furthermore, the workbook *Identity & Access* shows details about authentications in the VM

![identity workbook](https://i.imgur.com/afE2AmY.png "identity workbook"){: width="700" height="400" }

Also, we can create queries to see authentication failed attempts. In the Logs sections of the Log Analytics Workspace, we can run:

```
SecurityEvent
| where EventID == 4625 
```

![security events](https://i.imgur.com/IkgTcEf.png){: width="700" height="400" }


However, if we can go the overview tab in Microsoft Sentinel, it doesn't show in the map the connections of the VM even when those connections are made by public IP adressess.

![map](https://i.imgur.com/BZBSFGo.png "map"){: width="700" height="400" }

For that reason, we're going to use a custom script to see conections to the VM in the map. This is an idea from [Josh Madakor](https://youtu.be/RoZeVbbZ0o0), thank you for sharing your knowledge.

## Observing Auth Failed log

First of all, we need to ensure how to see the log when an authentication failed attempt in the Windows system. To do that, it's needed to open *Event Viewer* &rarr; *Windows Logs* folder &rarr; *Security* channel

![event viewer](https://i.imgur.com/9D7cmm5.png){: width="700" height="400" }

In the security channel, the event 4625 shows what we want, and it shows important information like:

* account name
* source workstation name
* source IP address

![4625 event](https://i.imgur.com/GTpdlkd.png){: width="700" height="400" }

Now, the idea is to take the source IP address with a script which creates a custom log and then it shows the country of the IP in the map with the help of ipgeolocation.io.

**NOTE**: for better results in the SIEM, Windows Firewall is disabled. However, in a production envirtonment, you should keep turn it on.

![Windows FW](https://i.imgur.com/ewZ0Ekn.png){: width="700" height="400" }

## Using the Script

to begin with, we have to download the script [here](https://github.com/joshmadakor1/Sentinel-Lab/blob/main/Custom_Security_Log_Exporter.ps1). Then, we must update the *API_KEY* varibale for using the script.

In order to get the API key, we have to create an account in [ipgeolocation.io](https://ipgeolocation.io/). Next, go to *Dashboard* and  you see the API Key.


![API key ipgeo](https://i.imgur.com/ASOASFB.png){: width="700" height="400" }

The script gathers all the information of failed authentications, it takes the IP address and creates a custom log.

Now, it's time to run the script and see its results:

![script-results](https://i.imgur.com/tpN2FVC.png){: width="700" height="400" }

As you can see, there are real failed authentication attempts from Rusia.

## Creating custom log in Log Analytics Workspace to bring the log from the script

Firstly, in the azure portal, we go to the Log Analytics workspace &rarr; custom log &rarr; add custom log

![creating custom log](https://i.imgur.com/6EmhmJO.png){: width="700" height="400" }

Second, a copy of the log file created by the script must be downloaded in our local computer because it should be uploaded in the azure portal.

The log has the following format:

```
latitude:47.91542,longitude:-120.60306,destinationhost:samplehost,username:fakeuser,sourcehost:24.16.97.222,state:Washington,country:United States,label:United States - 24.16.97.222,timestamp:2021-10-26 03:28:29
latitude:-22.90906,longitude:-47.06455,destinationhost:samplehost,username:lnwbaq,sourcehost:20.195.228.49,state:Sao Paulo,country:Brazil,label:Brazil - 20.195.228.49,timestamp:2021-10-26 05:46:20
latitude:52.37022,longitude:4.89517,destinationhost:samplehost,username:CSNYDER,sourcehost:89.248.165.74,state:North Holland,country:Netherlands,label:Netherlands - 89.248.165.74,timestamp:2021-10-26 06:12:56
latitude:40.71455,longitude:-74.00714,destinationhost:samplehost,username:ADMINISTRATOR,sourcehost:72.45.247.
latitude:55.88802,longitude:37.65136,destinationhost:vm-sw,username:Mel,sourcehost:94.232.44.12,state:Central Federal District, country:Russia,label:Russia - 94.232.44.12,timestamp:2022-05-10 00:16:26
latitude:55.88802,longitude:37.65136,destinationhost:vm-sw,username:db2admin,sourcehost:94.232.44.12,state:Central Federal District, country:Russia,label:Russia - 94.232.44.12,timestamp:2022-05-10 00:16:24
latitude:55.88802,longitude:37.65136,destinationhost:vm-sw,username:Administrator,sourcehost:94.232.44.12,state:Central Federal District, country:Russia,label:Russia - 94.232.44.12,timestamp:2022-05-10 00:16:22
```
Third, record delimiter &rarr; new line & &rarr; Next

![creating custom log II](https://i.imgur.com/W2xQyeU.png){: width="700" height="400" }

Next, in the collection paths section selects:

* Type: Windows
* Path (path of the log): ```C:\ProgramData\failed_rdp.log```

After, in the details section, we put a name for the custom log: ```FAILED_RDP_WITH_GEO``` and then create.

![custom log III](https://i.imgur.com/wYd7JjB.png){: width="700" height="400" }

After aprox 20 minutes, we can query those logs in the Log analytics Workspace:

![watching custom log](https://i.imgur.com/Y7KOZDK.png){: width="700" height="400" }

Nevertheless, those logs don't have their fields created. Therefore, we have to parse the log with the following:

* select a log &rarr; right clic &rarr; *Extract field from ...*

![watching custom log](https://i.imgur.com/srkaGnF.png){: width="700" height="400" }

Notice the log has this format:
```field1:value,field2:value,field3:value... ```

according to the log format, we have to select each value and save it in a field name. for example, with *latitude* field we have to save the extraction:

![latitude](https://i.imgur.com/8K53QFX.png){: width="700" height="400" }

For the rest of the field, we have to do the same process:
* select a log &rarr; right clic &rarr; *Extract field from ...* &rarr; select the value of the field &rarr; give field name &rarr; select field type &rarr; extract &rarr; save extraction.

After some minutes, if you run again the query, you'll see the new fields in the results sections for the new logs:

![results-parsed](https://i.imgur.com/SI9F5Iy.png){: width="700" height="400" }

**NOTE**: in case that some of your field values in the search results is not highlighted, you have to modify that registry to correct its parsing.

![modift-parsed](https://i.imgur.com/C2c9SPl.png){: width="700" height="400" }


Besides, in the Custom logs &rarr; Custom fields, we can see the field names we created previuosly. In case that one of those fields is presenting a wrong parsing, we have to delete the custom field and extract it again.

## Setup map in sentinel with latitude and longitude (or country)

Now, in the Microsoft Sentinel section, we visit Workbooks &rarr; Add Workbook. Then, in edition mode, removing all the widgets.

![workbook](https://i.imgur.com/KYh68gM.png){: width="700" height="400" }

Afterward, add &rarrr; add query

![query](https://i.imgur.com/82NFEqL.png){: width="400" height="400" }

paste this query:

```
FAILED_RDP_WITH_GEO_CL | summarize event_count=count() by sourcehost_CF, latitude_CF, longitude_CF, country_CF, label_CF, destinationhost_CF
| where destinationhost_CF != "samplehost"
| where sourcehost_CF != ""
```

the query shows real failed authentication attempts by counts

![query-results](https://i.imgur.com/UjMkwpr.png){: width="700" height="400" }

In order to create the map, we choose *map* as visualization and the map settings will be shown to configure properly the map:

* locations info using: we can choose *latitude/longitude* or *country/region*. For this example, we're using *country/region*
* Country/Region field: country_cf
* size by: event_count
* Metric value: event_count

Finally, apply to update the map and *save and close*

![map-results](https://i.imgur.com/Q1LxVM8.png){: width="700" height="400" }

Then, save the map, put a title: *Failed RDP map*, set the location, and save it.

At this moment, we have to wait some minutes or hours to see new failed authentications and see in the map.

![final-map](https://i.imgur.com/jEWQegV.png){: width="700" height="400" }




