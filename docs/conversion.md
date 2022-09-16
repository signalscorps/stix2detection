# Conversions

## STIX Properties

There 18 STIX SCOs, which between them have 255 property names that accept raw values (thus can be used in patterns).

These can be seen in the `global/stix_property_dictionary.json` file.

Here is an excerpt showing the `ipv4-addr` SCO and its `value` property;

```json
 {
   "stix2detect_ref": 153,
   "property_name": "ipv4-addr:value",
   "value_type": "string",
   "value_example": "198.51.100.3",
   "sco_property_definition": "Specifies the values of one or more IPv4 addresses expressed using CIDR notation. If a given IPv4 Address object represents a single IPv4 address, the CIDR /32 suffix MAY be omitted. Example: 10.2.4.5/24"
 },
```

It is these `property_name`s that are converted into target field names of target languages by each target language modules (e.g. `ipv4-addr:value` is translated to `src_ip` in the target rule format).

Each STIX property has a `value_type` defintion and a `value_example`. These are useful to understand how the STIX property should be mapped to the target format.

* string
    * encryption-algorithm-enum
    * hash-algorithm-ov
    * windows-pebinary-type-ov
    * network-socket-address-family-enum
    * network-socket-type-enum
    * windows-integrity-level-enum
    * windows-service-start-type-enum
    * windows-service-type-enum
    * windows-service-status-enum
    * account-type-ov
    * windows-registry-datatype-enum
* binary
* integer
* timestamp
* boolean
* list of type string
* dictionary
* hex
* float

## 1. Modular structure of translation

Each query language (and query product) has its own nuances and field mappings.

In order to ensure ease of extensibility of stix2detection (even for non-developers) a modular structure containing templated is defined. Each module contains;

* Charachter/command mappings: these define how characters in STIX patterns are represented in downstream products

* Unit conversions: this defines any conversions 
  * e.g. STIX uses bytes as a size unit, many other products use kilobytes, so bytes field values need to be converted for the target sodtware in kb.
* Field Mappings: this defines how STIX Pattern property names should be converted into the modules expected query names
  * e.g. STIX has the property `ipv4-addr:value` in a target language this field might be name `src_ip` and `dest_ip` therefore the property name needs to be translated for downstream product
* Operator Mappings: this defines how STIX Pattern operators should be convered into operators the downstream target understands.
  * e.g. STIX uses the Comparison Operator `!=` (does not equal) in some downstream products this operator might be `NOT` so needs to be translated on conversion
* Module Output (optional): this defines how the translated values should be written. Default is stdout

### 1.1 Module Field mappings

In many cases, products use different field names to represent the same thing. This means a mapping between source language (STIX 2.1 Patterns) and target language (represented by module) is required.

For example, in STIX 2.1 the property field name `ipv4-addr:value` exist. In Splunk CIM (a downstream target detection product/language), this property fieldname can be `["src_ip", "src", " dest_ip", "dest"]`.

The mapping also defines the operator that should be used in the case of source STIX 2.1 mapping having multiple field mappings in the output.

The `custom_from_stix_map.json` inside each module defines the mapping between pure STIX 2.1 and the modules target language in the following schema:

```json
[
 {
   "property_name": "<PROPERTY NAME FROM stix_property_dictionary.json>",
   "property_translation": ["<TARGET FIELD NAME [N]"],
   "property_comparison": "<OPERATOR>"
 }
]
```

For example, here is a snippet for the `ipv4-addr:value` object taken from the `sample_module`;

```json
[
 {
   "property_name": "ipv4-addr:value",
   "property_translation": ["src_ip", "src", " dest_ip", "dest"],
   "property_comparison": "OR"
 }
]
```

You will see in the above examples, one STIX property can have multiple target fieldnames. When translated, these are considered with the `property_comparison`.

To ensure each comparison is captured logically, it is wrapped in parenthesis (as defined by the `operator_map.json` file in the module).

Demonstrating the example from the Sample Module, if the input STIX 2.1 Pattern was;

```
[ ipv4-addr:value = 1.1.1.1 ]
```

The output would be;

```
((src_ip="1.1.1.1" OR src="1.1.1.1" OR dest_ip="1.1.1.1" OR dest="1.1.1.1"))
```

As you will see in the sample module, there is only one `property_name` compared to the 287 in `stix_property_dictionary.json`.

This is common because in some cases field mappings do not exist between STIX 2.1 Pattern and the target language. For example, the STIX 2.1 propetry field name `file:extensions:pdf-ext.is_optimized` does not map to any field name in the Splunk CIM data model.

In this case stix2detection can be instructed to do one of two things;

1. Ignore field name / value from the target query: in this case, the property will not be translated.
2. Use field value only as plaintext: in this case the property value is entered as plaintext in the translation (without the fieldname)

Many of you will also use custom STIX properties in their own [custom extensions definitions to define new Objects or new properties](https://docs.oasis-open.org/cti/stix/v2.1/os/stix-v2.1-os.html#_32j232tfvtly). stix2detection modules can handle these in a `custom_from_stix_map.json` used in the module.

The schema of `custom_from_stix_map.json` is exactly the same as `custom_from_stix_map.json`.

Note, if a `property_name` used in `custom_from_stix_map.json` is a default STIX property (used in `stix_property_dictionary.json`) you will recieve an error when running the script.

Using the `sample_module` as an example. Lets assume a user inputs a STIX 2.1 Bundle with the following pattern;

```
[ ipv4-addr:value = '5.5.5.5' AND x-some-ip = '6.6.6.6' ]
```

This contains a native STIX property mapped in `default_from_stix_map.json` (`ipv4-addr:value = '5.5.5.5'`) and a custom STIX property mapped in `custom_from_stix_map.json` (`x-some-ip = '6.6.6.6'`) which would output

```
((src_ip="5.5.5.5" OR src="5.5.5.5" OR dest_ip="5.5.5.5" OR dest="5.5.5.5")) AND ((src_ip="6.6.6.6" AND src="6.6.6.6"))
```

### 1.2. Module Operator/Qualifier Mappings

STIX 2.1 has a range of operators and qualifiers.

* Parenthesis / brackets (e.g. `(a AND b)`)
* Observation Expression Qualifiers (e.g. `a REPEATS x TIMES`)
* Observation Operators (e.g. `[ a ] AND [ b ]`)
* Comparison Expressions (e.g. `a AND b`)
* Comparison Operators (e.g `=`, `a MATCHES b`)

On total there are 24 available variations with translations defintions that need to be defined in a module.

Using the Splunk module again, lets imagine this Pattern is to be converted;

```
[ ipv4-addr:value = 1.1.1.1 ] REPEATS 5 TIMES
```

The following Operator/Qualifier Mappings in the Splunk CIM module exist that are relevant to this pattern;

* Parenthesis / brackets: `[` = `(`
* Parenthesis / brackets: `]` = `)`
* Comparison Expressions: `AND` = `AND`
* Observation Expression Qualifiers: `REPEATS X TIMES` = append following to end of translated query `| stats count | where count > X`

```
(src_ip="1.1.1.1" OR src="1.1.1.1" OR dest_ip="1.1.1.1" OR dest="1.1.1.1") | stats count | where count > 5
```

In some cases translations do not exist do not exist in a module (because they are not supported by the downstream target). In this case, stix2detect returns an unable to translate error, detailing with mapping failed.

### 1.3. Module Output Templates

All modules have a default output as stdout. However, a user can define the creation of th

For example, Splunk queries can be stored in a `savedsearches.conf` that can be directly synced with Splunk.

```
## The following rule was processed on: <DATETIME PROCESSED>
## The version of the object processed was: <OBJECT MODIFIED PROPERTY>

[<NAME OF THE INDICTOR OBJECT> (<INDICATOR OBJECT ID>)]
description = <DESCRIPTION INDICTOR OBJECT>
dispatch.earliest_time = <VALID FROM TIME IN INDICTOR OBJECT AS EPOCH>
dispatch.latest_time = now
request.ui_dispatch_view = search
search = <THE TRANSLATION OUTPUT>
```

Each time the query is run in the modules `output` directory a `savedsearches.conf` is appended to containing all rules processed by that module.

The custom file can take the following variables:

* `process.datetime`: the time the conversion was run
* `indicator.created`: 
* `indicator.modified`: 
* `indicator.title`: 
* `indicator.description`: 
* `indicator.id`: 
* `indicator.pattern`: 
* `indicator.value`

A user can 
in 
Module outputs are useful they can create (