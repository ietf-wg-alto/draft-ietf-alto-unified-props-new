# Examples {#examples}

## Network Map {#net-map-example}

The examples in this section use a very simple default network map:

``` text
defaultpid:  ipv4:0.0.0.0/0  ipv6:::0/0
pid1:        ipv4:192.0.2.0/25
pid2:        ipv4:192.0.2.0/28  ipv4:192.0.2.16/28
```

^[net-map-values-ex::Example Network Map]


## Property Definitions {#inet-prop-example}

The examples in this section use four additional properties beyond `pid`,
`ISP`, `ASN`, `country` and `state`, with the following values:

```
                        ISP    ASN   country   state
ipv4:192.0.2.0/24:    BitsRus   -      us       -
ipv4:192.0.2.0/28:       -    12345    -        NJ
ipv4:192.0.2.16/28:      -    12345    -        CT
ipv4:192.0.2.0:          -      -      -        PA
```

^[prop-map-values-ex::Example Property Values]


## Information Resource Directory (IRD) {#ird-example}

The following IRD defines the relevant resources of the ALTO server. It
provides two Property Maps, one for the `ISP` and `ASN` properties,
and another for the `country` and `state` properties. The server could have
provided a single Property Map for all four properties, but did not,
presumably because the organization that runs the ALTO server believes any
given client is not interested in all four properties.

The server provides two Filtered Property Maps. The first returns all four
properties, and the second just returns the `pid` property for the default
network map.

The Filtered Property Maps for the `ISP`, `ASN`, `country` and `state`
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
          "properties" : [ "pid" ]
        }
     },
     "location-property-map": {
       "uri": "http://alto.exmaple.com/propmap/location",
       "media-type": "application/alto-propmap+json",
       "accepts": "application/alto-propmapparams+json",
       "uses" : [ "default-network-map" ],
       "capabilities": {
         "domain-types": [ "pid" ],
         "properties": [ "country", "state" ]
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

The following example uses the properties and IRD defined above to retrieve
a Property Map for entities with the `ISP` and `ASN` properties. Note that the
response does not include the entity `ipv4:192.0.2.0`, because it does not have
a value for either of those properties. Also note that the entities
`ipv4:192.0.2.0/28` and `ipv4:192.0.2.16/28` are refinements of
`ipv4:192.0.2.0/24`, and hence inherit its value for `ISP` property. But
because that value is inherited, it is not explicitly listed in the Property Map.

```
GET /propmap/full/inet-ia HTTP/1.1
Host: alto.example.com
Accept: application/alto-propmap+json,application/alto-error+json


HTTP/1.1 200 OK
Content-Length: ###
Content-Type: application/alto-propmap+json

{
  "property-map": {
    "ipv4:192.0.2.0/24":   {"ISP": "BitsRus"},
    "ipv4:192.0.2.0/28":   {"ASN": "12345"},
    "ipv4:192.0.2.16/28":  {"ASN": "12345"}
  }
}
```

## Filtered Property Map Example #1 ## {#filt-prop-map-example-1}

The following example uses the Filtered Property Map resource to request the
`ISP`, `ASN` and `state` properties for several IPv4 addresses. Note that the
value of `state` for `ipv4:192.0.2.0` is the only explicitly defined property;
the other values are all derived by the inheritance rules for Internet address
entities.

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

The following example uses the Filtered Property Map resource to request the
`ASN`, `country` and `state` properties for several IPv4 prefixes. Note that
none of the returned property values is explicitly defined; all values are
derived by the inheritance rules for Internet address entities.

Also note the `ASN` property has the value `12345` for both the blocks
`ipv4:192.0.2.0/28` and `ipv4:192.0.2.16/28`, so every address in the block
`ipv4:192.0.2.0/27` has that property value. However the block
`ipv4:192.0.2.0/27` itself does not have a value for `ASN`: address blocks
cannot inherit properties from blocks with longer prefixes, even if every such
block has the same value.

```
POST /propmap/lookup/inet-iacs HTTP/1.1
Host: alto.example.com
Accept: application/alto-propmap+json,application/alto-error+json
Content-Length: ###
Content-Type: application/alto-propmapparams+json

{
  "entities" : [ "ipv4:192.0.2.0/26",
                 "ipv4:192.0.2.0/27",
                 "ipv4:192.0.2.0/28" ],
  "properties" : [ "ASN", "country", "state" ]
}


HTTP/1.1 200 OK
Content-Length: ###
Content-Type: application/alto-propmap+json

{
  "property-map": {
    "ipv4:192.0.2.0/26":  {"country": "us"},
    "ipv4:192.0.2.0/27":  {"country": "us"},
    "ipv4:192.0.2.0/28":  {"ASN": "12345",
                           "country": "us",
                           "state": "NJ"}
 }
}
```

## Filtered Property Map Example #3 ## {#filt-prop-map-example-3}

The following example uses the Filtered Property Map resource to request the
`pid` property for several IPv4 addresses and prefixes.

Note that the value of `pid` for the prefix `ipv4:192.0.2.0/26` is `pid1`, even
though all addresses in that block are in `pid2`, because `ipv4:192.0.2.0/25`
is the longest prefix in the network map which prefix-matches
`ipv4:192.0.2.0/26`, and that prefix is in `pid1`.

```
POST /propmap/lookup/pid HTTP/1.1
Host: alto.example.com
Accept: application/alto-propmap+json,application/alto-error+json
Content-Length: ###
Content-Type: application/alto-propmapparams+json

{
  "entities" : [
                "ipv4:192.0.2.0",
                "ipv4:192.0.2.16",
                "ipv4:192.0.2.64",
                "ipv4:192.0.2.128",
                "ipv4:192.0.2.0/26",
                "ipv4:192.0.2.0/30" ],
  "properties" : [ "pid" ]
}


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
    "ipv4:192.0.2.0":     {"pid": "pid2"},
    "ipv4:192.0.2.16":    {"pid": "pid2"},
    "ipv4:192.0.2.64":    {"pid": "pid1"},
    "ipv4:192.0.2.128":   {"pid": "defaultpid"},
    "ipv4:192.0.2.0/26":  {"pid": "pid1"},
    "ipv4:192.0.2.0/30":  {"pid": "pid2"}
  }
}
```

## Filtered Property Map Example #4 ## {#filt-prop-map-example-4}

The following example uses the Filtered Property Map resource to request the
`country` and `state` property for several PIDs defined in
`default-network-map`.

```
POST /propmap/lookup/location HTTP/1.1
Host: alto.example.com
Accept: application/alto-propmap+json,application/alto-error+json
Content-Length: ###
Content-Type: application/alto-propmapparams+json

{
  "entities" : ["pid:pid3",
                "pid:pid4",
                "pid:pid5",
                "pid:pid6",
                "pid:pid7"],
  "properties" : [ "country", "state" ]
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
    "pid:pid3": {
      "country": "us",
      "state": "CA"
    },
    "pid:pid4": {
      "country": "us",
      "state": "CT"
    },
    "pid:pid5": {
      "country": "ca",
      "state": "QC"
    },
    "pid:pid6": {
      "country": "ca",
      "state": "NT"
    },
    "pid:pid7": {
      "country": "fr"
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
