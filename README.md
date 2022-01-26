## Overview

Image Vulnerability Analysis (IVAN) is a command-line container scanning tool that looks for OS vulnerabilities in your Docker images. IVAN performs local scanning on images in your Docker daemon and sends the package metadata to the CrowdStrike cloud for analysis. When processing is complete, a summary report displays in your terminal with detailed information about identified vulnerabilities.

### Requirements

#### Docker

To use IVAN, the latest version of Docker must be installed on the executing machine. Currently, podman and other container runtimes are unsupported.

#### Supported operating systems

- Alpine Linux: Version 3.10 or later
- Red Hat Enterprise Linux (RHEL): Version 7 or later
- Ubuntu: All Falcon-supported versions
- CentOS: Version 6 or later
- Debian: Versions 9 and 10
- SUSE Linux Enterprise Server (SLES): Versions 11 SP4, 12 SP2, 12 SP3 12 SP4, 12 SP5, 15, 15 SP1, and 15 SP2
    
#### CrowdStrike subscription

- Cloud Workload Protection
    

#### API client

A one-time setup of an API client from Support > API Clients and Keys in the Falcon console is required to use IVAN. For more information about setting up an API client, see [CrowdStrike OAuth2-Based APIs](https://falcon.crowdstrike.com/documentation/46/crowdstrike-oauth2-based-apis#api-clients).

- Client ID and secret: IVAN requires an API client ID and secret to associate the CLI with your CrowdStrike account.
    
- Falcon Container CLI scope: Your API client must be scoped to Falcon Container CLI with both read and write permissions.
    

#### Images

An image must reside in the local Docker daemon of the system where IVAN is used (either built or pulled there) and follow the scheme `repo:tag` (such as `alpine:3.13.2`)

#### Region

When scanning an image, you pass the -region parameter to specify which CrowdStrike cloud you use (such as `-region us-1`). Available clouds include `us-1`, `us-2`, `us-gov-1`, `eu-1`.

## Install the IVAN CLI binary

Download the binary to install the IVAN CLI on your machine. Put the binary in a directory that is in your `$PATH` (such as `/usr/local/bin`) and make sure to give the binary execution permissions.

## Scan an image

Run IVAN to scan an image. Use the `./ivan` command and specify a region and the image repository name and tag to scan.

  
```bash
./ivan -region us-1 -image alpine:3.13.2
```
  
IVAN will prompt you to enter the API client ID and secret when you run it first. After that, your API client ID and secret are stored in `.crowdstrike/config.json` and automatically applied in subsequent scans.

## Understanding IVAN CLI results

When processing is complete, a JSON output of all detected vulnerabilities in the image displays in your terminal.

Example result output

```json
{
	"count": 34,
	"layerHash": "cb381a32b2296e4eb5af3f84092a2e6685e88adbc54ee0768a1a1010ce6376c7",
	"os": "Alpine 3.13.2",
	"vulnerabilities": [
		{
		"CVEID": "CVE-2021-28831",
		"Product": "busybox",
		"Severity": "HIGH",
		"Version": "1.32.1-r3",
		"Description": "decompress_gunzip.c in BusyBox through 1.32.1 mishandles the error bit on the huft_build result pointer, with a resultant invalid free or segmentation fault, via malformed gzip data."
		}
	]
}
```

The scan results include the following vulnerability details:

|Field  | Nested Field | Description |
|--|--|--|
|count  |  | The total number of identified vulnerabilities. |
|layerHash  |  | The hash for the layer that has the vulnerability. |
|os  |  | The operating system and version. |
|vulnerabilities  |  | An array of detected vulnerabilities and details. |
|  |CVEID  | The Common Vulnerabilities and Exposures (CVE) ID. |
|  |Product  | The name of the product/package associated with the vulnerability. |
|  |Severity  | Severity of the CVE. One of `CRITICAL`, `HIGH`, `MEDIUM`, `LOW`, `UNKNOWN`, or `NONE`. |
|  |Version | The version of the product associated with the vulnerability. |
|  |Description | A description of the CVE. |

## Additional options

Use `--help` to see other CLI options that might come in handy when using IVAN.

|Option | Description |
| -- | -- |
| `-image` | The image to check for vulnerabilities. Images must be Docker pulled before using ivan. |
| `-license` | Print the package MIT license.|
| `-reset-creds` | Remove the current API client ID and secret for the given region and provide a new client/secret pair. |
| `-timeout` | The timeout for calling the cloud API and waiting for its response (default is 30 seconds). To modify the timeout, enter the number of seconds (as an integer) for the new timeout period. |
