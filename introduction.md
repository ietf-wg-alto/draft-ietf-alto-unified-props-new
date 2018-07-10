# Introduction

The ALTO protocol [](#RFC7285) introduces the concept of `properties` attached
to `endpoint addresses`, and defines the Endpoint Property Service (EPS) to
allow ALTO clients to retrieve those properties. While useful, the EPS, as defined
in [](#RFC7285), has at least two limitations.

First, it only allows properties to be associated with a particular domain of
entities, namely individual IP addresses. It is reasonable to think that
collections of endpoints, as defined by CIDRs [](#RFC4632) or PIDs, may also
have properties.
<!-- Furthermore, a recent proposal
[](#I-D.ietf-alto-path-vector) has suggested new classes of entities
(ANE) with properties. -->
Since the EPS cannot be extended to new entity domains,
new services, with new request and response messages, would have to be
defined for new entity domains.

Second, the EPS is only defined as a POST-mode service. Clients must request the
properties for an explicit set of endpoint addresses. By contrast, [](#RFC7285) defines a
GET-mode Cost Map resource which returns all available costs, so a client can
get a full set of costs once, and then processes costs lookups without querying
the ALTO server. [](#RFC7285) does not define an equivalent service for endpoint
properties. At first a map of endpoint properties might seem impractical, because it could require
enumerating the property value for every possible endpoint. But in practice, it
is highly unlikely that properties will be defined for every endpoint address. It is much
more likely that properties will only be defined for a subset of endpoint addresses, and
that subset would be small enough to be enumerated. This is particularly true if
blocks of endpoint addresses with a common prefix (e.g., a CIDR) have the same value for
a property. Furthermore, entities in other domains may very well be enumerable.

This document proposes a new approach to retrieve ALTO properties.
Specifically, it defines two new types of resources, namely Property Maps (see
[](#prop-map)) and Filtered Property Maps (see [](#filter-prop-map)). The
former are GET-mode resources which return the property values for all entities
in a domain, and are analogous to the ALTO's Network Maps and Cost Maps. The
latter are POST-mode resources which return the values for a set of properties
and entities requested by the client, and are analogous to the ALTO's Filtered
Network Maps and Filtered Cost Maps.

Additionally, this document introduces ALTO Entity Domains, where entities
extend the concept of endpoints to objects that may be endpoints as defined in
[](#RFC7285) but also, for example, PIDs, Abstract Network Elements as defined
in [](#I-D.ietf-alto-path-vector) or cells. As a consequence, ALTO Entity
Domains are a super-set of ALTO Address Types and their relation is specified in
[](#consistency-procedure).

<!-- FIXME: Although it is understandable to clarify the property map can
represent generic entity domains, it is still not a good idea to introduce ANEs
and cells here. Because it may conduct dependency loop. -->

Entity domains and property names are extensible. New entity domains can be
defined without revising the messages defined in this document, in the same way
that new cost metrics and new endpoint properties can be defined without
revising the messages defined in [](#RFC7285).

This proposal would subsume the Endpoint Property Service defined in [](#RFC7285),
although that service may be retained for legacy clients (see [](#legacy)).
