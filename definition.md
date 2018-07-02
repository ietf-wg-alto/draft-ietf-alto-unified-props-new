# Definitions and Concepts

## Entity

The entity is an extended concept of the endpoint defined in Section 2.1 of [](#RFC7285). An entity is an object with a (possibly empty) set of properties. Every entity is in a domain, such as the IPv4 and IPv6 domains, and has a unique address.

## Entity Domain

An entity domain is a family of entities. Two examples are the Internet address and PID domain (see [](#inet-addr-domain) and [](#pid-domain)) that this document will define.
<!-- An additional example is the proposed domain of Abstract Network Elements associated with topology and routing, as suggested by [](#I-D.ietf-alto-path-vector). -->

## Domain Name {#domain-names}

Each entity domain has a unique name. A domain name MUST be no more than 32
characters, and MUST NOT contain characters other than US-ASCII alphanumeric characters (U+0030-U+0039, U+0041-U+005A, and U+0061-U+007A), hyphen ('-', U+002D), and low line ('\_', U+005F). For example, the names `ipv4` and `ipv6` identify objects in the Internet address domain (see [](#inet-addr-domain)).

The type DomainName is used in this document to denote a JSON string with a domain name in this format.

Domain names MUST be registered with the IANA, and the format of the entity addresses in that entity domain, as well as any hierarchical or inheritance rules for those entities, MUST be specified at the same time.

## Entity Address {#entity-addrs}

Each entity has a unique address of the format:

``` text
    domain-name : domain-specific-entity-address
```

Examples from the IP domain include individual addresses such as `ipv4:192.0.2.14` and `ipv6:2001:db8::12`, as well as address blocks such as `ipv4:192.0.2.0/26` and `ipv6:2001:db8::1/48`.

The type EntityAddr is used in this document to denote a JSON string with an entity address in this format.

The format of the second part of an entity address depends on the entity domain, and MUST be specified when registering a new entity domain. Addresses MAY be hierarchical, and properties MAY be inherited based on that hierarchy. Again, the rules defining any hierarchy or inheritance MUST be defined when the entity domain is registered.

Note that an entity address MAY have different textual representations, for a given entity domain. For example, the strings `ipv6:2001:db8::1` and `ipv6:2001:db8:0:0:0:0:0:1` refer to the same entity.

## Property Name ##

The space of property names associated with entities defined by this document is the same as, and is shared with, the endpoint property names defined by [](#RFC7285).  Thus entity property names are as defined in Section 10.8.2 of that document, and must be registered with the `ALTO Endpoint Property Type Registry` defined in [](#IANAEndpointProp) of that document. The type PropertyName denotes a JSON string with a property name in this format.

 This document defines uniform property names specified in a single property name space rather than being scoped by a specific entity domain, although some properties may only be applicable for particular entity domains.  This design decision is to enforce a design so that similar properties are named similarly.  The interpretation of the value of a property, however, may depend on the entity domain.  For example, suppose the `geo-location` property is defined as the coordinates of a point, encoded as (say) "latitude longitude [altitude]."  When applied to an entity that represents a specific host computer, such as an Internet address, the property defines the host’s location.  When applied to an entity that represents a set of computers, such as a CIDR, the property would be the location of the center of that set.  If it is necessary to represent the bounding box of a set of hosts, another property, such as `geo-region`, should be defined.

## Hierarchy and Inheritance {#def-hierarchy-and-inheritance}

Entities in a given domain MAY form hierarchy based on entity address.
Each entity domain MUST define its own hierarchy and inheritance rules when registered. The hierarchy and inheritance rule makes it possible for an entity to inherit a property value from another entity in the same domain. If and only if the property of an entity is undefined, the hierarchy and inheritance rules are applied.


## Relationship with Other ALTO Resources {#def-relationship-to-other-resources}

[](#RFC7285) recognizes that some properties MAY be specific to another ALTO resource, such as a network map. Accordingly [](#RFC7285) defines the concept of `resource-specific endpoint properties` (see Section 10.8.1), and indicates that dependency by prefixing the property name with the ID of the resource on which it depends. That document defines one resource-specific property, namely the `pid` property, whose value is the name of the PID containing that endpoint in the associated network map.

This document takes a different approach. Instead of defining the dependency by qualifying the property name, this document attaches the dependency to the entity domains. Thus all properties of a specific entity domain depend on the same resource, the properties of another entity domain may depend on another resource. For example, entities in the PID domain depend on a network map.
<!-- , entities in the ANE domain depend on a cost map or a endpoint cost map. -->

The `uses` field in an IRD entry defines the dependencies of a property map resource, and the `dependent-vtags` field in a property map response defines the dependencies of that map. These fields are defined in Sections 9.1.5 and 11.1 of [](#RFC7285), respectively.

The `uses` field in an IRD entry MUST NOT include two dependent resources with the same media type. This is similar to how [](#RFC7285) handles dependencies between cost maps and network maps. Recall that cost maps present the costs between PIDs, and PID names depend on a network map. If an ALTO server provides the `routingcost` metric for the network maps `net1` and `net2`, then the server defines two separate cost maps, one for `net1` and the other for `net2`.

According to [](#RFC7285), a legacy ALTO server with two network maps, with resource IDs `net1` and `net2`, could offer a single Endpoint Property Service for the two properties `net1.pid` and `net2.pid`. An ALTO server which supports the extensions defined in this document, would, instead, offer two different Property Maps for the `pid` property, one depending on `net1`, the other on `net2`.
