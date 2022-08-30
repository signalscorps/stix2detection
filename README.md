# stix2detection

stix2detection turns STIX Patterns into other detection rule languages.

stix2detection has been built in a way that most people (including non-developers) can easily add new detection languages via modules.

stix2detection has 4L

1. STIX Bunle uploaded and validated
2. Dealiasing of pattern (if needed)
3. Native STIX 2.1 object conversions applied by module
4. Custom STIX 2.1 object conversions applied by module
5. Rule translation saved

## Inputs

stix2detection accepts STIX 2.1 Indicator Objects containing a `pattern` with `pattern_type` equal to STIX.

These can be uploaded as a JSON file containing only the Indicator Object, or a STIX Bundle containing one or more Indicator Objects.

The STIX 2.1 Pattern inside the Indicator will be validated using [cti-stix-validator](https://github.com/oasis-open/cti-stix-validator) to ensure compliance.

In the case of STIX Bundles, if more than one valid Indicator SDO with a pattern, all will be processed and translated individually.

Here is an example Indicator SDO json file with a STIX Pattern;

```json
        {
            "type": "indicator",
            "spec_version": "2.1",
            "id": "indicator--a8f432cf-c7c3-40b8-9a0c-ec97d3252f65",
            "created": "2022-08-25T14:00:40.792391Z",
            "modified": "2022-08-25T14:00:40.792391Z",
            "name": "File name: test.exe",
            "indicator_types": [
                "unknown"
            ],
            "pattern": "[ file:name = 'test.exe' ]",
            "pattern_type": "stix",
            "pattern_version": "2.1",
            "valid_from": "2022-08-25T14:00:40.792391Z"
        }
```

Here is an example of a STIX bundle with two valid Indicator SDOs for conversion;


```json
{
    "type": "bundle",
    "id": "bundle--481234ef-77ed-4973-ad9a-06de1e6a66d6",
    "objects": [
        {
            "type": "indicator",
            "spec_version": "2.1",
            "id": "indicator--f7840d03-3cef-435c-b6e0-79a049434aef",
            "created": "2022-08-28T17:30:02.71357Z",
            "modified": "2022-08-28T17:30:02.71357Z",
            "name": "ipv4: 198.51.100.5",
            "indicator_types": [
                "unknown"
            ],
            "pattern": "[ ipv4-addr:value = '198.51.100.5' ]",
            "pattern_type": "stix",
            "pattern_version": "2.1",
            "valid_from": "2022-08-28T17:30:02.71357Z",
            "object_marking_refs": [
                "marking-definition--613f2e26-407d-48c7-9eca-b8e91df99dc9"
            ]
        },
        {
            "type": "indicator",
            "spec_version": "2.1",
            "id": "indicator--4557e91c-818f-4c8b-b1aa-67c41c6aa0d4",
            "created": "2022-08-28T17:30:03.016161Z",
            "modified": "2022-08-28T17:30:03.016161Z",
            "name": "ipv4: 198.0.103.12",
            "indicator_types": [
                "unknown"
            ],
            "pattern": "[ ipv4-addr:value = '198.0.103.12' ]",
            "pattern_type": "stix",
            "pattern_version": "2.1",
            "valid_from": "2022-08-28T17:30:03.016161Z",
            "object_marking_refs": [
                "marking-definition--613f2e26-407d-48c7-9eca-b8e91df99dc9"
            ]
        }
    ]
}
```

## STIX Properties

There 18 STIX SCOs, which between them have 255 property names that accept values (thus can be used in patterns).

These can be seen in the `stix_property_dictionary.json` file.

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

## Modular structure of translation

Each query language (and query product) has its own nuances and field mappings.

In order to ensure ease of extensibility of stix2detection (even for non-developers) a modular structure containing templated is defined. Each module contains;

* Field Mappings: this defines how STIX Pattern property names should be converted into the modules expected query names
* Operator Mappings: this defines how STIX Pattern operators should be convered into operators the downstream target understands.
* Module Output (optional): this defines how the translated values should be written. Default is stdout

### Module Field mappings

In many cases, products use different field names to represent the same thing. This means a mapping between source language (STIX 2.1 Patterns) and target language (represented by module) is required.

For example, in STIX 2.1 the property field name `ipv4-addr:value` exist. In Splunk CIM, this property fieldname can be `["src_ip", "src", " dest_ip", "dest"]`.

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

### Module Operator/Qualifier Mappings

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

### Module Output Templates

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


### Examples

The following examples use the `modules/sample_module`:

#### 1. Matching a File with a SHA-256 hash

Input:

```
[ file:hashes.'SHA-256' = 'aec070645fe53ee3b3763059376134f058cc337247c978add178b6ccdfb0019f' ]
```

Output:

```
search (file_hash="aec070645fe53ee3b3763059376134f058cc337247c978add178b6ccdfb0019f")
```

#### 2. Matching an Email Message with a particular From Email Address and Attachment File Name Using a Regular Expression

Input:

```
[ email-message:from_ref.value MATCHES '.+\\@example\\.com$' ]
```

Output

```
search (src_user=*) | ( rex src_user="(.+\\@example\\.com$)" )
``` 

#### 3. Matching a File with a SHA-256 hash and a PDF MIME type

Input:

```
[ file:hashes.'SHA-256' = 'aec070645fe53ee3b3763059376134f058cc337247c978add178b6ccdfb0019f' AND file:mime_type = 'application/x-pdf' ]
```

Output

```
search (file_hash="aec070645fe53ee3b3763059376134f058cc337247c978add178b6ccdfb0019f" AND mime_type = "application/x-pdf")
```

#### 4. Matching a File with SHA-256 or a MD5 hash (e.g., for the case of two different end point tools generating either an MD5 or a SHA-256), and a different File that has a different SHA-256 hash, against two different Observations

Input

```
[ file:hashes.'SHA-256' = 'bf07a7fbb825fc0aae7bf4a1177b2b31fcf8a3feeaf7092761e18c859ee52a9c' OR file:hashes.MD5 = 'cead3f77f6cda6ec00f57d76c9a6879f'] AND [file:hashes.'SHA-256' = 'aec070645fe53ee3b3763059376134f058cc337247c978add178b6ccdfb0019f' ]
```

Output

```
search (file_hash="bf07a7fbb825fc0aae7bf4a1177b2b31fcf8a3feeaf7092761e18c859ee52a9c" OR file_hash="cead3f77f6cda6ec00f57d76c9a6879f") AND (file_hash="aec070645fe53ee3b3763059376134f058cc337247c978add178b6ccdfb0019f")
```
 
#### 5. Matching a File with a MD5 hash, followed by (temporally) a Registry Key object that matches a value, within 5 minutes

Input

```
([ file:hashes.MD5 = '79054025255fb1a26e4bc422aef54eb4' ] FOLLOWEDBY [ windows-registry-key:key = 'HKEY_LOCAL_MACHINE\\foo\\bar' ]) WITHIN 300 SECONDS
```

Output

```
search (file_hash* AND registry_path=*) | transaction startswith=(file_hash="79054025255fb1a26e4bc422aef54eb4") endswith=(registry_path="HKEY_LOCAL_MACHINE\foo\bar") maxspan=300
```

#### 6. Matching three different, but specific Unix User Accounts

Input

```
[user-account:account_type = 'unix' AND user-account:user_id = '1007' AND user-account:account_login = 'Peter'] AND [user-account:account_type = 'unix' AND user-account:user_id = '1008' AND user-account:account_login = 'Paul'] AND [user-account:account_type = 'unix' AND user-account:user_id = '1009' AND user-account:account_login = 'Mary']
```

Output

```
search ( user_type="unix" AND user_id="1007" AND user="Peter" ) AND ( user_type="unix" AND user_id="1008" AND user="Paul" ) AND ( user_type="unix" AND user_id="1009" AND user="Mary" )
```

#### 7.Matching an Artifact object PCAP payload header

```
[ artifact:mime_type = 'application/vnd.tcpdump.pcap' AND artifact:payload_bin MATCHES '\\xd4\\xc3\\xb2\\xa1\\x02\\x00\\x04\\x00' ]
```

( mime_type="application/vnd.tcpdump.pcap" AND )


file:hashes.'SHA-256' = 'aec070645fe53ee3b3763059376134f058cc337247c978add178b6ccdfb0019f'

#### 8. Matching a File object with a Windows file path

Input

```
[ file:name = 'foo.dll' AND file:parent_directory_ref.path = 'C:\\Windows\\System32' ]
```

Output

```
search ( file_name="foo.dll" AND file_path="C:\Windows\System32" )
```

#### 9. Matching on a Windows PE File with high section entropy

Input

```
[ file:name = 'foo.dll' AND file:size > '10' ]
```

Output

```
search ( file_name="foo.dll" AND file_size>10
```

#### 10. Matching on Malware Beaconing to a Domain Name

[network-traffic:dst_ref.type = 'domain-name' AND network-traffic:dst_ref.value = 'example.com'] REPEATS 5 TIMES WITHIN 1800 SECONDS

 

#### 11. Matching on a Domain Name with IPv4 Resolution

```
[domain-name:value = 'www.5z8.info' AND domain-name:resolves_to_refs[*].value = '198.51.100.1/32']
```
 

#### 12. IP address or an identical IP subnet

Input

```
[ ipv4-addr:value ISSUBSET '198.51.100.0/24' ]
```

Output

Currently unsupported

#### 13. IP address or an identical IP superset

Input

```
[ ipv4-addr:value ISSUPERSET '198.51.100.0/24' ]
```

Output

Currently unsupported

#### 14. Matching on a URL

Input

```
[url:value = 'http://example.com/foo' OR url:value = 'http://example.com/bar']
```

Output

```
search ( url="http://example.com/foo" OR url="http://example.com/bar" )
```

#### 15. Matching on an X509 Certificate
 



[x509-certificate:issuer = 'CN=WEBMAIL' AND x509-certificate:serial_number = '4c:0b:1d:19:74:86:a7:66:b4:1a:bf:40:27:21:76:28']

 

#### 16. Matching on a Windows Registry Key

Input

```
[windows-registry-key:key = 'HKEY_CURRENT_USER\\Software\\CryptoLocker\\Files' OR windows-registry-key:key = 'HKEY_CURRENT_USER\\Software\\Microsoft\\CurrentVersion\\Run\\CryptoLocker_0388']
```

Output

```
search ( registry_path="HKEY_CURRENT_USER\Software\CryptoLocker\Files" OR registry_path="HKEY_CURRENT_USER\Software\Microsoft\CurrentVersion\Run\CryptoLocker_0388"
```


#### 17. Matching on a File with a set of properties

Input

```
[(file:name = 'pdf.exe' OR file:size = 371712) AND file:created = t'2014-01-13T07:03:17Z']
```

Output

```
search (( file_name="pdf.exe OR file_size=371712" )) AND
```

Matching on an Email Message with specific Sender and Subject

[email-message:sender_ref.value = 'jdoe@example.com' AND email-message:subject = 'Conference Info']

 

Matching on a Custom USB Device

[x-usb-device:usbdrive.serial_number = '575833314133343231313937']

 

Matching on Two Processes Launched with a Specific Set of Command Line Arguments Within a Certain Time Window

[process:command_line MATCHES '^.+>-add GlobalSign.cer -c -s -r localMachine Root$'] FOLLOWEDBY [process:command_line MATCHES'^.+>-add GlobalSign.cer -c -s -r localMachineTrustedPublisher$'] WITHIN 300 SECONDS

 

Matching on a Network Traffic IP that is part of a particular Subnet

[network-traffic:dst_ref.value ISSUBSET '2001:0db8:dead:beef:0000:0000:0000:0000/64']

 

Matching on several different combinations of Malware Artifacts. Note the following pattern requires that both a file and registry key exist, or that one of two processes exist.

([file:name = 'foo.dll'] AND [windows-registry-key:key = 'HKEY_LOCAL_MACHINE\\foo\\bar']) OR [process:image_ref.name = 'fooproc' OR process:image_ref.name = 'procfoo']

saved-ser

