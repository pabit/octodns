# OctoDNS records

## Record types

OctoDNS supports the following record types:

* `A`
* `AAAA`
* `CNAME`
* `MX`
* `NAPTR`
* `NS`
* `PTR`
* `SSHFP`
* `SPF`
* `SRV`
* `TXT`

Underlying provider support for each of these varies and some providers have extra requirements or limitations. In cases where a record type is not supported by a provider OctoDNS will ignore it there and continue to manage the record elsewhere. For example `SSHFP` is supported by Dyn, but not Route53. If your source data includes an SSHFP record OctoDNS will keep it in sync on Dyn, but not consider it when evaluating the state of Route53. The best way to find out what types are supported by a provider is to look for its `supports` method. If that method exists the logic will drive which records are supported and which are ignored. If the provider does not implement the method it will fall back to `BaseProvider.supports` which indicates full support.

Adding new record types to OctoDNS is relatively straightforward, but will require careful evaluation of each provider to determine whether or not it will be supported and the addition of code in each to handle and test the new type.

## GeoDNS support

GeoDNS is currently supported for `A` and `AAAA` records on the Dyn (via Traffic Directors) and Route53 providers. Records with geo information pushed to providers without support for them will be managed as non-geo records using the base values.

Configuring GeoDNS is complex and the details of the functionality vary widely from provider to provider. OctoDNS has an opinionated view of how GeoDNS should be set up and does its best to map that to each provider's offering in a way that will result in similar behavior. It may not fit your needs or use cases, in which case please open an issue for discussion. We expect this functionality to grow and evolve over time as it's more widely used.

## Config (`YamlProvider`)

OctoDNS records and `YamlProvider`'s schema is essentially a 1:1 match. Properties on the objects will match keys in the config.

### Names

Each top-level key in the yaml file is a record name. Two common special cases are the root record `''`, and a wildcard `'*'`.

```
---
'':
  type: A
  values:
    - 1.2.3.4
    - 1.2.3.5
'*':
  type: CNAME
  value: www.example.com.
www:
  type: A
  values:
    - 1.2.3.4
    - 1.2.3.5
www.sub:
  type: A
  values:
    - 1.2.3.6
    - 1.2.3.7
```

The above config lays out 4 records, `A`s for `example.com.`, `www.example.com.`, and `www.sub.example.com` and a wildcard `CNAME` mapping `*.example.com.` to `www.example.com.`.

### Multiple records

In the above example each name had a single record, but there are cases where a name will need to have multiple records associated with it. This can be accomplished by using a list.

```
---
'':
  - type: A
    values:
      - 1.2.3.4
      - 1.2.3.5
  - type: MX
    values:
      - priority: 10
        value: mx1.example.com.
      - priority: 10
        value: mx2.example.com.
```

### Record data

Each record type has a corresponding set of required data. The easiest way to determine what's required is probably to look at the record object in [`octodns/records.py`](/octodns/records.py). You may also utilize `octodns-validate` which will throw errors about what's missing when run.

`type` is required for all records. `ttl` is optional. When TTL is not specified the `YamlProvider`'s default will be used. In any situation where an array of `values` can be used you can opt to go with `value` as a single item if there's only one.
