---
title: The ADDR Pseudo Query Type
docname: draft-jabley-dnsop-addr-query-00

submissiontype: IETF
ipr: trust200902
area: Internet
wg: DNSOP Working Group
kw: Internet-Draft
cat: exp

coding: us-ascii
pi:
  - toc
  - symrefs
  - sortrefs

author:
  -
    ins: J. Abley
    name: Joe Abley
    org: Cloudflare
    city: Amsterdam
    country: NL
    phone: +31 6 45 56 36 34
    email: jabley@cloudflare.com

--- abstract

In the DNS, a single domain name can be associated with multiple
addresses.  Each address belongs to a single address family, such
as IPv4 or IPv6.  In the case where a name maps to addresses from
different address families, retrieving all addresses requires
multiple separate queries, one per address family, since the addresses
for each address family are published separately in the DNS in
different resource record sets, each with their own type. This
requires a client that wants to obtain all addresses to make multiple
queries.

This document describes the implementation and deployment a new,
meta query type that allows the initiator of a query to send a
single query and the responder to reply with the set of all IPv4
and IPv6 addresses that exist at the query name.

--- middle

# Introduction {#motivation}

The DNS protocol {{!RFC1034}} {{!RFC1035}} supports the publication
of addresses from a variety of different address families. Addresses
are published in RRSETs which have family-specific RRTYPEs. Multiple
such RRSETs can be published with the same owner name. For example,
the single owner name EXAMPLE.COM is associated with both an IPv4
and an IPv6 address (see {{examples}}).

This specification only addresses the case where the two address
families of interest are IPv4 and IPv6. It is common at the time
of writing for names to have mappings to addresses of both families,
and for software to use them interchangeably, e.g. as specified in
{{?RFC6555}}.

Software that wishes to obtain the full set of IPv4 and IPv6 addresses
associated with the same name must currently make two DNS queries,
one with each QTYPE.

This document specifies a mechanism by which a full set of IPv4 and
IPv6 addresses associated with a particular name can be obtained
in a single query using a new meta QTYPE.

# Terminology used in this document

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT",
"SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY",
and "OPTIONAL" in this document are to be interpreted as described
in BCP 14 {{!RFC2119}} {{!RFC8174}} when, and only when, they appear
in all capitals, as shown here.

"Initiator" refers to the side that sends a query.  "Responder"
refers to an authoritative-only server, recursive resolver or other
DNS component that receives queries and generates responses.  Other
DNS-specific terminology is used according to the definitions to
be found in {{!RFC8499}}.

# The ADDR Query Type {#specification}

This document defines the ADDR RRTYPE and specifies the manner in which
it is used. ADDR is a QTYPE {{!RFC6895}} and is available for use
with any CLASS.

## Sending ADDR Queries

The Initiator of a query with QTYPE=ADDR does so in order to obtain
a complete set of IPv4 and IPv6 addresses associated with the (QNAME,
QCLASS). Apart from the specified QTYPE, queries are constructed and
sent identically to queries with QTYPE=A or QTYPE=AAAA.

## Constructing and Sending ADDR Responses

Responses are constructed as follows:

1. QTYPE=ADDR;
1. The ANSWER, AUTHORITY and ADDITIONAL sections contain the union
of the RRSETS that the responder would include in separate responses
generated for QTYPE=A and QTYPE=AAAA;
1. In all other respects responses MUST otherwise be constructed normally,
including the use and interpretation of the DNS message header section
flags and EDNS(0) OPT meta-RR header.

A responder that is not authoritative for the zone containing the
QNAME (e.g.  a responder acting as a recursive resolver) that is
not able to construct a response exclusively from cached records
MUST send upstream queries in order for its response to be complete,
just as it would when responding to queries with QTYPE=A or QTYPE=AAAA
that could not be satisfied from the cache.

Where an owner name has either associated A or AAAA RRSETS but not
both and a query has requested a secure response with DO=1, the
response MUST include the proof of the non-existence of the missing
RRSET, e.g. by including a signed NSEC RR in the AUTHORITY section
of the response that confirms the absence of the missing RRTYPE at
the QNAME. This follows from (2) above.

## Receiving and Interpreting ADDR Responses

There are no special requirements for receiving and interpreting
responses to queries with QTYPE=ADDR.

## Discovering Support for ADDR Queries in Responders

Initiators SHOULD support local configuration that allows support
for particular responders to be specified based on knowledge of their
capabilities, for example where the initiator and responder are part
of a larger system and maintained by a common operator.

Initiators MAY also implement a discovery mechanism that allows
support in responders about which there is no local policy to be
inferred.  The following mechanism is provided as an illustration
of a possible mechanism that could be used. Implementors MAY use
different mechanisms.

1. An initiator maintains a cache of capabilities for DNS responders.
At startup this cache is empty.
1. An initiator of a query for QTYPE=A or QTYPE=AAAA to a responder
that is known from the cache to be capable uses QTYPE=ADDR to send
the upstream query.
1. If the capabilities of the responder are not known from the
cache, the initiator sends both QTYPE=A and QTYPE=AAAA and, in
parallel, QTYPE=ADDR. If all three queries generate responses and
the responses are compatible, the initiator adds the responder
address and a timestamp to the cache.
1. A responder included in the cache with a timestamp that is too
old to be considered current will no longer be considered current,
and the discovery mechanism should be repeated. The threshold
beyond which an entry should no longer be considered current is
a matter of local policy, and may be refined over time following
observation of caches in real-world operation.

Nameservers (responders) are often provisioned in a distributed
manner, e.g.  using anycast {{?RFC4786}} where multiple, autonomous
origin servers handle queries equivalently using the same nameserver
address. Where such nameservers are configured to support queries
with QTYPE=ADDR, it should be ensured that all origin servers
associated with the same nameserver address provide consistent
support.

## Examples {#examples}

The following three examples illustrate queries with the same QNAME
with QTYPEs A, AAAA and ADDR. Each example is provided in the form
of output from the common DNS troubleshooting tool "dig" as packaged
with the DNS nameserver BIND9. Some lines have been truncated to
improve readability. The example with QTYPE=ADDR is contrived since
at the time of writing the nameservers concerned do not support
this specification.

### Query and Response for (EXAMPLE.COM, IN, A)

~~~
  ; <<>> DiG 9.10.6 <<>> @A.IANA-SERVERS.NET EXAMPLE.COM A
    +multiline +dnssec +norec
  ; (1 server found)
  ;; global options: +cmd
  ;; Got answer:
  ;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 4055
  ;; flags: qr aa; QUERY: 1, ANSWER: 2, AUTHORITY: 0, ADDITIONAL: 1
    
  ;; OPT PSEUDOSECTION:
  ; EDNS: version: 0, flags: do; udp: 4096
  ;; QUESTION SECTION:
  ;EXAMPLE.COM.		IN A

  ;; ANSWER SECTION:
  EXAMPLE.COM.    86400   IN A 93.184.216.34
  EXAMPLE.COM.    86400   IN RRSIG A 13 2 86400 (
                       20231028144229 20231008002139 37939 example.com.
                       BfSk6wqpibs9a4AUa8gXHFzhBGiO9f0QuFLVaMYkPu0P
                       gKrDUl94VEfWlJ5pOMalgL1RzFwngd51b3FFiLUp3g== )
~~~

### Query and Response for (EXAMPLE.COM, IN, AAAA)

~~~
  ; <<>> DiG 9.10.6 <<>> @A.IANA-SERVERS.NET EXAMPLE.COM AAAA
    +multiline +dnssec +norec
  ; (1 server found)
  ;; global options: +cmd
  ;; Got answer:
  ;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 15495
  ;; flags: qr aa; QUERY: 1, ANSWER: 2, AUTHORITY: 0, ADDITIONAL: 1
    
  ;; OPT PSEUDOSECTION:
  ; EDNS: version: 0, flags: do; udp: 4096
  ;; QUESTION SECTION:
  ;EXAMPLE.COM.		IN AAAA
    
  ;; ANSWER SECTION:
  EXAMPLE.COM.    86400   IN AAAA 2606:2800:220:1:248:1893:25c8:1946
  EXAMPLE.COM.    86400   IN RRSIG AAAA 13 2 86400 (
                       20231027223840 20231007082139 37939 example.com.
                       IbunZHqZyc6umaDr0zsam9+hR/wSGm6BqHARrdCi58hf
                       Bi0tbzRFV+mOsdlUl8nyNqvfDg6tn/NIBgsUjg6vZg== )
~~~

### Query and Response for (EXAMPLE.COM, IN, ADDR)

~~~
  ; <<>> DiG 9.10.6 <<>> @A.IANA-SERVERS.NET EXAMPLE.COM ADDR
    +multiline +dnssec +norec
  ; (1 server found)
  ;; global options: +cmd
  ;; Got answer:
  ;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 28915
  ;; flags: qr aa; QUERY: 1, ANSWER: 2, AUTHORITY: 0, ADDITIONAL: 1
    
  ;; OPT PSEUDOSECTION:
  ; EDNS: version: 0, flags: do; udp: 4096
  ;; QUESTION SECTION:
  ;EXAMPLE.COM.		IN ADDR
    
  ;; ANSWER SECTION:
  EXAMPLE.COM.    86400   IN A 93.184.216.34
  EXAMPLE.COM.    86400   IN RRSIG A 13 2 86400 (
                       20231028144229 20231008002139 37939 example.com.
                       BfSk6wqpibs9a4AUa8gXHFzhBGiO9f0QuFLVaMYkPu0P
                       gKrDUl94VEfWlJ5pOMalgL1RzFwngd51b3FFiLUp3g== )
  EXAMPLE.COM.    86400   IN AAAA 2606:2800:220:1:248:1893:25c8:1946
  EXAMPLE.COM.    86400   IN RRSIG AAAA 13 2 86400 (
                       20231027223840 20231007082139 37939 example.com.
                       IbunZHqZyc6umaDr0zsam9+hR/wSGm6BqHARrdCi58hf
                       Bi0tbzRFV+mOsdlUl8nyNqvfDg6tn/NIBgsUjg6vZg== )
~~~

# Security Considerations

This specification describes a new meta QTYPE which causes DNS
responses to be generated that are larger than if QTYPE=A or
QTYPE=AAAA were used. This provides some amplification potential,
although it is modest and in a practical sense much less severe
than with QTYPE=ANY.

In the case of QTYPE=ANY the amplification potential was very large
without needing to contrive particular owner names in the DNS, e.g.
due to the common practice of stacking many RRSETS with significant
RDATA at the zone apex. Some operators choose to address these risks
by providing minimal responses to queries with QTYPE=ANY, as described
in {{?RFC8482}}.

In the case of QTYPE=ANY, only RRSETS of type A and AAAA are able
to contribute to the amplification. The size of the corresponding
RDATA for these RRs is fixed and relatively small. While it is not
unusual for multiple RRs of type A and AAAA to be associated with
a single owner name, in practice the total size of such RRSETS
is dramatically smaller than the zone apex example alluded to above.

A nameserver operator that is concerned about the amplification
potential of this specification can simply choose not to implement it.

# IANA Considerations

The IANA has allocated the code point TBA1 for QTYPE ADDR from the
range of available code-points designated for use by "Q and Meta-TYPEs"
{{!RFC6895}} and recorded it in the Resource Record (RR) TYPEs
sub-registry of the DNS Parameters registry as follows:

| TYPE  | Value  | Reference      | Template          | Registration Date  |
| ---   | ---    | ---            |                   |                    |
| TBA1  | ADDR   | This document  | See {{template}}  | As appropriate     |

The IANA has updated the "iana-dns-class-rr-type" YANG Module
to reflect this registration, as described in {{!RFC9108}}.

# Acknowledgements

This is not the first attempt to specify some kind of multi-QTYPEs
functionality. The bravery and foolish optimism of all those that
came before is hereby acknowledged.

An early implementation of this mechanism was implemented in
authoritative-only DNS servers by Olafur Gudmundsson, Christian
Elmerot and Pavel Odintsov  at Cloudflare starting around 2016,
using the private-use code-point 65535 for ADDR.

--- back

# RRTYPE Allocation Template {#template}

~~~
                 DNS RRTYPE PARAMETER ALLOCATION TEMPLATE
   
   When ready for formal consideration, this template is to be submitted
   to IANA for processing by emailing the template to dns-rrtype-
   applications@ietf.org.
   
   A. Submission Date: 2023-10-10
   
   B.1 Submission Type:  [X] New RRTYPE  [ ] Modification to RRTYPE
   B.2 Kind of RR:  [ ] Data RR  [X] Meta-RR
   
   C. Contact Information for submitter (will be publicly posted):
      Name: Joe Abley                  Email Address: jabley@cloudflare.com
      International telephone number: +31 6 45 56 36 34
      Other contact handles:
   
   D. Motivation for the new RRTYPE application.
      Please keep this part at a high level to inform the Expert and
      reviewers about uses of the RRTYPE.  Most reviewers will be DNS
      experts that may have limited knowledge of your application space.
   
      See draft-jabley-dnsop-addr-query-00 section 1.
   
   E. Description of the proposed RR type.
      This description can be provided in-line in the template, as an
      attachment, or with a publicly available URL.
   
      See draft-jabley-dnsop-addr-query-00 section 3.
   
   F. What existing RRTYPE or RRTYPEs come closest to filling that need
      and why are they unsatisfactory?
   
      The RRTYPEs A and AAAA provide a means of publishing IPv4
      and IPv6 addresses in the DNS. However, in order to obtain
      all IPv4 and IPv6 addresses for a particular owner name, two
      queries are required.  This specification allows the same
      functionality with a single query.
   
      The RRTYPE ANY (referred to as * in RFC 1035) provides a
      means of obtaining all RRTYPEs associated with a single owner
      name in a single query. However, queries with QTYPE=ANY
      provide unacceptable amplification potential for some operators
      and in many cases their use has been curtailed, e.g. see
      RFC 8482.
   
   G. What mnemonic is requested for the new RRTYPE (optional)?
   
      Note: If a mnemonic is not supplied, not allowed, or duplicates an
      existing RRTYPE or CLASS mnemonic, the Expert will assign a
      mnemonic.
   
      ADDR
   
   H. Does the requested RRTYPE make use of any existing IANA registry
      or require the creation of a new IANA subregistry in DNS
      Parameters?  If so, please indicate which registry is to be used
      or created.  If a new subregistry is needed, specify the
      allocation policy for it and its initial contents.  Also include
      what the modification procedures will be.
   
      No.
   
   I. Does the proposal require/expect any changes in DNS
      servers/resolvers that prevent the new type from being processed
      as an unknown RRTYPE (see [RFC3597])?
   
      No.
   
   J. Comments:
~~~

