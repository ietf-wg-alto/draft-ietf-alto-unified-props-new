# Definitions and Concepts

## Entity

The entity is a generalized concept of the endpoint defined in Section 2.1 of
[](#RFC7285). An entity is an object with a (possibly empty) set of properties.
Each entity MUST be in one and only one domain, such as the IPv4 domain or the
IPv6 domain, and has a unique address.

## Entity Domain

An entity domain is a set of entities. Examples of domains are the Internet
address domains (see [](#inet-addr-domain) and the PID domain (see
[](#pid-domain)). This document will define the domains precisely below.
<!-- An additional example is the proposed domain of Abstract Network Elements
associated with topology and routing, as suggested by
[](#I-D.ietf-alto-path-vector). -->

## Domain Name {#domain-names}

Each entity domain has a unique name. A domain name MUST be no more than 32
characters, and MUST NOT contain characters other than US-ASCII alphanumeric
characters (U+0030-U+0039, U+0041-U+005A, and U+0061-U+007A), hyphen ('-',
U+002D), and low line ('\_', U+005F). For example, the names `ipv4` and `ipv6`
identify entities in the Internet address domains (see [](#inet-addr-domain)).

The type DomainName is used in this document to denote a JSON string with
a domain name in this format.

Domain names MUST be registered with the IANA, and the format of the entity
addresses (see [](#entity-addrs)) in that entity domain, as well as any
hierarchical or inheritance rules (see [](#def-hierarchy-and-inheritance)) for
those entities, MUST be specified at the same time.

## Entity Address {#entity-addrs}

Each entity has a unique address of the format:

``` text
    EntityAddr ::= DomainName : DomainSpecificEntityAddr
```

Examples from the IP domains include individual addresses such as
`ipv4:192.0.2.14` and `ipv6:2001:db8::12`, as well as address blocks such as
`ipv4:192.0.2.0/26` and `ipv6:2001:db8::1/48`.

The type EntityAddr is used in this document to denote a JSON string with an
entity address in this format.

The format of the second part of an entity address depends on the entity
domain, and MUST be specified when registering a new entity domain. Addresses
MAY be hierarchical, and properties MAY be inherited based on that hierarchy.
Again, the rules defining any hierarchy or inheritance MUST be defined when the
entity domain is registered.

Note that an entity address MAY have different textual representations, for
a given entity domain. For example, the strings `ipv6:2001:db8::1` and
`ipv6:2001:db8:0:0:0:0:0:1` refer to the same entity.

## Property Name ##

<!-- FIXME: This is not correct. Because the ALTO entity domain is not a strict
superset of the ALTO address type. Revise it! -->
The space of entity property names associated with entities defined by this
document is a superset of the endpoint property names defined by [](#RFC7285).
Thus endpoint property names registered with the `ALTO Endpoint Property Type
Registry` MUST be defined in [](#IANAEndpointProp) of this document. The type
PropertyName denotes a JSON string with a property name in this format.

This document defines property names in the domain-specific context. This
design is to enforce that each property name MUST be registered for every
applicable entity domains individually. This design decision is adopted because
of the following considerations:

- Some properties may only be applicable for particular entity domains, not
  all. For example, the `pid` property is not applicable for entities in the
  `pid` domain.
- The interpretation of the value of a property may depend on the entity
  domain. For different entity domains, not only the intended semantics but
  also the dependent resource types may be totally different. For example,
  suppose that the `geo-location` property is defined as the coordinates of
  a point, encoded as (say) "latitude longitude [altitude]." When applied to an
  entity that represents a specific host computer, such as an Internet address,
  the property defines the host's location and has no required dependency.
  However, when applied to an entity in the `pid` domain, the property would
  indicate the location of the center of all hosts in this `pid` entity and
  depend on a Network Map defining this `pid` entity.

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

## Hierarchy and Inheritance {#def-hierarchy-and-inheritance}

Entities in a given domain MAY form a hierarchy based on entity addresses, and
introducing hierarchy allows the introduction of inheritance. Each
entity domain MUST define its own hierarchy and inheritance rules when
registered. The hierarchy and inheritance rule makes it possible for an entity
to inherit a property value from another entity in the same domain.
<!--If and only
if the property of an entity is undefined, the hierarchy and inheritance rules
are applied. [YRY: Do we need this?] [Jensen: I think this feature is for reducing the response size.] -->

## Relationship with Other ALTO Resources {#def-relationship-to-other-resources}

[](#RFC7285) recognizes that some properties for some entity domains MAY be
specific to an ALTO resource, such as a network map. Accordingly Section 10.8.1
of [](#RFC7285) defines the concept of `resource-specific endpoint properties`,
and indicates that dependency by prefixing the property name with the ID of the
resource on which it depends. That document defines one resource-specific
property, namely the `pid` property, whose value is the name of the PID
containing that endpoint in the associated network map.

This document takes a different approach. Instead of defining the dependency by
qualifying the property name, this document attaches the dependency to the
entity domains. Thus each resource-specific property of all entities in a
specific domain depends on the same resources; the properties of entities in
another domain may depend on another resource. For example, in a single property
map, the `pid` property of all entities in an Internet address domain MUST
depend on the same network map. Each property of all entities in the PID domain
MUST also depend on a network map; but different properties may depend on
different network maps.

<!--
This document takes a different approach. Instead of defining the dependency by
qualifying the property name, this document attaches the dependency to the
entity domains. Thus all properties of a specific entity domain depend on the
same resources (see below); the properties of another entity domain may depend on another
resource. For example, entities in the PID domain depend on a network map.
-->

Specifically, this document uses the `uses` and `dependent-vtags` fields defined
in Sections 9.1.5 and 11.1 of [](#RFC7285), respectively, to specify the
preceding dependency: the `uses` field of an IRD entry providing entity domain
related resources (see Property Map and Filtered Property Map resources below)
specifies the dependent resources, and the `dependent-vtags` field specifies
dependency in message responses.
