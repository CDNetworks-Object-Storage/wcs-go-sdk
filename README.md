# User Guide for Go SDK

## Instructions

wcs-go-sdk is a basic encapsulation with no JSON libs in it. If the returned result is a JSON sring, you have to read the result with an extra JSON lib.


## Install

1. Install with go module: `go get -u github.com/Wangsu-Cloud-Storage/wcs-go-sdk/src/lib`
2. Install with source code: copy `/src/lib` to your project.

## Initialization

AK/SK, Domain name and Upload name are required to accesss object storage. You can get them as following steps:

- Apply for CDNetworks cloud storage service.
- Log in CDNetworks SI portal, get the AccessKey and SecretKey in Security Console - AK/SK Management
- Log in CDNetworks SI portal, get Upload Domain (puturl) and Manage Domain (mgrurl) in Bucket Overview -> Bucket Settings

 Initialize as follows after you get AK/SK, Domain name and Upload name:

 ```
 auth := utility.NewAuth("<AccessKey>", "<SecretKey>")

 //config := core.NewDefaultConfig()

 config := core.NewConfig(false, "<UploadHost>", "<ManageHost>")

 ```

4. See more examples at `src/examples`.
