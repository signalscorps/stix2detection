# stix2detection

stix2detection turns STIX Patterns into other detection rule languages.

stix2detection has been built in a way that most people (including non-developers) can easily add new detection languages via modules.

stix2detection has 4L

1. STIX Bunle uploaded and validated
2. Dealiasing of pattern (if needed)
3. Native STIX 2.1 object conversions applied by module
4. Custom STIX 2.1 object conversions applied by module
5. Rule translation saved








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

