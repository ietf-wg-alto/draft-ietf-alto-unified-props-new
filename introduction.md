# Introduction

The ALTO protocol [](#RFC7285) introduces the concept of `properties` attached
to `endpoint addresses`, and defines the Endpoint Property Service (EPS) to
allow ALTO clients to retrieve those properties. While useful, the EPS, as defined
in [](#RFC7285), has at least three limitations.

<!-- FIXME: revise the requirements -->

First, the EPS allows properties to be associated with only endpoints which are
identified by individual communication addresses like IPv4 and IPv6 addresses.
It is reasonable to think that collections of endpoints, as defined by CIDRs
[](#RFC4632) or PIDs, may also have properties. Furthermore, recent ALTO use
cases show that properties of network flows [](#RFC7011) and routing elements
[](#RFC7921) are also very useful. Since the EPS cannot be extended to those
generic entities, new services, with new request and response messages, would
have to be defined for them.

<!--
First, it allows properties to be associated with only a particular domain of
entities, namely individual IP addresses. It is reasonable to think that
collections of endpoints, as defined by CIDRs [](#RFC4632) or PIDs, may also
have properties.
Furthermore, a recent proposal
[](#I-D.ietf-alto-path-vector) has suggested new classes of entities
(ANE) with properties.
Since the EPS cannot be extended to new entity domains,
new services, with new request and response messages, would have to be
defined for new entity domains.
-->

Second, the EPS only allows endpoints identified by global communication
addresses. However, an endpoint address may be a local IP address or an anycast
IP address which is also not globally unique. Additionally, a generic entity
such as a PID may have an identifier that is not globally unique. For example, a
PID identifier may be used in several network maps, where in each network map,
this PID identifier points to a different set of addresses.

Third, the EPS is only defined as a POST-mode service. Clients must request the
properties for an explicit set of endpoint addresses. By contrast, [](#RFC7285)
defines a GET-mode cost map resource which returns all available costs, so a
client can get a full set of costs once, and then processes costs lookups
without querying the ALTO server. [](#RFC7285) does not define a similar service
for endpoint properties. At first, a map of endpoint properties might seem
impractical, because it could require enumerating the property value for every
possible endpoint. However, in practice, it is highly unlikely that properties will
be defined for every endpoint address. It is much more likely that properties
may be defined for only a subset of endpoint addresses, and the specification of
properties uses an aggregation representation to allow enumeration. This is
particularly true if blocks of endpoint addresses with a common prefix (e.g., a
CIDR) have the same value for a property. Entities in other domains may very
well allow aggregated representation and hence be enumerable as well.

<!-- FIXME: revise the design overview -->

To address the three limitations, this document specifies a protocol extension
for defining and retrieving ALTO properties:

- The first limitation is adressed by introducing a generic concept called ALTO
  Entity, which generalizes an endpoint and may represent a PID, a network
  element, a cell in a cellular network, an abstracted network element as
  defined in [REF path-vector], or other physical or logical objects used by
  ALTO. Each entity is included in a collection called ALTO Entity Domain. Also,
  each Entity Domain includes only one type of entities. Thus, each entity
  domain has a type.

- The second limitation is addressed by using resource-specific entity domains.
  A resource-specific entity domain contains entities defined and identified
  with respect to a given ALTO information resource. For example, an entity
  domain containing PIDs is identified w.r.t. the network map in which these
  PIDs are defined. Likewise an entity domain containing local IP addresses may
  be defined w.r.t. a local network.

- Finally, the third limitation is addressed by defining two new types of ALTO
  information resources: Property Map, detailed in [](#prop-map) and Filtered
  Property Map, detailed in [](#filter-prop-map). The former is a GET-mode
  resource which returns the property values for all entities in some entity
  domains, and is analogous to a network map or a cost map in [](#RFC7285). The
  latter is a POST-mode resource which returns the values for a set of
  properties and entities requested by the client, and is analogous to a
  filtered network map or a filtered cost map.

This approach is extensible, because new entity domain types can be defined
without revising the protocol specification defined in this document. In the
same way, new cost metrics and new endpoint properties can be defined
without revising the protocol specification defined in [](#RFC7285).

This document subsumes the Endpoint Property Service defined in [](#RFC7285),
although that service may be retained for legacy clients (see [](#legacy)).
