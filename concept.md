
# Basic Features of the Entity Property Map Extension {#basic-features-of-the-unified-property-extension}

This section gives a high-level overview of the basic features involved in
ALTO Entity Property Maps. It assumes the reader is familiar with the ALTO
protocol {{RFC7285}}. The purpose of this extension is to allow conveying
properties on objects that extend ALTO Endpoints and are called ALTO
Entities, or entities for short.

The features introduced in this section can be used as standalone. However,
in some cases, these features may depend on particular information resources
and need to be defined with respect to them. To this end,
[](#advanced-features-of-the-unified-property-extension) introduces
additional features that extend the ones presented in the present section.

The Entity Property Maps extension described in this document introduces a
number of features that are summarized in
[](#features-introduced-with-epm-extension), where [](#TableUPFeatures) lists
the features and references the sections in this document that give a
high-level and normative description thereof.

## Entity {#con-entity}

The concept of an ALTO Entity generalizes the concept of an ALTO Endpoint
defined in Section 2.1 of {{RFC7285}}. An entity is an object that can be an
endpoint that is defined by its network address, but can also be an object
that has a defined mapping to a set of one or more network addresses or an
object that is not even related to any network address. Thus, whereas all
endpoints are entities, not all entities are endpoints.

<!-- Examples of entities -->

Examples of entities are:

* an ALTO endpoint, defined in {{RFC7285}}, that represents an application or
  a host identified by a communication address (e.g., an IPv4 or IPv6
  address) in a network,
* a PID, defined in {{RFC7285}}, that has a provider defined human-readable
  identifier specified by an ALTO network map, which maps a PID to a set of
  IPv4 and IPv6 addresses,
* an autonomous system (AS), that has an AS number (ASN) as its identifier
  and maps to a set of IPv4 and IPv6 addresses,
* a country with a code as specified in {{ISO3166-1}}, to which applications
  such as CDN providers associate properties and capabilities,
* a TCP/IP network flow, that is identified by a TCP/IP 5-Tuple specifying
  its source and destination addresses and port numbers and the utilized
  protocol,
* a routing element, that is specified in {{RFC7921}} and is associated with
  routing capabilities information,
* an abstract network element, that represents an abstraction of a network
  part such as a router, one or more links, a network domain or their
  aggregation.

## Entity Domain {#con-entity-domain}

An entity domain defines a set of entities of the same semantic type. An
entity domain is characterized by its type and identified by its name.

In this document, an entity must be owned by exactly one entity domain name.
An entity identifier must point to exactly one entity. If two entities in two
different entity domains refer to the same physical or logical object, they
are treated as different entities. For example, if an object has both an IPv4
and an IPv6 address, these two addresses will be treated as two entities,
defined respectively in the "ipv4" and "ipv6" entity domains.

### Entity Domain Type {#con-entity-domain-type}

The type of an entity domain type defines the semantics of a type of entity.
Entity domain types can be defined in different documents. For example: the
present document defines entity domain types "ipv4", "ipv6" and "pid" in
sections [](#inet-addr-domain) and [](#pid-domain). The entity domain type
"ane", that defines Abstract Network Elements (ANEs), is introduced in
{{I-D.ietf-alto-path-vector}}. The entity domain type that defines country
codes is introduced in {{I-D.ietf-alto-cdni-request-routing-alto}}. An entity
domain type MUST be registered at the IANA, as specified in section
[](#dom-reg-process) and similarly to an ALTO address type.

### Entity Domain Name {#con-entity-domain-name}

The name of an entity domain is defined in the scope of an ALTO server. An
entity domain name can sometimes be identical to the name of its relevant
entity domain type. This is the case when the entities of a domain have an
identifier that points to the same object throughout all the information
resources of the Server that provide entity properties for this domain. For
example, a domain of type "ipv4" containing entities identified by a public
IPv4 address can be named "ipv4" because its entities are uniquely identified
by all the Server resources.

In some cases, a domain type and domain name must be different. Indeed, for
some domain types, entities are defined relative to a given information
resource. This is the case for entities of domain type "pid". A PID is
defined relative to a network map. For example: an entity "mypid10" of domain
type "pid" may be defined in a given network map and be undefined in other
network maps. Or "mypid10" may even be defined in two different network maps
and map, in each of these network maps, to a different set of endpoint
addresses. In this case, naming an entity domain only by its type "pid" does
not guarantee that its set of entities is owned by exactly one entity domain.

[](#rsed-name) and [](#domain-names) of this
document describe how a domain is uniquely identified, across the ALTO
Server, by a name that associates the domain type and the related information
resource.

## Entity Property Type {#con-property}

An entity property defines a property of an entity. This is similar to the
endpoint property defined in Section 7.1 of {{RFC7285}}. An entity property
can convey either network-aware or network-agnostic information. Similarly to
an entity domain, an entity property is characterized by its type and
identified by its name. An entity property type MUST be registered at the
IANA, as specified in section [](#IANAEntityProp).

Below are some examples with real and fictitious entity domain and property
names:

* an entity in the "ipv4" domain type may have a property whose value is an
  Autonomous System (AS) number indicating the AS that owns this IPv4 address
  and another property named "countrycode" indicating a country code mapping
  to this address,
* an entity identified by its country code in the entity domain type
  "countrycode", defined in {{I-D.ietf-alto-cdni-request-routing-alto}} may
  have a property indicating what delivery protocol is used by a CDN,
* an entity in the "netmap1.pid" domain may have a property that indicates
  the central geographical location of the endpoints it includes.

It should be noted that some identifiers may be used for both an entity
domain type and a property type. For example:

* the identifier "countrycode" may point to both the entity domain type
  "countrycode" and the fictitious property type "countrycode".
* the identifier "pid" may point to both the entity domain type "pid" and the
  property type "pid".

Likewise, a same identifier may point to both a domain name and a property
name. For example: the identifier "netmap10.pid" may point to either the
domain defined by the PIDs of network map "netmap10" or to a property that
returns, for an entity defined by its IPv4 address, the PID of netmap10 that
contains this entity. Such cases will be further explained in
[](#advanced-features-of-the-unified-property-extension).

## New information resource and media type: ALTO Property Map {#con-propmap}

This document introduces a new ALTO information resource named Property Map.
An ALTO property map provides a set of properties on one or more sets of
entities. A property may apply to different entity domain types and names.
For example, an ALTO property map may define the "ASN" property for both
"ipv4" and "ipv6" entity domains.

The present extension also introduces a new media type.

This document uses the same definition of an information resource as Section
9.1 of {{RFC7285}}. ALTO uses media types to uniquely indicate the data
format used to encode the content to be transmitted between an ALTO server
and an ALTO client in the HTTP entity body. In the present case, an ALTO
property map resource is
<!-- represented by a JSON object of type InfoResourcePropertyMap and -->
defined by the media type "application/alto-propmap+json".

A Property Map can be queried as a GET-mode resource, thus conveying all
properties on all entities indicated in its capabilities. It can also be
queried as a POST-mode resource, thus conveying a selection of properties on
a selection of entities.

# Advanced Features of the Entity Property Map Extension {#advanced-features-of-the-unified-property-extension}

## Entity Identifier and Entity Domain Name {#entity-identifier-and-entity-domain}

In {{RFC7285}}, an endpoint has an identifier that is explicitly associated
with the "ipv4" or "ipv6" address domain. Examples are "ipv4:192.0.2.14" and
"ipv6:2001:db8::12".

In this document, an entity must be owned by exactly one entity domain name
and an entity identifier must point to exactly one entity. To ensure this, an
entity identifier is explicitly attached to the name of its entity domain and
an entity domain type characterizes the semantics and identifier format of
its entities.

The encoding format of an entity identifier is further specified in
[](#entity-addrs) of this document.

For instance:

* if an entity is an endpoint with example routable IPv4 address
  "192.0.2.14", its identifier is associated with domain name "ipv4" and is
  "ipv4:192.0.2.14",
* if an entity is a PID named "mypid10" in network map resource "netmap2",
  its identifier is associated with domain name "netmap2.pid" and is
  "netmap2.pid:mypid10".

## Resource-Specific Entity Domain Name {#rsed-name}

Some entities are defined and identified uniquely and globally. This is the
case for instance when entities are endpoints that are identified by a
routable IPv4 or IPv6 address. The entity domain for such entities can be
globally defined and named "ipv4" or "ipv6". Those entity domains are called
resource-agnostic entity domains in this document, as they are not associated
with any specific ALTO information resources.

Some other entities and entity types are only defined relatively to a given
information resource. This is the case for entities of domain type "pid",
that can only be understood with respect to the network map where they are
defined. For example, a PID named "mypid10" may be defined to represent a set
S1 of IP addresses in a network map resource named "netmap1". Another network
map "netmap2" may use the same name "mypid10" and define it to represent
another set S2 of IP addresses. The identifier "pid:mypid10" may thus point
to different objects because the information on the originating information
resource is lost.

To solve this ambiguity, the present extension introduces the concept of
resources-specific entity domain. This concept applies to domain types where
entities are defined relatively to a given information resource. It can also
apply to entity domains that are defined locally, such as local networks of
objects identified with a local IPv4 address.

In such cases, an entity domain type is explicitly associated with an
identifier of the information resource where these entities are defined. Such
an information resource is referred to as the "specific information
resource". Using a resource-aware entity domain name, an ALTO property map
can unambiguously identify distinct entity domains of the same type, on which
entity properties may be queried. Examples of resource-specific entity domain
names may look like: "netmap1.pid" or "netmap2.pid". Thus, a name association
such as "netmap1.pid:mypid10" and "netmap2.pid:mypid10" allows to distinguish
the two abovementioned PIDs that are both named "mypid10" but in two
different resources, "netmap1" and "netmap2".

An information resource is defined in the scope of an ALTO Server and so is
an entity domain name. The format of a resource-specific entity domain name
is further specified in [](#domain-names).

## Resource-Specific Entity Property Value {#rsep}

Like entity domains, some types of properties are defined relatively to an
information resource. That is, an entity may have a property of a given type,
whose values are associated to different information resources.

For example, suppose entity "192.0.2.34" defined in the "ipv4" domain has a
property of type "pid", whose value is the PID to which address "192.0.2.34"
is attached in a network map. The mapping of network addresses to PIDs is
specific to a network map and probably different from one network map
resource to another one. Thus, if a property "pid" is defined for entity
"192.0.2.34" in two different network maps "netmap1" and "netmap2", the value
for this property will likely be a different value in "netmap1" and
"netmap2".

To support information resource dependent property values, this document uses
the same approach as in Section 10.8.1 of {{RFC7285}} entitled
"Resource-Specific Endpoint Properties". When a property value depends on a
given information resource, the name of this property must be explicitly
associated with the information resource that defines it.

For example, the property "pid" queried on entity "ipv4:192.0.2.34" and
defined in both "netmap1" and "netmap2", can be named "netmap1.pid" and
"netmap2.pid". This allows a Client to get a property of the same type but
defined in different information resources with a single query.
Specifications on the property name format are provided in [](#def-property).

## Entity Hierarchy and Property Inheritance {#con-hni}

For some domain types, entities can be grouped in a set and be defined by the
identifier of this set. This is the case for domain types "ipv4" and "ipv6",
where individual Internet addresses can be grouped in blocks. When a same
property value applies to a whole set, a Server can define a property for the
identifier of this set instead of enumerating all the entities and their
properties. This allows a substantial reduction of transmission payload both
for the Server and the Client. For example, all the entities included in the
set defined by the address block "ipv6:2001:db8::1/64" share the same
properties and values defined for this block.

Additionally, entity sets sometimes are related by inclusion, hierarchy or
other relations. This allows defining inheritance rules for entity properties
that propagate properties among related entity sets. The Server and the
Client can use these inheritance rules for further payload savings. Entity
hierarchy and property inheritance rules are specified in the documents that
define the applicable domain types. The present document defines these rules
for the "ipv4" and "ipv6" domain types.

This document introduces, for applicable domain types, "Entity Property
Inheritance rules", with the following concepts: Entity Hierarchy, Property
Inheritance and Property Value Unicity. A detailed specification of entity
hierarchy and property inheritance rules is provided in
[](#def-hierarchy-and-inheritance).

### Entity Hierarchy

An entity domain may allow using a single identifier to identify a set of
individual entities. For example, a CIDR block can be used to identify a set
of IPv4 or IPv6 entities. A CIDR block is called a hierarchical entity
identifier, as it can reflect inclusion relations among entity sets. For
example, the CIDR "ipv4:192.0.1.0/24" includes all the individual IPv4
entities identified by the CIDR "ipv4:192.0.1.0/26".

### Property Inheritance

A property may be defined for a hierarchical entity identifier, while it may
be undefined for individual entities covered by this identifier. In this
case, these individual entities inherit the property value defined for the
identifier that covers them. For example, suppose a property map defines a
property P for which it assigns value V1 only for the hierarchical entity
identifier "ipv4:192.0.1.0/24" but not for individual entities in this block.
Suppose also that inheritance rules are specified for CIDR blocks in the
"ipv4" domain type. When receiving this property map, a Client can infer that
entity "ipv4:192.0.1.1" inherits the property value V1 of block
"ipv4:192.0.1.0/24" because the address "ipv4:192.0.1.1" is included by the
CIDR block "ipv4:192.0.1.0/24".

Property value inheritance rules also apply among entity sets. A property map
may define values for an entity set belonging to a hierarchy but not for
"sub" sets that are covered by this set identifier. In this case, inheritance
rules must specify how entities in "sub" sets inherit property values from
their "super" set. For instance, if a property P is defined only for the
entity set identified by address block "ipv4:192.0.1.0/24", the entity set
identified by "ipv4:192.0.1.0/30" and thus included in the former set, may
inherit the property P value from set "ipv4:192.0.1.0/24".

### Property Value Unicity

The inheritance rules must ensure that an entity belonging to a hierarchical
set of entities inherits no more than one property value, for the sake of
consistency. Indeed, a property map may define a property on a hierarchy of
entity sets that inherit property values from one or more subsets (upper
levels). On the other hand, a property value, defined on a superset (lower
level) may be different from the value defined in a subset. In such a case,
supersets may potentially end up with different property values. This may be
the case for address blocs with increasing prefix length, on which a property
value gets increasingly accurate and thus may differ. For example, a
fictitious property such as "geo-location" or "average transfer volume" may
be defined at a progressively finer grain for entity sets defined with
progressively longer CIDR prefixes. It seems more interesting to have
property values of progressively higher accuracy. A unicity rule, applied to
the entity domain type must specify an arbitration rule among the different
property values for an entity. An example illustrating the need for such
rules is provided in [](#inet-inheritance).

## Supported Properties on Entity Domains in Property Map Capabilities {#applicable-entity-domains-and-properties-in-the-property-map-capabilities}

A property type is not necessarily applicable to any domain type, or an ALTO
Server may choose not to provide a property on all applicable domains. For
instance, a property type reflecting link bandwidth is likely not defined on
entities of a domain of type "country-code". Therefore an ALTO server
providing Property Maps needs to specify the properties that can be queried
on the different entity domains it supports.

This document explains how the Information Resources Directory (IRD)
capabilities of a Property Map resource unambiguously expose what properties
a Client can query on a given entity domain.

* a field named "mappings" lists the names of the entity domains supported by
  the Property Map,
* for each listed entity domain, a list of the names of the applicable
  properties is provided.

An example is provided in [](#ird-example). The "mappings" field associates
entity domains and properties that can be resource-agnostic or
resource-specific. This allows a Client to formulate compact and unambiguous
entity property queries, possibly relating to one or more information
resources. In particular:

* it avoids a Client to query a property on entity domains on which it is not
  defined,
* it allows a Client to query, for an entity E, values for a property P that
  are defined in several information resources,
* it allows a Client to query a property P on entities that are defined in
  several information resources.

Further specifications are provided in [](#FullPropMapCapabilities).

## Defining Information Resource for Resource-Specific Entity Domains {#def-ir}

A Client willing to query properties on entities belonging to a domain needs
to know how to retrieve these entities. To this end, the Client can look up
the "mappings" field exposed in IRD capabilities of a property map, see
[](#applicable-entity-domains-and-properties-in-the-property-map-capabilities).
This field, in its keys, exposes all the entity domains supported by the
property map. The syntax of the entity domain identifier specified in
[](#domain-names) allows the client to infer whether the entity domain is
resource-specific or not. The Client can extract, if applicable, the
identifier of the specific resource, query the resource and retrieve the
entities. For example:

* an entity domain named "netmap1.ipv4" includes the IPv4 addresses that
  appear in the "ipv4" field of the endpoint address group of each PID in the
  network map "netmap1", and that cannot be recognized outside "netmap1"
  because, for instance, these are local non-routable addresses,
* an entity domain named "netmap1.pid" includes the PIDs listed in network
  map "netmap1".
* an entity domain named "ipv4" is resource-agnostic and covers all the
  routable IPv4 addresses.

Besides, it is also necessary to inform a Client about which associations of
specific resources and entity domain types are allowed, because it is not
possible to prevent a Server from exposing inappropriate associations. An
informed Client will just ignore inappropriate associations exposed by a
Server and avoid error-prone transactions with the Server.

For example, the association "costmap3.pid" is not allowed for the following
reason: although a cost map exposes PID identifiers, it does not define the
set of addresses included in this PID. Neither does a cost map list all the
PIDs on which properties can be queried, because a cost map only exposes PID
pairs on which a queried cost type is defined. Therefore, the resource
"costmap3" does not enable a Client to extract information on the existing
PID entities or on the addresses they contain.

Instead, the cost map uses a network map, where all the PIDs used in a cost
map are defined together with the addresses contained by the PIDs. This
network map is qualified in this document as the Defining Information
Resource for the entity domain of type "pid" and this concept is explained in
[](#defining-information-resource-and-media-type).

### Defining Information Resource and its Media Type {#defining-information-resource-and-media-type}

For the reasons explained in the previous section, this document introduces
the concept of "Defining Information Resource and its Media Type".

A defining information resource for an entity domain D is the information
resource where entities of D are defined. That is, all the information on the
entities of D can be retrieved in this resource. This concept applies to
resource-specific domains. This is useful for entity domain types that are by
essence domain-specific, such as "pid" and "ane" domain types. It is also
useful for resource-specific entity domains constructed from
resource-agnostic domain types, such as network map specific domains of local
IPv4 addresses.

The defining information resource of an entity domain D is unique and has the
following specificities:

* it has an entry in the IRD,
* it defines the entities of D,
* it does not use another information resource that defines these entities,
* it defines and exposes entity identifiers that are all persistent.
* its media type is unique and equal to the one that is specified for the
  defining information resource of an entity domain type.

A fundamental attribute of a defining information resource is its media type.
There is a unique association between an entity domain type and the media
type of its defining information resource. When an entity domain type allows
associations with defining information resources, the document that defines
this entity domain type specifies the media type of the potential defining
information resource. Likewise, the IANA registration of an entity domain
type also specifies the media type of the potential defining information
resource.

When the Client wants to use a resource-specific entity domain, it needs to
be cognizant of the media-type of its defining information resource. If the
Server exposes resources a resource specific entity domain with a
non-compliant media type for the domain type, the Client can avoid
transaction errors by ignoring them.

<!--
The same holds for property types whose values are defined relatively to an
information resource. Similarly to resource specific entity domains, the
Client needs to be cognizant of appropriate associations of information
resource and property types. Therefore, when specifying and registering a
property type whose values are resource-specific, it is necessary to specify
the media type of its defining information resource. For example: the
defining information resource for property type "pid" is a network map; the
defining information resource for property type "cdnifci-capability", defined in
{{I-D.ietf-alto-cdni-request-routing-alto}} is a "cdnifci-map" information
resource, defined in that same document.
-->

### Examples of Defining Information Resources and Their Media Types {#example-specific-ir-mediatype}

Here are examples of defining information resource types and their media
types associated to different entity domain types.

* For entity domain type "pid": the media type of the specific resource is
  "application/alto-networkmap+json", because PIDs are defined in network map
  resources.
* For entity domain types "ipv4" and "ipv6": the media type of the specific
  resource is "application/alto-networkmap+json", because IPv4 and IPv6
  addresses covered by the Server are defined in network map resources.
* For entities of domain type "ane": {{I-D.ietf-alto-path-vector}} defines
  entities named "ANE", where ANE stands for Abstracted Network Element, and
  the entity domain type "ane". An ANE may have a persistent identifier, say,
  "entity-4", that is provided by the Server as a value of the
  "persistent-entity-id" property of this ANE. Further properties may then be
  queried on an ANE by using its persistent entity ID. These properties are
  available from a persistent property map, that defines properties on a
  specific "ane" domain. Together with the persistent identifier, the Server
  also provides the property map resource identifier where the "ane" domain
  containing "entity-4" is defined. The definition of the "ane" entity domain
  containing "entity-4" is thus specific to the property map. Therefore, for
  entities of domain type "ane" that have a persitent identifier, the media
  type of the specific information resource is
  "application/alto-propmap+json".
* Last, the entity domain types "asn" and "countrycode" defined in
  {{I-D.ietf-alto-cdni-request-routing-alto}} do not have a defining
  information resource. Indeed, the entity identifiers in these two entity
  domain types are already standardized in documents that the Client can use.

## Defining Information Resource for Resource-Specific Property Values {#def-ir-for-irsp}

As explained in [](#rsep), a property type may
take values that are resource-specific. This is the case for property type
"pid", whose values are by essence defined relatively to a specific network
map. That is, the PID value returned for an IPv4 address is specific to the network
map defining this PID and may differ from one network map to another one.

<!--
Property values may be specific to different types of information resources.
For example: the value for property "pid" is specific to a network map. The
value for property type "cdnifci-capab" is specific to the information
resource "cdnifci-map", defined in
{{I-D.ietf-alto-cdni-request-routing-alto}}, while network maps do not define
property "fci-capability" for IPv4 addresses and a cdnifci-map does not
define "pid" values for IPv4 addresses.
-->

Another example is provided in {{I-D.ietf-alto-cdni-request-routing-alto}}
that defines property type "cdni-capabilities". The value of this property is
specific to a CDNI Advertisement resource, that provides a list of CDNI
capabilities. The property is provided for entity domain types "ipv4",
"ipv6", "asn" and "countrycode". A CDNI Advertisement resource does however
not define PID values for IPv4 addresses while a network map does not define
CDNI capabilities for IPv4 addresses.

<!--
Thus, similarly to resource specific entity domains, the Client needs to be
cognizant of appropriate associations of information resource and property
types.
-->

Similarly to resource-specific entity domains, the Client needs to be
cognizant of appropriate associations of information resource and property
types. Therefore, when specifying and registering a property type whose
values are resource-specific, the media type of its defining information
resource needs to be specified. For example:

<!--
### Examples of defining resources media-types for properties {#example-specific-ir-mediatype-prop}
-->

<!--
Here are some examples of specific information resources types associated to
entity property types and their media type.
-->

* The media type of the defining information resource for property type "pid"
  is "application/alto-networkmap+json".
* The media type of the defining information resource for property type
  "cdni-capabilities" defined in {{I-D.ietf-alto-cdni-request-routing-alto}}
  is "application/alto-cdni+json".
