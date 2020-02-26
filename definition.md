# Protocol Specification: Basic Data Type

## Entity Domain {#def-domain}

### Entity Domain Type {#domain-types}

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

### Entity Domain Name {#domain-names}

Each entity domain is identified by an entity domain name, a string of the
following format:

``` text
EntityDomainName ::= [ [ ResourceID ] '.' ] EntityDomainType
```

This document distinguishes three types of entity domains: resource-specific
entity domains, self-defined entity domains, and resource-agnostic entity domains. Their
entity domain names are derived as follows.

Each ALTO information resource MAY define a resource-specific entity domain
(which could be empty) in a given entity domain type. A resource-specific entity
domain is identified by an entity domain name derived as follows. It MUST start
with a resource ID using the ResourceID type defined in [](#RFC7285), followed
by the '.' separator (U+002E), followed by an EntityDomainType typed string. For
example, if an ALTO server provides two network maps `netmap-1` and `netmap-2`,
they can define two different `pid` domains identified by `netmap-1.pid` and
`netmap-2.pid` respectively. To be simplified, in the scope of a specific
information resource, the resource-specific entity domain defined by itself can
be identified by the '.' EntityDomainTyep without the ResourceID.

When the associated information resource of a resource-specific entity domain is
the current information resource itself, this resource-specific entity domain is
a self-defined entity domain, and its ResourceID SHOULD be ignored from its
entity domain name.

Given a set of ALTO information resources, there MAY be a resource-agnostic entity
domain in a given entity domain type amongst them. A resource-agnostic entity domain
is simply identified by its entity domain type. For example, given two network
maps `net-map-1` and `net-map-2`, `ipv4` and `ipv6` identify two resource-agnostic
Internet address entity domains (see [](#inet-addr-domain)) between them.

<!--
Each entity domain type may have a global entity domain. For a global entity
domain (i.e., not resource-specific), its entity domain name is an
EntityDomainType typed string. For example, the `ipv4` and `ipv6` entity domain
types identify two Internet address entity domains (see [](#inet-addr-domain)).
-->

Note that the '.' separator is not allowed in EntityDomainType and hence there
is no ambiguity on whether an entity domain name refers to a global entity
domain or a resource-specific entity domain.

<!--
For an EntityDomainType which allows resource-specific entity domains, the valid
type(s) resources MUST be specified.
-->

### Entity Identifier {#entity-addrs}

<!-- FIXME: The entity identifier is not global unique. -->

Entities in an entity domain are identified by entity identifiers (EntityID) of
the following format:

``` text
EntityID ::= EntityDomainName ':' DomainTypeSpecificEntityID
```

Examples from the Internet address entity domains include individual IP
addresses such as `net1.ipv4:192.0.2.14` and `net1.ipv6:2001:db8::12`, as well
as address blocks such as `net1.ipv4:192.0.2.0/26` and
`net1.ipv6:2001:db8::1/48`.

The format of the second part of an entity identifier depends on the entity
domain type, and MUST be specified when registering a new entity domain type.
Identifiers MAY be hierarchical, and properties MAY be inherited based on that
hierarchy. Again, the rules defining any hierarchy or inheritance MUST be
defined when the entity domain type is registered.

The type EntityID is used in this document to denote a JSON string
representing an entity identifier in this format.

Note that two entity identifiers with different textual representations may
refer to the same entity, for a given entity domain. For example, the strings
`net1.ipv6:2001:db8::1` and `net1.ipv6:2001:db8:0:0:0:0:0:1` refer to the same
entity in the 'ipv6' entity domain.

### Hierarchy and Inheritance {#def-hierarchy-and-inheritance}

To make the representation efficient, some types of entity domains MAY allow the
ALTO client/server to use a hierarchical format entity identifier to represent a
block of individual entities. e.g., In an IPv4 domain `net1.ipv4`, a cidr
`net1.ipv4:192.0.2.0/26` represents 64 individual IPv4 entities. In this case,
the corresponding property inheritance rule MUST be defined for the entity
domain type. The hierarchy and inheritance rule MUST have no ambiguity.

<!--If and only
if the property of an entity is undefined, the hierarchy and inheritance rules
are applied. [YRY: Do we need this?] [Jensen: I think this feature is for reducing the response size.] -->

## Entity Property {#def-property}

Each entity property has a type to indicate the encoding and the semantics of
the value of this entity property, and has a name to be identified. One entity
MAY have multiple properties in the same type.

### Entity Property Type {#def-property-type}

The type EntityPropertyType is used in this
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
- The intended semantics of the value of an entity property may also depend on
  the entity domain type of this entity. For example, suppose that the
  `geo-location` property is defined as the coordinates of a point, encoded as
  (say) "latitude longitude [altitude]." When applied to an entity that
  represents a specific host computer, identified by an address in the `ipv4` or
  `ipv6` entity domain, the property defines the host's location. However, when
  applied to an entity in a `pid` domain, the property would indicate the
  location of the center of all hosts in this `pid` entity.

### Entity Property Name

<!-- FIXME: remove most. Use RFC 7285 Section 10.8, for resource-specific
properties and global properties. -->

Each entity property is identified by an entity property name, which is a string
of the following format:

``` text
EntityPropertyName ::= [ ResourceID ] '.' EntityPropertyType
```

Similar to the endpoint property type defined in Section 10.8 of [](#RFC7285),
each entity property may be defined by either the property map itself
(self-defined) or some other specific information resource (resource-specific).

The entity property name of a resource-specific entity property starts with a
string of the type ResourceID defined in [](#RFC7285), followed by the '.'
separator (U+002E) and a EntityDomainType typed string. For example, the `pid`
properties of an `ipv4` entity defined by two different maps `net-map-1` and
`net-map-2` are identified by `net-map-1.pid` and `net-map-2.pid` respectively.

When the associated information resource of the entity property is the current
information resource itself, the ResourceID in the property name SHOULD be
ignored. For example, the `.asn` property of an `ipv4` entity indicates the AS
number of the AS which this IPv4 address is owned by.

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

<!--
## Resource

A resource indicates an ALTO information resource in this document.

### Resource Type {#def-resource-type}

Each resource has a type identified by a JSON string, which aliases to a media
type of the response of an ALTO information resource.

When a new ALTO service is defined and introduces a new media type of response,
a new resource type SHOULD be defined as well.

Each resource type MUST be registered with the IANA. The aliased media type MUST
be specified.
-->

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
