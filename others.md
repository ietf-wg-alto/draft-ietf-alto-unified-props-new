# Security Considerations {#SecSC}

<!-- How the SC of the base protocol fits the new service. -->

Both Property Map and Filtered Property Map defined in this document
fit into the architecture of the ALTO base protocol, and hence the Security
Considerations (Section 15 of [](#RFC7285)) of the base protocol fully apply:
authenticity and integrity of ALTO information (i.e., authenticity and integrity
of Property Maps), potential undesirable guidance from authenticated ALTO
information (e.g., potentially imprecise or even wrong value of a property
such as geo-location), confidentiality of ALTO information (e.g., exposure of
a potentially sensitive entity property such as geo-location), privacy for
ALTO users, and availability of ALTO services should all be considered.

A particular fundamental security consideration when an ALTO server provides
a Property Map is to define precisely the policies on who can access what
properties for which entities. Security mechanisms such as authentication and
confidentiality mechanisms then should be applied to enforce the policy. For
example, a policy can be that a property P can be accessed only by its owner
(e.g., the customer who is allocated a given IP address).
Then, the ALTO server will need to deploy corresponding mechanisms to
realize the policy. The policy may allow non-owners to access a coarse-grained
value of the property P. In such a case, the ALTO server may provide a different
URI to provide the information.

<!-- How about the additional considerations. -->

<!-- For confidentiality of ALTO information, similar to the discussion in Section
15.3 of [](#RFC7285), properties in a Property Map resource may expose sensitive
customer-specific information. If in this case, an ALTO server may limit access
to those properties by providing several different Property Maps: For
non-sensitive properties, the ALTO server would provide a URI which accepts
requests from any ALTO client; Sensitive properties, on the other hand, would
only be available via a private URI which would require the authentication to
the ALTO client. -->

<!--
Also, while technically this document does not introduce any security risks not
inherent in the Endpoint Property Service defined by [](#RFC7285),
the GET-mode Property Map resource defined in this document does make it easier
for an ALTO client to download large numbers of property values. Accordingly, an
ALTO server SHOULD limit GET-mode Property Maps to properties which do not
contain sensitive data.
-->

# IANA Considerations

This document defines additional application/alto-* media types, and extends
the ALTO endpoint property registry.

## application/alto-* Media Types

This document registers two additional ALTO media types, listed in
[](#TableMediaTypes).

-------------------------------------------------------------
Type        Subtype                 Specification
----------- ----------------------- -------------------------
application alto-propmap+json       [](#FullPropMapMediaType)

application alto-propmapparams+json [](#filter-prop-map-params)

-------------------------------------------------------------

^[TableMediaTypes::Additional ALTO Media Types.]

Type name:
~ application

Subtype name:
~ This document registers multiple subtypes, as listed in [](#TableMediaTypes).

Required parameters:
~ n/a

Optional parameters:
~ n/a

Encoding considerations:
~ Encoding considerations are identical to those specified for the
`application/json` media type. See [](#RFC7159).

Security considerations:
~ Security considerations related to the generation and consumption of ALTO
Protocol messages are discussed in Section 15 of [](#RFC7285).

Interoperability considerations:
~ This document specifies formats of conforming messages and the interpretation
thereof.

Published specification:
~ This document is the specification for these media types; see
[](#TableMediaTypes) for the section documenting each media type.

Applications that use this media type:
~ ALTO servers and ALTO clients either stand alone or are embedded within other
applications.

Additional information:
~   Magic number(s):
    ~ n/a

    File extension(s):
    ~ This document uses the mime type to refer to protocol messages and thus does
    not require a file extension.

    Macintosh file type code(s):
    ~ n/a

Person &amp; email address to contact for further information:
~ See Authors' Addresses section.

Intended usage:
~ COMMON

Restrictions on usage:
~ n/a

Author:
~ See Authors' Addresses section.

Change controller:
~ Internet Engineering Task Force (mailto:iesg@ietf.org).

## ALTO Entity Domain Registry {#IANADomain}

This document requests IANA to create and maintain the `ALTO Entity Domain
Registry`, listed in [](#TableDomainNames).

-------------------------------------------------------------------------------------------
Identifier Entity Address Encoding Hierarchy &amp; Inheritance Mapping to ALTO Address Type
---------- ----------------------- --------------------------- ----------------------------
ipv4       See [](#ipv4-domain)    See [](#inet-inheritance)   Yes

ipv6       See [](#ipv6-domain)    See [](#inet-inheritance)   Yes

pid        See [](#pid-domain)     None                        No

-------------------------------------------------------------------------------------------

^[TableDomainNames::ALTO Entity Domains.]

This registry serves two purposes. First, it ensures uniqueness of identifiers
referring to ALTO entity domains. Second, it states the requirements for allocated
entity domains.

<!--
This registry is considered as an extension of the `ALTO Address Type Registry`
defined in Section 14.4 of [](#RFC7285). It must be noted that:

- An entity MAY or MAY NOT be an endpoint. For example, `pid` is registered as
an entity domain in [](#TableDomainNames), but it is not an endpoint address
type.
- An endpoint MUST be an entity. For example, `ipv4` and `ipv6` are already
registered in `ALTO Address Type Registry` in [](#RFC7285), so they MUST have
the same identifier when registered as entity domains.
-->

### Consistency Procedure between ALTO Address Type Registry and ALTO Entity Domain Registry {#consistency-procedure}

One potential issue of introducing the `ALTO Entity Domain Registry` is its
relationship with the `ALTO Address Types Registry` already defined in Section
14.4 of [](#RFC7285). In particular, the entity address of an entity domain
registered in the `ALTO Entity Domain Registry` MAY match an address type
defined in `ALTO Address Type Registry`. It is necessary to precisely define and
guarantee the consistency between `ALTO Address Type Registry` and `ALTO Entity
Domain Registry`.

<!--
This section specifies the consistency procedure between `ALTO Address Type
Registry` and `ALTO Domain Registry`.
-->

<!-- there is therefore a risk of separately registering the address type
and the domain when they point to the same thing. This section specifies how to
ensure the consistency between `ALTO Address Type Registry` (See Section 14.4 of
[](#RFC7285)) and `ALTO Domain Registry`.-->


We define that the ALTO Entity Domain Registry is consistent with ALTO Address
Type Registry if two conditions are satisfied:

- When an address type is already or able to be registered in the ALTO Address
  Type Registry [](#RFC7285), the same identifier MUST be used when a
  corresponding entity domain is registered in the ALTO Entity Domain Registry.
- If an ALTO entity domain has the same identifier as an ALTO address type, their
  addresses encoding MUST be compatible.

To achieve this consistency, the following items MUST be checked before
registering a new ALTO entity domain in a future document:

- Whether the ALTO Address Type Registry contains an address type that can be
  used as an entity address for the candidate domain identifier. This has been
  done for the identifiers `ipv4` and `ipv6` in [](#TableDomainNames).
- Whether the candidate entity address of the entity domain is able to be an
  endpoint address, as defined in Sections 2.1 and 2.2 of [](#RFC7285).

<!-- It is RECOMMANDED that a new ALTO entity domain be registered when the
corresponding address type is registered based on ALTO Address Type Registry
[](#RFC7285). -->

<!-- TODO: PLACE HOLDER for consistency -->

<!--
One potential issue of introducing the ALTO Domain Registry is its relationship
with the ALTO Address Types Registry already defined in [RFC7285].

To achieve consistency, this document considers each ALTO address type defines
an ALTO domain. For example, the address types ipv4 and ipv6 define the ipv4 and
ipv6 domains respectively. To achieve consistency for future ALTO address types,
it is required that each new ALTO address type to be registered in the ALTO
Address Type Registry MUST also be registered as a domain in the ALTO Domain
Registry. And the Entity Address Encoding of the corresponding domain MUST
include both Address Encoding and Prefix Encoding of this address type.
-->

When a new ALTO entity domain is registered, the consistency with the ALTO
Address Type Registry MUST be ensured by the following procedure:

- Test: Do corresponding entity addresses match a known `network` address type?
    - If yes (e.g., cell, MAC or socket addresses):
        - Test: Is such an address type present in the ALTO Address Type
          Registry?
            - If yes: Set the new ALTO entity domain identifier to be the found
              ALTO address type identifier.
            - If no: Define a new ALTO entity domain identifier and use it to
              register a new address type in the ALTO Address Type Registry
              following Section 14.4 of [](#RFC7285).
        - Use the new ALTO entity domain identifier to register a new ALTO
          entity domain in the ALTO Entity Domain Registry following
          [](#dom-reg-process) of this document.
    - If no (e.g., pid name, ane name or country code): Proceed with the ALTO
      Entity Domain registration as described in [](#dom-reg-process).

<!--
Other extensions that introduce new ALTO address types and domains that
are also cognizant of the present extensions can directly proceed as follows:

- When a new address type is registered in the ALTO Address Type Registry
[](#RFC7285), the same identifier MUST be also registered in the ALTO Domain
Registry.
- Proceed as described in [](#dom-reg-process).
-->

<!--
This registry is considered as an extension of the "ALTO Address Type Registry"
defined in Section 14.4 of [](#RFC7285). In particularly,

- An entity MAY or MAY NOT be an endpoint. For example, "pid" is registered as
an entity domain in [](#TableDomainNames), but it is not an endpoint address
type.
- An endpoint MUST be an entity. For example, "ipv4" and "ipv6" are already
registered in "ALTO Address Type Registry" in [](#RFC7285), so they MUST be
registered as entity domains.

When a new address type is registered in the ALTO Address Type Registry
[](#RFC7285), the same identifier MUST be also registered in the ALTO Entity
Domain Registry. And the Entity Address Encoding of this entity domain
identifier MUST include both Address Encoding and Prefix Encoding of the same
identifier registered in the ALTO Address Type Registry [](#RFC7285). For the
purpose of defining properties, an individual entity address and the
corresponding full-length prefix MUST be considered aliases for the same entity.
-->

### ALTO Entity Domain Registration Process {#dom-reg-process}

New ALTO entity domains are assigned after IETF Review [](#RFC5226) to ensure
that proper documentation regarding the new ALTO entity domains and their
security considerations has been provided. RFCs defining new entity domains
SHOULD indicate how an entity in a registered domain is encoded as an
EntityAddr, and, if applicable, the rules defining the entity hierarchy and
property inheritance. Updates and deletions of ALTO entity domains follow the
same procedure.

Registered ALTO entity domain identifiers MUST conform to the syntactical
requirements specified in [](#domain-names). Identifiers are to be recorded and
displayed as strings.

Requests to the IANA to add a new value to the registry MUST include the
following information:

- Identifier: The name of the desired ALTO entity domain.

- Entity Address Encoding: The procedure for encoding the address of an entity
  of the registered type as an EntityAddr (see [](#entity-addrs)). If
  corresponding entity addresses of an entity domain match a known `network`
  address type, the Entity Address Encoding of this domain identifier MUST
  include both Address Encoding and Prefix Encoding of the same identifier
  registered in the ALTO Address Type Registry [](#RFC7285). For the purpose of
  defining properties, an individual entity address and the corresponding
  full-length prefix MUST be considered aliases for the same entity.

- Hierarchy: If the entities form a hierarchy, the procedure for determining
  that hierarchy.

- Inheritance: If entities can inherit property values from other entities, the
  procedure for determining that inheritance.

- Mapping to ALTO Address Type: A boolean value to indicate if the entity domain
  can be mapped to the ALTO address type with the same identifier.

- Security Considerations: In some usage scenarios, entity addresses carried in
  ALTO Protocol messages may reveal information about an ALTO client or an ALTO
  service provider. Applications and ALTO service providers using addresses of
  the registered type should be made aware of how (or if) the addressing scheme
  relates to private information and network proximity.

This specification requests registration of the identifiers `ipv4`, `ipv6` and
`pid`, as shown in [](#TableDomainNames).

## ALTO Entity Property Type Registry {#IANAEndpointProp}

<!--
The ALTO Endpoint Property Type Registry was created by [](#RFC7285). If
possible, the name of that registry SHOULD be changed to "ALTO Entity Property
Type Registry", to indicate that it is not restricted to Endpoint Properties.
If it is not feasible to change the name, the description MUST be amended to
indicate that it registers properties in all entity domains, rather than just the
Internet address domain.
-->

This document requests IANA to create and maintain the `ALTO Entity Property
Type Registry`.

To distinguish with the `ALTO Endpoint Property Type Registry`, this new registry
is for properties in all possible entity domains, rather than just the Internet
address domain. So it is necessary to define and process the consistency
between the two registries.

In the meanwhile, because every entity property is owned by some entity
domains, for each registered entity property, the ALTO Entity Property Registry
MUST define its accepted entity domains and its semantics in every different
accepted entity domains.

Also, an entity property MAY depend on several other ALTO resources. The ALTO
Entity Property Registry MUST specify the media types of all dependent
resources and how the ALTO client uses them in order.

TODO: The registry format and the initial content

<!--
This document specifies the process for the ALTO Entity Property Type Registry,
similar to the "ALTO Entity Domain Registry". This process follows the
following principles:
-->

<!--
- **Superset**: An endpoint property MUST be an entity property; an entity
  property MAY NOT be an endpoint property.
-->

<!--
- **Consistency**: An entity property has no inconsistent meaning with an
  already defined endpoint property.
- **Domain-specific**: An entity property MUST be registered for some entity
  domains explicitly.
- **Multiple dependencies**: An entity property MAY depend on a sequence of
  resources. The registry MUST specify how the client uses them in order.
-->

