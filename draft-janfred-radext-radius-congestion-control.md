---
entity:
  SELF: "[RFCXXXX]"
title: "Methods for Mitigation of Congestion and Load Issues on RADIUS Servers"
abbrev: "RADIUS Congestion Mitigations"
category: exp

docname: draft-janfred-radext-radius-congestion-control-latest
submissiontype: IETF
v: 3
area: ""
workgroup: "RADIUS EXTensions"
keyword:
 - RADIUS
venue:
  group: "RADIUS EXTensions"
  type: ""
  mail: "radext@ietf.org"

author:
  - name: Jan-Frederik Rieckers
    org: Deutsches Forschungsnetz | German National Research and Education Network
    street: Alexanderplatz 1
    code: 10178
    city: Berlin
    country: Germany
    email: rieckers@dfn.de
    abbrev: DFN
    uri: www.dfn.de

normative:

informative:


--- abstract

The RADIUS protocol as defined in {{!RFC2865}} does not have a means to signal server overload or congesition to the clients.
This can lead to load problems, especially in a federated RADIUS proxy fabric.
This document attempts to fix this.

--- middle

# Introduction

The RADIUS protocol {{RFC2865}} does not have a means to signal a server overload or a congestion to RADIUS clients.
These overload situations may be a result of a high load of legitimate traffic and might even be worsened by retransmissions of packets the server failed to answer due to the high load.
These situation can happen in a lost of scenarios.
In RADIUS proxy fabric, a server overload may even result from a single RADIUS client, for example when an EAP supplicant immediately starts a new authentication try without delay when getting a reject.

Especially in RADIUS proxy fabrics, the impact of misbehaving clients on the whole proxy chain can be reduced by reducing the packet load at the entry level or as early in the proxy chain as possible.
Since the end user device cannot be controlled, we have to rely on the RADIUS proxies to implement coutermeasures.

These countermeasures can be used to reduce the load by one of two methods.

First, the response to requests can be delayed. By delaying RADIUS responses, the client has to wait for the answer to send its next request, which decreases the packet load on the server.
This method can also be used to slow down clients that immediately retry the authentication once they receive a reject.

When a home server knows that an authentication of this client cannot succeed (for example because it used an expired certificate with EAP-TLS), and the client keeps retrying, any RADIUS actor along the proxy chain could generate a reject for this specific user.


# Conventions and Definitions

{::boilerplate bcp14-tagged}

# Protocol Description

TODO

# Security Considerations

TODO Security


# IANA Considerations

This document will have IANA actions.
They are still TODO in detail.

Roughly the following things should be allocated

* Attribute Type (possibly from extended attributes) for Delay-Capability of type string
* Attribute Type (possibly from extended attributes) for Response-Delay of type tlv
* New registry table for types in the Response-Delay attributes
  * 0 - reserved
  * 1 - Response-Delay, Type integer, delay request in milliseconds
  * 2 - Response-Block, Type integer, request to stop sending data for this particular user for period of time, time in seconds
  * 3 - Response-Block-Attributes


--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
