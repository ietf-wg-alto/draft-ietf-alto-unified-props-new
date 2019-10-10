# Overview: Basic Concepts

Before we define the specification of unified properties, there are several basic concepts which we need to introduce.

<!--
~~~
                                                          (Define)
      +----------+                        +----------+   +-------------+
    ->| Property |<-----------------------| Internal |---| asn  | load |
   /  |   Map 1  |                        |   Map    |   |-------------|
  /   +----------+                        +----------+   | 1234 | 95%  |
 |         ^                                    |        | 5678 | 70%  |
 |         |                                     \       +-------------+
 |         |          (Export)                    \       (Extend)
 |    +---------+    +------------------------+    \     +--------------+
 |    | Network |----| ipv4           | pid   |     -----| geo-location |
 |    |  Map 1  |    |------------------------|          +--------------+
 |    +---------+    | 192.168.0.0/24 | pid1  | - - - -> | New York     |
 |                   | 192.168.1.0/24 | pid2  | - - - -> | Shanghai     |
 |                   +------------------------+          +--------------+
 |                    (Export)
  \   +---------+    +------------------------+
   ---| Network |----| ipv4           | pid   |
      |  Map 2  |    |------------------------|
      +---------+    | 192.168.0.0/24 | Paris |
                     | ...            | ...   |
                     +------------------------+
~~~
-->

<!-- Entity -> Property -> Resource -> Entity Domain -> Aggreated Entity Domain -->

## Entity {#con-entity}

<!-- FIXME:
This section introduces a new concept and deserves more explanations before
diving in the technical "how to handle".
For instance:
- what kind of object other than an addressable ALTO endpoint is eligible to be
  an ALTO entity?
- an entity must be related to network address(es) e.g. entity such as PID, ASN
  nb, Country code have a defined mapping to IP or cellular addresses.
-->

The entity concept generalizes the concept of the endpoint defined in Section
2.1 of [](#RFC7285). An entity is an object that can be an endpoint and is
identified by its network address, but can also be an object that has a defined
mapping to a set of one or more network addresses or is even not related to any
network address.

<!-- Examples of entities -->

Examples of eligible entities are:

- a PID, defined in [](#RFC7285), that has a provider defined human-readable
  abstract identifier defined by an ALTO network map, which maps a PID to a set
  of ipv4 and ipv6 addresses;
- an autonomous system (AS), that has an AS number (ASN) as its identifier
  and maps to a set of ipv4 and ipv6 addresses;
- a region representing a country, that is identified by its country code defined
  by ISO 3166 and maps to a set of cellular addresses;
- a TCP/IP network flow, that has a server defined identifier consisting of the
  defining TCP/IP 5-Tuple, which is an example that all endpoints
  are entities while not all entities are endpoints;
- a routing element, that is specified in [](#RFC7921) and includes routing
  capability information;
- an abstract network element, that has a server defined identifier and
  represents a network node, link or their aggregation.

<!--
An entity is an object with a (possibly empty) set of properties.
It MAY or MAY NOT be related to a network address. There are a lot of examples
of entities, such as applications or end-hosts in a communication network, cells
in a wireless network, network flows, network functions, routing elements in a
routing system or even general network elements.
-->

<!--
Each entity MUST be in one and only one domain, such as the IPv4 domain, the
IPv6 domain or PID domain (defined in this document), and has a unique identifier.
-->

## Entity Property {#con-property}

An entity property defines a property of an entity. It is similar to the
endpoint property defined by Section 7.1 of [](#RFC7285), but can be general
besides network-aware.

For example,

- an `ipv4` entity may have a property whose value is an Autonomous System (AS)
  number indicating the AS which this IPv4 address is owned by;
- a `pid` entity may have a property which indicates the central geographical
  location of endpoints included by it.

## Property Map {#con-propmap}

An ALTO property map provides a set of properties for a set of entities. These
entities may be in different types. For example, an ALTO property map may define
the ASN property for both `ipv4` and `ipv6` entities.

## Information Resource {#con-resource}

This document uses the same definition of the information resource as defined by
[](#RFC7285). Each information resource usually has a JSON format representation
following a specific schema defined by its media type.

For example, an ALTO network map resource is represented by a JSON object of type
InfoResourceNetworkMap defined by the media type
`application/alto-networkmap+json`.

## Entity Domain {#con-entity-domain}

An entity domain defines a set of entities in the same type. This type is also
called the type of this entity domain.

Using entity domains, an ALTO property map can indicate which entities the ALTO
client can query to get their properties.

### Resource-Specific Entity Domain

To define an entity domain, one naive solution is to enumerate all entities in
this entity domain. However, it is inefficient when the size of the entity domain
is large.

To avoid enumerating all entities, this document introduces an approach called
`Resource-Specific Entity Domain` to define entity domains:

Each information resource may define several types of entity domains. Also, for
each type of entity domain, an information resource can define at most one
entity domain. For example, an ALTO network map resource can define an IPv4
domain, an IPv6 domain, and a pid domain. In this document, these entity domains
are called resource-specific entity domains. An ALTO property map only needs to
indicate which types of entity domain defined by which information resources can
be queried, the ALTO client will know which entities are effective to be queried.

### Relationship between Entity and Entity Domain

In this document, an entity is owned by exact one entity domain. It requires
that when an ALTO client or server references an entity, it must indicate its
entity domain explicitly. Even two entities in two different entity domains may
reflect to the same physical or logical object, we treat them as different
entities.

Because of this rule, although the resource-specific entity domain approach has
no ambiguity, it may introduce redundancy.

### Aggregated Entity Domain

Two entities in two different resource-specific entity domains may reflect to
the same physical or logical object. For example, the IPv4 entity `192.0.2.34`
in the IPv4 domain of the network map `netmap1` and the IPv4 entity `192.0.2.34`
in the IPv4 domain of the network map `netmap2` should indicate the same
Internet endpoint addressed by the IPv4 address `192.0.2.34`.

Each entity in each resource-specific entity domain may only have part of
properties of its associated physical or logical object. For example, the IPv4
entity in the IPv4 domain of the network map `netmap1` only has the PID property
defined by `netmap1`; same to the IPv4 entity in the IPv4 domain of the network
map `netmap2`. If the ALTO client wants to get the complete properties, using
the resource-specific entity domain, the ALTO client has to query the IPv4
entity `192.0.2.34` twice.

To simplify the query process of the ALTO client, this document introduces the
concept of `Aggregated Entity Domain`. An aggregated entity domain defines a union
set of entities coming from multiple resource-specific entity domains in the
same type. An entity in the aggregated entity domain inherits all properties
defined for its associated entity in each associated resource-specific entity
domains. For example, the IPv4 entity `192.0.2.34` in the aggregated entity
domain between the IPv4 domain of `netmap1` and the IPv4 domain of `netmap2` has
PID properties defined by both `netmap1` and `netmap2`.

Note that some resource-specific entity domains may not be able to be aggregated
even if they are in the same type. For example, a property map `propmap1` may
define the `asn` property on both PID domains `netmap1.pid` and `netmap2.pid`.
But the PID `pid1` in `netmap1.pid` and the PID with the same name in
`netmap2.pid` have different `asn` property values. It does not make sense to
define an aggregated PID domain between `netmap1.pid` and `netmap2.pid` to
provide the `propmap1.asn` property because it is ambiguous.

### Resource-Specific Entity Property

According to the example of the aggregated entity domain, an entity may have
multiple properties in the same type but associated to different information
resources. To distinguish them, this document uses the same approach proposed by
Section 10.8.1 of [](#RFC7285), which is called `Resource-Specific Entity Property`.

## Scope of Property Map

Using entity domains to organize entities, an ALTO property map resource
can be regarded as given sets of properties for given entity domains. If we ignore the
syntax sugar of the aggregated entity domain, we can regard an ALTO property
map resource as a set of (ri, di) => (ro, po) mappings, where (ri,
di) means a resource-specific entity domain of type di defined by the
information resource ri, and (ro, po) means a resource-specific entity property
po defined by the information resource ro.

For each (ri, di) => (ro, po) mapping, the scope of an ALTO property map
resource must be one of the cases in the following diagram:

``` text
                    domain.resource   domain.resource
                    (ri) = r          (ri) = this
                  +-----------------|-----------------+
    prop.resource | Export          | Non-exist       |
    (ro) = r      |                 |                 |
                  +-----------------|-----------------+
    prop.resource | Extend          | Define          |
    (ro) = this   |                 |                 |
                  +-----------------|-----------------+
```

where `this` represents the resulting property map resource, and `r` represents an
existing ALTO information resource other the resulting property map resource.

- ri = ro = r (`export` mode): the property map resource just transforms the
  property mapping di => po defined by r into the unified representation format
  and exports it. For example: r = `netmap1`, di = `ipv4`, po = `pid`. The
  property map resource exports the `ipv4 => pid` mapping defined by `netmap1`.
- ri = r, ro = this (`extend` mode): the property map extends properties of
  entities in the entity domain (r, di) and defines a new property po on them.
  For example: the property map resource (`this`) defines a `geolocation`
  property on domain `netmap1.pid`.
- ri = ro = this (`define` mode): the property map defines a new intrinsic
  entity domain and defines property po for each entity in this domain. For
  example: the property map resource (`this`) defines a new entity domain `asn`
  and defines a property `ipprefixes` on this domain.
- ri = this, ro = r: in the scope of a property map resource, it does not make
  sense that another existing ALTO information resource defines a property for
  this property map resource.

## Entity Hierarchy and Property Inheritance {#con-hierarchy-and-inheritance}

Enumerating all individual effective entities are inefficient. Some types of
entities have the hierarchy format, e.g., cidr, which stand for sets of
individual entities. Many entities in the same hierarchical format entity sets
may have the same proprety values. To reduce the size of the property map
representation, this document introduces an approach called `Property
Inheritance`. Individual entities can inherit the property from its hierarchical
format entity set.
