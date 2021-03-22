
# Security Considerations {#SecSC}

Both Property Map and Filtered Property Map defined in this document fit into
the architecture of the ALTO base protocol, and hence the Security
Considerations (Section 15 of {{RFC7285}}) of the base protocol fully apply:
authenticity and integrity of ALTO information (i.e., authenticity and
integrity of Property Maps), potential undesirable guidance from
authenticated ALTO information (e.g., potentially imprecise or even wrong
value of a property such as geo-location), confidentiality of ALTO
information (e.g., exposure of a potentially sensitive entity property such
as geo-location), privacy for ALTO users, and availability of ALTO services
should all be considered.

A particular fundamental security consideration when an ALTO server provides
a Property Map is to define precisely the policies on who can access what
properties for which entities. Security mechanisms such as authentication and
confidentiality mechanisms should then be applied to enforce the policy. For
example, a policy can be that a property P can be accessed only by its owner
(e.g., the customer who is allocated a given IP address). Then, the ALTO
server will need to deploy corresponding mechanisms to realize the policy.
The policy may allow non-owners to access a coarse-grained value of the
property P. In such a case, the ALTO server may provide a different URI to
provide the information.

# IANA Considerations

This document defines additional application/alto-* media types. It defines
an ALTO Entity Domain Type Registry that extends the ALTO Address Type
Registry defined in {{RFC7285}}. It also defines an ALTO Entity Property Type
Registry that extends the ALTO endpoint property registry defined in
{{RFC7285}}.

## application/alto-* Media Types

This document registers two additional ALTO media types, listed in
[](#TableMediaTypes).

|---
| Type | Subtype | Specification
|:-|:-|:-
| application | alto-propmap+json | [](#FullPropMapMediaType)
|---
| application | alto-propmapparams+json | [](#filter-prop-map-params)
{: #TableMediaTypes title="Additional ALTO Media Types."}

Type name:
: application

Subtype name:
: This document registers multiple subtypes, as listed in [](#TableMediaTypes).

Required parameters:
: n/a

Optional parameters:
: n/a

Encoding considerations:
: Encoding considerations are identical to those specified for the
  `application/json` media type. See {{RFC8259}}.

Security considerations:
: Security considerations related to the generation and consumption of ALTO
  Protocol messages are discussed in Section 15 of {{RFC7285}}.

Interoperability considerations:
: This document specifies formats of conforming messages and the interpretation
  thereof.

Published specification:
: This document is the specification for these media types; see
  [](#TableMediaTypes) for the section documenting each media type.

Applications that use this media type:
: ALTO servers and ALTO clients either stand alone or are embedded within other
  applications.

Additional information:
:
  Magic number(s):
  : n/a

  File extension(s):
  : This document uses the mime type to refer to protocol messages and thus does
  not require a file extension.

  Macintosh file type code(s):
  : n/a

Person &amp; email address to contact for further information:
: See Authors' Addresses section.

Intended usage:
: COMMON

Restrictions on usage:
: n/a

Author:
: See Authors' Addresses section.

Change controller:
: Internet Engineering Task Force (mailto:iesg@ietf.org).

## ALTO Entity Domain Type Registry {#IANADomain}

This document requests IANA to create and maintain the "ALTO Entity Domain Type
Registry", listed in [](#TableEntityDomainNames).

|---
| Identifier | Entity Identifier Encoding | Hierarchy &amp; Inheritance | Media Type of Defining Resource
|:-|:-|:-|:-
| ipv4 | See [](#ipv4-domain) | See [](#inet-inheritance) | application/alto-networkmap+json
|---
| ipv6 | See [](#ipv6-domain) | See [](#inet-inheritance) | application/alto-networkmap+json
|---
| pid | See [](#pid-domain) | None | application/alto-networkmap+json
{: #TableEntityDomainNames title="ALTO Entity Domain Types"}

This registry serves two purposes. First, it ensures uniqueness of
identifiers referring to ALTO entity domain types. Second, it states the
requirements for allocated entity domain types.

### Consistency Procedure between ALTO Address Type Registry and ALTO Entity Domain Type Registry {#consistency-procedure}

One potential issue of introducing the "ALTO Entity Domain Type Registry" is
its relationship with the "ALTO Address Types Registry" already defined in
Section 14.4 of {{RFC7285}}. In particular, the entity identifier of a type
of an entity domain registered in the "ALTO Entity Domain Type Registry" MAY
match an address type defined in "ALTO Address Type Registry". It is
necessary to precisely define and guarantee the consistency between "ALTO
Address Type Registry" and "ALTO Entity Domain Registry".

We define that the ALTO Entity Domain Type Registry is consistent with ALTO
Address Type Registry if two conditions are satisfied:

* When an address type is already or able to be registered in the ALTO Address
  Type Registry {{RFC7285}}, the same identifier MUST be used when a
  corresponding entity domain type is registered in the ALTO Entity Domain Type
  Registry.
* If an ALTO entity domain type has the same identifier as an ALTO address type,
  their addresses encoding MUST be compatible.

To achieve this consistency, the following items MUST be checked before
registering a new ALTO entity domain type in a future document:

* Whether the ALTO Address Type Registry contains an address type that can be
  used as an identifier for the candidate entity domain type identifier. This has been
  done for the identifiers "ipv4" and "ipv6" of [](#TableEntityDomainNames).
* Whether the candidate entity domain type identifier can potentially be an
  endpoint address type, as defined in Sections 2.1 and 2.2 of {{RFC7285}}.

When a new ALTO entity domain type is registered, the consistency with the ALTO
Address Type Registry MUST be ensured by the following procedure:

* Test: Do corresponding entity domain type identifiers match a known "network" address type?
    * If yes (e.g., cell, MAC or socket addresses):
        * Test: Is such an address type present in the ALTO Address Type
          Registry?
            * If yes: Set the new ALTO entity domain type identifier to be the
              found ALTO address type identifier.
            * If no: Define a new ALTO entity domain type identifier and use it to
              register a new address type in the ALTO Address Type Registry
              following Section 14.4 of {{RFC7285}}.
        * Use the new ALTO entity domain type identifier to register a new ALTO
          entity domain type in the ALTO Entity Domain Type Registry following
          [](#dom-reg-process) of this document.
    * If no (e.g., pid name, ane name or country code): Proceed with the ALTO
      Entity Domain Type registration as described in [](#dom-reg-process).

### ALTO Entity Domain Type Registration Process {#dom-reg-process}

New ALTO entity domain types are assigned after IETF Review {{RFC8126}} to
ensure that proper documentation regarding the new ALTO entity domain types
and their security considerations has been provided. RFCs defining new entity
domain types SHOULD indicate how an entity in a registered type of domain is
encoded as an EntityID, and, if applicable, the rules defining the entity
hierarchy and property inheritance. Updates and deletions of ALTO entity
domains types follow the same procedure.

Registered ALTO entity domain type identifiers MUST conform to the
syntactical requirements specified in [](#domain-names). Identifiers are to
be recorded and displayed as strings.

Requests to the IANA to add a new value to the Entity Domain Type registry
MUST include the following information:

* Identifier: The name of the desired ALTO entity domain type.
* Entity Identifier Encoding: The procedure for encoding the identifier of an
  entity of the registered domain type as an EntityID (see
  [](#entity-addrs)). If corresponding entity identifiers of an entity domain
  type match a known "network" address type, the Entity Identifier Encoding
  of this domain identifier MUST include both Address Encoding and Prefix
  Encoding of the same identifier registered in the ALTO Address Type
  Registry {{RFC7285}}. To define properties, an individual entity identifier
  and the corresponding full-length prefix MUST be considered aliases for the
  same entity.
* Hierarchy: If the entities form a hierarchy, the procedure for determining
  that hierarchy.
* Inheritance: If entities can inherit property values from other entities, the
  procedure for determining that inheritance.
* Media type of defining information resource: some entity domain types allow
  an entity domain name to be combined with an information resource name to
  define a resource-specific entity domain. Such an information resource is
  called "defining information resource". In this case, the authorized media
  type of the defining information resources MUST be specified in the
  document defining the entity domain type. When an entity domain type allows
  combinations with defining resources, this must be indicated and the
  conditions fully specified in the document.
  The defining information resource for an entity domain type is the one
  that:
  * has an entry in the IRD,
  * defines these entities,
  * does not use another information resource that defines these entities,
  * defines and exposes entity identifiers that are all persistent.
  * has a unique media type equal to the one specified in the document
    defining the entity domain type.
* Knowing the media type of the defining information resource is useful when
  Servers indicate resource specific entity domains in the property map
  capabilities. Clients need to know if the combination of information
  resource type and entity domain type is allowed. See also,
  [](#def-ir) and [](#def-domain) for more
  information.
* Mapping to ALTO Address Type: A boolean value to indicate if the entity domain
  type can be mapped to the ALTO address type with the same identifier.
* Security Considerations: In some usage scenarios, entity identifiers carried in
  ALTO Protocol messages may reveal information about an ALTO client or an ALTO
  service provider. Applications and ALTO service providers using addresses of
  the registered type should be cognizant of how (or if) the addressing scheme
  relates to private information and network proximity.

This specification requests registration of the identifiers "ipv4", "ipv6" and
"pid", as shown in [](#TableEntityDomainNames).

## ALTO Entity Property Type Registry {#IANAEntityProp}

This document requests IANA to create and maintain the "ALTO Entity Property
Type Registry", listed in [](#TablePropertyTypes).

This registry extends the "ALTO Endpoint Property Type Registry", defined in
{{RFC7285}}, in that a property type is defined on one or more entity
domains, rather than just on IPv4 and IPv6 Internet address domains. An entry
in this registry is an ALTO entity property type defined in
[](#def-property-type). Thus, a registered ALTO entity property type
identifier MUST conform to the syntactical requirements specified in that
section.

|---
| Identifier | Intended Semantics | Media Type of Defining Resource
|:-|:-|:-
| pid | See Section 7.1.1 of [](#RFC7285) | application/alto-networkmap+json
{: #TablePropertyTypes title="ALTO Entity Property Types."}

Requests to the IANA to add a new value to the registry MUST include the
following information:

* Identifier: The unique id for the desired ALTO entity property type. The
  format MUST be as defined in [](#def-property-type) of this document. It
  includes the information of the applied ALTO entity domain and the property
  name.
* Intended Semantics: ALTO entity properties carry with them semantics to guide
  their usage by ALTO clients. Hence, a document defining a new type SHOULD
  provide guidance to both ALTO service providers and applications utilizing
  ALTO clients as to how values of the registered ALTO entity property should
  be interpreted.
* Media type of defining information resource: when the property type allows
  values to be defined relatively to a given information resource, the latter
  is referred to as the "defining information resource", see also description
  in [](#def-ir-for-irsp). The media type
  of the possibly used defining information resource MUST be indicated.
* Security Considerations: ALTO entity properties expose information to ALTO
  clients. ALTO service providers should be cognizant of the security
  ramifications related to the exposure of an entity property.

In security considerations, the request should also discuss the sensitivity
of the information, and why it is required for ALTO-based operations.
Regarding this discussion, the request SHOULD follow the recommendations of
Section 14.3. ALTO Endpoint Property Type Registry in {{RFC7285}}.

This document requests registration of the identifier "pid", listed in
[](#TablePropertyTypes). Semantics for this property are documented in
Section 7.1.1 of {{RFC7285}}. No security issues related to the exposure of a
"pid" identifier are considered, as it is exposed with the Network Map
Service defined and mandated in {{RFC7285}}.

# Acknowledgments {#ack}

The authors would like to thank Dawn Chen (Tongji University), and Shenshen
Chen (Tongji/Yale University) for their contributions to earlier drafts.
Thank you also to Qiao Xiang (Yale University), Shawn Lin, Xin Wang and Vijay
Gurbani (Vail systems) for fruitful discussions. Last, big thanks to Danny
Perez (University of Campinas) and Luis Contreras (Telefonica) for their
substantial review feedback and suggestions to improve this document.
