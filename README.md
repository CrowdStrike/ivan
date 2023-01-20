## Overview

Falcon Image Vulnerability Analysis (IVAN) is a command-line image assessment tool. It works by creating an inventory of packages on an image and then sending the package metadata to the CrowdStrike cloud for assessment.

IVAN results are returned as a JSON report in the terminal. IVAN differs from other methods of image assessment because only the image metadata is uploaded to the CrowdStrike cloud. The image and metadata do not appear anywhere in the Falcon Console.

### Requirements

- **Docker**: You must have the latest version of Docker.
- **CrowdStrike subscription**: Falcon Cloud Workload Protection
- **API client**: You can create a new API client on [API Client and keys](https://falcon.crowdstrike.com/support/api-clients-and-keys).
	- Your API client must have **Falcon Container CLI** scope with `Write` permission.

> **Note**
> To use IVAN, the latest version of Docker must be installed on the executing machine. Currently, podman and other container runtimes are unsupported.

#### Supported operating systems

| OS | Supported versions |
| ------ | ------ |
| Alpine Linux      | 3.9 through 3.17.9      |
|Amazon Linux|1, 2|
| CentOS      | 7 through 8.3      |
| Debian GNU      | 9, 10, 11      |
|Oracle Linux|6.0 through 8.9|
| Red Hat Enterprise Linux (RHEL)      | 7 through 8.6      |
| SUSE Linux Enterprise Server (SLES)      | 11.4, 12.2, 12.3, 12.4, 12.5, 15, 15.0, 15.1, 15.2 |
| Ubuntu      | 16.04, 18.04, 20.04, 22.04      |

#### IVAN releases

You can download the latest IVAN release at [https://github.com/CrowdStrike/ivan/releases](https://github.com/CrowdStrike/ivan/releases).

### Install IVAN
Download IVAN and make it executable.

1. Download the latest version of IVAN for your OS from [here](https://github.com/CrowdStrike/ivan/releases).
2. Extract the archive.
	In a terminal, run:
	```sh
	tar xvzf ivan_<version>.tar.gz
	```
3. Make the binary executable.
	In a terminal, run:
	```sh
	chmod +ux ivan
	```
4. (Optional) Move the binary into `$PATH` (example:_/usr/local/bin_).

### Authenticate IVAN
Provide IVAN with your CrowdStrike API client ID and secret. You are prompted for these credentials the first time you run IVAN or when you use the `-reset-creds` option.

If you want to set up non-interactive shell login, set the API client ID and secret as environment variables:
```sh
export FALCON_CLIENT_ID=<clientID>
export FALCON_CLIENT_SECRET=<clientSecret>
```

> **Note**
> To create an API client, see [API Client and keys](https://falcon.crowdstrike.com/support/api-clients-and-keys).

Your API credentials are applied automatically for subsequent image assessments. The credentials are stored in _$HOME/crowdstrike/config.json_.

```json
{
 "region": {
   "client_id": "e2f…d06",
   "client_secret": "aba…4To"
 },
 "region2": {
   "client_id": "l9f…d06",
   "client_secret": "cdc…j4To"
 },
 "region3": {
   "client_id": "p6f…d06",
   "client_secret": "plo…nj4To"
 }
}
```

#### Image assessment location

IVAN assesses images through the Docker daemon. Use docker pull to make images available for IVAN, or load local images to Docker by running the following command:

```sh
docker load < <image_name>
```

### IVAN synopsis

Use this syntax to run IVAN image assessment on a Docker image.

`ivan [options] [region] [image]`

**Image (required)**

`-image <imageName:tag>`

Specifies the image to assess. If a tag is not specified, Docker appends latest tag to the image name.

**Region (required)**

`-region <string>`

Sets the CrowdStrike cloud region. Possible values are `us-1`, `us-2`, `eu-1`, `us-gov-1`.

**Options**

`-dry-run`

Lists the image packages but doesn’t send it to the CrowdStrike cloud for image assessment.

`-license`

Prints the IVAN license to the terminal.

`-timeout <integer>`

Sets the client timeout duration. The default is 30 seconds.

`-reset-creds`

Initiates terminal prompt to re-enter API client ID and password.

### Image assessment report

The report returns the following info in JSON format:

| Object          | Type    | Description                                                                           |
| --------------- | ------- | ------------------------------------------------------------------------------------- |
| count           | integer | The count of vulnerabilities on image                                                 |
| layerHash       | string  | The layer hash containing the vulnerabilities                                         |
| os              | string  | The OS and version on the image                                                       |
| vulnerabilities | array   | An array of vulnerabilities and their info                                            |
| CVEID           | string  | The Common Vulnerabilities and Exposures (CVE) ID of the vulnerability                |
| Product         | string  | The product name associated with the vulnerability                                    |
| Severity        | string  | The CVE severity of the vulnerability: CRITICAL, HIGH, MEDIUM, LOW, UNKNOWN, or NONE. |
| Version         | string  | The version of the product associated with the vulnerability                          |
| Description     | string  | The CVE description                                                                   |

### Examples using IVAN

Here are some examples of the input and output for assessing images with IVAN.

**Assess an image**

```sh
ivan -region us-1 -image alpine:3.17.0
```

_Output when vulnerabilities are found:_
```json
{
  "count": 2,
  "layerHash": "7528…c933",
  "os": "Alpine 3.17.0",
  "vulnerabilities": [
   {
    "CVEID": "CVE-2022-3996",
    "Product": "libcrypto3",
    "Severity": "HIGH",
    "Version": "3.0.7-r0",
    "Description": "If an X.509 certificate … functions."
   },
   {
    "CVEID": "CVE-2022-3996",
    "Product": "libssl3",
    "Severity": "HIGH",
    "Version": "3.0.7-r0",
    "Description": "If an X.509 certificate … functions."
   }
  ]
 }
```

_Output when no vulnerabilities are found:_

```json
{
  "count": 0,
  "layerHash": "b1a6…7392",
  "os": "Ubuntu 20.04",
  "vulnerabilities": null
 }
```

**List the inventory of packages on an image**
```sh
ivan -dry-run -region us-1 -image myApp:latest
```

> **Note**
> The `-dry-run` option blocks the inventory from being sent to the CrowdStrike cloud for image assessment. The inventory shows a complete list of packages found on the image. It does not show package vulnerabilities.

```json
{
  "osversion": "Ubuntu 16.04",
  "packages": [
    {
      "Vendor": "Ubuntu Core developers",
      "Product": "libquadmath0",
      "MajorVersion": "5.4.0-6ubuntu1~16.04.12",
      "SoftwareArchitecture": "amd64",
      "PackageProvider": "DPKG",
      "PackageSource": "libquadmath0 5.4.0-6ubuntu1~16.04.12"
    },
    
    ...
    
],
"applicationPackages": [
    {
      "type": "PYTHON",
      "libraries": [
        {
          "Name": "pip",
          "Version": "19.0.3",
          "License": "Unknown",
          "LayerHash": "2fcf…c367f"
        },
        {
          "Name": "PyYAML",
          "Version": "5.4.1",
          "License": "Unknown",
          "LayerHash": "ea8d…507e1"
        }
      ]
    }
  ]
}
```
