## Inputs

stix2detection accepts STIX 2.1 Indicator Objects containing a `pattern` with `pattern_type` equal to STIX.

These can be uploaded as a JSON file containing only the Indicator Object, or a STIX Bundle containing one or more Indicator Objects. Bundles can be used to translate one  or more Indicator SDO patterns in a single command

The STIX 2.1 Pattern inside the Indicator(s) will be validated using [cti-stix-validator](https://github.com/oasis-open/cti-stix-validator) to ensure compliance.

If any patterns are not compliant, stix2detection will flag a warning and will not convert the pattern.

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