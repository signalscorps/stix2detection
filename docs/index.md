# stix2detection Overview

STIX 2.1 has a concept of [patterns](https://docs.oasis-open.org/cti/stix/v2.1/os/stix-v2.1-os.html#_e8slinrhxcc9). STIX Patterns help to detect potentially malicious activity in logs.

Patterns are typically stored inside STIX 2.1 Indicator SDOs. For example;

```json
        {
            "type": "indicator",
            "spec_version": "2.1",
            "id": "indicator--6001e31f-34fc-4252-8a32-0e8dc3268265",
            "created": "2022-09-13T18:49:49.827719Z",
            "modified": "2022-09-13T19:05:46.759616Z",
            "name": "ipv4: 198.0.103.12:800",
            "indicator_types": [
                "unknown"
            ],
            "pattern": "[ ipv4-addr:value = '198.0.103.12' AND network-traffic:dst_port = '800' ]",
            "pattern_type": "stix",
            "pattern_version": "2.1",
            "valid_from": "2022-09-13T18:49:49.827719Z",
            "object_marking_refs": [
                "marking-definition--613f2e26-407d-48c7-9eca-b8e91df99dc9"
            ],
		    "extensions": {
		        "extension-definition--604c32d5-20f5-4f93-8547-e490e512f8d0": {
		            "extension_type": "property-extension",
		            "status": "experimental",
		            "license": "MIT",
		            "fields": ["ipv4-addr"],
		            "falsepositives": "Some software piracy tools (key generators, cracks) are classified as hack tools",
		            "level": "high",
		            "logsource": {
		   				"product": "windows",
		   				"service": "application"
		   			}
		   		}
		   	}
        }
```

You can identify the `pattern_type` as `stix`, and the `pattern` itself

```
[ ipv4-addr:value = '198.0.103.12' AND network-traffic:dst_port = '800' ]
```

The problem is, there are lots of log searching tools on the market, and each has its own detection (pattern) languague.

stix2detection makes it easy to convert Indicator SDO patterns into other target languages using modules that contain a series of configoration files.