---
title: QUIC Reliable Messages
category: std

docname: draft-seemann-quic-reliable-message-latest
submissiontype: IETF
number:
date:
consensus: true
v: 3
area: AREA
workgroup: WG Working Group
keyword:
 - quic
 - message
 - reliabe
venue:
  group: "QUIC"
  type: "Working Group"
  mail: "quic@ietf.org"
  arch: "https://mailarchive.ietf.org/arch/browse/quic/"
  github: "marten-seemann/draft-seemann-quic-reliable-message"
  latest: "https://marten-seemann.github.io/draft-seemann-quic-reliable-message/draft-seemann-quic-reliable-message.html"

author:
 -
    fullname: Marten Seemann
    email: martenseemann@gmail.com

normative:

informative:


--- abstract

This document introduces a new way to transmit reliable messages on top of QUIC.

--- middle

# Introduction

Applications running on top of QUIC ({{!RFC9000}}) have two options to transmit
data. First, they can send data on top of (unidirectional or bidirectional) QUIC
streams. Second, with {{!RFC9221}} defines an unreliable datagram extension to
QUIC. In the case of packet loss, data sent on QUIC streams is automatically
retransmitted, unless the stream was reset. DATAGRAM frames are not
retransmitted.

This document defines an extension to QUIC that introduces a new frame, the
MESSAGE frame. This frame is intended for short (sub-MTU) messages that need to
be delivered reliably. However, implementations may allow the application to
update the message before sending a retransmission.

# Using MESSAGE frames

If the extension is negotiated, an endpoint is allowed to send MESSAGE frames up
to the maximum sequence number the peer advertised in the max_message_id
transport paramter. Endpoints SHOULD regularly increase the maximum sequence
number by sending MAX_MESSAGE frames as MESSAGE frames are consumed.

When a MESSAGE frame is declared lost, implemenations MUST do one of two things:
They either MUST retransmit the frame, or they MUST inform the application. THe
application MAY decide that the frame does not need to be retransmitted, or it
MAY update the contents of the message. If the contents of the message are
updated, it MUST be transmitted using a new sequence nubmer.

The receiver of a MESSAGE frame MUST deliver the frame to the application. It
MUST NOT drop the frame, unless the connection is closed.

# Frames

There are two frames, the MESSAGE frame, and the MAX_MESSAGE frame.

## MESSAGE Frame

~~~
MESSAGE Frame {
  Type (i) = TBD,
  Sequence Number (i),
  Message Data (...),
}
~~~

The MESSAGE frame contains the following fields:

Sequence Number:

: A variable-length integer encoding of the sequence number of the message.
Sequence numbers start at 0 and SHOULD be used sequentially.

Message Data:

: The message data to be delivered.


## MAX_MESSAGE Frame

~~~
MAX_MESSAGE Frame {
  Type (i) = TBD+1,
  Maximum Sequence Number (i),
}
~~~

The MAX_MESSAGE frame contains the following fields:

Maximum Sequence Number:

: The maximum sequence number that the peer is willing to receive.

The Maximum Sequence Number MUST NOT be decreased. However, due to reordering on
the wire, MAX_MESSAGE frames might be received out of order. The receiver MUST
ignore MAX_MESSAGE frames that do not increase the Maximum Sequence Number.

# Negotiating Extension Use

Endpoints advertise their support of the extension described in this document by
sending the max_message_id (TBD) transport parameter ({{Section 7.4 of
RFC9000}}) with variable-length integer value specifiying the maximum sequence
number of MESSAGE frames it is willing to receive. An implementation that
understands this transport parameter MUST treat the receipt of an invalid value
as a connection error of type TRANSPORT_PARAMETER_ERROR.

When using 0-RTT, both endpoints MUST remember the value of this transport
parameter. This allows use of this extension in 0-RTT packets. When the server
accepts 0-RTT data, the server MUST NOT disable this extension or reduce the
value on the resumed connection.


# Conventions and Definitions

{::boilerplate bcp14-tagged}


# Security Considerations

A malicious node could attempt to overwhelm an endpoint by sending a large
number of MESSAGE frames. Endpoints SHOULD make sure to define reasonable limits
using the transport parameter and the MAX_MESSAGE frame.

# IANA Considerations

TODO: consider transport parameter and frame types

--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
