# Definitions and Concepts

## Entity

The entity is an extended concept of the endpoint defined in Section 2.1 of
[](#RFC7285). An entity is an object with a (possibly empty) set of properties.
Every entity is in one and only one domain, such as the IPv4 domain or the
IPv6 domain, and has a unique address.

## Entity Domain

An entity domain is a set of entities. Examples of domains are the 
Internet address domains (see [](#inet-addr-domain) and the PID 
domain (see [](#pid-domain)). This document will define the domains
precisely below.
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
addresses (see [](entity-addrs)) in that entity domain, as well as any
hierarchical or inheritance rules (see [](#def-hierarchy-and-inheritance)) for
those entities, MUST be specified at the same time.

## Entity Address {#entity-addrs}

Each entity has a unique address of the format:

``` text
    domain-name : domain-specific-entity-address
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

The space of entity property names associated with entities defined by this document
is the same as, and is shared with, the endpoint property names defined by
[](#RFC7285).  Thus entity property names are as defined in Section 10.8.2 of
that document, and MUST be registered with the `ALTO Endpoint Property Type
Registry` defined in [](#IANAEndpointProp) of that document. The type
PropertyName denotes a JSON string with a property name in this format.

This document defines uniform property names specified in a single property
name space rather than being scoped by a specific entity domain, although some
properties may only be applicable for particular entity domains.  This design
decision is to enforce a design so that similar properties are named similarly.
The interpretation of the value of a property, however, may depend on the
entity domain.  For example, suppose that the `geo-location` property is defined as
the coordinates of a point, encoded as (say) "latitude longitude [altitude]."
When applied to an entity that represents a specific host computer, such as an
Internet address, the property defines the host’s location.  When applied to an
entity that represents a set of computers, such as a CIDR, the property would
be the location of the center of that set.  If it is necessary to represent the
bounding box of a set of hosts, another property, such as `geo-region`, should
be defined.

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

[](#RFC7285) recognizes that some properties MAY be specific to an ALTO
resource, such as a network map. Accordingly Section 10.8.1 of [](#RFC7285) defines the concept
of `resource-specific endpoint properties`, and indicates
that dependency by prefixing the property name with the ID of the resource on
which it depends. That document defines one resource-specific property, namely
the `pid` property, whose value is the name of the PID containing that endpoint
in the associated network map.

This document takes a different approach. Instead of defining the dependency by
qualifying the property name, this document attaches the dependency to the
entity domains. Thus each resource-specific property of all entities in a
specific domain depends on the same resources; the properties of entities in
another domain may depend on another resource. For example, in a single property
map, the `pid` property of all entities in an Internet address domain MUST
depend on a unique network map.

Meanwhile, some entity domains may be resource-specific, which means an entity
address in such domains MUST depend on another resource. For example, entities
in the PID domain depend on a network map.
<!-- , entities in the ANE domain depend on a cost map or a endpoint cost map. -->

Specifically, this document uses the `uses` and `dependent-vtags` fields 
defined in Sections 9.1.5 and 11.1 of [](#RFC7285), 
respectively, to specify the preceding dependency: the `uses` field of an 
IRD entry providing entity domain related resources (see Property Map and Filtered Property Map resources below) specifies the dependent resources, 
and the `dependent-vtags` field specifies dependency in message responses.
