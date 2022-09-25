---
title: Analyzing Packets with Wireshark and Python
date: 2022-09-24 22:20:00 -0500
categories: [python, wireshark]
tags: [wireshark,security,python]     # TAG names should always be lowercase
---

Some cases, as a analysts we need to review network packets to find the root cause of an error or incident. for this reason, in this post we're going to analyze packets using wireshark and see if we can analyze them with Python.

For this post I use [this pcap file](https://github.com/WOSECPA/AnalistaSOC2022/blob/main/Modulo%204/Taller%20de%20Wireshark/PCAP01.pcapng) from a course I have taken called "Bootcamp Analista SOC Nivel 1". It's a free and excellent course (taugh in spanish) made by DOJO community.

for that cap file we´re going to answer those questions:

1. What protocol did it use the port 3942?
2. What is the IP address of the host that was pinged twice?
3. How many DNS query response packets have been captured?
4. What is the source IP address which has sent more packets?

## Using Wireshark

1. in the filter section we can use this query to see the results:
```
udp.port==3942 || tcp.port ==3942
```

we can observe that in the column called *Protocol* we see the answer: SSDP. this protocol stands for *Simple Service Discovery Protocol*.

![1w](https://i.imgur.com/dTP1b3b.png)

2. in the filter section we should only write *icmp* and then observe the info column which packets begin with *request*

![2w](https://i.imgur.com/etGnkw9.png)

as a result, the answer is **8.8.4.4**

3. here we write *dns.resp.type* in the filter section and see the bottom of the windows to show the number of packets

![3w](https://i.imgur.com/bDDlZMk.png)

the answer is **99 packets**

4. here, I started to filter with ip address manually. Then, I could observer that *192.168.1.7* has a big part of the percentage

![4w](https://i.imgur.com/Cmd69bo.png)

Answering this question with python is better because we can automate with better filters and graph.

## Using python

First, we have to export the pcap file to csv in order to read with pandas module with the use of tshark:

```
 !tshark -r 'PCAP01.pcapng' -T fields -E header=y -E separator=, -E quote=d -E occurrence=f -e ip.version -e ip.hdr_len -e ip.tos -e ip.id -e ip.flags -e ip.flags.rb -e ip.flags.df -e ip.flags.mf -e ip.frag_offset -e ip.ttl -e ip.proto -e ip.checksum -e ip.src -e ip.dst -e ip.len -e ip.dsfield -e tcp.port -e tcp.srcport -e tcp.dstport -e tcp.seq -e tcp.ack -e tcp.len -e tcp.hdr_len -e tcp.flags -e tcp.flags.fin -e tcp.flags.syn -e tcp.flags.reset -e tcp.flags.push -e tcp.flags.ack -e tcp.flags.urg -e tcp.flags.cwr -e tcp.window_size -e tcp.checksum -e tcp.urgent_pointer -e tcp.options.mss_val -e udp.port -e _ws.col.Protocol -e _ws.col.Info -e dns.qry.name -e dns.resp.type > output.csv
```

Then, we can read it with:

```
df = pd.read_csv('output.csv',on_bad_lines='skip')
```

In python, the filters are similar to Wireshark but the advantage it's we can do more with the data.

1. To know what is the name of the protocol with number 3942, we can use this filter:

```
df[(df['tcp.port']==3942) | (df['udp.port']==3942)][['_ws.col.Protocol','_ws.col.Info']]
```
![1p](https://i.imgur.com/FIjzhDj.png)

2. to filter by ICMP protocol we use:

```
df[(df['_ws.col.Protocol']=='ICMP')][['ip.src','ip.dst','_ws.col.Info']]
```

![2p](https://i.imgur.com/LSwvNYp.png)

the table shows us a column called *_ws.col.Info* where we can see the echo request packets and confirm the destination ip address is **8.8.4.4**

3. we use the following filter:

```
df[df['dns.resp.type'].notnull()].count()
```
![3p](https://i.imgur.com/IiNuBaH.png)

we confirm that **99 packets** is the answer

4. Here we can use the groupby method to answer this question.

```
df.groupby(by='ip.src').count().sort_values('_ws.col.Protocol',ascending=False)['_ws.col.Protocol'].head(5).plot.bar()
```

![4p](https://i.imgur.com/YQf8Y7M.png)

according to the graph, we can see that the ip address **192.168.1.7** has more packets than the rest.

If you want to see my jupyter noteebook, you can see [here](https://github.com/jean0828/Analying-packets-with-Python/blob/main/Analyzing_pcap_files_with_python.ipynb).