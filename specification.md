
# Entity Domain Types Defined in this Document {#entity-domain-types}

This document requires the definition of each entity domain type MUST include
(1) the entity domain type name and (2) domain-specific entity identifiers,
and MAY include (3) hierarchy and inheritance semantics optionally. This
document defines three initial entity domain types as follows.

## Internet Address Domain Types {#inet-addr-domain}

The document defines two entity domain types (IPv4 and IPv6) for Internet
addresses. Both types are resource-agnostic entity domain types and hence
define corresponding resource-agnostic entity domains as well. Since the two
domains use the same hierarchy and inheritance semantics, we define the
semantics together, instead of repeating for each.

### Entity Domain Type: IPv4 {#ipv4-domain}

#### Entity Domain Type Identifier {#ipv4-edt}

ipv4

#### Domain-Specific Entity Identifiers {#ipv4-dsei}

Individual addresses are strings as specified by the IPv4Addresses rule of
Section 3.2.2 of {{RFC3986}}; Hierarchical addresses are prefix-match strings
as specified in Section 3.1 of {{RFC4632}}. To define properties, an
individual Internet address and the corresponding full-length prefix are
considered aliases for the same entity. An individual Internet address and
the corresponding full-length prefix are considered aliases for the same
entity on which to define properties. Thus, "ipv4:192.0.2.0" and
"ipv4:192.0.2.0/32" are equivalent.

### Entity Domain Type: IPv6 {#ipv6-domain}

#### Entity Domain Type Identifier {#ipv6-edt}

ipv6

#### Domain-Specific Entity Identifiers {#ipv6-dsei}

Individual addresses are strings as specified by Section 4 of {{RFC5952}};
Hierarchical addresses are prefix-match strings as specified in Section 7 of
{{RFC5952}}. To define properties, an individual Internet address and the
corresponding 128-bit prefix are considered aliases for the same entity. That
is, "ipv6:2001:db8::1" and "ipv6:2001:db8::1/128" are equivalent, and have
the same set of properties.

### Hierarchy and Inheritance of Internet Address Domains {#inet-inheritance}

Both Internet address domains allow property values to be inherited.
Specifically, if a property P is not defined for a specific Internet address
I, but P is defined for a hierarchical Internet address C which
prefix-matches I, then the address I inherits the value of P defined for the
hierarchical address C. If more than one such hierarchical addresses define a
value for P, I inherits the value of P in the hierarchical address with the
longest prefix. Note that this longest prefix rule ensures no multiple value
inheritances, and hence no ambiguity.

Hierarchical addresses can also inherit properties: if a property P is not
defined for the hierarchical address C, but is defined for another
hierarchical address C' which covers all IP addresses in C, and C' has a
shorter prefix length than C, then C MAY inherit the property from C'. If
there are multiple such hierarchical addresses like C', C MUST inherit from
the hierarchical address having the longest prefix length.

As an example, suppose that a server defines a property P for the following
entities:

~~~
                          ipv4:192.0.2.0/26: P=v1
                          ipv4:192.0.2.0/28: P=v2
                          ipv4:192.0.2.0/30: P=v3
                          ipv4:192.0.2.0:    P=v4
~~~
{: #fig:def-prop-val title="Defined Property Values."}

Then the following entities have the indicated values:

~~~
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
~~~
{: #fig:inh-prop-val title="Inherited Property Values."}

<!-- Improve words of this paragraph. (Done, waiting for review) -->

An ALTO server MAY explicitly indicate a property as not having a value for
a particular entity. That is, a server MAY say that property P of entity X is
"defined to have no value", instead of "undefined". To indicate "no value",
a server MAY perform different behaviours:

* If that entity would inherit a value for that property, then the ALTO server
  MUST return a `null` value for that property. In this case, the ALTO client
  MUST recognize a `null` value as "no value" and "do not apply the inheritance
  rules for this property."
* If the entity would not inherit a value, then the ALTO server MAY return
  `null` or just omit the property. In this case, the ALTO client cannot infer
  the value for this property of this entity from the Inheritance rules. So the
  client MUST interpret that this property has no value.

If the ALTO server does not define any properties for an entity, then the
server MAY omit that entity from the response.

### Defining Information Resource Media Type for domain types IPv4 and IPv6 {#defining-IR-media-type-ipv4-ipv6}

Entity domain types "ipv4" and "ipv6" both allow to define resource specific
entity domains. When resource specific domains are defined with entities of
domain type "ipv4" or "ipv6", the defining information resource for an entity
domain of type "ipv4" or "ipv6" MUST be a Network Map. The media type of a
defining information resource is therefore:

application/alto-networkmap+json

## Entity Domain Type: PID {#pid-domain}

The PID domain associates property values with the PIDs in a network map.
Accordingly, this entity domain always depends on a network map.

### Entity Domain Type Identifier

pid

### Domain-Specific Entity Identifiers

The entity identifiers are the PID names of the associated network map.

### Hierarchy and Inheritance

There is no hierarchy or inheritance for properties associated with PIDs.

### Defining Information Resource Media Type for Domain Type PID {#defining-IR-media-type-pid}

The entity domain type "pid" allows to define resource specific entity
domains. When resource specific domains are defined with entities of domain
type "pid", the defining information resource for entity domain type "pid"
MUST be a Network Map. The media type of a defining information resource is
therefore:

application/alto-networkmap+json

### Relationship To Internet Addresses Domains

The PID domain and the Internet address domains are completely independent;
the properties associated with a PID have no relation to the properties
associated with the prefixes or endpoint addresses in that PID. An ALTO
server MAY choose to assign all the properties of a PID to the prefixes in
that PID or only some of these properties.

For example, suppose "PID1" consists of the prefix "ipv4:192.0.2.0/24", and
has the property "P" with value "v1". The Internet address entities
"ipv4:192.0.2.0" and "ipv4:192.0.2.0/24" in the IPv4 domain MAY have a value
for the property "P", and if they do, it is not necessarily "v1".

## Internet Address Properties vs. PID Properties

Because the Internet address and PID domains relate to completely distinct domain types, the
question may arise as to which entity domain type is the best for a property. In general, the
Internet address domain types are RECOMMENDED for properties that are closely related
to the Internet address, or are associated with, and inherited through, hierarchical addresses.

The PID domain type is RECOMMENDED for properties that arise from the definition of
the PID, rather than from the Internet address prefixes in that PID.

For example, because Internet addresses are allocated to service providers by
blocks of prefixes, an "ISP" property would be best associated with Internet
address domain types. On the other hand, a property that explains why a PID
was formed, or how it relates to a provider's network, would best be
associated with the PID domain type.

# Property Map {#prop-map}

A property map returns the properties defined for all entities in one or more
domains, e.g., the "location" property of entities in "pid" domain, and the
"ASN" property of entities in "ipv4" and "ipv6" domains.

[](#prop-map-example) gives an example of a property map request and its
response.

## Media Type {#FullPropMapMediaType}

The media type of a property map is `application/alto-propmap+json`.

## HTTP Method

The property map is requested using the HTTP GET method.

## Accept Input Parameters

None.

## Capabilities {#FullPropMapCapabilities}

The capabilities are defined by an object of type PropertyMapCapabilities:

~~~
    object {
      EntityPropertyMapping mappings;
    } PropertyMapCapabilities;
    
    object-map {
      EntityDomainName -> EntityPropertyName<1..*>;
    } EntityPropertyMapping
~~~

with fields:

mappings:
: A JSON object whose keys are names of entity domains and values are the
  supported entity properties of the corresponding entity domains.

## Uses {#FullPropMapUses}

The `uses` field of a property map resource in an IRD entry specifies
dependent resources of this property map. It is an array of the resource ID(s)
of the resource(s).

## Response {#FullPropMapResponse}

If the entity domains in this property map depend on other resources, the
`dependent-vtags` field in the `meta` field of the response MUST be an array
that includes the version tags of those resources, and the order MUST be
consistent with the `uses` field of this property map resource. The data
component of a property map response is named `property-map`, which is a JSON
object of type PropertyMapData, where:

~~~
    object {
      PropertyMapData property-map;
    } InfoResourceProperties : ResponseEntityBase;

    object-map {
      EntityID -> EntityProps;
    } PropertyMapData;

    object {
      EntityPropertyName -> JSONValue;
    } EntityProps;
~~~

The ResponseEntityBase type is defined in Section 8.4 of {{RFC7285}}.

Specifically, a PropertyMapData object has one member for each entity in the
property map. The entity's properties are encoded in the corresponding
EntityProps object. EntityProps encodes one name/value pair for each property,
where the property names are encoded as strings of type PropertyName.
A protocol implementation SHOULD assume that the property value is either
a JSONString or a JSON "null" value, and fail to parse if it is not, unless the
implementation is using an extension to this document that indicates when and
how property values of other data types are signaled.

For each entity in the property map:

* If the entity is in a resource-specific entity domain, the ALTO server SHOULD
  only return self-defined properties and resource-specific properties which
  depend on the same resource as the entity does. The ALTO client SHOULD ignore
  the resource-specific property in this entity if their mapping is not
  registered in the ALTO Resource Entity Property Transfer Registry of the type
  of the corresponding resource.
* If the entity is in a shared entity domain, the ALTO server SHOULD return
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

~~~
  object {
    EntityID             entities<1..*>;
    EntityPropertyName   properties<1..*>;
  } ReqFilteredPropertyMap;
~~~

with fields:

entities:
: List of entity identifiers for which the specified properties are to be
  returned. The ALTO server MUST interpret entries appearing multiple times
  as if they appeared only once. The domain of each entity MUST be included
  in the list of entity domains in this resource's `capabilities` field (see
  [](#FilteredPropMapCapabilities)).

properties:
: List of properties to be returned for each entity. Each specified property
  MUST be included in the list of properties in this resource's
  `capabilities` field (see [](#FilteredPropMapCapabilities)). The ALTO
  server MUST interpret entries appearing multiple times as if they appeared
  only once.

  Note that the `entities` and `properties` fields MUST have at least one entry
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

## Filtered Property Map Response {#FilteredPropMapResponse}

The response MUST indicate an error, using ALTO protocol error handling, as
defined in Section 8.5 of {{RFC7285}}, if the request is invalid.

Specifically, a filtered property map request can be invalid in the following cases:

* An entity identifier in the `entities` field of the request is invalid if:

    * The domain of this entity is not defined in the `entity-domains`
      capability of this resource in the IRD,
    * The entity identifier is not valid for the entity domain.

    A valid entity identifier does never generate an error, even if the filtered property
    map resource does not define any properties for it.

    If an entity identifier in the `entities` field of the request is invalid, the ALTO
    server MUST return an `E_INVALID_FIELD_VALUE` error defined in Section
    8.5.2 of {{RFC7285}}, and the `value` field of the error message SHOULD
    indicate the provided invalid entity identifier.

* A property name in the `properties` field of the request is invalid if this
  property name is not defined in the `properties` capability of this
  resource in the IRD.

    When a filtered property map resource does not define a value for a
    property requested on a particular entity, it is not an error. In this
    case, the ALTO server MUST omit that property from the response for that
    endpoint.

    If a property name in `properties` in the request is invalid, the ALTO
    server MUST return an `E_INVALID_FIELD_VALUE` error defined in Section
    8.5.2 of {{RFC7285}}. The `value` field of the error message SHOULD
    indicate the property name.
    
The response to a valid request is the same as for the Property Map
(see [](#FullPropMapResponse)), except that:

* If the requested entities include entities in the shared entity domain, the
  `dependent-vtags` field in its `meta` field MUST include version tags of all
  dependent resources appearing in the `uses` field.
* If the requested entities only include entities in resource-specific entity
  domains, the `dependent-vtags` field in its `meta` field MUST include the version
  tags of the resources on which the requested resource-specific entity domains and
  the requested resource-specific properties are dependent on.
* The response only includes the entities and properties requested by the
  client. If an entity in the request is identified by a hierarchical identifier
  (e.g., a "ipv4" or "ipv6" prefix), the response MUST cover properties
  for all identifiers in this hierarchical identifier.

The filtered property map response MUST include all the inherited property
values for the requested entities and all the entities which are able to
inherit property values from the requested entities. To achieve this goal,
the ALTO server MAY follow three rules:

* If a property for a requested entity is inherited from another entity not
  included in the request, the response SHOULD include this property for the
  requested entity. For example, A full property map may skip a property P for
  an entity A (e.g., ipv4:192.0.2.0/31) if P can be derived using inheritance
  from another entity B (e.g., ipv4:192.0.2.0/30). A filtered property map
  request may include only A but not B. In such a case, the property P SHOULD be
  included in the response for A.
* If there are entities covered by a requested entity but having different
  values for the requested properties, the response SHOULD include all those
  entities and the different property values for them. For example, considering
  a request for property P of entity A (e.g., ipv4:192.0.2.0/31), if P has value
  v1 for A1=ipv4:192.0.2.0/32 and v2 for A2=ipv4:192.0.2.1/32, then, the
  response SHOULD include A1 and A2.
* If an entity identifier in the response is already covered by other
  entities identifiers in the same response, it SHOULD be removed from the
  response, for the sake of compactness. In the previous example, the entity
  A = ipv4:192.0.2.0/31 SHOULD be removed because A1 and A2 cover all the
  addresses in A.

An ALTO client should be aware that the entities in the response MAY be
different from the entities in its request.

## Entity property type defined in this document {#prop-type-pid}

This document defines the entity property type "pid". This property type
extends the ALTO Endpoint Property Type "pid" defined in section 7.1.1 of
{{RFC7285}} as follows: the property has the same semantics and applies to
IPv4 and IPv6 addresses; the difference is that the IPv4 and IPv6 addresses
have evolved from the status of endpoints to the status of entities.

The defining information resource for property type MUST be a network map.
This document requests a IANA registration for this property

### Entity Property Type: pid

1. Identifier: pid
2. Semantics: the intended semantics are the same as in {{RFC7285}} for the
   ALTO Endpoint Property Type "pid"
3. Media type of defining information resource: application/alto-networkmap+json
4. Security considerations: for entity property type "pid" are the same as
   documented in {{RFC7285}} for the ALTO Endpoint Property Type "pid".

# Impact on Legacy ALTO Servers and ALTO Clients {#legacy}

## Impact on Endpoint Property Service

Since the Property Map and the Filtered Property Map defined in this document
provide a functionality that covers the Endpoint Property Service (EPS)
defined in Section 11.4 of {{RFC7285}}, ALTO servers may prefer to provide
Property Map and Filtered Property Map in place of EPS. However, for the
legacy endpoint properties, it is recommended that ALTO servers also provide
EPS so that legacy clients can still be supported.

## Impact on Resource-Specific Properties

Section 10.8 of {{RFC7285}} defines two categories of endpoint properties:
"resource-specific" and "global". Resource-specific property names are
prefixed with the ID of the resource they depend on, while global property
names have no such prefix. The property map and the filtered property map
defined in this document define similar categories of entity properties. The
difference is that entity property maps do not define "global" entity
properties. Instead, they define "self-defined" entity properties as a
special case of "resource-specific" entity properties, where the specific
resource is the property map itself. This means that "self-defined"
properties are defined within the scope of the property map.

## Impact on Other Properties

In the present extension, properties can now be defined on sets of entity
addresses, rather than just individual endpoint addresses as is is the case
in RFC7285. This might change the semantics of a property. These sets can be
for example hierachical IP address blocks. For instance, a property such as
fictitious "geo-location", defined on a set of IP addresses would have a
value corresponding to the barycenter of this set of addresses.
