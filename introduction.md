
# Introduction

The ALTO protocol {{RFC7285}} introduces the concept of "properties" attached
to "endpoint addresses", and defines the Endpoint Property Service (EPS) to
allow ALTO clients to retrieve those properties. While useful, the EPS, as
defined in {{RFC7285}}, has at least three limitations.

First, the EPS allows properties to be associated with only endpoints that
are identified by individual communication addresses like IPv4 and IPv6
addresses. It is reasonable to think that collections of endpoints, as
defined by CIDRs {{RFC4632}} or PIDs, may also have properties. Furthermore,
recent ALTO use cases show that properties of entities such as network flows
{{RFC7011}} and routing elements {{RFC7921}} are also useful. Such cases are
documented in {{I-D.gao-alto-fcs}}. The current EPS however is restricted to
individual endpoints and cannot be applied to those entities.

Second, the EPS only allows endpoints identified by global communication
addresses. However, an endpoint address may be a local IP address or an
anycast IP address that may not be globally unique. Additionally, an entity
such as a PID may have an identifier that is not globally unique. That is, a
same PID identifier may be used in multiple network maps, while in each
network map, this PID identifier points to a different set of addresses. For
example, PID "mypid10" may be defined in "netmap1" and "netmap2" while in
each network map, "mypid10" covers a different set of addresses.

Third, the EPS is only defined as a POST-mode service. Clients must request
the properties for an explicit set of endpoint addresses. By contrast,
{{RFC7285}} defines a GET-mode cost map resource which returns all available
costs, so a client can get a full set of costs once, and then process cost
lookups without querying the ALTO server. {{RFC7285}} does not define a
similar service for endpoint properties. At first, a map of endpoint
properties might seem impractical, because it could require enumerating the
property value for every possible endpoint. However, in practice, it is
highly unlikely that properties will be defined for every endpoint address.
It is much more likely that properties may be defined for only a subset of
endpoint addresses, and the specification of properties uses an aggregation
representation to allow enumeration. This is particularly true if blocks of
endpoint addresses with a common prefix (e.g., a CIDR) have the same value
for a property. Entities in other domains may very well allow aggregated
representation and hence be enumerable as well.

To address the three limitations, this document specifies a protocol
extension for defining and retrieving ALTO properties:

* The first limitation is addressed by introducing a generic
  concept called ALTO Entity, which generalizes an endpoint and may represent
  a PID, a network element, a cell in a cellular network, an abstracted
  network element as defined in {{I-D.ietf-alto-path-vector}}, or other
  physical or logical objects involved in a network topology. Each entity is
  included in a collection called an ALTO Entity Domain. Since each ALTO
  Entity Domain includes only one type of entities, each Entity Domain can be
  classified by the type of entities in it.
* The second limitation is addressed by using resource-specific
  entity domains. A resource-specific entity domain contains entities that
  are defined and identified with respect to a given ALTO information
  resource, which provides scoping. For example, an entity domain containing
  PIDs is identified with respect to the network map in which these PIDs are
  defined. Likewise, an entity domain containing local IP addresses may be
  defined with respect to a local network map.
* The third limitation is addressed by defining two new types of ALTO
  information resources: Property Map, detailed in [](#prop-map) and Filtered
  Property Map, detailed in [](#filter-prop-map). The former is a GET-mode
  resource that returns the property values for all entities in one or more
  entity domains, and is analogous to a network map or a cost map in
  {{RFC7285}}. The latter is a POST-mode resource that returns the values for
  sets of properties and entities requested by the client, and is analogous
  to a filtered network map or a filtered cost map.

The protocol extension defined in this document is augmentable. New entity
domain types can be defined without revising the specification defined in
this document. Similarly, new cost metrics and new endpoint properties can be
defined in other documents without revising the protocol specification
defined in {{RFC7285}}.

## Terminology

This document uses the following terms and abbreviations, that will be
further defined in the document. While this document introduces the feature
"entity property map", it will use both the term "property map" and "entity
property map" to refer to this feature.

* Transaction: A request/response exchange between an ALTO Client and an ALTO Server.
* Client: When used with a capital "C", this term refers to an ALTO Client.
* Server: When used with a capital "S", this term refers to an ALTO Server.
* EPM: An abbreviation for entity property map.
* FPM: An abbreviation for filtered property map.
* EPS: An abbreviation for Endpoint Property Service.

# Requirements Language

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD",
"SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this
document are to be interpreted as described in BCP 14 {{RFC2119}} {{RFC8174}}
when, and only when, they appear in all capitals, as shown here. When the
words appear in lower case, they are to be interpreted with their natural
language meanings.
