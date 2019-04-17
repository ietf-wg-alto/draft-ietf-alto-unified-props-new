# Examples {#examples}

<!-- FIXME: Examples should be revised -->

## Network Map {#net-map-example}

The examples in this section use a very simple default network map:

``` text
defaultpid:  ipv4:0.0.0.0/0  ipv6:::0/0
pid1:        ipv4:192.0.2.0/25
pid2:        ipv4:192.0.2.0/28  ipv4:192.0.2.16/28
pid3:        ipv4:192.0.3.0/28
pid4:        ipv4:192.0.3.16/28
```

^[net-map-values-ex::Example Network Map]


## Property Definitions {#inet-prop-example}

Beyond `pid`, the examples in this section use four additional properties for
Internet address domains, `ISP`, `ASN`, `country` and `state`, with the
following values:

```
                        ISP    ASN   country   state
ipv4:192.0.2.0/23:    BitsRus   -      us       -
ipv4:192.0.2.0/28:       -    12345    -        NJ
ipv4:192.0.2.16/28:      -    12345    -        CT
ipv4:192.0.2.0:          -      -      -        PA
ipv4:192.0.3.0/28:       -    12346    -        TX
ipv4:192.0.3.16/28:      -    12346    -        MN
```

^[prop-map-values-ip-ex::Example Property Values for Internet Address Domains]

And the examples in this section use the property `region` for PID domain with
the following values:

```
                   region
pid:defaultpid:     -
pid:pid1:           west
pid:pid2:           east
pid:pid3:           south
pid:pid4:           north
```

^[prop-map-values-pid-ex::Example Property Values for PID Domain]

Note that `-` means the value of the property for the entity is "undefined". So
the entity would inherit a value for this property by the inheritance rule if
possible. For example, the value of the `ISP` property for `ipv4:192.0.2.0` is
`BitsRus` because of `ipv4:192.0.2.0/24`. But the `region` property for
`pid:defaultpid` has no value because no entity from which it can inherit.

## Information Resource Directory (IRD) {#ird-example}

The following IRD defines the relevant resources of the ALTO server. It
provides two property maps, one for the `ISP` and `ASN` properties,
and another for the `country` and `state` properties. The server could have
provided a single property map for all four properties, but did not,
presumably because the organization that runs the ALTO server believes any
given client is not interested in all four properties.

The server provides two filtered property maps. The first returns all four
properties, and the second just returns the `pid` property for the default
network map.

The filtered property maps for the `ISP`, `ASN`, `country` and `state`
properties do not depend on the default network map (it does not have a `uses`
capability), because the definitions of those properties do not depend on the
default network map. The Filtered Property Map for the `pid` property does have
a `uses` capability for the default network map, because that defines the
values of the `pid` property.

Note that for legacy clients, the ALTO server provides an Endpoint Property
Service for the `pid` property for the default network map.

```
  "meta" : {
      ...
      "default-alto-network-map" : "default-network-map"
  },
  "resources" : {
     "default-network-map" : {
        "uri" : "http://alto.example.com/networkmap",
        "media-type" : "application/alto-networkmap+json"
     },
     .... property map resources ....
     "country-state-property-map" : {
        "uri" : "http://alto.example.com/propmap/full/inet-cs",
        "media-type" : "application/alto-propmap+json",
        "capabilities" : {
          "entity-domains": [ "ipv4", "ipv6" ],
          "properties" : [  "country", "state" ]
        }
     },
     "isp-asn-property-map" : {
        "uri" : "http://alto.example.com/propmap/full/inet-ia",
        "media-type" : "application/alto-propmap+json",
        "capabilities" : {
          "entity-domains": [ "ipv4", "ipv6" ],
          "properties" : [ "ISP", "ASN" ]
        }
     },
     "iacs-property-map" : {
        "uri" : "http://alto.example.com/propmap/lookup/inet-iacs",
        "media-type" : "application/alto-propmap+json",
        "accepts" : "application/alto-propmapparams+json",
        "capabilities" : {
          "entity-domains": [ "ipv4", "ipv6" ],
          "properties" : [ "ISP", "ASN", "country", "state" ]
        }
     },
     "pid-property-map" : {
        "uri" : "http://alto.example.com/propmap/lookup/pid",
        "media-type" : "application/alto-propmap+json",
        "accepts" : "application/alto-propmapparams+json",
        "uses" : [ "default-network-map" ]
        "capabilities" : {
          "entity-domains" : [ "ipv4", "ipv6" ],
          "properties" : [ "default-network-map.pid" ]
        }
     },
     "region-property-map": {
       "uri": "http://alto.exmaple.com/propmap/region",
       "media-type": "application/alto-propmap+json",
       "accepts": "application/alto-propmapparams+json",
       "uses" : [ "default-network-map" ],
       "capabilities": {
         "domain-types": [ "default-network-map.pid" ],
         "properties": [ "region" ]
       }
     },
     "legacy-pid-property" : {
        "uri" : "http://alto.example.com/legacy/eps-pid",
        "media-type" : "application/alto-endpointprop+json",
        "accepts" : "application/alto-endpointpropparams+json",
        "capabilities" : {
          "properties" : [ "default-network-map.pid" ]
        }
     }
  }
```
^[example-ird::Example IRD]


## Property Map Example {#prop-map-example}

The following example uses the properties and IRD defined above to retrieve a
Property Map for entities with the `ISP` and `ASN` properties.

Note that, to be compact, the response does not includes the entity
`ipv4:192.0.2.0`, because values of all those properties for this entity are
inherited from other entities.

Also note that the entities `ipv4:192.0.2.0/28` and `ipv4:192.0.2.16/28` are
merged into `ipv4:192.0.2.0/27`, because they have the same value of the `ASN`
property. The same rule applies to the entities `ipv4:192.0.3.0/28` and
`ipv4:192.0.3.0/28`. Both of `ipv4:192.0.2.0/27` and `ipv4:192.0.3.0/27` omit
the value for the `ISP` property, because it is inherited from
`ipv4:192.0.2.0/23`.

<!-- Done: Revise the description to match the new property map data. -->

```
GET /propmap/full/inet-ia HTTP/1.1
Host: alto.example.com
Accept: application/alto-propmap+json,application/alto-error+json
```

```
HTTP/1.1 200 OK
Content-Length: ###
Content-Type: application/alto-propmap+json

{
  "property-map": {
    "ipv4:192.0.2.0/23":   {"ISP": "BitsRus"},
    "ipv4:192.0.2.0/27":   {"ASN": "12345"},
    "ipv4:192.0.3.0/27":   {"ASN": "12346"}
  }
}
```

## Filtered Property Map Example #1 ## {#filt-prop-map-example-1}

The following example uses the filtered property map resource to request the
`ISP`, `ASN` and `state` properties for several IPv4 addresses.

Note that the value of `state` for `ipv4:192.0.2.0` is the only explicitly
defined property; the other values are all derived by the inheritance rules for
Internet address entities.

```
POST /propmap/lookup/inet-iacs HTTP/1.1
Host: alto.example.com
Accept: application/alto-propmap+json,application/alto-error+json
Content-Length: ###
Content-Type: application/alto-propmapparams+json

{
  "entities" : [ "ipv4:192.0.2.0",
                 "ipv4:192.0.2.1",
                 "ipv4:192.0.2.17" ],
  "properties" : [ "ISP", "ASN", "state" ]
}
```

```
HTTP/1.1 200 OK
Content-Length: ###
Content-Type: application/alto-propmap+json

{
  "property-map": {
    "ipv4:192.0.2.0":
           {"ISP": "BitsRus", "ASN": "12345", "state": "PA"},
    "ipv4:192.0.2.1":
           {"ISP": "BitsRus", "ASN": "12345", "state": "NJ"},
    "ipv4:192.0.2.17":
           {"ISP": "BitsRus", "ASN": "12345", "state": "CT"}
  }
}
```

## Filtered Property Map Example #2 ## {#filt-prop-map-example-2}

The following example uses the filtered property map resource to request the
`ASN`, `country` and `state` properties for several IPv4 prefixes.

Note that the property values for both entities `ipv4:192.0.2.0/26` and
`ipv4:192.0.3.0/26` are not explicitly defined. They are inherited from the
entity `ipv4:192.0.2.0/23`.

Also note that some entities like `ipv4:192.0.2.0/28` and `ipv4:192.0.2.16/28`
in the response are not listed in the request explicitly. The response includes
them because they are refinements of the requested entities and have different
values for the requested properties.

The entity `ipv4:192.0.4.0/26` is not included in the response, because there
are neither entities which it is inherited from, nor entities inherited from it.

<!--
Also note the `ASN` property has the value `12345` for both the blocks
`ipv4:192.0.2.0/28` and `ipv4:192.0.2.16/28`, so every address in the block
`ipv4:192.0.2.0/27` has that property value. However the block
`ipv4:192.0.2.0/27` itself does not have a value for `ASN`: address blocks
cannot inherit properties from blocks with longer prefixes, even if every such
block has the same value.
-->

<!-- Done: Remove the overlap of requested entities. -->

```
POST /propmap/lookup/inet-iacs HTTP/1.1
Host: alto.example.com
Accept: application/alto-propmap+json,application/alto-error+json
Content-Length: ###
Content-Type: application/alto-propmapparams+json

{
  "entities" : [ "ipv4:192.0.2.0/26",
                 "ipv4:192.0.3.0/26",
                 "ipv4:192.0.4.0/26" ],
  "properties" : [ "ASN", "country", "state" ]
}
```

```
HTTP/1.1 200 OK
Content-Length: ###
Content-Type: application/alto-propmap+json

{
  "property-map": {
    "ipv4:192.0.2.0/26":  {"country": "us"},
    "ipv4:192.0.2.0/28":  {"ASN": "12345",
                           "state": "NJ"},
    "ipv4:192.0.2.16/28": {"ASN": "12345",
                           "state": "CT"},
    "ipv4:192.0.2.0":     {"state": "PA"},
    "ipv4:192.0.3.0/26":  {"country": "us"},
    "ipv4:192.0.3.0/28":  {"ASN": "12345",
                           "state": "TX"},
    "ipv4:192.0.3.16/28": {"ASN": "12345",
                           "state": "MN"}
 }
}
```

## Filtered Property Map Example #3 ## {#filt-prop-map-example-3}

The following example uses the filtered property map resource to request the
`pid` property for several IPv4 addresses and prefixes.

Note that the entity `ipv4:192.0.3.0/27` is redundant in the response. Although
it can inherit a value of `defaultpid` for the `pid` property from the entity
`ipv4:0.0.0.0/0`, none of addresses in it is in `defaultpid`. Because blocks
`ipv4:192.0.3.0/28` and `ipv4:192.0.3.16/28` have already cover all addresses in
that block. So an ALTO server who wants a compact response can omit this entity.

<!--
Note that the value of `pid` for the prefix `ipv4:192.0.2.0/26` is `pid1`, even
though all addresses in that block are in `pid2`, because `ipv4:192.0.2.0/25`
is the longest prefix in the network map which prefix-matches
`ipv4:192.0.2.0/26`, and that prefix is in `pid1`.
-->

<!-- Done: Remove the overlap of requested entities and show the entity
addresses decomposition. -->

```
POST /propmap/lookup/pid HTTP/1.1
Host: alto.example.com
Accept: application/alto-propmap+json,application/alto-error+json
Content-Length: ###
Content-Type: application/alto-propmapparams+json

{
  "entities" : [
                "ipv4:192.0.2.128",
                "ipv4:192.0.3.0/27" ],
  "properties" : [ "default-network-map.pid" ]
}
```

```
HTTP/1.1 200 OK
Content-Length: ###
Content-Type: application/alto-propmap+json

{
  "meta" : {
    "dependent-vtags" : [
       {"resource-id": "default-network-map",
        "tag": "7915dc0290c2705481c491a2b4ffbec482b3cf62"}
    ]
  },
  "property-map": {
    "ipv4:192.0.2.128":   {"default-network-map.pid": "defaultpid"},
    "ipv4:192.0.2.0/27":  {"default-network-map.pid": "defaultpid"},
    "ipv4:192.0.3.0/28":  {"default-network-map.pid": "pid3"},
    "ipv4:192.0.3.16/28": {"default-network-map.pid": "pid4"}
  }
}
```

<!-- TODO: A new example to illustrate decomposition -->

## Filtered Property Map Example #4 ## {#filt-prop-map-example-4}

The following example uses the filtered property map resource to request the
`region` property for several PIDs defined in `default-network-map`. The value
of the `region` property for each PID is not defined by `default-network-map`,
but the reason why the PID is defined by the network operator.

```
POST /propmap/lookup/region HTTP/1.1
Host: alto.example.com
Accept: application/alto-propmap+json,application/alto-error+json
Content-Length: ###
Content-Type: application/alto-propmapparams+json

{
  "entities" : ["default-network-map.pid:pid1",
                "default-network-map.pid:pid2"],
  "properties" : [ "region" ]
}
```

```
HTTP/1.1 200 OK
Content-Length: ###
Content-Type: application/alto-propmap+json

{
  "meta" : {
    "dependent-vtags" : [
       {"resource-id": "default-network-map",
        "tag": "7915dc0290c2705481c491a2b4ffbec482b3cf62"}
    ]
  },
  "property-map": {
    "default-network-map.pid:pid1": {
      "region": "west"
    },
    "default-network-map.pid:pid2": {
      "region": "east"
    }
  }
}
```

<!--
The following example uses the Filtered Property Map resource to request the
"availbw" property for several abstract network elements.

```
POST /propmap/lookup/availbw HTTP/1.1
Host: alto.example.com
Accept: application/alto-propmap+json,application/alto-error+json
Content-Length: ###
Content-Type: application/alto-propmapparams+json

{
  "entities" : [
                "ane:L001",
                "ane:Lae0",
                "ane:L3eb ],
  "properties" : [ "availbw" ]
}


HTTP/1.1 200 OK
Content-Length: ###
Content-Type: application/alto-propmap+json

{
  "property-map": {
    "ane:L001":    {"availbw": "55"},
    "ane:Lae0":    {"availbw": "70"},
    "ane:L3eb":    {"availbw": "40"}
  }
}
```
-->
