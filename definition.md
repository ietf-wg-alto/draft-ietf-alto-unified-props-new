# Definitions and Concepts

## Entity

<!-- FIXME:
This section introduces a new concept and deserves more explanations before
diving in the technical "how to handle".
For instance:
- what kind of object other than an addressable ALTO endpoint is eligible to be
  an ALTO entity?
- an entity must be related to network address(es) e.g. entity such as PID, ASN
  nb, Country code have a defined mapping to IP or cellular addresses.
-->

The entity concept generalizes the concept of the endpoint defined in Section 2.1 of
[](#RFC7285). An entity is an object that can be an endpoint and is identified
by its network address, but can also be an object that has a defined mapping to
a set of one or more network addresses or is even not related to any network
address.

<!-- Examples of entities -->

Examples of eligible entities are:

- a PID, defined in [](#RFC7285), that has a provider defined human readable
  abstract identifier defined by a ALTO network map, which maps a PID to a set of ipv4 and ipv6 addresses;
- an autonomous system (AS), that has an AS number (ASN) as its identifier 
  and maps to a set of ipv4 and ipv6 addresses;
- a region representing a country, that is identified by its country code defined 
  by ISO 3166 and maps to a set of cellular addresses;
- a TCP/IP network flow, that has a server defined identifier consisting of the 
  defining TCP/IP 5-Tuple, , which is an example that all endpoints 
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

### Entity Domain

<!-- FIXME: Illustrate what means by "global" here. -->

Each entity MUST belong to one and only one entity domain, where an entity
domain is defined as a set of entities. An entity domain can be a global entity
domain; this document defines two global entity domains, for two Internet
address domains (see [](#inet-addr-domain)). An entity domain can also be
defined by an ALTO resource; this document defines PID entity domains to be
derived from ALTO network maps (see [](#pid-domain)). Future documents can
define additional entity domains to satisfy their additional requirements such
as cellular network information and routing capability exposure. But they are
not in the scope of this document.

<!-- This document will define the domains precisely below. -->
<!-- An additional example is the proposed domain of Abstract Network Elements
associated with topology and routing, as suggested by
[](#I-D.ietf-alto-path-vector). -->

#### Entity Domain Type {#domain-types}

An entity domain has a type, which is defined by a string that MUST be no more
than 64 characters, and MUST NOT contain characters other than US-ASCII
alphanumeric characters (U+0030-U+0039, U+0041-U+005A, and U+0061-U+007A),
hyphen ('-', U+002D), and low line ('\_', U+005F). For example, the strings
`ipv4`, `ipv6`, and `pid` are valid entity domain types.

The type EntityDomainType is used in this document to denote a JSON string
confirming to the preceding requirement.

An entity domain type defines the semantics of a type of entity domains. Each
entity domain type MUST be registered with the IANA. The format of the entity
identifiers (see [](#entity-addrs)) in that type of entity domains, as well as any
hierarchical or inheritance rules (see [](#def-hierarchy-and-inheritance)) for
those entities, MUST be specified at the same time.

#### Entity Domain Name {#domain-names}

<!-- FIXME: What is a global entity domain here? Requrie the definition. -->
Each entity domain is identified by an entity domain name, a string of the
following format:

``` text
EntityDomainName ::= [ ResourceID '.' ] EntityDomainType
```

This document distinguish two types of entity domains: resource-specific entity
domains and shared entity domains. Their entity domain names are derived as
follows.

Each ALTO information resource MAY define a resource-specific entity domain
(which could be empty) in a given entity domain type.
A resource-specific entity domain is identified by an entity domain name derived
as follows. It MUST start with a resource ID using the ResourceID type defined
in [](#RFC7285), followed by the '.' separator (U+002E), followed by an
EntityDomainType typed string. Hence, there can be as many entity domains as the
number of ALTO information resources for each entity domain type. For example,
if an ALTO server provides two network maps `net-map-1` and `net-map-2`, they
can define two different `pid` domains identified by `net-map-1.pid` and
`net-map-2.pid` respectively.

Given a set of ALTO information resources, there MAY be a shared entity domain
in a given entity domain type amongst them. A shared entity domain is simply
identified by its entity domain type. To avoid ambiguity, the shared entity
domain can only be used when a scope of a set of ALTO information resources is
given. For example, given two network maps `net-map-1` and `net-map-2`, the
`ipv4` and `ipv6` entity domain types identify two shared Internet address
entity domains (see [](#inet-addr-domain)) for them.

<!--
Each entity domain type may have a global entity domain. For a global entity
domain (i.e., not resource-specific), its entity domain name is an
EntityDomainType typed string. For example, the `ipv4` and `ipv6` entity domain
types identify two Internet address entity domains (see [](#inet-addr-domain)).
-->

Note that the '.' separator is not allowed in EntityDomainType and hence there
is no ambiguity on whether an entity domain name refers to a global entity
domain or a resource specific entity domain.

For an EntityDomainType which allows resource-specific entity domains, the valid
type(s) resources MUST be specified.

### Entity Identifier {#entity-addrs}

<!-- FIXME: The entity identifier is not global unique. -->

Entities in an entity domain are identified by entity identifiers (EntityID) of
the following format:

``` text
EntityID ::= EntityDomainName ':' DomainTypeSpecificEntityID
```

Examples from the Internet address entity domains include individual IP
addresses such as `ipv4:192.0.2.14` and `ipv6:2001:db8::12`, as well as address
blocks such as `ipv4:192.0.2.0/26` and `ipv6:2001:db8::1/48`.

The format of the second part of an entity identifier depends on the entity
domain type, and MUST be specified when registering a new entity domain type.
Identifiers MAY be hierarchical, and properties MAY be inherited based on that
hierarchy. Again, the rules defining any hierarchy or inheritance MUST be
defined when the entity domain type is registered.

The type EntityID is used in this document to denote a JSON string
representing an entity identifier in this format.

Note that two entity identifiers with different textual representations may refer
to the same entity, for a given entity domain. For example, the strings
`ipv6:2001:db8::1` and `ipv6:2001:db8:0:0:0:0:0:1` refer to the same entity in
the 'ipv6' entity domain.

### Entity Property

An entity property defines a property of an entity. It is similar to the
endpoint property defined by Section 7.1 of [](#RFC7285), but can be general
besides network-aware.

For example, an `ipv4` entity may have a property whose value is an Autonomous
System (AS) number indicating the AS which this IPv4 address is owned by.

#### Entity Property Type {#def-property-type}

Each entity property has a type to indicate the encoding and the semantics of
the value of this entity property. The type EntityPropertyType is used in this
document to indicate a string denoting an entity property type. The string MUST
be no more than 32 characters, and it MUST NOT contain characters other than
US-ASCII alphanumeric characters (U+0030-U+0039, U+0041-U+005A, and
U+0061-U+007A), the hyphen ('-', U+002D), the colon (':', U+003A), or the low
line ('_', U+005F).

Each entity property type MUST be registered with the IANA. The intended
semantics of the entity property type MUST be specified at the same time.
<!-- , as well as the media types of dependent resources and the interpretation, -->

<!-- FIXME: this part is used to motivate the mapping of resource type. may need
to move to below. -->

To distinguish with the endpoint property type, the entity property type has the
following features.

- Some entity property types may be applicable to entities in only particular
  types of entity domains, not all. For example, the `pid` property is not
  applicable to entities in a `pid` typed entity domain, but is applicable to
  entities in the `ipv4` or `ipv6` domains.
- The intended semantics of the value of a entity property may also depend on
  the the entity domain type of this entity. For example, suppose that the
  `geo-location` property is defined as the coordinates of a point, encoded as
  (say) "latitude longitude [altitude]." When applied to an entity that
  represents a specific host computer, identified by an address in the `ipv4` or
  `ipv6` entity domain, the property defines the host's location. However, when
  applied to an entity in a `pid` domain, the property would indicate the
  location of the center of all hosts in this `pid` entity.

#### Entity Property Name

<!-- FIXME: remove most. Use RFC 7285 Section 10.8, for resource-specific
properties and global properties. -->

Each entity property is identified by an entity property name, which is a string
of the following format:

``` text
EntityPropertyName ::= [ ResourceID '.' ] EntityPropertyType
```

Similar to the endpoint property type defined in Section 10.8 of [](#RFC7285),
each entity property may be defined by either the property map itself
(self-defined) or some other specific resource (resource-specific).

The entity property name of a self-defined entity property is an
EntityPropertyType typed string. For example, the `asn` property of an `ipv4`
entity indicates the AS number of the AS which this IPv4 address is owned by.

The entity property name of a resource-specific entity property starts with a
string of the type ResourceID defined in [](#RFC7285), followed by the '.'
separator (U+002E) and a EntityDomainType typed string. For example, the `pid`
properties of an `ipv4` entity defined by two different maps `net-map-1` and
`net-map-2` are identified by `net-map-1.pid` and `net-map-2.pid` respectively.

<!-- ## Property Type and Property Name {#def-property-type} -->

<!-- FIXME: Section needs be reorganized to first motivate the attachment of
address to address domain before setting rules. -->

<!-- OLD-0 -->
<!--
This document defines property types in the domain-specific semantics. This
design is to enforce that each property type MUST be registered for a single
specific entity domain. But multiple property types with the similar semantics
MAY share the same Property Name in different entity domains. This design
decision is adopted because of the following considerations:
-->

<!-- NEW-0 -->
<!--
An entity in an entity domain MAY have one or more properties, where each
property is defined by a Property Type. This document defines properties to be
entity-domain-type specific for the following reasons:

Therefore, each property type has a unique identifier encoded with the following
format:
-->

<!--
- The `EntityDomainName` indicates which entity domain the property type applies
  to.
- The `PropertyName` SHOULD relate to the semantics of this property type. It
  does not have to be globally unique. In other words, different property types
  could have the same property name applied to different entity domains, if
  they have the similar semantics. For example, the property types `ipv4:pid`
  and `ipv6:pid` have the same property name `pid` applied to both `ipv4` and
  `ipv6` domains.
-->

<!-- ## Property Name ## -->

<!-- FIXME: This is not correct. Because the ALTO entity domain is not a strict
superset of the ALTO address type. Revise it! -->

<!--
The space of entity property names associated with entities defined by this
document is a superset of the endpoint property names defined by [](#RFC7285).
Thus endpoint property names registered with the `ALTO Endpoint Property Type
Registry` MUST be defined in [](#IANAEndpointProp) of this document. The type
PropertyName denotes a JSON string with a property name in this format.
-->

<!-- FIXED: Change the single name space design to the domain-specific design -->

<!--
This document defines uniform property names specified in a single property
name space rather than being scoped by a specific entity domain, although some
properties may only be applicable for particular entity domains. This design
decision is to enforce a design so that similar properties are named similarly.
The interpretation of the value of a property, however, may depend on the
entity domain. (FIXME: This design decision will mess up the dependency
declaration.) For example, suppose that the `geo-location` property is defined
as the coordinates of a point, encoded as (say) "latitude longitude
[altitude]." When applied to an entity that represents a specific host
computer, such as an Internet address, the property defines the host's
location.  When applied to an entity that represents a set of computers, such
as a CIDR, the property would be the location of the center of that set.  If it
is necessary to represent the bounding box of a set of hosts, another property,
such as `geo-region`, should be defined.
-->

### Hierarchy and Inheritance {#def-hierarchy-and-inheritance}

Entities in a given domain MAY form a hierarchy based on entity identifiers, and
introducing hierarchy allows the introduction of inheritance. Each entity domain
type MUST define its own hierarchy and inheritance rules when registered. The
hierarchy and inheritance rule makes it possible for an entity to inherit a
property value from another entity in the same domain.
<!--If and only
if the property of an entity is undefined, the hierarchy and inheritance rules
are applied. [YRY: Do we need this?] [Jensen: I think this feature is for reducing the response size.] -->

## Resource

A resource indicates an ALTO information resource in this document.

### Resource Type {#def-resource-type}

Each resource has a type identified by a JSON string, which aliases to a media
type of the response of an ALTO information resource.

When a new ALTO service is defined and introduces a new media type of response,
a new resource type SHOULD be defined as well.

Each resource type MUST be registered with the IANA. The aliased media type MUST
be specified.

### Entity Domain Export {#def-epe}

Each type of resource MAY export several entity domains in some entity domain
types. For example, a network map resource defines a `pid` domain, a `ipv4`
domain and a `ipv6` domain (which may be empty).

When a new resource type is registered, if this type of resource can export an
entity domain in an existing entity domain type, the corresponding document MUST
define how to export such type of entity domain from such type of resource.

When a new entity domain type is defined, if an existing type of resource can
export an entity domain in this entity domain type, the corresponding document
MUST define how to export such type of entity domain from such type of resource.

### Entity Property Transfer {#def-ept}

For each entity domain which could be exported by a resource, this resource MAY
be transferred to a property map mapping entities in this entity domain to some
entity property. For example, a network map resource can map an `ipv4` entity to
its `pid` property.

When a new resource type is registered, if this type of resource can export an
entity domain in an existing entity domain type, and be transferred to a
property map mapping entities in this entity domain to an existing type of
entity property, the corresponding document MUST define how to transfer such
type of resource to such a property map.

When a new entity domain type or a new entity property type is defined, if an
existing type of resource can export an entity domain in this entity domain
type, and be transferred to a property map mapping entities in this entity
domain to this type of entity property, the corresponding document MUST define
how to transfer such type of resource to such a property map.

<!--
## Relationship with Other ALTO Resources {#def-relationship-to-other-resources}

FIXME: very messy below. Delete most.

[](#RFC7285) recognizes that some properties for some entity domains MAY be
specific to an ALTO resource, such as a network map. Accordingly Section 10.8.1
of [](#RFC7285) defines the concept of `resource-specific endpoint properties`,
and indicates that dependency by prefixing the property name with the ID of the
resource on which it depends. That document defines one resource-specific
property, namely the `pid` property, whose value is the name of the PID
containing that endpoint in the associated network map.

However, a property may be associated to more than one information resources
within an entity domain. For example, the fictitious property
`pid:cdni-fci-capabilities` indicates CDNI capabilities (see [](#RFC8008)) of a
set of `ipv4` or `ipv6` typed CDNI footprints included by some entities in PID
domain. It depends on two resources:

- the network map in which the input PID entities have been defined,
- the fictitious CDNI FCI map in which the CDNI footprints and capabilities
  advertisement objects (see [](#RFC8008)) are included.

A `resource-specific property` can only indicate one of them. Thus, using
`resource-specific properties` cannot handle multiple dependencies very well.

To address this issue, this document takes a different approach as follows:

- Firstly, instead of defining the dependency by prefixing the property name
  with a specific dependent resource identifier, this document introduces a
  Property Type that appends a property name to an entity domain name, and
  registers the dependency types for this Property Type. This gives a hint on
  the types of dependent resources. For example, the fictitious property
  `pid:region` applying to entities in the PID domain depends on the network map
  in which the input PID entities have been defined; but the fictitious property
  `ipv4:region` applying to entities in IPv4 domain does not depend on any
  information resource.
- Secondly, it sets a rule saying that in a property map, all provided property
  types MUST have the same media types of dependent information resources
  (denoted as "dependency types" in short). For example, the fictitious
  property types `pid:region` and `ipv4:region` cannot be provided in the same
  property map, as they have different dependency types.
- Finally, it identifies, in the IRD and Server responses, the sequence of
  information resources associated to all provided properties of entities in a
  particular property map. In other words, the ALTO server MUST NOT mix
  properties depending on different resources into the same resources, even if
  they have the same dependency types. There are two kinds of examples:

  - Assume there are a set of entities in IPv4 and IPv6 domain, and all of them
    have the property `pid`. The `pid` properties of entities in IPv4 domain
    depend on the network map `net1`, but the `pid` properties of entities in
    IPv6 domain depend on another network map `net2`. Although the property
    types `ipv4:pid` and `ipv6:pid` have the same dependency type sequence
    `["application/alto-networkmap+json"]`, the ALTO server cannot put them into
    the same property map.
  - Assume there are a set of entities in IPv4 domain, and each of them have two
    `pid` properties. One `pid` property depends on the network map `net1`, the
    other `pid` property depends on another network map `net2`. To distinguish
    them, the ALTO server can provide two `resource-specific endpoint
    properties` called `pid.net1` and `pid.net2` for each IPv4 entities in the
    same endpoint property map. But using the property map service defined in
    this document, the ALTO server has to define two individual property maps.
    Both property maps provide the property type `ipv4:pid`, but one depends on
    `net1` and the other one depends on `net2`.

To specify the aforementionned dependencies, this document uses the "uses" and
"dependent-vtags" fields defined respectively in Sections 9.1.5 and 11.1 of
[RFC7285].

- the "uses" field is included in the IRD entry of a resources-dependent
  information resource and specifies the dependent IRD resource.
- the "dependent-vtags" member is used in a Server response message to specify
  the dependent resource.
-->
