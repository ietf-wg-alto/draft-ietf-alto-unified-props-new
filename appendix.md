# Scope of Property Map

Using entity domains to organize entities, an ALTO property map resource can be
regarded as given sets of properties for given entity domains. If we ignore the
resource-agnostic entity domains, we can regard an ALTO property map resource as
a set of (ri, di) => (ro, po) mappings, where (ri, di) means a resource-specific
entity domain of type di defined by the information resource ri, and (ro, po)
means a resource-specific entity property po defined by the information resource
ro.

For each (ri, di) => (ro, po) mapping, the scope of an ALTO property map
resource must be one of the cases in the following diagram:

``` text
                    domain.resource   domain.resource
                    (ri) = r          (ri) = this
                  +-----------------|-----------------+
    prop.resource | Export          | Non-exist       |
    (ro) = r      |                 |                 |
                  +-----------------|-----------------+
    prop.resource | Extend          | Define          |
    (ro) = this   |                 |                 |
                  +-----------------|-----------------+
```

where `this` represents the resulting property map resource, and `r` represents an
existing ALTO information resource other the resulting property map resource.

- ri = ro = r (`export` mode): the property map resource just transforms the
  property mapping di => po defined by r into the unified representation format
  and exports it. For example: r = `netmap1`, di = `ipv4`, po = `pid`. The
  property map resource exports the `ipv4 => pid` mapping defined by `netmap1`.
- ri = r, ro = this (`extend` mode): the property map extends properties of
  entities in the entity domain (r, di) and defines a new property po on them.
  For example: the property map resource (`this`) defines a `geolocation`
  property on domain `netmap1.pid`.
- ri = ro = this (`define` mode): the property map defines a new intrinsic
  entity domain and defines property po for each entity in this domain. For
  example: the property map resource (`this`) defines a new entity domain `asn`
  and defines a property `ipprefixes` on this domain.
- ri = this, ro = r: in the scope of a property map resource, it does not make
  sense that another existing ALTO information resource defines a property for
  this property map resource.

## Example Property Map

The following figure shows an example property map called Property Map 1, which
depends on two network maps and provides three sets of mappings by

- exporting a mapping from ipv4 entities to PIDs defined by two different network maps,
- extending geo-location properties to ipv4 entities defined by Network Map 1,
- and defining a new mapping from ASNs to traffic load properties.

~~~
                                                          (Define)
      +----------+                                     +-------------+
    ->| Property |<---------------------------+--------| asn  | load |
   /  |   Map 1  |                            |        |-------------|
  /   +----------+                            |        | 1234 | 95%  |
 |         ^                                  |        | 5678 | 70%  |
 |         |                                   \       +-------------+
 |         |          (Export)                  \       (Extend)
 |    +---------+    +------------------------+  \     +--------------+
 |    | Network |----| ipv4           | pid   |   -----| geo-location |
 |    |  Map 1  |    |------------------------|        +--------------+
 |    +---------+    | 192.168.0.0/24 | pid1  | - - -> | New York     |
 |                   | 192.168.1.0/24 | pid2  | - - -> | Shanghai     |
 |                   +------------------------+        +--------------+
 |                    (Export)
  \   +---------+    +------------------------+
   ---| Network |----| ipv4           | pid   |
      |  Map 2  |    |------------------------|
      +---------+    | 192.168.0.0/24 | Paris |
                     | ...            | ...   |
                     +------------------------+
~~~

More detailed examples are shown in [](#examples).
