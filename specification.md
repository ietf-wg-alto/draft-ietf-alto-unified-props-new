<!-- FIXME: Make the specification coherent with the definition -->

# Entity Domain Types

This document defines three entity domain types. The definition of each entity
domain type below includes the following: (1) entity domain type name, (2)
entity domain-specific entity identifiers, and (3) hierarchy and inheritance
semantics. Since a global entity domain type defines a single global entity
domain, we say entity domain instead of entity domain type.

## Internet Address Domain Types {#inet-addr-domain}

The document defines two entity domain types (IPv4 and IPv6) for Internet addresses. Both
types are global entity domain types and hence define a corresponding global entity domain as well. 
Since the two domains use the same hierarchy and inheritance semantics, we define the semantics 
together, instead of repeating for each.

### IPv4 Domain {#ipv4-domain}

#### Entity Domain Type

ipv4

#### Domain-Specific Entity Identifiers

Individual addresses are strings as specified by the IPv4Addresses rule of
Section 3.2.2 of [](#RFC3986); blocks of addresses are prefix-match strings as
specified in Section 3.1 of  [](#RFC4632). For the purpose of defining
properties, an individual Internet address and the corresponding full-length
prefix are considered aliases for the same entity. Thus `ipv4:192.0.2.0` and
`ipv4:192.0.2.0/32` are equivalent.

### IPv6 Domain {#ipv6-domain}

#### Entity Domain Type

ipv6

#### Domain-Specific Entity Identifiers

Individual addresses are strings as specified by Section 4 of [](#RFC5952);
blocks of addresses are prefix-match strings as specified in Section 7 of
[](#RFC5952). For the purpose of defining properties, an individual Internet
address and the corresponding 128-bit prefix are considered aliases for the
same entity. That is, `ipv6:2001:db8::1` and `ipv6:2001:db8::1/128` are
equivalent, and have the same set of properties.

### Hierarchy and Inheritance of Internet Address Domains {#inet-inheritance}

Both Internet address domains allow property values to be inherited. Specifically,
if a property P is not defined for a specific Internet address I, but P is defined for some
block C which prefix-matches I, then the address I inherits the value of
P defined for block C. If more than one such block defines a value for P, I
inherits the value of P in the block with the longest prefix. It is important
to notice that this longest prefix rule will ensure no multiple inheritance,
and hence no ambiguity.

Address blocks can also inherit properties: if a property P is not defined for
a block C, but is defined for some block C' which covers all IP addresses in C,
and C' has a shorter mask than C, then block C inherits the property from C'.
If there are several such blocks C', C inherits from the block with the longest
prefix.

As an example, suppose that a server defines a property P for the following entities:

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

An ALTO server MAY explicitly indicate a property as not having a value for
a particular entity. That is, a server MAY say that property P of entity X is
"defined to have no value", instead of "undefined". To indicate "no value",
a server MAY perform different behaviours:

- If that entity would inherit a value for that property, then the ALTO server
  MUST return a `null` value for that property. In this case, the ALTO client
  MUST recognize a `null` value as "no value" and "do not apply the inheritance
  rules for this property."
- If the entity would not inherit a value, then the ALTO server MAY return
  `null` or just omit the property. In this case, the ALTO client cannot infer
  the value for this property of this entity from the Inheritance rules. So the
  client MUST interpret that this property has no value.

If the ALTO server does not define any properties for an entity, then the
server MAY omit that entity from the response.

<!--

### Relationship to Network Maps

Entities in an Internet address domain MAY be associated with multiple ALTO
network map resources. But according to
[](#def-relationship-to-other-resources), in a single property map, an Internet
address domain MUST be associated with at most one ALTO network map resource. To
achieve this, if there are n network maps in an ALTO server, the server may
provide at least n+1 property maps for entities an in Internet address domain.
i.e., for each network map, the ALTO server may provide an individual property
map associated with it. And then the ALTO server may provide a property map not
associated with any network maps.
-->

<!-- FIXED: (YRY: not fully clear) -->
<!--
An Internet address domain MAY be associated with an ALTO network map resource.
Logically, there is a map of Internet address entities to property values for
each network map defined by the ALTO server, plus an additional Property Map
for Internet address entities which are not associated with a network map.
So, if there are n network maps, the server can provide n+1 maps of Internet
address entities to property values. These maps are separate from each other.
The prefixes in the Property Map do not have to correspond to the prefixes
defining the network map's PIDs. For example, the Property Map for a network
map MAY assign properties to `ipv4:192.0.2.0/24` even if that prefix is not
associated with any PID in the network map.
-->

## PID Domain {#pid-domain}

The PID domain associates property values with the PIDs in a network map.
Accordingly, this entity domain always depends on a network map.

### Entity Domain Type

pid

### Domain-Specific Entity Identifiers

The entity identifiers are the PID names of the associated network map.

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
Internet address domains are RECOMMENDED for properties that are closely related
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

# Resource Types

<!-- TODO: May need to be moved to behind of property map spec. -->

## Network Map Resource

### Resource Type

networkmap

### Media Type

application/alto-networkmap+json

### Entity Domain Export {#netmap-ede}

A `networkmap` typed resource defines a `pid` domain, an `ipv4` domain
and an `ipv6` domain by follows:

- The defined `pid` domain includes all PIDs in keys of the `network-map`
  object.
- The defined `ipv4` domain includes all IPv4 addresses appearing in the `ipv4`
  field of the endpoint address group of each PID.
- The defined `ipv6` domain includes all IPv6 addresses appearing in the `ipv6`
  field of the endpoint address group of each PID.

### Entity Property Transfer {#netmap-ept}

For each of the preceding entity domains, an `networkmap` typed resource
provides the properties mapping as follows:

`ipv4 -> pid`:
~ An `networkmap` typed resource can map an `ipv4` entity to a `pid` property
  whose value is a PID defined by this `networkmap` resource and including the
  IPv4 address of this entity.

`ipv6 -> pid`:
~ An `networkmap` typed resource can map an `ipv6` entity to a `pid` property
  whose value is a PID defined by this `networkmap` resource and including the
  IPv6 address of this entity.
  
## Cost Map Resource

### Resource Type

costmap

### Media Type

application/alto+costmap+json
  
## Endpoint Cost Resource

### Resource Type

endpointcost

### Media Type

application/alto+endpointcost+json

## Endpoint Property Resource

### Resource Type

endpointprop

### Media Type

application/alto-endpointprop+json

### Entity Domain Export {#ep-ede}

TBD.

### Entity Property Transfer {#ep-ept}

TBD.

## Property Map Resource

### Resource Type

propmap

### Media Type

application/alto-propmap+json (See [](#FullPropMapMediaType).)

### Entity Domain Export {#up-ede}

TODO: property map is special and should be able to export any entity domains.

### Entity Property Transfer {#up-ept}

TODO: property map is special and should be able to transfer any entity properties.

# Property Map {#prop-map}

A property map returns the properties defined for all entities in one or more
domains, e.g., the `location` property of entities in `pid` domain, and the
`ASN` property of entities in `ipv4` and `ipv6` domains.

<!-- Note that Property Map Resource is not applicable to ANE domain. -->
<!-- It is not RECOMMENDED. But it depends on the implementation. -->

[](#prop-map-example) gives an example of a property map request and its
response.

## Media Type {#FullPropMapMediaType}

The media type of a property map is
`application/alto-propmap+json`.

## HTTP Method

The property map is requested using the HTTP GET method.

## Accept Input Parameters

None.

## Capabilities {#FullPropMapCapabilities}

The capabilities are defined by an object of type PropertyMapCapabilities:

``` text
    object {
      EntityDomainName entity-domains<1..*>;
      EntityPropertyName properties<1..*>;
    } PropertyMapCapabilities;
```

with fields:

entity-domains:
~ List of entity domain names of entity domains supported by this property map.
Each entity domain can be either a resource-specific entity domain defined by
one of dependent resource in the `uses` field, or the shared entity domain
amongst all dependent resources in the `uses` field. The ALTO client SHOULD
ignore a resource-specific entity domain if its entity domain type is not
registered in the ALTO Resource Entity Domain Export Registry of the type of the
corresponding resource.

properties:
~ List of entity property names to be returned for each entity in this property
map.

<!--
where the value of `entity-domains` is an array of entity domain names specifying the entity domains, and
the value of `properties` is an array of entity property names specifying the properties returned for entities in
those domains. The semantics is that this property map provides all property
types generated by the cross product of `entity-domains` and `properties`. If a
property in `properties` is NOT supported by a domain in `entity-domains`, the
server can declare different property maps to conform to the semantics.

For example, the capability {"entity-domains": ["ipv4", "ipv6"], "properties":
["pid"]} means the property map provides both property types `ipv4:pid` and
`ipv6:pid`.
-->

## Uses {#FullPropMapUses}

<!--
This is similar to how [](#RFC7285)
handles dependencies between cost maps and network maps. Recall that cost maps
present the costs between PIDs, and PID names depend on a network map. If an
ALTO server provides the `routingcost` metric for the network maps `net1` and
`net2`, then the server defines two separate cost maps, one for `net1` and the
other for `net2`.-->

The `uses` field of a property map resource in an IRD entry specifies
dependent resources of this property map. It is an array of the resource ID(s)
of the resource(s).

<!--
In a single property map, every property value of every entity depends on the
same array of resources. Thus, if properties depending on different resources
arrays would be provided, they MUST be split into different property maps.
-->

<!--
=== Below TBD: ===

The `uses` field of a property map resource in an IRD entry specifies
dependencies as discussed in Section 2.7. It is an array of the resource ID(s)
of the resource(s) that each domain in `entity-domains` depends on, in order to
provide the properties specified in the `properties` capability.

For example, the `pid` property for an ipv4 entity is a resource-specific
property depending on a specific network map. Assume that the `entity-domains`
capability of a property map resource includes `ipv4`, and the `properties`
capability includes `pid`. Then, the `uses` field MUST include the resource ID
of the specific network map resource.

In the general case, the `uses` field should not have ambiguity in specifying
dependencies. To achieve this goal, the ALTO server MUST ensure the ALTO client
can interpret the resource dependencies by the following `uses` rule:

- For each domain in `entity-domains`, take the full `uses` list, and then,
    - go over each property in `properties` capability in array order:
        - If the property is a resource-specific property for the current
          domain, and it needs a sequence of S resources, then
            - take the S resource ID(s) at the beginning of `uses` to
              interpret the property;
            - and remove the S resource ID(s) from `uses`.
-->

<!--
  go over each property in `properties` in array order
    if the property is a resource-specific property
                       and needs a sequence of S resources,
      the S resource ID(s) at the beginning of `uses` are used to
          interpret the property
      the S resource ID(s) are removed from `uses`
-->

<!--
In the special case when all entity domains or resource-specific properties
depend on the same resource, the `uses` list can only include this single
resource. For example, a property map providing `country` and `state` properties
for all entities in the `pid` domain can specify the `uses` as [
`default-networkmap` ], which means both `country` and `state` properties for
the `pid` domain are associated with the resource `default-networkmap`.

To simplify client verifying the `uses` rule, it is RECOMMENDED that
a single resource-specific property is specified in `properties` in each property
map resource.
-->

<!--
Note that according to [](#RFC7285), a legacy ALTO server with two network maps,
with resource IDs `net1` and `net2`, could offer a single Endpoint Property Service
for the two properties `net1.pid` and `net2.pid`. An ALTO server which supports
the property map resource defined in this document, would, instead, offer two different
property maps for the `pid` property, one depending on `net1`, and the other on
`net2`.
-->

<!-- Deprecate this process. See the principles of ALTO Entity Property Type Registry.

To make the client can understand the resource dependencies of a property map
correctly, the following processes are required:

1. Each combination of "entity-domain" and "property" SHOULD specify its
   dependent resource type explicitly. For example, "<ipv4, pid>" or "<ipv6, pid>"
   depends on a network map; <ane, pid> depends on a network map and a cost
   map.
2. Each combination of "entity-domain" and "property" SHOULD specify how to use
   the dependent resources to interpret this combination. For example, for
   "<pid4, pid>", the dependent network map is used to validate and interpret the
   pid property values; for "<ane, pid>", the dependent cost map is used to
   validate and interpret the entities in ane domain, and the dependent network
   map is used to validate and interpret the pid property values.
-->

## Response {#FullPropMapResponse}

If the entity domains in this property map depend on other resources, the
`dependent-vtags` field in the `meta` field of the response MUST be an array
that includes the version tags of those resources, and the order MUST be
consistent with the `uses` field of this property map resource. The data
component of a property map response is named `property-map`, which is a JSON
object of type PropertyMapData, where:

``` text
    object {
      PropertyMapData property-map;
    } InfoResourceProperties : ResponseEntityBase;

    object-map {
      EntityID -> EntityProps;
    } PropertyMapData;

    object {
      EntityPropertyName -> JSONValue;
    } EntityProps;
```

The ResponseEntityBase type is defined in Section 8.4 of [](#RFC7285).

Specifically, a PropertyMapData object has one member for each entity in the
property map. The entity's properties are encoded in the corresponding
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

For each entity in the property map:

- If the entity is in a resource-specific entity domain, the ALTO server SHOULD
  only return self-defined properties and resource-specific properties which
  depend on the same resource as the entity does. The ALTO client SHOULD ignore
  the resource-specific property in this entity if their mapping is not
  registered in the ALTO Resource Entity Property Transfer Registry of the type
  of the corresponding resource.
- If the entity is in a shared entity domain, the ALTO server SHOULD return
  self-defined properties and all resource-specific properties defined for all
  resource-specific entities which have the same domain-specific entity
  identifier as this entity does.

<!--
For each entity in the Property Map, the ALTO server returns the value defined
for each of the properties specified in this resource's `capabilities` list.
-->

For efficiency, the ALTO server SHOULD omit property values that are inherited
rather than explicitly defined; if a client needs inherited values, the client
SHOULD use the entity domain's inheritance rules to deduce those values.

# Filtered Property Map {#filter-prop-map}

A filtered property map returns the values of a set of properties for a set of
entities selected by the client.

[](#filt-prop-map-example-1), [](#filt-prop-map-example-2),
[](#filt-prop-map-example-3) and [](#filt-prop-map-example-4) give examples of
filtered property map requests and responses.

## Media Type {#FilterPropMapMediaType}

The media type of a property map resource is
`application/alto-propmap+json`.

## HTTP Method

The filtered property map is requested using the HTTP POST method.

## Accept Input Parameters {#filter-prop-map-params}

The input parameters for a filtered property map request are supplied in the
entity body of the POST request. This document specifies the input parameters
with a data format indicated by the media type
`application/alto-propmapparams+json`, which is a JSON object of type
ReqFilteredPropertyMap:

``` text
  object {
    EntityID             entities<1..*>;
    EntityPropertyName   properties<1..*>;
  } ReqFilteredPropertyMap;
```

with fields:

entities:
~ List of entity identifiers for which the specified properties are to be
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

Same to the `uses` field of the Property Map resource (see
[](#FullPropMapUses)).

<!--
The `uses` field of a filtered property map is an array with the resource ID(s)
of resource(s) that each domain in `entity-domains` depends on, in order to
provide the properties specified in the `properties` capability. The same `uses`
rule as defined by the property map resource applies (see [](#FullPropMapUses)).
-->

<!-- YRY: say refer to the same consistency of uses in Section 4.5. -->

## Response {#FilteredPropMapResponse}

The response MUST indicate an error, using ALTO protocol error handling, as
defined in Section 8.5 of [](#RFC7285), if the request is invalid.

Specifically, a filtered property map request can be invalid as follows:

- An entity identifier in `entities` in the request is invalid if:

    - The domain of this entity is not defined in the `entity-domains`
      capability of this resource in the IRD;
    - The entity identifier is an invalid identifier in the entity domain.

    A valid entity identifier is never an error, even if this filtered property
    map resource does not define any properties for it.

    If an entity identifier in `entities` in the request is invalid, the ALTO
    server MUST return an `E_INVALID_FIELD_VALUE` error defined in Section
    8.5.2 of [](#RFC7285), and the `value` field of the error message SHOULD
    indicate this entity identifier.

- A property name in `properties` in the request is invalid if this
  property name is not defined in the `properties` capability of this
  resource in the IRD.

    It is not an error that a filtered property map
    resource does not define a requested property's value for a particular
    entity. In this case, the ALTO server MUST omit that property from the
    response for that endpoint.

    If a property name in `properties` in the request is invalid, the ALTO
    server MUST return an `E_INVALID_FIELD_VALUE` error defined in Section
    8.5.2 of [](#RFC7285). The `value` field of the error message SHOULD
    indicate the property name.
    
The response to a valid request is the same as for the Property Map
(see [](#FullPropMapResponse)), except that:

- If the requested entities include entities in the shared entity domain, the
  `dependent-vtags` field in its `meta` field MUST include version tags of all
  dependent resources appearing in the `uses` field.
- If the requested entities only include entities in resource-specific entity
  domains, the `dependent-vtags` field in its `meta` field MUST include version
  tags of resources which requested resource-specific entity domains and
  requested resource-specific properties are dependent on.
- The response only includes the entities and properties requested by the
  client. If an entity in the request is identified by a hierarchical identifier
  (e.g., an `ipv4` or `ipv6` address block), the response MUST cover properties
  for all identifiers in this hierarchical identifier.

<!-- FIXME: do we define the term "block" / "address block"? it seems to be
always used to describe an entity from which there are other entities
inheriting. -->

It is important that the filtered property map response MUST include all
inherited property values for the requested entities and all the entities which
are able to inherit property values from them. To achieve this goal, the ALTO
server MAY follow three rules:

<!-- NOTE: We use "MAY" here because there may be multiple solutions achieving
this goal. As long as the ALTO client can interpret all properties for all
requested singleton entities correctly. -->

- If a property for a requested entity is inherited from another entity not
  included in the request, the response SHOULD include this property for the
  requested entity. For example, A full property map may skip a property P for
  an entity A (e.g., ipv4:192.0.2.0/31) if P can be derived using inheritance
  from another entity B (e.g., ipv4:192.0.2.0/30). A filtered property map
  request may include only A but not B. In such a case, the property P SHOULD be
  included in the response for A.
- If there are entities covered by a requested entity but having different
  values for the requested properties, the response SHOULD include all those
  entities and the different property values for them. For example, considering
  a request for property P of entity A (e.g., ipv4:192.0.2.0/31), if P has value
  v1 for A1=ipv4:192.0.2.0/32 and v2 for A2=ipv4:192.0.2.1/32, then, the
  response SHOULD include A1 and A2.
- If an entity in the response is already covered by some other entities in the
  same response, it SHOULD be removed from the response for compactness. For
  example, in the previous example, the entity A=ipv4:192.0.2.0/31 SHOULD be
  removed because A1 and A2 cover all the addresses in A.

An ALTO client should be aware that the entities in the response MAY be
different from the entities in its request.

<!--
It is possible that the entities in the response are different from the entities
in the request. Consider a request for property P of entity A (e.g.,
ipv4:192.0.2.0/31). Assume that P has value v1 for A1=ipv4:192.0.2.0/32 and v2
for A2=ipv4:192.0.2.1/32. Then, the response will include entities A1 and A2,
instead of the request entity A.
-->

<!--
An ALTO client should be aware that the entities in the response MAY be
different from the ones it requests. If entities in the requested domain can be
inherited, the ALTO server MAY decompose a requested entity address into several
entities which could inherit it. One example is the Internet Address domains:
Considering a request for property P of entity A (e.g., ipv4:192.0.2.0/31), if P
has value v1 for A1=ipv4:192.0.2.0/32 and v2 for A2=ipv4:192.0.2.1/32, then the
ALTO server could return the response including entities A1 and A2, instead of
the requested entity A. The ALTO server could also return v1 for A1 and v2 for
A, and the ALTO client can also deduce v2 for A2 from the inheritance.
-->

<!--
An operator should be aware that if the ALTO server supports the entities
decomposition, there will be potential security considerations. [](#SecSC)
discusses the details and potential solutions.
-->

<!--
YRY: Need to make a decision. It is possible that the entities in the
response are different from the entities in the request. Consider
a request for property P of entity A (e.g., ipv4:192.0.2.0/31). Assume
that P has value v1 for A1=ipv4:192.0.2.0/32 and v2 for A2=ipv4:192.0.2.1/32.
Then, the response will include entities A1 and A2, instead of the request
entity A.

Jensen: This process looks better. But if accept it, we need to add more
security considerations, e.g., if a client request a full wildcard 0.0.0.0/0,
the server has to actually return the full map.
-->

<!-- Errors must follow RFC7285 Section 8.5.2... -->
<!-- Check Section 11.4.1.6 of RFC7285. We need to make the behavior of UP
consistent with EPS. -->

<!--Discussion Needed: sometimes the client can compute some inherited property values.
In this case, can the Filter Property Map response only contain the uncomputable
inherited property values instead of all of them?-->

# Impact on Legacy ALTO Servers and ALTO Clients {#legacy}

## Impact on Endpoint Property Service

Since the property map and the filtered property map defined in this document provide the
functionality of the Endpoint Property Service (EPS) defined in Section 11.4
of [](#RFC7285), it is RECOMMENDED that the EPS be deprecated in favor of Property
Map and Filtered Property Map. However, ALTO servers MAY provide an EPS for
the benefit of legacy clients.

## Impact on Resource-Specific Properties

Section 10.8 of [](#RFC7285) defines two categories of endpoint properties:
`resource-specific` and `global`. Resource-specific property names are prefixed
with the ID of the resource they depend upon, while global property names have
no such prefix. The property map and the filtered property map defined in this
document do not distinguish between those two types of properties. Instead, if
there is a dependency, it is indicated by the `uses` capability of a property
map, and is shared by all properties and entity domains in that map.
Accordingly, it is RECOMMENDED that resource-specific endpoint properties be
deprecated, and no new resource-specific endpoint properties be defined.

## Impact on the `pid` Property

Section 7.1.1 of [](#RFC7285) defines the resource-specific endpoint property
name `pid`, whose value is the name of the PID containing that endpoint. For
compatibility with legacy clients, an ALTO server which provides the `pid`
property via the EPS MUST use that definition, and that
syntax.

However, when used with property maps, this document amends the definition of
the `pid` property as follows.

First, the name of the property is simply `pid`; the name is not prefixed with
the resource ID of a network map. The `uses` capability of the property map
indicates the associated network map. This implies that a property map
can only return the `pid` property for one network map; if an ALTO server
provides several network maps, it MUST provide a Property Map for each
of the network maps.

Second, a client MAY request the `pid` property for a block of
Internet addresses. An ALTO server determines the value of `pid` for an
address block C as the rules defined in [](#FilteredPropMapResponse).

<!--
Let CS be the set of all address blocks
in the network map. If C is in CS, then the value of `pid` is the
name of the PID associated with C. Otherwise, find the longest block
C' in CS such that C' prefix-matches C, but is shorter than C. If
there is such a block C', the value of `pid` is the name of the PID
associated with C'.
YRY: Handle the issue of decomposition.
If not, the ALTO server has two optional ways to determines the value:

- The ALTO server may just return no value for the `pid` property of block C;
- Or the ALTO server may find a subset CS' in CS such that C prefix-matches each
  C' in CS' and is longer than each C'. If CS' can fully cover C, the ALTO
  server returns `pid` property of each C' in CS' instead of `pid` property of
  C, and the value of `pid` for each C' is the name of the PID associated with
  C'; otherwise, the ALTO server returns associated PID name for the `pid`
  property of each C' in CS' and also the no value for the `pid` property of C.
  Note that the CS' may not be unique.
  
The determination depends on the implementation.
-->

Note that although an ALTO server MAY provide a GET-mode property map
which returns the entire map for the `pid` property, there is no need
to do so, because that map is simply the inverse of the network map.

## Impact on Other Properties

In general, there should be little or no impact on other previously defined
properties. The only consideration is that properties can now be defined on
blocks of identifiers, rather than just individual identifiers, which might change
the semantics of a property.
