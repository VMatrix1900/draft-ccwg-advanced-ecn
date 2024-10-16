---
stand_alone: true
category: std
submissionType: IETF
ipr: trust200902
lang: en

title: Data Fields for Congestion Measurement
abbrev: CM
docname: draft-shi-ippm-congestion-measurement-data-latest
obsoletes:
updates:

area: "Operations and Management"
workgroup: "IP Performance Measurement"

kw:
  - Congestion Measurement

author:
 -
  ins: H. Shi
  name: Hang Shi
  organization: Huawei
  email: shihang9@huawei.com
  role: editor
  city: Beijing
  country: China
 -
  ins: T. Zhou
  name: Tianran Zhou
  organization: Huawei
  city: Beijing
  country: China
  email: zhoutianran@huawei.com
 -
  ins: G. Zhao
  name: Guangyu Zhao
  organization: China Mobile
  country: China
  email: zhaoguangyu@chinamobile.com
 -
  ins: Z. Li
  name: Zhenqiang Li
  organization: China Mobile
  city: Beijing
  country: China
  email: li_zhenqiang@hotmail.com

informative:
  CONGA:
    title: "CONGA: distributed congestion-aware load balancing for datacenters"
    author:
      -
        initials: M.
        surname: Alizadeh
        fullname: Mohammad Alizadeh
      -
        initials: T.
        surname: Edsall
        fullname: Tom Edsall
      -
        initials: S.
        surname: Dharmapurikar
        fullname: Sarang Dharmapurikar
      -
        initials: R.
        surname: Vaidyanathan
        fullname: Ramanan Vaidyanathan
      -
        initials: K.
        surname: Chu
        fullname: Kevin Chu
      -
        initials: A.
        surname: Fingerhut
        fullname: Andy Fingerhut
      -
        initials: V.
        surname: Lam
        fullname: Vinh The Lam
      -
        initials: F.
        surname: Matus
        fullname: Francis Matus
      -
        initials: R.
        surname: Pan
        fullname: Rong Pan
      -
        initials: N.
        surname: Yadav
        fullname: Navindra Yadav
      -
        initials: G.
        surname: Varghese
        fullname: George Varghese
    date: 2014-8
    refcontent:
      - "Proceedings of the 2014 ACM conference on SIGCOMM"
    seriesinfo:
      DOI: 10.1145/2619239

--- abstract

Congestion Measurement collects the congestion information in the packet while the packet traverses a path. The sender sets the congestion measurement command in the packet header indicating the network device along the path to update the congestion information field in the packet. When the packet arrives at the receiver, the congestion information field will reflect the degree of congestion across network path. Congestion Measurement can enable precise congestion control, aids in effective load balancing, and simplifies network debugging. This document defines data fields for Congestion Measurement. Congestion Measurement Data-Fields can be encapsulated into a variety of protocols, such as Network Service Header (NSH), Segment Routing, Generic Network Virtualization Encapsulation (Geneve), or IPv6.

--- middle

# Introduction {#intro}

To effectively manage network congestion, a detailed understanding of congestion levels across the network is imperative. Congestion control algorithms, therefore, necessitate precise congestion measurements to adapt and optimize data flow. This approach involves monitoring various metrics such as packet loss, delay variations, and throughput, which can provide a glimpse of the network's congestion state. Enhanced congestion metrics allow for a more nuanced response to congestion, enabling algorithms to adjust sending rates with greater precision, thereby improving overall network performance and efficiency.

Furthermore, the detailed congestion measurements obtained are not solely beneficial for congestion control; they serve multifaceted purposes, including load balancing and network operations debugging. By analyzing congestion data, network operators can identify and resolve bottlenecks, optimize traffic distribution, and ensure a balanced load across the network. This data-driven approach facilitates proactive network management, allowing for timely interventions that can preempt potential disruptions and enhance network reliability and performance.

Addressing the limitations of High Precision Congestion Control (HPCC){{?I-D.draft-an-ccwg-hpcc}}, which leverages in-band telemetry for detailed congestion signal collection but faces challenges with packet size increases and computational redundancy, our proposed solution introduces data fields for Congestion Measurement. Congestion Measurement expands the conventional single-bit ECN to multiple bits, allowing network devices to update congestion information at each hop more granularly. Consequently, when packets reach the receiver, the congestion information field in the packet accurately not just the presence of congestion but the degree of congestion across the link's path. This nuanced approach facilitates a richer set of data for decision-making, supporting not only more precise congestion control but also improving load balancing and network debugging efforts. By overcoming HPCC's shortcomings, our approach enhances network efficiency, reduces computational overhead at endpoints, and offers a scalable solution to managing congestion in complex network environments. Congestion Measurement Data-Fields can be encapsulated into a variety of protocols, such as Network Service Header (NSH), Segment Routing, Generic Network Virtualization Encapsulation (Geneve), or IPv6.

## Terminology

- ECN: Explicit Congestion Notification
- HPCC: High Precision Congestion Control{{?I-D.draft-an-ccwg-hpcc}}
- DRE: Discounting Rate Estimator{{CONGA}}

## Requirements Language

{::boilerplate bcp14-tagged}

# Overview

{{CM-procedure}} shows the overview procedure of Congestion Measurement. First the sender MUST marks the packet with data fields for Congestion Measurement (see {{format}}) which specifies what kind of the congestion information that the sending node intends to collect from transit nodes. As the packet traverses through the network, each router should inspect the data fields and update the Congestion Info field accordingly. Upon reaching the receiver, the updated congestion info data within the packet is extracted and then send back to the sender. The sender, now equipped with the congestion information reflective of the packet's journey, uses this data to make informed adjustments to its sending rate or load balancing decisions.

~~~
   Mark              Update            Update             Export
Congestion         Congestion        Congestion         Congestion
Measurement          Info               Info               Info
    |                 |                  |                  |
    |                 |                  |                  |
    |                 |                  |                  |
+-------+         +-------+          +-------+         +---------+
|Sending|========>|Transit|=========>|Transit|======= >|Receiving|
|  Node |         | Node1 |          | Node2 |         |  Node   |
+-------+  Link-1 +-------+  Link-2  +-------+ Link-3  +---------+
~~~
{: #CM-procedure title="Overview of Congestion Measurement"}

# Data fields for Congestion Measurement {#format}

{{CM-header}} shown the format of data fields for Congestion Measurement.

~~~
 0                   1                   2                   3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|U| Reserved  |C|           Congestion Info Type                |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                     Congestion Info Data                      |
~                            ....                               ~
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
~~~
{: #CM-header  title="Data Fields for Congestion Measurement"}

where:

- Flags: An 8-bit field.
  - The first bit(U) indicates whether the Congestion Info Data field needs to be updated by transit nodes. If set, the transit nodes will update the Congestion Info Data. If not, the transit node will not update it.
  - The last bit(C) indicates the Congestion Info Data is customized and used only in limited domain such as Data center network. If the C is 0, the Congestion Info Type is a bitmap. Other bits are reserved.
- Congestion Info Type: A 24-bit map that specifies the present Congestion Info Data. Supported Congestion Info Data is listed in {{congestion-info}}. Note that it is possible for multiple Congestion Info Data to coexist in one packet for the endpoint to collect the detailed raw congestion information.
- Congestion Info Data: A variable length field including the congestion information data. Router MUST update this field based on local load status. The length and the update operation is listed in {{congestion-info}}.

| Bit | Congestion Info Data | Length | Operation |
|:------:+:-----+:------:+---------|
| 0 | Inflight Ratio | 8 | Max |
| 1 | DRE | 8 | Max |
| 2 | Queue Utilization Ratio | 8 | Max |
| 3 | Queue Delay | 8 | Add |
| 4 | Congested Hops | 8 |Add |
| 5 | Available Bandwidth | 8 | Min
{: #congestion-info title="Congestion Info Data"}

# Example 1: HPCC with Congestion Measurement

HPCC calculates the inflight ratio of each link(represent the link utilization of the link) from the collected raw load information carried in the INT. Then maximum inflight ratio along the path is identified and used to adjust the sending rate. The formula to calculate the inflight ratio of each link is shown below:

~~~
txRate = (txBytes_1 - txBytes_2)/(t_1-t_2)
inflight ratio = qlen/(B*T) + txRate/B
~~~

where:

- txBytes: link total transmitted bytes associated with timestamp ts
- qlen: link queue length
- B: link bandwidth
- T: Baseline RTT

Leveraging Congestion Measurement, the router participates in calculation of the maximum inflight ratio. Each router MUST calculate the inflight ratio of the down link and then compare it to the one in the Congestion Info Data field and keep the larger one. When the packet arrives at the endpoint, the Congestion Info Data field already contains the maximum inflight ratio. The sending rate adjustment algorithm remains unchanged. By allowing routers to conduct these calculations, the computing overhead is reduced for the endpoint. Since the update of value is in-place, the packet size remains unchanged regardless of the hops count.

# Example 2: Available bandwidth

The ABW(available bandwidth) of links can be applied in existing CC algorithms to optimize their throughput performance, such as TCP Reno and CUBIC. The sending rate and congestion window can be dynamically adjusted during the CC's slow-start and loss recovery phases. The BBR algorithm, which detects link bottleneck bandwidth based on rate and round-trip time (RTT), can utilize the ABW to obtain the bottleneck bandwidth of the link and optimize data throughput efficiency. Alternatively, a completely new CC algorithm can be designed based on ABW to predict and avoid congestion in advance.

The method for obtaining the ABW of a link is shown as follows:

1. The sending node can obtain the ABW of its egress port, mark the packet with data fields for ABW Measurement, and then send the packet to the Receiving node.
2. Transit Node identify the ABW probe action based on the Congestion Measurement header, compare the ABW of their egress port with the ABW in the packet. If the ABW of the current node is smaller than that in the packet, it updates to the link's ABW and forwards the packet; otherwise, it directly forwards the packet.
3. After receiving the ABW packet, the receiving node parses the link's ABW, constructs an ABW response packet, and sends it back to the sending node.

The calculation of the current node's ABW can be referenced as follows:
~~~
ABW = B - T - R
~~~

where B is the bandwidth of the egress port where the flow passes, T is the traffic size of that egress port, and R is the reserved bandwidth. The reserved bandwidth takes into account the fairness of the CC algorithm, facilitating the entry of newly added flow. The value of R can be set according to the specific circumstances of each node, allowing TOR switches and backbone routers to reserve different percentages of bandwidth.

# Security Considerations

TBD.

# IANA Considerations

TBD.
