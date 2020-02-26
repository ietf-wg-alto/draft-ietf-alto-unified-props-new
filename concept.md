# Basic Features of the Unified Property Extension

The purpose of this extension is to convey properties on objects that extend
ALTO Endpoints and are called ALTO Entities, entities for short. This section
introduces the basic features involved in ALTO Property Maps.

<!-- Entity -> Property -> Resource -> Entity Domain -> Aggreated Entity Domain -->

<!--
## Information Resource {#con-resource}

This document uses the same definition of the information resource as defined by
[](#RFC7285). Each information resource usually has a JSON format representation
following a specific schema defined by its media type.

For example, an ALTO network map resource is represented by a JSON object of type
InfoResourceNetworkMap defined by the media type
`application/alto-networkmap+json`.
-->

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

The concept of ALTO Entity generalizes the concept of an ALTO Endpoint defined
in Section 2.1 of [](#RFC7285). An entity is an object that can be an endpoint
and is identified by its network address, but can also be an object that has a
defined mapping to a set of one or more network addresses or is even not related
to any network address.

<!-- Examples of entities -->

Examples of eligible entities are:

- a PID, defined in [](#RFC7285), that has a provider defined human-readable
  abstract identifier specified by an ALTO network map, which maps a PID to a set
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
  represents a routable network node, a link, a network domain or their aggregation.

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

## Entity Domain {#con-entity-domain}

An entity domain defines a set of entities of the same type. This type is also
called the type of this entity domain. Thus, an entity domain type defines the
type semantics and the identifier format of its entities. An entity domain also
has a name. The name and type of an entity domain could be the same.
Example of such entity domains are: "ipv4", "pid", which are defined in
[](#inet-addr-domain) and [](#pid-domain).

Using entity domains, the ALTO property map capabilities indicate on which
entity domains an ALTO client can query properties.

## Entity Property {#con-property}

An entity property defines a property of an entity. It is similar to the
endpoint property defined by Section 7.1 of [](#RFC7285). It can convey
either network-aware or network-agnostic information.

For example:

- an entity in the `ipv4` domain may have a property whose value is an
  Autonomous System (AS) number indicating the AS that owns this IPv4 address,

- an entity in the `pid` domain may have a property that indicates the central
  geographical location of endpoints it includes.


It should be noted that some objects may be both entities and properties. For
example, a PID may be both a property of an "ipv4" entity and an entity on which
a Client may query properties such as geographical location.

## New information resource and media type: ALTO Property Map {#con-propmap}

This document introduces a new ALTO information resource named
Property Map. An ALTO property map provides a set of properties on a set of
entities. These entities may be of different types. For example, an ALTO
property map may define the ASN property for both "ipv4" and "ipv6" type of
entities.

The present extension also introduces a new media type.

This document uses the same definition of the information resource as defined by
[](#RFC7285). Each information resource usually has a JSON format
representation following a specific schema defined by its media type. In the
present case, an ALTO property map resource is represented by a JSON object of
type InfoResourcePropertyMap and defined by the media type
`application/alto-propmap+json`.

A Property Map can be queried as a GET-mode resource, thus conveying values of
all properties on all entities indicated in its capabilities. It can also be
queried as a POST-mode resource, thus conveying a selection of properties on a
selection of entities.

<!--
An ALTO property map provides a set of properties for a set of entities. These
entities may be in different types. For example, an ALTO property map may define
the ASN property for both `ipv4` and `ipv6` entities.
-->

# Advanced Features of the Unified Property Extension

## Entity Identifier and Entity Domain

In [](#RFC7285), an endpoint has an identifier explicitly associated with the
"ipv4" or "ipv6" address domain. Examples are "ipv4:192.0.2.14" and
"ipv6:2001:db8::12". In this extension, an entity domain characterizes the
type semantics and identifier format of its entities. And the identifier of
an entity is explicitly associated with its entity domain. For instance: an
entity that is an endpoint with an IPv4 address will have an identifier
associated with the "ipv4" domain, like "ipv4:192.0.2.14"; an entity which is a
PID will have an identifier associated with a "pid" domain, like "pid:mypid10".
      
In this document, an entity must be owned by exactly one entity domain. And an
entity identifier must point to exactly one entity. Even if two entities in two
different entity domains refer to the same physical or logical object, they are
treated as different entities. For example, an IPv4
and IPv6 address.

## Resource-Specific Entity Domain Name

Some entities are defined and identified in a unique and global way. This is the
case for instance for entities that are endpoints identified by a routable IPv4
or IPv6 address. The entity domain for such entities can be globally defined and
named "ipv4" or "ipv6". Those entity domains are also called resource-agnostic
entity domains in this document, as they are not associated to any specific ALTO
information resources.

Some other entities and entity types are only defined relatively to a given
information resource. This is the case for entities of domain "pid", that can
only be understood with respect to the network map where they are defined.
For example, a PID named "mypid10" may be defined to represent a set S1 of IP
addresses in an information resource of type Network Map and named "netmap1".
Another Network Map "netmap2" may use the same name "mypid10" and define it
to represent another set S2 of IP addresses. The identifier "pid:mypid10" may
thus point to different objects because the information on the originating
information resource is lost. The reason is that "pid" denotes an entity
domain type rather than an unambiguous identifier.

To solve this ambiguity, the present extension introduces the concept of
resources-specific entity domain. This concept applies to domains where entities
are defined relatively to a given information resource. It can also apply to
domains of entities that are defined locally, such as local networks of objects
identified with a local IPv4 address.

In such cases, an entity domain name is explicitly associated with an identifier
of the information resource where these entities are defined. Using a
resource-specific entity domain name, an ALTO Property Map may unambiguously
indicate entity domains of the same type, on which entity properties may be
queried. Example resource-specific entity domain names may look like:
"netmap1.pid" or "netmap2.pid". This allows to identify two distinct PID
entities such as "netmap1.pid:mypid10" or "netmap1.pid:mypid10".
Resource-specific entity domain name will be specified in [](#domain-names).

<!--
## Resource-Specific Entity Domain

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
-->

<!--
## Relationship between Entity and Entity Domain

In this document, an entity is owned by exact one entity domain. It requires
that when an ALTO client or server references an entity, it must indicate its
entity domain explicitly. Even two entities in two different entity domains may
reflect to the same physical or logical object, we treat them as different
entities.

Because of this rule, although the resource-specific entity domain approach has
no ambiguity, it may introduce redundancy.
-->

<!-- ## Aggregated Entity Domain -->
<!-- ## Resource-Agnostic Entity Domain Name -->

<!-- TODO: Go over the whole document and rename aggregated entity domain to
resource-agnostic entity domain -->

<!--
Two entities in two different resource-specific entity domains may refer to
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
-->

## Resource-Specific Entity Property

<!--
According to the example of the aggregated entity domain, an entity may have
multiple properties in the same type but associated to different information
resources. To distinguish them, this document uses the same approach proposed by
Section 10.8.1 of [](#RFC7285), which is called `Resource-Specific Entity Property`.
-->

An entity may have properties of same type, whose values are associated to
different information resources. For instance, entity "192.0.2.34" defined in
the "ipv4" domain may have two "pid" properties defined in two different network
maps "netmap1" and "netmap2". These properties will likely have different values in
"netmap1" and "netmap2". To distinguish between them, this document uses the
same approach proposed as in Section 10.8.1 of [](#RFC7285), which is called
"Resource-Specific Entity Property". When a property value depends on a given
information resource, the identifier of the property must be explicitly
associated with the information resource that defines it.

For example, the "pid" property queried on entity "ipv4:192.0.2.34" and defined
in "netmap1" and "netmap2" respectively, may be named "netmap1.pid" and
"netmap2.pid". This allows a Client to get a property of the same type but
defined in different information resources in a single query.
Specifications are provided in [](#def-property).

## Entity Hierarchy and Property Inheritance {#con-hierarchy-and-inheritance}

Enumerating all individual entities is inefficient for both the Client and
the Server.

- For the Client, even if it only wants to request properties for a "/24"
ipv4 subnet, using the individual ipv4 entity, it has to enumerate all 256
entities.
- For the Server, in some cases, most of the entities may have the same
properties. Simply enumerating all of them may introduce a lot of reduncency
in the payload. For example, all the individual ipv4 entities in a "/24" ipv4
subnet is usually owned by the same AS. When a Client requests the ASN
property for this ipv4 subnet, using the individual ipv4 address, the Server
has to repeat the same ASN property for 256 times in the worst case.

To reduce the size of the property map request and response payloads, this
document introduces, when appicable, an approach called "Property
Inheritance". This approach consists of two parts: Entity Hierarchy and
Property Inheritance.

- Entity Hierarchy: This approach allows an entity domain to support using a
single identifier to identify a set of indivudial entities. For example, a
CIDR can be used to identify a set of ipv4 or ipv6 entities. Such an
identifier is called a hierarchical entity identifier, as the set identified
by it can be hierarachical. For example, the CIDR "ipv4:192.0.1.0/24" covers
all the individual ipv4 entities identified the CIDR "ipv4:192.0.1.0/26".
- Property Inheritance: This approach allows a property map to define a
property for a hierarchical entity identifier. An undefined property of an
entity may inherit the value of the property defined for a hierarchical
entity identifier. For example, a property map only defines the ASN property
for a hierarchical ipv4 entity identifier "ipv4:192.0.1.0/24". When receiving
this property map, a Client could infer that the ipv4 entity "ipv4:192.0.1.1"
inherits the same ASN property even if it does not appear in the property
map.

The detailed specification will be specified in [](#def-hierarchy-and-inheritance).

<!--
Enumerating all individual effective entities are inefficient. Some types of
entities have the hierarchy format, e.g., cidr, which stand for sets of
individual entities. Many entities in the same hierarchical format entity sets
may have the same proprety values. To reduce the size of the property map
representation, this document introduces an approach called `Property
Inheritance`. Individual entities can inherit the property from its hierarchical
format entity set.
-->

## Applicable Entity Domains and Properties in the Property Map Capabilities

This section explains how the IRD capabilities of a Property Map unambiguously
expose what type of properties on what entity domains a Client can query. A
field called "mapping" enumerates the entity domains supported by the Property
Map; For each entity domain, a list of applicable properties is provided. An
example can be found in [](#ird-example). Using resource-agnostic or resource-specific
entity domains and properties allows to formulate compact an unambiguous
entity property queries relating to one or more information resources, in
particular:

- avoid a Client to query a property on entity domains on which P is not
  defined,
- query for en entity E property values defined in different information
  resources,
- query a property P on entities E defined in different information resources.

Specifications will be provided in [](#FullPropMapCapabilities).

## Connection between Resource-Specific Entity Domain/Entity Property Mapping and Information Resources

Although the IRD capabilities of a Property Map can expose the supported mappings in this property map, it may still not be clear to a Client what a resource-specific entity domain is, and what an applicable resource-specific entity property means, as those concepts are not defined in other ALTO information resources. For example, a Client should understand that:

- a local IPv4 entity domain `netmap1.ipv4` includes the IPv4 addresses
  appearing in the `ipv4` field of the endpoint address group of each PID in the
  network map `netmap1`;
- a `netmap1.pid` property of an IPv4 entity `ipv4:192.0.1.1` indicates the PID defined by the network map `netmap1` and including the IPv4 address `ipv4:192.0.1.1` in its endpoint address group.

To help the client understanding these connections, this document requests two
new IANA registries for each information resource to define the connection to
each supported resource-specific entity domain and entity property mapping
respectively. Such a connection is called `Information Resource Export`, to
explain what is an resource-specific entity domain or an entity property mapping
exported by an information resource. Examples of `Information Resource Exports`
of existing ALTO information resources are provided in [](#ed-pm-export).
Specifications are provided in [](#def-ire). The details of these new IANA
registries are provided in [](#IANAResourceEDE) and [](#IANAResourceEPT).
