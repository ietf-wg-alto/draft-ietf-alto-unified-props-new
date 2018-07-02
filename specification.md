# Entity Domains

This document defines the following entity domains. For the definition of each
entity domain, it includes the following template: domain name, domain-specific
addresses, and hierarchy and inheritance semantics.

## Internet Address Domains {#inet-addr-domain}

The document defines two entity domains (IPv4 and IPv6) for Internet addresses. Both
entity domains include individual addresses and blocks of addresses.

### IPv4 Domain {#ipv4-domain}

#### Domain Name

ipv4

#### Domain-Specific Entity Addresses

Individual addresses are strings as specified by the IPv4Addresses rule of
Section 3.2.2 of [](#RFC3986). Blocks of addresses are prefix-match strings as
specified in Section 3.1 of  [](#RFC4632). For the purpose of defining
properties, an individual Internet address and the corresponding full-length
prefix are considered aliases for the same entity. Thus `ipv4:192.0.2.0` and
`ipv4:192.0.2.0/32` are equivalent.

### IPv6 Domain {#ipv6-domain}

#### Domain Name

ipv6

#### Domain-Specific Entity Addresses

Individual addresses are strings as specified by Section 4 of [](#RFC5952).
Blocks of addresses are prefix-match strings as specified in Section 7 of
[](#RFC5952). For the purpose of defining properties, an individual Internet
address and the corresponding 128-bit prefix are considered aliases for the
same entity. That is, `ipv6:2001:db8::1` and `ipv6:2001:db8::1/128` are
equivalent, and have the same set of properties.

### Hierarchy and Inheritance of ipv4/ipv6 Domains {#inet-inheritance}

Both entity domains allow property values to be inherited. Specifically, if a property
P is not defined for a specific Internet address I, but P is defined for some
block C which prefix-matches I, then the address I inherits the value of
P defined for block C. If more than one such block defines a value for P, I
inherits the value of P in the block with the longest prefix. It is important
to notice that this longest prefix rule will ensure no multiple inheritance,
and hence no ambiguity.

Address blocks can also inherit properties: if property P is not defined for
a block C, but is defined for some block C' which prefix-matches C, and C' has
a shorter mask than C, then block C inherits the property from C'. If there are
several such blocks C', C inherits from the block with the longest prefix.

As an example, suppose that a server defines the property P for the following entities:

``` text
       ipv4:192.0.2.0/26: P=v1
       ipv4:192.0.2.0/28: P=v2
       ipv4:192.0.2.0/30: P=v3
       ipv4:192.0.2.0:    P=v4
```
^[fig:def-prop-val::Defined Property Values.]

Then the following entities have the indicated values:

``` text
       ipv4:192.0.2.0:    P=v4
       ipv4:192.0.2.1:    P=v3
       ipv4:192.0.2.16:   P=v1
       ipv4:192.0.2.32:   P=v1
       ipv4:192.0.2.64:   (not defined)
       ipv4:192.0.2.0/32: P=v4
       ipv4:192.0.2.0/31: P=v3
       ipv4:192.0.2.0/29: P=v2
       ipv4:192.0.2.0/27: P=v1
       ipv4:192.0.2.0/25: (not defined)
```

^[fig:inh-prop-val::Inherited Property Values.]

<!-- Improve words of this paragraph. (Done, waiting for review) -->

An ALTO Server MAY explicitly indicate a property as not having a value for
a particular entity. That is, a server MAY say that property A of entity X is
"defined to have no value", instead of "undefined". To indicate "no value",
a server MAY perform different behaviours:

- If that entity would inherit a value for that property, then the ALTO server
  MUST return a `null` value for that property. In this case, the ALTO client
  MUST recognize a `null` value as "no value" and "do not apply the inheritance
  rules for this property."
- If the entity would not inherit a value, then the ALTO server MAY return
  `null` or just omit the property. In this case, the ALTO client cannot infer
  the value for this property of this entity from the Inheritance rules. So the
  client MUST interpret this property has no value.

If the ALTO Server does not define any properties for an entity, then the
server MAY omit that entity from the response.

### Relationship to Network Maps

An Internet address domain MAY be associated with an ALTO network map resource. Logically, there is a map of Internet address entities to property values for each network map defined by the ALTO server, plus an additional property map for Internet address entities which are not associated with a network map. So, if there are n network maps, the server can provide n+1 maps of Internet address entities to property values. These maps are separate from each other. The prefixes in the property map do not have to correspond to the prefixes defining the network map's PIDs. For example, the property map for a network map MAY assign properties to `ipv4:192.0.2.0/24` even if that prefix is not associated with any PID in the network map.

## PID Domain {#pid-domain}

The PID domain associates property values with the PIDs in a network map.
Accordingly, this entity domain always depends on a network map.

### Domain Name

pid

### Domain-Specific Entity Addresses

The entity addresses are the PID names of the associated network map.

### Hierarchy and Inheritance

There is no hierarchy or inheritance for properties associated with PIDs.

### Relationship To Internet Addresses Domains

The PID domain and the Internet address domains are completely independent; the
properties associated with a PID have no relation to the properties associated
with the prefixes or endpoint addresses in that PID. An ALTO server MAY choose
to assign some or all properties of a PID to the prefixes in that PID.

For example, suppose `PID1` consists of the prefix `ipv4:192.0.2.0/24`, and has
the property `P` with value `v1`. The Internet address entities
`ipv4:192.0.2.0` and `ipv4:192.0.2.0/24`, in the IPv4 domain MAY have a value
for the property `P`, and if they do, it is not necessarily `v1`.

## Internet Address Properties vs. PID Properties

Because the Internet address and PID domains are completely separate, the
question may arise as to which entity domain is the best for a property. In general, the
Internet address domain is RECOMMENDED for properties that are closely related
to the Internet address, or are associated with, and inherited through, blocks
of addresses.

The PID domain is RECOMMENDED for properties that arise from the definition of
the PID, rather than from the Internet address prefixes in that PID.

For example, because Internet addresses are allocated to service providers by
blocks of prefixes, an `ISP` property would be best associated with the
Internet address domain. On the other hand, a property that explains why a PID
was formed, or how it relates a provider's network, would best be
associated with the PID domain.

<!-- Begin of removing -->
<!--
## ANE Domain {#ane-domain}

### Domain Name

ane

### Domain-Specific Entity Addresses

The entity address of ane domain is encoded as a JSON string.  The string MUST
be no more than 64 characters, and it MUST NOT contain characters other than
US-ASCII alphanumeric characters (U+0030-U+0039, U+0041-U+005A, and
U+0061-U+007A), the hyphen ('-', U+002D), the colon (':', U+003A), the at sign
('@', code point U+0040), the low line ('\_', U+005F), or the '.' separator
(U+002E). The '.' separator is reserved for future use and MUST NOT be used
unless specifically indicated in this document, or an extension document.

### Hierarchy and Inheritance

There is no hierarchy or inheritance for properties associated with ANEs.
-->
<!-- End of removing -->

# Property Map Resource {#prop-map}

A Property Map returns the properties defined for all entities in one or more
domains.

<!-- Note that Property Map Resource is not applicable to ANE domain. -->
<!-- It is not RECOMMENDED. But it depends on the implementation. -->

[](#prop-map-example) gives an example of a property map request and its
response.

## Media Type {#FullPropMapMediaType}

The media type of an ALTO Property Map resource is
`application/alto-propmap+json`.

## HTTP Method

An ALTO Property Map resource is requested using the HTTP GET method.

## Accept Input Parameters

None.

## Capabilities {#FullPropMapCapabilities}

The capabilities are defined by an object of type PropertyMapCapabilities:

``` text
    object {
      DomainName entity-domain-types<1..*>;
      PropertyName prop-types<1..*>;
    } PropertyMapCapabilities;
```

where `entity-domain-types` is an array with the domains of the entities in this
property map, and `prop-types` is an array with the names of the properties
returned for entities in those domains.

<!-- TODO: Maybe add some recommendation. Low priority -->
<!--
If the server declares multiple domain-types and multiple prop-types in the capability, each prop-type MUST be supported in each domain in the "capabilities" field. In other words, if a prop-type is NOT supported in a particular domain, the property map MUST be divided into several maps.
-->

## Uses

An array with the resource ID(s) of resource(s) with which the entity domains in this
map are associated. In most cases, this array will have at most one ID, for
example, for a network map resource. However, the `uses` field MUST NOT contain
two resources of the same resource type. For example, if a property map depends
on network map resource, the `uses` field MUST include exactly one network map
resource.

## Response {#FullPropMapResponse}

If the entity domains in this property map depend on other resources, the
`dependent-vtags` field in the `meta` field of the response MUST be an array
that includes the version tags of those resources. The data component of
a Property Map response is named `property-map`, which is a JSON object of type
PropertyMapData, where:

``` text
    object {
      PropertyMapData property-map;
    } InfoResourceProperties : ResponseEntityBase;

    object-map {
      EntityAddr -> EntityProps;
    } PropertyMapData;

    object {
      PropertyName -> JSONValue;
    } EntityProps;
```

The ResponseEntityBase type is defined in Section 8.4 of [](#RFC7285).

Specifically, a PropertyMapData object has one member for each entity in the
Property Map. The entity's properties are encoded in the corresponding
EntityProps object. EntityProps encodes one name/value pair for each property,
where the property names are encoded as strings of type PropertyName.
A protocol implementation SHOULD assume that the property value is either
a JSONString or a JSON `null` value, and fail to parse if it is not, unless the
implementation is using an extension to this document that indicates when and
how property values of other data types are signaled.

<!--An ALTO Server MAY explicitly define a property as not having a value for
a particular entity. That is, a server MAY say that a property is "defined to
have no value", as opposed to the property being "undefined".
If that entity would inherit a value for that property, then the ALTO server
MUST return a "null" value for that property, and an ALTO client MUST recognize
a "null" value means "do not apply the inheritance rules for this property."
If the entity would not inherit a value, the ALTO server MAY return "null" or
MAY just omit the property.-->

For each entity in the Property Map, the ALTO Server returns the value defined
for each of the properties specified in this resource's `capabilities` list.
For efficiency, the ALTO Server SHOULD omit property values that are inherited
rather than explicitly defined; if a client needs inherited values, the client
SHOULD use the entity domain's inheritance rules to deduce those values.

# Filtered Property Map Resource {#filter-prop-map}

A Filtered Property Map returns the values of a set of properties for a set of
entities selected by the client.

[](#filt-prop-map-example-1), [](#filt-prop-map-example-2) and
[](#filt-prop-map-example-3) give examples of filtered property map requests
and responses.

## Media Type {#FilterPropMapMediaType}

The media type of an ALTO Property Map resource is
`application/alto-propmap+json`.

## HTTP Method

An ALTO Filtered Property Map resource is requested using the HTTP POST method.

## Accept Input Parameters {#filter-prop-map-params}

The input parameters for a Filtered Property Map request are supplied in the
entity body of the POST request. This document specifies the input parameters
with a data format indicated by the media type
`application/alto-propmapparams+json`, which is a JSON object of type
ReqFilteredPropertyMap:

``` text
  object {
    EntityAddr     entities<1..*>;
    PropertyName   properties<1..*>;
  } ReqFilteredPropertyMap;
```

with fields:

entities:
~ List of entity addresses for which the specified properties are to be
returned. The ALTO server MUST interpret entries appearing multiple times as if
they appeared only once. The domain of each entity MUST be included in the list
of entity domains in this resource's `capabilities` field
(see [](#FilteredPropMapCapabilities)).

properties:
~ List of properties to be returned for each entity. Each specified property
MUST be included in the list of properties in this resource's `capabilities`
field (see [](#FilteredPropMapCapabilities)). The ALTO server MUST interpret
entries appearing multiple times as if they appeared only once.
~ Note that the `entities` and `properties` fields MUST have at least one entry
each.

## Capabilities {#FilteredPropMapCapabilities}

The capabilities are defined by an object of type PropertyMapCapabilities, as
defined in [](#FullPropMapCapabilities).

## Uses

An array with the resource ID(s) of resource(s) with which the entity domains in this
map are associated. In most cases, this array will have at most one ID, and it
will be for a network map resource.

## Response {#FilteredPropMapResponse}

The response is the same as for the property map (see [](#FullPropMapResponse)),
except that it only includes the entities and properties requested by the
client.

Also, the Filtered Property Map response MUST include all inherited
property values for the specified entities (unlike the Full Property
Map, the Filtered Property Map response does not include enough
information for the client to calculate the inherited values).

If an entity in `entities` in the request is invalid, the ALTO server MUST
return an `E_INVALID_FIELD_VALUE` error defined in Section 8.5.2 of
[](#RFC7285). An entity can be invalid if the domain of the entity is not
defined in the IRD for this service or the entity address is an invalid address
of the entity domain. On the other hand, a valid entity address is not an error, even
if the server does not define a value for a requested property. In this case,
the server MUST omit that property from the response for only that entity. If
a request contains a property in `properties` and the property is not specified
in the IRD for the service, the ALTO server MUST return an
`E_INVALID_FIELD_VALUE` error defined in Section 8.5.2 of [](#RFC7285). The
`value` of the error message SHOULD indicate the wrong property.

If the ALTO server does not define a requested property's value for
a particular entity, then it MUST omit that property from the response for only
that endpoint.

<!-- TODO: Errors must follow RFC7285 Section 8.5.2... -->
<!-- Check Section 11.4.1.6 of RFC7285. We need to make the behavior of UP
consistent with EPS. -->

If the ALTO server does not support a requested entity's domain, then it MUST
return an E_INVALID_FIELD_VALUE error defined in Section 8.5.2 of
[](#RFC7285).

<!--Discussion Needed: sometimes the client can compute some inherited property values.
In this case, can the Filter Property Map response only contain the uncomputable
inherited property values instead of all of them?-->

# Impact on Legacy ALTO Servers and ALTO Clients {#legacy}

## Impact on Endpoint Property Service

The Property Maps defined in this document provide the same functionality as
the Endpoint Property Service (EPS) defined in Section 11.4 of [](#RFC7285).
Accordingly, it is RECOMMENDED that the EPS be deprecated in favor of Property
Maps. However, ALTO servers MAY provide an EPS for the benefit of legacy
clients.

## Impact on Resource-Specific Properties

Section 10.8 of [](#RFC7285) defines two categories of endpoint properties:
`resource-specific` and `global`. Resource-specific property names are prefixed
with the ID of the resource they depend upon, while global property names
have no such prefix. The property map resources defined in this document do not
distinguish between those two types of properties. Instead, if there is
a dependency, it is indicated by the `uses` capability of a property map, and
is shared by all properties and entity domains in that map. Accordingly, it is
RECOMMENDED that resource-specific endpoint properties be deprecated, and no
new resource-specific endpoint properties be defined.

## Impact on the `pid` Property

Section 7.1.1 of [](#RFC7285) defines the resource-specific endpoint property
name `pid`, whose value is the name of the PID containing that endpoint. For
compatibility with legacy clients, an ALTO server which provides the `pid`
property via the Endpoint Property Service MUST use that definition, and that
syntax, in the EPS resource.

However, when used with Property Maps, this document amends the definition of
the `pid` property as follows.

First, the name of the property is simply `pid`; the name is not prefixed with
the resource ID of a network map. The `uses` capability of the property map
resource indicates the associated network map. This implies that a property map
can only return the `pid` property for one network map; if an ALTO server
provides several network maps, it MUST provide a property map resource for each
one.

Second, a client MAY request the `pid` property for a block of
addresses. An ALTO server determines the value of `pid` for an
address block C as follows. Let CS be the set of all address blocks
in the network map. If C is in CS, then the value of `pid` is the
name of the PID associated with C. Otherwise, find the longest block
C' in CS such that C' prefix-matches C, but is shorter than C. If
there is such a block C', the value of `pid` is the name of the PID
associated with C'. If not, then `pid` has no value for block C.

Note that although an ALTO server MAY provide a GET-mode property map resource
which returns the entire map for the `pid` property, there is no need to do so,
because that map is simply the inverse of the network map.

## Impact on Other Properties

In general, there should be little or no impact on other previously defined
properties. The only consideration is that properties can now be defined on
blocks of addresses, rather than just individual addresses, which might change
the semantics of a property.
