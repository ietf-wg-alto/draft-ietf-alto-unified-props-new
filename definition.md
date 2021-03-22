
# Protocol Specification: Basic Data Types

## Entity Domain {#def-domain}

### Entity Domain Type {#domain-types}

An entity domain has a type, which is uniquely identified by a string that
MUST be no more than 64 characters, and MUST NOT contain characters other
than US-ASCII alphanumeric characters (U+0030-U+0039, U+0041-U+005A, and
U+0061-U+007A), the hyphen ('-', U+002D), or the low line ('\_', U+005F).

For example, the strings "ipv4", "ipv6", and "pid" are valid entity domain
types. "ipv4.anycast" and "pid.local" are invalid.

The type EntityDomainType is used in this document to denote a JSON string
meeting the preceding requirements.

An entity domain type defines the semantics of a type of entity,
independently of any specifying resource. Each entity domain type MUST be
registered with the IANA. The format of the entity identifiers (see
[](#entity-addrs)) in that type of entity domain, as well as any hierarchical
or inheritance rules (see [](#def-hierarchy-and-inheritance)) for those
entities, MUST be specified at the same time.

### Entity Domain Name {#domain-names}

As said in [](#con-entity-domain) when introducing entity domains, an entity
domain is characterized by its type and identified by its name.

This document distinguishes three categories of entity domains:
resource-specific entity domains, resource-agnostic entity domains and
self-defined entity domains. Their entity domain names are constructed as
specified in the following sub-sections.

Each entity domain is identified by a unique entity domain name which is a
string of the following format:

~~~
    EntityDomainName ::= [ [ ResourceID ] '.' ] EntityDomainType
~~~

Where the presence and construction of component:

~~~
    "[ [ ResourceID ] '.' ]"
~~~

depends on the category of entity domain.

Note that the '.' separator is not allowed in EntityDomainType and hence
there is no ambiguity on whether an entity domain name refers to a
resource-agnostic entity domain or a resource-specific entity domain.

Note also that Section 10.1 of {{RFC7285}} specifies the format of the PID
Name which is the format of the resource ID including the following
specification: "the '.' separator is reserved for future use and MUST NOT be
used unless specifically indicated in this document, or an extension
document". The present extension keeps the format specification of
{{RFC7285}}, hence the '.' separator MUST NOT be used in an information
resource ID.

#### Resource-specific Entity Domain {#resource-specific-ED}

A resource-specific entity domain is identified by an entity domain name
constructed as follows. It MUST start with a resource ID using the ResourceID
type defined in Section 10.2 of {{RFC7285}}, followed by the '.' separator
(U+002E), followed by a string of the type EntityDomainType specified in
[](#domain-types).

For example, if an ALTO server provides two network maps "netmap-1" and
"netmap-2", these network maps can define two resource-specific domains of
type "pid", respectively identified by "netmap-1.pid" and "netmap-2.pid".

#### Resource-agnostic Entity Domain {#resource-agnostic-ED}

A resource-agnostic entity domain contains entities that are identified
independently of any information resource. Hence, the identifier of a
resource-agnostic entity domain is simply the identifier of its entity domain
type. For example, "ipv4" and "ipv6" identify the two resource-agnostic
Internet address entity domains defined in [](#inet-addr-domain).

#### Self-defined Entity Domain {#self-defined-ED}

A property map can define properties on entities that are specific to a
unique information resource, which is the property map itself. This may be
the case when an ALTO Server provides properties on a set of entities that
are defined only in this property map, are not relevant to another one and do
not depend on another specific resource.

For example: a specialised property map may define a domain of type "ane",
defined in {{I-D.ietf-alto-path-vector}}, that contains a set of ANEs
representing data centers, that each have a persistent identifier and are
relevant only to this property map.

In this case, the entity domain is qualified as "self-defined". The
identifier of a self-defined entity domain can be of the format:

~~~
    EntityDomainName ::= '.' EntityDomainType
~~~

where '.' indicates that the entity domain only exists within the property
map resource using it.

A self-defined entity domain can be viewed as a particular case of
resource-specific entity domain, where the specific resource is the current
resource that uses this entity domain. In that case, for the sake of
simplification, the component "ResourceID" SHOULD be omitted in its entity
domain name.

### Entity Identifier {#entity-addrs}

<!-- FIXME: The entity identifier is not global unique. -->

Entities in an entity domain are identified by entity identifiers (EntityID) of
the following format:

~~~
EntityID ::= EntityDomainName ':' DomainTypeSpecificEntityID
~~~

Examples from the Internet address entity domains include individual IP
addresses such as `net1.ipv4:192.0.2.14` and `net1.ipv6:2001:db8::12`, as well
as address blocks such as `net1.ipv4:192.0.2.0/26` and
`net1.ipv6:2001:db8::1/48`.

The format of the second part of an entity identifier depends on the entity
domain type, and MUST be specified when defining a new entity domain type and
registering it with the IANA. Identifiers MAY be hierarchical, and properties
MAY be inherited based on that hierarchy. The rules defining any hierarchy or
inheritance MUST be defined when the entity domain type is registered.

The type EntityID is used in this document to denote a JSON string
representing an entity identifier in this format.

Note that two entity identifiers with different valid textual representations
may refer to the same entity, for a given entity domain. For example, the
strings "net1.ipv6:2001:db8::1" and "net1.ipv6:2001:db8:0:0:0:0:0:1" refer to
the same entity in the "ipv6" entity domain. Such equivalences should be
established by the object represented by DomainTypeSpecificEntityID, for
example, {{RFC5952}} establishes equivalence for IPv6 addresses, while
{{RFC4632}} does so for IPv4 addresses.

### Hierarchy and Inheritance {#def-hierarchy-and-inheritance}

To simplify the representation, some types of entity domains allow the ALTO
Client and Server to use a hierarchical entity identifier format to represent
a block of individual entities. For instance, in an IPv4 domain "net1.ipv4",
a CIDR "net1.ipv4:192.0.2.0/26" covers 64 individual IPv4 entities. In this
case, the corresponding property inheritance rule MUST be defined for the
entity domain type. The hierarchy and inheritance rule MUST have no
ambiguity.

## Entity Property {#def-property}

Each entity property has a type to indicate the encoding and the semantics of
the value of this entity property, and has a name to identify it.

### Entity Property Type {#def-property-type}

The type EntityPropertyType is used in this document to indicate a string
denoting an entity property type. The string MUST be no more than 32
characters, and it MUST NOT contain characters other than US-ASCII
alphanumeric characters (U+0030-U+0039, U+0041-U+005A, and U+0061-U+007A),
the hyphen ('-', U+002D), the colon (':', U+003A), or the low line ('\_',
U+005F). Note that the '.' separator is not allowed because it is reserved to
separate an entity property type and an information resource identifier when
an entity property is resource-specific.

Each entity property type MUST be registered with the IANA. The intended
semantics of the entity property type MUST be specified at the same time.

Identifiers prefixed with "priv:" are reserved for Private Use {{RFC8126}}
without a need to register with IANA. All other identifiers for entity
property types appearing in an HTTP request or response with an
"application/alto-*" media type MUST be registered in the "ALTO Entity
Property Type Registry", defined in [](#IANAEntityProp). For an entity
property identifier with the "priv:" prefix, an additional string (e.g.,
company identifier or random string) MUST follow the prefix to reduce
potential collisions, that is, the string "priv:" alone is not a valid
endpoint property identifier.

To distinguish from the endpoint property type, the entity property type has
the following characteristics:

* Some entity property types are applicable to entities in particular entity
  domain types only. For example, the property type "pid" is applicable to
  entities in the entity domain types "ipv4" or "ipv6" while is not
  applicable to entities in an entity domain of type "pid".
* The intended semantics of the value of an entity property may also depend
  on the entity domain type. For example, suppose that a property named
  "geo-location" is defined as the coordinates of a point, encoded as:
  "latitude longitude \[altitude\]." When applied to an entity that represents
  a specific host computer, identified by an address in an entity domain of
  type "ipv4" or "ipv6", the "geo-location" property would define the host's
  location. However, when applied to an entity in a "pid" domain type, the
  property would indicate the location of the center of all hosts in this
  "pid" entity.

### Entity Property Name

Each entity property is identified by an entity property name, which is a
string of the following format:

~~~
EntityPropertyName ::= [ ResourceID ] '.' EntityPropertyType
~~~

Similar to the endpoint property type defined in Section 10.8 of {{RFC7285}},
each entity property may be defined by either the property map itself
(self-defined) or some other specific information resource
(resource-specific).

The entity property name of a resource-specific entity property starts with a
string of the type ResourceID defined in {{RFC7285}}, followed by the '.'
separator (U+002E) and a EntityDomainType typed string. For example, the
"pid" properties of an "ipv4" entity defined by two different maps
"net-map-1" and "net-map-2" are identified by "net-map-1.pid" and
"net-map-2.pid" respectively.

The specific information resource of an entity property may be the current
information resource itself, that is, the property map defining the property.
In that case, the ResourceID in the property name SHOULD be ignored. For
example, the property name ".asn" applied to an entity identitifed by its
IPv4 address, indicates the AS number of the AS that "owns" the entity, where
the returned AS number is defined by the property map itself.

### Format for Entity Property Value {#format-entity-property-value}

{{RFC7285}} in Section 11.4.1.6, specifies that an implementation of the
Endpoint Property Service specified in {{RFC7285}} SHOULD assume that the
property value is a JSONString and fail to parse if it is not. This document
extends the format of a property value by allowing it to be a JSONValue
instead of just a JSONString.
