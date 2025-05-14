# pGina.Fork 3.9.9.12

The [pGina.Fork 3.9.9.12](https://github.com/MutonUfoAI/pgina/releases/tag/3.9.9.12) software and earlier versions are vulnerable to an authentication bypass via DNS poisoning when the `HttpAuth` plugin is used for authentication. During each authentication attempt, the `HttpAuth` plugin performs a DNS TXT record query for the hardcoded domain `pginaloginserver`. If the response to this query can be controlled, it is possible to authenticate to the target system and permanently overwrite the existing authentication server address, thereby redirecting all plaintext credentials to an adversary-controlled server.

When the `HttpAuth` plugin is enabled for authentication, it verifies whether a user is valid by issuing an HTTP POST request in the `getResponse()` method, located in the file `Plugins/HttpAuth/HttpAuth/Accessor.cs`. The only validation performed by this method is to ensure that no exception (`WebException webx`) occurs. Therefore, if a [200 OK](https://github.com/MutonUfoAI/pgina/blob/master/Plugins/HttpAuth/HttpAuth/Accessor.cs#L44) response is received, the application will confirm the authenticity of the user. Given this, the [return value](https://github.com/MutonUfoAI/pgina/blob/master/Plugins/HttpAuth/HttpAuth/Accessor.cs#L26) from the `Settings.resolveSettings()` method becomes the exploition point.

![[images/01.png]]

The `Settings.resolveSettings()` method, located in `Plugins/HttpAuth/HttpAuth/Settings.cs`, is called each time a user attempts to authenticate. The application performs a series of checks to determine the current location of the logon server. Four distinct sources can be used to identify which logon server should be used when sending the authentication request: `_urlByEnvVar()`, `_getTxtRecords()`, `@DEFAULT_URL`, or `m_settings.Loginserver`. Technically, since no mutual authentication (mTLS) functionality is present, any of these four methods can be leveraged for exploitation, as if one fails, the others will be checked in sequence.

![[images/02.png]]

Focusing on the `_getTxtRecords()` method, the application reaches out to a [hardcoded value](https://github.com/MutonUfoAI/pgina/blob/master/Plugins/HttpAuth/HttpAuth/Settings.cs#L44) of `pginaloginserver`. During authentication, it attempts to resolve the logon server by creating a new process and executing the command `nslookup -type=TXT pginaloginserver`. Depending on the system’s DNS configuration and how `nslookup` leverages DNS search suffixes defined in the host resolver settings, the actual domain queried may vary slightly across environments, but it will always contain the string `pginaloginserver`.

![[images/03.png]]

The vulnerability in this instance was exploited using ARP poisoning to perform a DNS resolution takeover on the lookup request, allowing an adversary to return a fake response to the application. 

If `HttpAuth` is configured for `Authentication`, and `Gateway`, alongside `Local Machine` being configured for `Gateway`, it is possible to assign the unauthenticated user any [local group permissions](https://github.com/MutonUfoAI/pgina/blob/master/Plugins/HttpAuth/README.md?plain=1#L13). With the DNS resolution taken over, when a user attempts to authenticate, given the hierarchy of `Settings.resolveSettings()`, `nslookup.exe` is executed using the hardcoded value of `pginaloginserver`.

![[images/05.png]]

With both sending and receiving controlled due to the positioned resolution, DNS lookup requests can be altered. This works both as a means to capture plaintext credentials of users authenticating to devices on the network, as well as a method to gain administrative access to a device.

![[images/06.png]]

Behind the scenes, looking at the pGina.Fork log files, the content from the malicious HTTP server (chosen from DNS) can be seen being processed by the application. Eventually, based on the configuration, groups are assigned based on information present within the response from the server.

![[images/07.png]]
![[images/09.png]]

### Affected Assets
**pGina.Fork <= 3.9.9.12**
* https://github.com/MutonUfoAI/pgina/archive/3.9.9.12.zip

### CVE-ID


### CWE
CWE-290: Authentication Bypass by Spoofing, CWE-350: Reliance on \*DNS Resolution for a Security-Critical Action

### Credit
Kyle Burns (kburns@packetlabs.net), Jordan Guy (jguy@packetlabs.net), Ian Lin (ilan@packetlabs.net)
