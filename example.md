# Examples {#examples}

## Network Map {#net-map-example}

The examples in this section use a very simple default network map:

``` text
defaultpid:  ipv4:0.0.0.0/0  ipv6:::0/0
pid1:        ipv4:192.0.2.0/25
pid2:        ipv4:192.0.2.0/27
pid3:        ipv4:192.0.3.0/28
pid4:        ipv4:192.0.3.16/28
```

^[net-map-values-ex::Example Default Network Map]

And another simple alternative network map:

``` text
defaultpid:  ipv4:0.0.0.0/0  ipv6:::0/0
pid1:        ipv4:192.0.2.0/27
pid2:        ipv4:192.0.3.0/27
```

^[alt-net-map-values-ex::Example Alternative Network Map]


## Property Definitions {#inet-prop-example}

Beyond `pid`, the examples in this section use four additional properties for
Internet address domains, `ISP`, `ASN`, `country` and `state`, with the
following values:

```
                        ISP    ASN   country   state
ipv4:192.0.2.0/23:    BitsRus   -      us       -
ipv4:192.0.2.0/28:       -    12345    -        NJ
ipv4:192.0.2.16/28:      -    12345    -        CT
ipv4:192.0.2.1:          -      -      -        PA
ipv4:192.0.3.0/28:       -    12346    -        TX
ipv4:192.0.3.16/28:      -    12346    -        MN
```

^[prop-map-values-ip-ex::Example Property Values for Internet Address Domains]

And the examples in this section use the property `region` for the PID domain of
the default network map with the following values:

```
                   region
pid:defaultpid:     -
pid:pid1:           us-west
pid:pid2:           us-east
pid:pid3:           us-south
pid:pid4:           us-north
```

^[prop-map-values-pid-ex::Example Property Values for Default Network Map's PID Domain]

Note that `-` means the value of the property for the entity is "undefined". So
the entity would inherit a value for this property by the inheritance rule if
possible. For example, the value of the `ISP` property for `ipv4:192.0.2.1` is
`BitsRus` because of `ipv4:192.0.2.0/24`. But the `region` property for
`pid:defaultpid` has no value because no entity from which it can inherit.

Similar to the PID domain of the default network map, the examples in this
section use the property `ASN` for the PID domain of the alternative network map
with the following values:

```
                   ASN
pid:defaultpid:     -
pid:pid1:         12345
pid:pid2:         12346
```

^[alt-prop-map-values-pid-ex::Example Property Values for Alternative Network Map's PID Domain]

## Information Resource Directory (IRD) {#ird-example}

The following IRD defines ALTO Server information resources that are relevant
to the Entity Property Service. It provides two property maps: one for the
"ISP" and "ASN" properties, and another one for the "country" and "state"
properties. The server could have provided a single property map for all four
properties, but does not, presumably because the organization that runs the
ALTO server believes that a client is not necessarily interested in getting
all four properties.

The server provides several filtered property maps. The first returns all
four properties, and the second returns only the "pid" property for the
default network map.

The filtered property maps for the "ISP", "ASN", "country" and "state"
properties do not depend on the default network map (it does not have a
`uses` capability), because the definitions of those properties do not depend
on the default network map. The Filtered Property Map providing the "pid"
property does have a `uses` capability for the default network map, because
the default network map defines the values of the "pid" property.

Note that for legacy clients, the ALTO server provides an Endpoint Property
Service for the "pid" property defined on the endpoints of the default
network map.

The server provides another filtered Property map resource, named
"ane-dc-property-map", that returns a fictitious properties named
"storage-capacity", "ram" and "cpu" for ANEs that have a persistent
identifier. The entity domain to which the ANEs belong is "self-defined" and
valid only within the property map.

```
GET /directory HTTP/1.1
Host: alto.example.com
Accept: application/alto-directory+json,application/alto-error+json

HTTP/1.1 200 OK
Content-Length: 2827
Content-Type: application/alto-directory+json

{
  "meta" : {
    "default-alto-network-map" : "default-network-map"
  },
  "resources" : {
    "default-network-map" : {
      "uri" : "http://alto.example.com/networkmap/default",
      "media-type" : "application/alto-networkmap+json"
    },
    "alt-network-map" : {
      "uri" : "http://alto.example.com/networkmap/alt",
      "media-type" : "application/alto-networkmap+json"
    },
    "ia-property-map" : {
      "uri" : "http://alto.example.com/propmap/full/inet-ia",
      "media-type" : "application/alto-propmap+json",
      "uses": [ "default-network-map", "alt-network-map" ],
      "capabilities" : {
        "mappings": {
          "ipv4": [ ".ISP", ".ASN" ],
          "ipv6": [ ".ISP", ".ASN" ]
        }
      }
    },
    "iacs-property-map" : {
      "uri" : "http://alto.example.com/propmap/lookup/inet-iacs",
      "media-type" : "application/alto-propmap+json",
      "accepts": "application/alto-propmapparams+json",
      "uses": [ "default-network-map", "alt-network-map" ],
      "capabilities" : {
        "mappings": {
          "ipv4": [ ".ISP", ".ASN", ".country", ".state" ],
          "ipv6": [ ".ISP", ".ASN", ".country", ".state" ]
        }
      }
    },
    "region-property-map": {
      "uri": "http://alto.example.com/propmap/lookup/region",
      "media-type": "application/alto-propmap+json",
      "accepts": "application/alto-propmapparams+json",
      "uses" : [ "default-network-map", "alt-network-map" ],
      "capabilities": {
        "mappings": {
          "default-network-map.pid": [ ".region" ],
          "alt-network-map.pid": [ ".ASN" ]
        }
      }
    },
    "ip-pid-property-map" : {
      "uri" : "http://alto.example.com/propmap/lookup/pid",
      "media-type" : "application/alto-propmap+json",
      "accepts" : "application/alto-propmapparams+json",
      "uses" : [ "default-network-map", "alt-network-map" ],
      "capabilities" : {
        "mappings": {
          "ipv4": [ "default-network-map.pid",
                    "alt-network-map.pid" ],
          "ipv6": [ "default-network-map.pid",
                    "alt-network-map.pid" ]
        }
      }
    },
    "legacy-endpoint-property" : {
      "uri" : "http://alto.example.com/legacy/eps-pid",
      "media-type" : "application/alto-endpointprop+json",
      "accepts" : "application/alto-endpointpropparams+json",
      "capabilities" : {
        "properties" : [ "default-network-map.pid",
                         "alt-network-map.pid" ]
      }
    },
    "ane-dc-property-map": {
      "uri" : "http://alto.example.com/propmap/lookup/ane-dc",
      "media-type" : "application/alto-propmap+json",
      "accepts": "application/alto-propmapparams+json",
      "capabilities": {
        "mappings": {
          ".ane" : [ "storage-capacity", "ram", "cpu" ]
        }
      }
    }
  }
}
```
^[example-ird::Example IRD]


## Full Property Map Example {#prop-map-example}

The following example uses the properties and IRD defined above in
[](#ird-example) to retrieve a Property Map for entities with the "ISP" and
"ASN" properties.

Note that, to be compact, the response does not include the entity
"ipv4:192.0.2.0", because values of all those properties for this entity are
inherited from other entities.

Also note that the entities "ipv4:192.0.2.0/28" and "ipv4:192.0.2.16/28" are
merged into "ipv4:192.0.2.0/27", because they have the same value of the
"ASN" property. The same rule applies to the entities "ipv4:192.0.3.0/28" and
"ipv4:192.0.3.0/28". Both of "ipv4:192.0.2.0/27" and "ipv4:192.0.3.0/27" omit
the value for the "ISP" property, because it is inherited from
"ipv4:192.0.2.0/23".

```
GET /propmap/full/inet-ia HTTP/1.1
Host: alto.example.com
Accept: application/alto-propmap+json,application/alto-error+json
```

```
HTTP/1.1 200 OK
Content-Length: 418
Content-Type: application/alto-propmap+json

{
  "meta": {
    "dependent-vtags": [
      {"resource-id": "default-network-map",
       "tag": "3ee2cb7e8d63d9fab71b9b34cbf764436315542e"},
      {"resource-id": "alt-network-map",
       "tag": "c0ce023b8678a7b9ec00324673b98e54656d1f6d"}
    ]
  },
  "property-map": {
    "ipv4:192.0.2.0/23":   {".ISP": "BitsRus"},
    "ipv4:192.0.2.0/27":   {".ASN": "12345"},
    "ipv4:192.0.3.0/27":   {".ASN": "12346"}
  }
}
```

## Filtered Property Map Example #1 ## {#filt-prop-map-example-1}

The following example uses the filtered property map resource to request the
"ISP", "ASN" and "state" properties for several IPv4 addresses.

Note that the value of "state" for "ipv4:192.0.2.0" is the only explicitly
defined property; the other values are all derived by the inheritance rules
for Internet address entities.

```
POST /propmap/lookup/inet-iacs HTTP/1.1
Host: alto.example.com
Accept: application/alto-propmap+json,application/alto-error+json
Content-Length: 158
Content-Type: application/alto-propmapparams+json

{
  "entities" : [ "ipv4:192.0.2.0",
                 "ipv4:192.0.2.1",
                 "ipv4:192.0.2.17" ],
  "properties" : [ ".ISP", ".ASN", ".state" ]
}
```

```
HTTP/1.1 200 OK
Content-Length: 540
Content-Type: application/alto-propmap+json

{
  "meta": {
    "dependent-vtags": [
      {"resource-id": "default-network-map",
       "tag": "3ee2cb7e8d63d9fab71b9b34cbf764436315542e"},
      {"resource-id": "alt-network-map",
       "tag": "c0ce023b8678a7b9ec00324673b98e54656d1f6d"}
    ]
  },
  "property-map": {
    "ipv4:192.0.2.0":
           {".ISP": "BitsRus", ".ASN": "12345", ".state": "PA"},
    "ipv4:192.0.2.1":
           {".ISP": "BitsRus", ".ASN": "12345", ".state": "NJ"},
    "ipv4:192.0.2.17":
           {".ISP": "BitsRus", ".ASN": "12345", ".state": "CT"}
  }
}
```

## Filtered Property Map Example #2 ## {#filt-prop-map-example-2}

The following example uses the filtered property map resource to request the
"ASN", "country" and "state" properties for several IPv4 prefixes.

Note that the property values for both entities "ipv4:192.0.2.0/26" and
"ipv4:192.0.3.0/26" are not explicitly defined. They are inherited from the
entity "ipv4:192.0.2.0/23".

Also note that some entities like "ipv4:192.0.2.0/28" and
"ipv4:192.0.2.16/28" in the response are not explicitly listed in the
request. The response includes them because they are refinements of the
requested entities and have different values for the requested properties.

The entity "ipv4:192.0.4.0/26" is not included in the response, because there
are neither entities which it is inherited from, nor entities inherited from
it.

```
POST /propmap/lookup/inet-iacs HTTP/1.1
Host: alto.example.com
Accept: application/alto-propmap+json,application/alto-error+json
Content-Length: 170
Content-Type: application/alto-propmapparams+json

{
  "entities" : [ "ipv4:192.0.2.0/26",
                 "ipv4:192.0.3.0/26",
                 "ipv4:192.0.4.0/26" ],
  "properties" : [ ".ASN", ".country", ".state" ]
}
```

```
HTTP/1.1 200 OK
Content-Length: 766
Content-Type: application/alto-propmap+json

{
  "meta": {
    "dependent-vtags": [
      {"resource-id": "default-network-map",
       "tag": "3ee2cb7e8d63d9fab71b9b34cbf764436315542e"},
      {"resource-id": "alt-network-map",
       "tag": "c0ce023b8678a7b9ec00324673b98e54656d1f6d"}
    ]
  },
  "property-map": {
    "ipv4:192.0.2.0/26":  {".country": "us"},
    "ipv4:192.0.2.0/28":  {".ASN": "12345",
                           ".state": "NJ"},
    "ipv4:192.0.2.16/28": {".ASN": "12345",
                           ".state": "CT"},
    "ipv4:192.0.2.0":     {".state": "PA"},
    "ipv4:192.0.3.0/26":  {".country": "us"},
    "ipv4:192.0.3.0/28":  {".ASN": "12345",
                           ".state": "TX"},
    "ipv4:192.0.3.16/28": {".ASN": "12345",
                           ".state": "MN"}
  }
}
```

## Filtered Property Map Example #3 ## {#filt-prop-map-example-3}

The following example uses the filtered property map resource to request the
"default-network-map.pid" property and the "alt-network-map.pid" property for
a set of IPv4 addresses and prefixes.

Note that the entity "ipv4:192.0.3.0/27" is decomposed into two entities
"ipv4:192.0.3.0/28" and "ipv4:192.0.3.16/28", as they have different
"default-network-map.pid" property values.

```
POST /propmap/lookup/pid HTTP/1.1
Host: alto.example.com
Accept: application/alto-propmap+json,application/alto-error+json
Content-Length: 221
Content-Type: application/alto-propmapparams+json

{
  "entities" : [
                "ipv4:192.0.2.128",
                "ipv4:192.0.2.0/27",
                "ipv4:192.0.3.0/27" ],
  "properties" : [ "default-network-map.pid",
                   "alt-network-map.pid ]
}
```

```
HTTP/1.1 200 OK
Content-Length: 774
Content-Type: application/alto-propmap+json

{
  "meta": {
    "dependent-vtags": [
      {"resource-id": "default-network-map",
       "tag": "3ee2cb7e8d63d9fab71b9b34cbf764436315542e"},
      {"resource-id": "alt-network-map",
       "tag": "c0ce023b8678a7b9ec00324673b98e54656d1f6d"}
    ]
  },
  "property-map": {
    "ipv4:192.0.2.128":   {"default-network-map.pid": "defaultpid",
                           "alt-network-map.pid": "defaultpid"},
    "ipv4:192.0.2.0/27":  {"default-network-map.pid": "pid2",
                           "alt-network-map.pid": "pid1"},
    "ipv4:192.0.3.0/28":  {"default-network-map.pid": "pid3",
                           "alt-network-map.pid": "pid2"},
    "ipv4:192.0.3.16/28": {"default-network-map.pid": "pid4",
                           "alt-network-map.pid": "pid2"}
  }
}
```

## Filtered Property Map Example #4 ## {#filt-prop-map-example-4}

Here is an example of using the filtered property map to query the regions
for several PIDs in "default-network-map". The "region" property is specified
as a "self-defined" property, i.e., the values of this property are defined
by this property map resource.

```
POST /propmap/lookup/region HTTP/1.1
Host: alto.example.com
Accept: application/alto-propmap+json,application/alto-error+json
Content-Length: 132
Content-Type: application/alto-propmapparams+json

{
  "entities" : ["default-network-map.pid:pid1",
                "default-network-map.pid:pid2"],
  "properties" : [ ".region" ]
}
```

```
HTTP/1.1 200 OK
Content-Length: 326
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
      ".region": "us-west"
    },
    "default-network-map.pid:pid2": {
      ".region": "us-east"
    }
  }
}
```

## Filtered Property Map for ANEs Example #5 ## {#ane-example}

The following example uses the filtered property map resource
"ane-dc-property-map" to request properties "storage-capacity" and "cpu" on
several ANEs defined in this property map.

```
POST /propmap/lookup/ane-dc HTTP/1.1
Host: alto.example.com
Accept: application/alto-propmap+json,application/alto-error+json
Content-Length: 155
Content-Type: application/alto-propmapparams+json

{
  "entities" : [".ane:dc21",
                ".ane:dc45.srv9",
                ".ane:dc6.srv-cluster8"],
  "properties" : [ "storage-capacity", "cpu"]
}
```

```
HTTP/1.1 200 OK
Content-Length: 295
Content-Type: application/alto-propmap+json

{
  "meta" : {
  },
  "property-map": {
    ".ane:dc21":
      {"storage-capacity" : 40000 Gbytes, "cpu" : 500 Cores},
    ".ane:dc45.srv9":
      {"storage-capacity" : 100 Gbytes, "cpu" : 20 Cores},
    ".ane:dc6.srv-cluster8":
      {"storage-capacity" : 6000 Gbytes, "cpu" : 100 Cores}
  }
}
```
