---
title: "Requirements for HTTPS for Local Domains"
abbrev: "HTTPS for Local: Requirements"
category: info

docname: draft-nygren-requirements-https-local-domains-latest
submissiontype: IETF  # also: "independent", "editorial", "IAB", or "IRTF"
number:
date:
consensus: true
v: 3
# area: AREA
# workgroup: WG Working Group
keyword:
 - https
 - local domains
 - pki
 - pkix
 - certificate authority
 - tls
 - identity
 - authentication
venue:
  group: SETTLE
  type: ""
  mail: settle@ietf.org
  arch: https://mailarchive.ietf.org/arch/browse/settle/
  github: "danwing/requirements-https-local-domains"
  latest: "https://danwing.github.io/requirements-https-local-domains/draft-nygren-requirements-https-local-domains.html"

author:
 -
    fullname: Erik Nygren
    organization: Akamai Technologies
    email: erik+ietf@nygren.org
    uri: http://erik.nygren.org/
 -
    ins: D. Wing
    name: Dan Wing
    organization: Cloud Software Group Holdings, Inc.
    abbrev: Cloud Software Group
    email: danwing@gmail.com



normative:

informative:
  thomson-hld:
    title: "HTTPS for Local Domains"
    author:
      name: Martin Thomson
    date: September 2017
    target: https://docs.google.com/document/u/0/d/170rFC91jqvpFrKIqG4K8Vox8AL4LeQXzfikBQXYPmzU/edit

  tpac:
    title: "HTTPS for Local Networks"
    author:
      org: W3C
      name: Carlos IL
    date: September 2024
    target:  https://github.com/w3c/tpac2024-breakouts/issues/78

  phb-mesh:
     title: "Mathematical Mesh"
     author:
       org:
       name: Phillip Hallam-Baker
     date: 2022
     target: https://github.com/hallambaker/Mathematical-Mesh

  iotops-suib-prezo:
     title: "SUIB: Browsing local web resources in a secure usable manner"
     author:
       -
         name: Jan Geertsma
       -
         name: Christian Amsüss
       -
         name: Micheal Richardson
       -
         name: Nick Allott
     date: November 2021
     ann: "Presentation of IOT Security Foundation SUIB to IETF112 IOTOPS working group"
     target: https://datatracker.ietf.org/meeting/112/materials/slides-112-iotops-suib-browsing-local-web-resources-in-a-secure-usable-manner-iot-device-configuration-as-a-special-case-00.pdf

  iotops-suib:
     title: "SUIB: Router and IoT Vulnerabilities: Insecure by Design"
     author:
       org: IOT Security Foundation
     date: August 2021
     target: https://iotsecurityfoundation.org/wp-content/uploads/2021/08/ManySecured-SUIB-White-Paper.pdf

  iotsf:
     title: "IOT Security Foundation"
     date: September 2015
     target: https://iotsecurityfoundation.org

  stark:
     title: "When a web PKI certificate won't cut it"
     date: December 2021
     author:
       name: Emily M. Stark
     target: https://emilymstark.com/2021/12/24/when-a-web-pki-certificate-wont-cut-it.html

  shared:
     title: "APPROACH-2: Using Shared Secret"
     date: September 2019
     author:
       org: W3C
     target: https://httpslocal.github.io/proposals/#approach-2

  plex:
     title: How Plex Is Doing Https for All Its Users
     date: June 2015
     author:
       name: Filippo Valsorda
     target: https://words.filippo.io/how-plex-is-doing-https-for-all-its-users/

  w3c-httpslocal:
     title: "HTTPS in Local Network Community Group"
     date: 2019
     author:
       org: W3C
     target: https://github.com/httpslocal

  w3c-pna:
     title: Private Network Access
     date: September 2024
     author:
       org: W3C
     target: https://wicg.github.io/private-network-access/

  sec-context:
     title: "Secure Contexts"
     date: 2023
     author:
       org: W3C
     target: https://w3c.github.io/webappsec-secure-contexts/

  not-secure:
     title: "A secure web is here to stay"
     date: 2018
     author:
       org: Google
     target: https://blog.chromium.org/2018/02/a-secure-web-is-here-to-stay.html

--- abstract

When connecting to servers on their local network, users are
surprised to encounter user interfaces that display errors,
show insecure connections, and block some HTTP features
when missing a secure context.  However, obtaining PKIX
certificates for those servers is difficult for a variety
of reasons.

This document explores requirements for authenticating local servers
without relying on PKIX certificates.

--- middle

# Introduction

Servers on local networks have historically used and encouraged
unencrypted communications -- printers, routers, network attached
storage (NAS).  However, browsers disadvantage unencrypted
communications (e.g., {{not-secure}}, {{sec-context}}) which increases
importance of a secure context (HTTPS) to local domains.  Today, a
secure context is obtained with a PKIX certificate ({{?RFC5280}})
signed by a Certification Authority (CA) that is trusted by the client.

However, servers on a local network cannot easily get PKIX
certificates signed by a Certification Authority because of their
firewall or NAT (to prove domain ownership), lack of domain name
delegation, and need for ongoing certificate renewal.

The problem has been well recognized since about 2017 and several
proposals have been suggested to solve this problem, each with their
own drawbacks.  This document is not intended to summarize these proposals
or their drawbacks; for that detail see the pointers to previous work
in {{related}}.  At a high level, the proposals have involved
solutions such as:

  * pre-shared secrets (scanned, printed, or displayed by the server)

  * Public DNS pointing at local domain's IP address (e.g., {{plex}})

  * Local Certification Authority, where a CA is
    added to client's certificate trust list and that CA signs
    certificates for devices within the local network

  * Trust On First Use (TOFU), where a user verifies the first
    connection to a server and the client remembers that verification,
    similar to common use of ssh

  * WebRTC and WebTransport, where a PKI-signed server provides a
    public key fingerprint of another server that it has previously
    bootstrapped

  * Encoding server's public key into the hostname {{thomson-hld}}

This document explores IETF requirements for an alternative server
authentication system for local hosts.

# Conventions and Definitions

{::boilerplate bcp14-tagged}

# Technical Requirements

## Naming

PKIX certificates are a centralized naming scheme derived from DNS.
These names have (the possibility of) being human-readable names.  But
the most significant property is uniqueness -- each name has its own
identity and that identity can be proven.

A system that does not rely on centralized naming lacks this inherient
uniqueness property.

Without a centralized naming scheme, name collisions are possible and
likely.  For example, it is likely that many networks will have a
printer named, simply, "printer", much like many people might share a
common name such as "John".  Humans prefer simple, human-readable
names, but a strong identity cannot be created with such names: if
two networks both have a printer named "printer", they are indistinguishable
and if one responds when the other was expected, the mismatch will
appear identical to an attack.  This would be unacceptable.

> R-UNIQUE-NAME: The system MUST have a way to uniquely identify
  servers.


## Cryptographic Binding

A server's name has to be mapped to its cryptographic identity.

> R-BINDING: The Web Origin MUST be cryptographically bound to one or
  more key pairs, where the private keying material is on the service
  endpoint and where an attacker without the private key(s) is unable
  to access any state associated with the Web Origin.

A client has to be able to validate the name maps to the cryptographic
identity.

> R-VALIDATE: Clients MUST be able to cryptographically validate that
  the authenticating server matches the identity in the URI / Web
  Origin.

Web browsers and modern users both expect a URI.

> R-URI: It MUST be possible to construct a URI that encapsulates a
  Web Origin and its cryptographically-bound identity information.


## Abstract Naming

Using IP addresses in names is problematic if the server's IP address
changes due to ISP renumbering or internal network DHCP server
reconfiguration.

> R-ABSTRACT: The solution SHOULD abstract names from IP addresses.

Any given name should be able to have a mixture of IPv4, IPv6
Link-Local (on an Interface), IPv6 ULA, and IPv6 Globally-Routable
addresses. Operating a local DNS is beyond the scope of many
administrators, so being able to advertise the server using
{{?DNS-SD=RFC6763}} is necessary.

> R-DNS-SD: The name MUST be advertisable using {{?DNS-SD=RFC6763}}



## Avoid Central Authority

A solution needs to be self-contained and not use the central
authority of PKIX.

> R-AVOID-CENTRAL: A solution SHOULD NOT (MUST NOT?) rely on central
  trust hierarchy.

Vendors go out of business or lose interest in continuing to service
old products. The products may still be operational.

> R-AVOID-VENDOR: A solution SHOULD NOT (MUST NOT?) have continued
  reliance on a service operated by a vendor, including if the device
  is reset to factory defaults (e.g., reset for troubleshotting or
  because sold).


## Multiple Application Protocols

> R-MULT-APP: A solution MUST support various application-level
  protocols such as HTTPS, IPPS, DoH, DoT, SMB over QUIC, and so on.


## Cryptographic Agility

> R-AGILITY: A solution SHOULD support crypto agility (such as
  supporting more than one active key type).

## TLS Server Name Indication

> R-TLS-SNI: A solution SHOULD support TLS SNI so a server knows which
key pair/cert is expected.

## Localhost

> R-LOCALHOST: A solution SHOULD support localhost uses as well (e.g.,
for sending a user to connect to a local service)


## W3C Private Network Access

> R-PNA: A solution SHOULD integrate well with an evolution of
{{w3c-pna}} and both allow for an improved model there but should also
provide more robust solutions to vulnerabilities that it tries to
address

## Constrain to Local Resources

> R-LOCAL: A solution SHOULD be constrained to .local and .internal.

> Discuss: MAY constrain to the DHCP domain-search value??  Should we
also allow any arbitrary name if the IP address is local (RFC1918
address), too?


## Technical Requirements

1. It SHOULD be possible to have a way to represent a URI that
includes a single specific IP address and the cryptographic identity
of the service endpoint.

> Discuss: the above requirement needs to be re-written.


1. SHOULD support key rotation (even if via 301 redirect) — Q: is it
acceptable to state to be lost here?  Note: likely can’t do 301 if
doing TLS (HTTPS).  Is this suggestion to start HTTP and upgrade to
HTTPS?  Could be useful for HTTPS but redirect unavailable for IPP,
SMB, DoH.

> Discuss: the above requirement needs to be re-written.


1. SHOULD support building trust relationships within devices in the
local environment

> Discuss: the above requirement needs to be re-written.


1. Could this help with HTTPS access to Wi-Fi login portals
({{?RFC8952}}, {{?RFC8910}})?

> Discuss: the above requirement needs to be re-written.

# Human Factors Requirements


## Discoverable

> R-DISCOVER: A solution SHOULD have a way to do discovery of
endpoints and their identities (for example, via {{?DNS-SD=RFC6763}}).


## Easy to Use

> R-EASY: A solution SHOULD have human factors and adversarial testing
on proposed solutions to make sure that this solution provides a
reasonable experience to average and novice end-users and does not
introduce new security exploitation vectors

## Bookmarkable

> R-BOOKMARK: A solution SHOULD have a URI that users can Bookmark to create an association
to a friendly name.

> Discussion: Can URL bar of the browser honor mDNS/DNSSD advertised
names, or give a pull-down of them similar to how the “add printer”
dialog does for printers?  This would help ease the use of long FQDN
so it’s almost as easy as router.local.  Especially if it could show a
nickname that is configured by the printer.

## Human-friendly Name

> R-CONSISTENT: A solution SHOULD represent these URIs to humans in a
consistent, readable, and non-confusing fashion.  (In a browser,
users shouldn’t see the key fingerprint by default but rather a
representation of its presence)


# Big Open Questions

## Key Rotation

1. Is it acceptable for the Web Origin to change as part of key rotations?
A: no, this doesn’t happen today and changing the web origin would violate the principle of least surprise.

## Trust on First Use (TOFU)

Is TOFU acceptable?

> Note: TOFU is arguably what we have today with self-signed certificates
     which can be trusted (after the user accepts the warning message and
     adds the certificate to their client's trust store).


## User Experience

For a solution, what is the User Experience for any trust relationship / web-of-trust?

## Trust Relationship

For a solution, what is the nature of the trust relationship?

   * Peer trust web?

   * Central CA within the local environment / trust clearing house?

   * Client establishes its own trust to the server

## Interaction with Matter/Thread

How does a solution tie into systems like Matter/Thread that have their own trust establishment frameworks?



# Use Cases

For the below, “Secure communications” means being able to make a TLS
connection to a service such that the service is able to authenticate
itself in a way to prevent MitM attacks.  The security model must be
TOFU at a minimum, but when the identity of a service is none it
should be possible to send it as a URI in such as a way to present a
secure association rooted in the connection that sent it:

* Secure communications via HTTPS to admin interfaces on CPEs for
both initial and ongoing configuration tasks of various servers
(router, printer, NAS, etc.).

* Secure communications to DoH/DoT servers on CPEs

* Secure communications to printers (IPPS {{?RFC7472}} printing)

* Secure communications to other local services (SMB over QUIC to
  another workstation or a NAS) and IoT devices

* Secure communications to localhost processes from a browser (e.g.,
  admin tools)


# Related {#related}

Martin Thomson wrote a document on HTTPS for Local Domains which covers requirements,
discusses several solutions and their tradeoffs, and suggests a solution with hostnames
encoding the server's public key {{thomson-hld}} in November 2017.

W3C worked on this problem from 2017 through 2021 {{w3c-httpslocal}}. More recently,
W3C had a workshop on the problem in September 2024 {{tpac}}.

The boundaries of a limited domain -- such as the local domain described in this
document -- are explored in {{Section 6 of ?RFC8799}}.

The IOTOPS working group and the associated IOT Security Foundation
{{iotsf}} discussed the problem and some requirements in their white
paper {{iotops-suib}} and presentation to IOTOPS working group at
IETF112 {{iotops-suib-prezo}}.

A threshold key system is described and implemented at {{phb-mesh}} with the following
description:

> The Mesh is designed to provide users with the highest level of
  security that is possible without asking them to do anything at
  all. For this to become possible, the Mesh will have to be shipped
  to users as part of the machine Operating System.

A summary of the problem and analysis of several solutions
(Locally-installed CAs, Plex, WebRTC and WebTransport, TOFU, shared
secrets) and some drawbacks of those solutions is at {{stark}}.

A method using PAKE and a shared secret (displayed on the server) is
explained at {{shared}}.

# Security Considerations

TODO Security


# IANA Considerations

This document has no IANA actions.


--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
