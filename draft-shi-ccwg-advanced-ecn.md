---
stand_alone: true
category: std
submissionType: IETF
ipr: trust200902
lang: en

title: Advanced Explicit Congestion Notification
abbrev: AECN
docname: draft-shi-ccwg-advanced-ecn-latest
obsoletes:
updates:
# date: 2022-02-02 -- date is filled in automatically by xml2rfc if not given

area: "Transport"
workgroup: "Congestion Control Working Group"

kw:
  - Congestion Notification

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
  email: zhoutianran@huawei.com

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

This document proposes Advanced Explicit Congestion Notification mechanism enabling host to obtain the congestion information at the bottleneck. The sender sets the congestion information collection command in the packet header indicating the network device to update the congestion information field per hop. The receiver carries the updated congestion information back to the sender in the ACK. The sender then leverage the rich congestion information to do congestion control.

--- middle

# Introduction {#intro}

Traditionally, congestion control has depended on implicit congestion detection by the host, where hosts gauge congestion primarily through packet loss or variations in round-trip times. Explicit Congestion Notification (ECN) represents a substantial improvement, as it facilitates network devices to explicitly signal congestion to the endpoints before packet loss occurs. Low Latency, Low Loss, Scalable throughput (L4S) leverages ECN to meticulously control the queuing delay. It uses ECN markings to maintain low queuing delays and avoid bufferbloat. However, ECN is limited by the use of a single bit of information. This limitation constrains the granularity of congestion information that can be conveyed. L4S's requirement for more detailed congestion signals demands an enhanced utilization of ECN, which could involve employing additional bits for a more precise representation of congestion levels and better control over delay and throughput in contemporary network environments.

HPCC{{?I-D.draft-an-ccwg-hpcc}} leverages more extensive congestion signals from the network by utilizing in-band telemetry, which facilitates the gathering of detailed load information from each switch it traverses. This enhanced approach enables HPCC to make more informed decisions on controlling network congestion and converge fast. However, one caveat associated with this approach is that HPCC utilizes an append mode for in-band telemetry. In append mode, as the packet traverses the network, it accumulates data from each switch, which consequently increases the size of the packet. This growth in packet size can potentially lead to issues such as exceeding the Maximum Transmission Unit (MTU) size which makes it unsuitable for the internet. Another caveat is that each sender need to repeat the computation to get the bottleneck information even if they shares the same path.

This document defines Advanced ECN which expands the 1 bit congestion notification to multiple bits and enables network device to update the congestion information per hop. When the packet arrives at the receiver, the congestion information field will reflect the congestion status of the path. By offloading the congestion information calculation to the network device, the computing burden of the endpoint can be reduced.

## Terminology

- ECN: Explicit Congestion Notification
- AECN: Advanced Explicit Congestion Notification
- HPCC: High Precision Congestion Control{{?I-D.draft-an-ccwg-hpcc}}
- DRE: Discounting Rate Estimator{{CONGA}}

## Requirements Language

{::boilerplate bcp14-tagged}

# Overview

{{AECN-procedure}} shows the overview procedure of AECN. First the sender MUST marks the packet with AECN command and initial Congestion Info(called AECN header, see {{format}}). The AECN Command specified what kind of the congestion information that the endpoint intend to collect from network devices. As the packet traverses through the network, each router MUST update the Congestion Info field based on the AECN command and the router's local load condition. Upon reaching the receiver, the updated congestion information within the packet is extracted and then communicated back to the sender, typically using the transport protocol's acknowledgment mechanism. The sender, now equipped with the congestion information reflective of the packet’s journey, uses this data to make informed adjustments to its sending rate.

~~~
              pkt+                     pkt+                     pkt+
         AECN Command+            AECN Command+            AECN Command+
+------+Congestion Info0+-------+Congestion Info1+-------+Congestion Info2+--------+
|Sender|===============>|Router1|===============>|Router2|===============>|Receiver|
+------+     Link-1     +-------+     Link-2     +-------+     Link-3     +--------+
  /|\                                                                         |
   |                                                                          |
   +--------------------------------------------------------------------------+
                                        ACKs
~~~
{: #AECN-procedure title="Overview of Advanced ECN"}

# AECN header format and encapsulation {#format}

{{AECN-header}} shown the format of AECN. The AECN header SHOULD be encapsulated in IPv6 extension header{{!RFC8200}} such as SRH, Hop by Hop Options Header etc.

~~~
 0                   1                   2                   3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|     Flags     |           Congestion Info Type                |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                     Congestion Info Data                      |
~                            ....                               ~
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
~~~
{: #AECN-header  title="AECN header format"}

where:

Flags: An 8-bit field. The Bit 7 of Flags indicates the Congestion Info is customized and used only in limited domain such as Data center network. If the Bit 7 is 0, the Congestion Info Type is a bitmap. Other bits are reserved.

Congestion Info Type: A 24-bit map that specifies the present Congestion Info Data. Supported Congestion Info Data is listed in {{congestion-info}}. Note that it is possible for multiple Congestion Info Data to coexist in one packet.

Congestion Info Data: A variable length field including the congestion information data. Router MUST update this field based on local load status.

| Bit | Congestion Info Data | Length | Operation |
|:------:+:-----+:------:+---------|
| 0 | Inflight Ratio | 8 | Max |
| 1 | DRE | 8 | Max |
| 2 | Queue Utilization Ratio | 8 | Max |
| 3 | Queue Delay | 8 | Add |
| 4 | Congested Hops | 8 |Add |
{: #congestion-info title="Congestion Info Data"}

# Example: HPCC with AECN

HPCC calculates the inflight ratio of each link(represent the link utilization of the link) from the collected raw load information carried in the INT. Then maximum inflight ratio along the path is identified and used to adjust the sending rate. The formula to calculate the inflight ratio of each link is shown below:

~~~
txRate = (txBytes_1 - txBytes_2)/(t_1-t_2)
inflight ratio = qlen/(B*T) + txRate/B
~~~

where:
txBytes: link total transmitted bytes associated with timestamp ts

qlen: link queue length

B: link bandwidth

T: Baseline RTT

Leveraging AECN, the router participates in calculation of the maximum inflight ratio. Each router MUST calculate the inflight ratio of the down link and then compare it to the one in the AECN header and keep the larger one. When the packet arrives at the endpoint, the Congestion Info field of the AECN header already contains the maximum inflight ratio. The sending rate adjustment algorithm remains unchanged. By allowing routers to conduct these calculations, the computing overhead is reduced for the endpoint. Since the update of value is in-place, the packet size remains unchanged regardless of the hops count.

# Security Considerations

TBD.

# IANA Considerations

TBD.
