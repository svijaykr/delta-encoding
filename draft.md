---
title: Delta Encoding
abbrev:
date: 2014
category: info

ipr: trust200902
area: General
workgroup:
keyword: Internet-Draft

stand_alone: yes
pi: [toc, tocindent, sortrefs, symrefs, strict, compact, comments, inline]

author:
 -
    ins: P. Kedia
    name: Pranav Kedia
    organization: Google
    email: pranavk@google.com
    uri: https://www.google.com/

author:
 -
    ins: S Vijayakrishnan
    name: Siddharth Vijayakrishnan
    organization: Google
    email: sidv@google.com
    uri: https://www.google.com/

normative:
  RFC2119:
  RFC5234:
  RFC5226:
  RFC7230:
  RFC7231:


informative:

--- abstract

Delta encoding mechanism to reduce the payload size by only transferring the diff.

--- middle

# Introduction
HTTP/1.1 supports response compression via Accept-Encoding and Content-encoding headers. However, it does not provide a efficient mechanism for compressing data that is repeated across responses. [RFC 3229](http://www.ietf.org/rfc/rfc3229.txt) limits support to only responses that originate from the same URL. To get around this limitation, Google proposed a specfication called [Shared Dictionary Compression over HTTP](http://lists.w3.org/Archives/Public/ietf-http-wg/2008JulSep/att-0441/Shared_Dictionary_Compression_over_HTTP.pdf) which extends support for cross-payload compression. The document tries to merge and extend these existing proposals with both cross-response payload compression and removing the need for an out-of-band dictionary download.

## Goals
- Enable incremental update one of one URL repsonse from a previous response for the same URL.
- Enable incremental update of one URL response from a previous response for another URL.
- Use properties inherent to the content to determine the content signature. i.e. use SHA256 or MD5. Do not use unreliable server side signatures like time-stamp or entity-tag.
- Support multiple delta transformation algorithms. e.g. vcdiff for text and [courgette](http://www.chromium.org/developers/design-documents/software-updates-courgette) for binary.
enable negotiation of delta transform algorithms to accommodate yet to be discovered delta algorithms. e.g. new image diff algorithm to cater to change in image sprite.
- Be efficient by not introducing extra round-trips or bloating request payload.

##Non-Goals
- Specify new caching semantics. The goal is to respect the behavior of existing caching headers
- Specify a mechanism to generate the delta diff
- More ?
 
#Request and Response Flow
- When a user agent that is requesting a resource for the first time, it advertises its support for delta encoding via the "Accept-Encoding: delta" header along with a list of supported algorithms via "A-IM" header (for e.g. A-IM: vcdiff).
- When a user agent is requesting a resource that exists in its cache, in addition to the step above, it advertises the dictionaries that are available for that resource via the "Delta-Base" header (for e.g. "Delta-Base: content-signature1, content-signature2"). The user-agent generates dictionaries for the resource based on previous responses. These responses can be for either the same resource or for a different resource, but linked together by the new "Delta-Related" headers (see below). The user-agent MUST advertise support for delta encoding via "Accept-Encoding" if it sends a Delta-Base header. 


- When the origin server sees a request without a "Delta-Base" but with an "Accept-Encoding: delta" header, it does nothing differently with the body of the response.- i.e. it sends back the response as it would have before. However, it MAY choose to indicate to the client that this response can be used as a delta-base for future requests via a "Delta-Related" header. For e.g. a request for http://somesite.com might generate a response with the headers 
    Delta-Related: "domain:.somesite.com"
and a request for http://anothersite.com/foo might generate a response with the headers 
    Delta-Related: "path:/foo, /bar"
This allows dictionaries to be defined on responses at both a domain and path level. Domain matching is as per RFC2965. (TODO) Path matching rules are specified below

- When the origin server sees a request with a "Delta-Base" header, it MAY choose to send back the response encoded as a delta to one of the entities specified in the "Delta-Base" header in the request. If it chooses to send back a delta-encoded response, it MUST respond with "Content-Encoding : delta" and MUST pick one of the user-agent advertised transformation algorithms. The algorithm by the server is sent back in the response in the IM header (for e.g. "IM: vcdiff"). Further, it MUST pick an entity from the user-agent supplied list of entities in the "Delta-Base" header and return that in the response.

- The user-agent computes the delta after it receives the response by applying the selected transformation algorithm to the base that has been picked by the server 

- The resource corresponding to the decoded response is then cached according to standard HTTP semantics. 

- This flow is expected to work with existing HTTP validation semantics including If-Modified-Since and If-None-Match. 


#Generating Content Signatures

#Validating the Response 


 

#Examples
+Client does not have a cached "Delta-Base" for requested URL.
  -Server does not indicate use of the response in future requests.
  -Server indicates use of the response in future requests.
##
## Notational Conventions
The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in {{RFC2119}}.

This document uses the Augmented Backus-Naur Form (ABNF) notation of {{RFC5234}} with the list rule extension defined in {{RFC7230}}, Appendix B. It includes by reference the DIGIT rule from {{RFC5234}}; the OWS, field-name and quoted-string rules from {{RFC7230}}; and the parameter rule from {{RFC7231}}.


# IANA Considerations

This document defines the "A", "B", "C", and "D" HTTP request fields and registers them in the Permanent Message Header Fields registry.

- Header field name: A
- Applicable protocol: HTTP
- Status: standard
- Author/Change controller: Pranav Kedia, pranavk@google.com
- Specification document(s): [this document]
- Related information: for Delta Encoding


# Security Considerations

Text goes here.

--- back
