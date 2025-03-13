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

Pushing these countermeasures to the the earliest possible proxy inside the proxy chain has multiple advantages over rejecting it at the home server.
First, it reduces the load on all proxies in the proxy chain, since they do not need to forward traffic that will get rejected anyway.
Secondly, when the response should get delayed, pushing this delay as far down the proxy chain prevents RADIUS retransmissions.
When the RADIUS proxy already has the response, it then does not need to proxy the retransmitted RADIUS packets, which reduces the load for the RADIUS proxies in the later proxy chain.
Instead, the RADIUS proxy just ignores the retransmission, since it already has an answer for this RADIUS packet, it just delays its response.

# Conventions and Definitions

{::boilerplate bcp14-tagged}

Additionally, we use the following terms in this document, in the meaning as defined here:

RADIUS Instance
: A single device or software module that implements the RADIUS protocol

RADIUS Proxy
: A single device or software module that acts as RADIUS server and RADIUS client at the same time. It receives RADIUS requests and forwards them towards the next RADIUS proxy, usually based on the realm of the User-Name attribute.

RADIUS Proxy Chain
: the list of RADIUS Instances that a RADIUS Request traverses from the first RADIUS Client across any number of RADIUS proxies to the final RADIUS Server that responds to the RADIUS Request.

# Protocol Description

The protocol extension consists of two parts:
First, any RADIUS proxy in the proxy chain capable of either of the two countermeasures needs to signal this capability to the following RADIUS proxies and the home server, so they know whether or not they can use this feature.
Second, for the reply, the home server or RADIUS proxy needs to signal the reply policy back to the previous RADIUS proxies.

## Proxy-Capability Attribute

The Proxy-Capability Attribute is used to signal the capability of a RADIUS proxy to any RADIUS entity in the later proxy chain.
With the help of this, on the reply path, a RADIUS proxy can determine whether the requested action should be performed by itself or the packet will pass through another capable proxy later which can then perform the actions.

The Proxy-Capability Attribute is of type string as defined in {{!RFC8044, Section 3.5}}.
The value of the Proxy-Capability Attribute is a concatenated list of the proxy capabilities the RADIUS Instance has.

Correct formal description: TODO

Informal description: concatenate all capabilities. values up to 127 are encoded in one byte, extended capabilities are encoded as two bytes.
For parsing, the receiver can look at the first bit, if it is a 0 it is a single-byte value, if it is a 1, then the capability is a two-byte value. This allows for simple extension, while keeping it as simple and short as possible. The attribute MUST NOT include a capability multiple times.


Each capable RADIUS Instance in the RADIUS Proxy Chain SHOULD add the Proxy-Capability Attribute for Access-Request and Accounting-Request packets before forwarding the RADIUS packet to the next RADIUS instance.
Future capabilityes MAY specify capabilities for other RADIUS packet types. The capabilities defined in this document SHOULD only be added for Access-Request and Accounting-Request packets when Proxy-Capabilitiy is used with other RADIUS packets.

When a capable RADIUS proxy receives a RADIUS packet with the Proxy-Capability Attribute, the RADIUS Proxy SHOULD add its own capabilities to the Attribute if the capability is not yet included. The RADIUS Proxy MUST NOT remove existing capabilities, unless explicitly configured to remove them. As a hint, administrators SHOULD only configure the removal of capabilities when they know that the capability is not honored.

In this document, we define two capabilities:

Capability Response-Delay-Capable
: The Capability Response-Delay-Capable with value 1 is used to signal that the RADIUS Instance is capable of delaying RADIUS Response packets.

Capability Response-Block-Capable
: The capability Response-Block-Capable with value 2 is used to signal that the RADIUS Instance is capable of blocking RADIUS Requests that match specific criteria and sending an Access-Reject instead on behalf of the home server.

## Response-Delay Attribute

The Response-Delay Attribute is used to signal the desire of the home server that sending of the RADIUS response should be delayed for a certain amount.
The Response-Delay Attribute is of type integer as defined in {{RFC8044, Section 3.1}}.
The value is the deplay in milliseconds.

## Response-Block Attribute

The Response-Block Attribute is used to signal the desire of the home server that future requests that match certain criteria should be rejected by a RADIUS Instance on behalf of the home server.

# Security Considerations

TODO Security


# IANA Considerations

This document will have IANA actions.

They are still TODO in detail.

Roughly the following things should be allocated:

* Attribute Type (possibly from extended attributes) for Proxy-Capability of type string (Extended-Attribute-1, TBD1)
* New registry table for for types in the Proxy-Capability attribute
  * 0 - reserved
  * 1 - Response-Delay-Capable
  * 2 - Response-Block-Capable
  * 3-125 - reserved for future use
  * 126 , 127 - experimental
  * 128 - 255 - Extended Capability
* Attribute Type for Response-Delay of type integer (Extended-Attribute-1, TBD2)
* Attribute Type for Response-Block of type tlv (Extended-Attribute-1, TBD3)
* New registry table for types in the Response-Delay attributes
  * 0 - reserved
  * 1 - Response-Block, Type integer, request to stop sending data for this particular user for period of time, time in seconds
  * 2 - Response-Block-Attributes, type string
  * 3-250 - reserved for future use
  * 251 - private use
  * 252-255 - experimental
* New entry in the registry for Values for RADIUS Attribute 101, Error-Cause Attribute
  * 4XX (TBD4) Request ratelimited

--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
