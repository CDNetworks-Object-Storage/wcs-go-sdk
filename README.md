# User Guide for Go SDK

## Instructions

The wcs-go-sdk provides a relatively raw encapsulation. The lib part does not include a JSON library. If the operation returns a JSON string, you need to choose a JSON library yourself, such as the built-in json package in Golang, and follow the documentation to interpret it correctly.


## Install

1. Using Go modules: `go get -u github.com/Wangsu-Cloud-Storage/wcs-go-sdk/src/lib`
2. Using source code: Copy `/src/lib` to your code directory

## Initialization

AK/SK, Domain name and Upload name are required to accesss object storage. You can get them as following steps:

- Apply for CDNetworks cloud storage service.
- Log in CDNetworks SI portal, get the AccessKey and SecretKey in Security Console - AK/SK Management
- Log in CDNetworks SI portal, get Upload Domain (puturl) and Manage Domain (mgrurl) in Bucket Overview -> Bucket Settings

 Initialize as follows after you get AK/SK, Domain name and Upload name:

Use the following code for initialization:

- Log in CDNetworks SI portal, get the AccessKey and SecretKey in Security Console - AK/SK Management.
- Log in CDNetworks SI portal, get Upload Domain (puturl) and Manage Domain (mgrurl) in Bucket Overview -> Bucket Settings

 ```
package main

import (
    "fmt"
    "github.com/CDNetworks-Object-Storage/wcs-go-sdk/src/lib/core"
    "github.com/CDNetworks-Object-Storage/wcs-go-sdk/src/lib/utility"
    "io/ioutil"
    "net/http"
    "strconv"
    "time"
)

func main() {
    // Configure authentication with AK and SK
    auth := utility.NewAuth("<AccessKey>", "<SecretKey>")
    
    // Configure whether to use HTTPS, upload domain, and management domain
    config := core.NewConfig(false, "<UploadHost>", "<ManageHost>")
    
    // Perform operations
}
 ```
## How to use
### Calculate Upload Token
 ```
package main

import (
    "fmt"
    "github.com/CDNetworks-Object-Storage/wcs-go-sdk/src/lib/utility"
)

func main() {
    // Assemble put_policy as needed
    put_policy := "{\"scope\":\"aaa\",\"deadline\":\"1893427200000\"}"
    
    auth := utility.NewAuth("aaasdfasf", "bbbsdfdsfdsafsdf")
    fmt.Printf(auth.CreateUploadToken(put_policy))
}
 ```
### Simple Upload
Directly uploading the entire file in a single request is suitable for files smaller than 2GB. If the file size exceeds 2GB, you must use multipart (segmented) uploading.
 ```
package main

import (
    "fmt"
    "github.com/CDNetworks-Object-Storage/wcs-go-sdk/src/lib/core"
    "github.com/CDNetworks-Object-Storage/wcs-go-sdk/src/lib/utility"
    "io/ioutil"
    "net/http"
    "strconv"
    "time"
)

func main() {
    auth := utility.NewAuth("<AccessKey>", "<SecretKey>")
    config := core.NewConfig(false, "<UploadHost>", "<ManageHost>")
    
    // Create a SimpleUpload instance
    su := core.NewSimpleUpload(auth, config, nil)
    
    // Set token expiration
    deadline := time.Now().Add(time.Second*3600).Unix() * 1000
    put_policy := "{\"scope\": \"bucketName\",\"deadline\": \"" + strconv.FormatInt(deadline, 10) + "\"}"
    
    // Upload file
    response, err := su.UploadFile(`C:\Windows\WindowsShell.Manifest`, put_policy, "WindowsShell.txt", nil)
    if nil != err {
        fmt.Println("UploadFile() failed:", err)
        return
    }
    
    body, _ := ioutil.ReadAll(response.Body)
    if http.StatusOK == response.StatusCode {
        fmt.Println(string(body))
    } else {
        fmt.Println("Failed, StatusCode =", response.StatusCode)
        fmt.Println(string(body))
    }
}
 ```

### Multipart Upload

 ```
package main

import (
    "fmt"
    "github.com/CDNetworks-Object-Storage/wcs-go-sdk/src/lib/core"
    "github.com/CDNetworks-Object-Storage/wcs-go-sdk/src/lib/utility"
    "io/ioutil"
    "net/http"
    "strconv"
    "time"
)

func main() {
    bucket := "bucketName"
    key := "keyName"

    auth := utility.NewAuth("<AccessKey>", "<SecretKey>")
    config := core.NewConfig(false, "<UploadHost>", "<ManageHost>")

    su := core.NewSliceUpload(auth, config, nil)
    put_extra := core.NewPutExtra(days_remain_to_delete)

    // Set token expiration
    deadline := time.Now().Add(time.Second*3600).Unix() * 1000
    put_policy := "{\"scope\": \"" + bucket + "\",\"deadline\": \"" + strconv.FormatInt(deadline, 10) + "\"}"

    var response *http.Response
    var err error
    // Upload with specified block size
    response, err = su.UploadFileWithBlockSize("localfilename", put_policy, key, put_extra, block_size)

    // Upload with specified concurrency
    response, err = su.UploadFileConcurrent(localfilename, put_policy, key, put_extra, pool_size)

    if nil != err {
        fmt.Println("UploadFile() failed:", err)
        return
    }
    body, _ := ioutil.ReadAll(response.Body)
    if http.StatusOK == response.StatusCode {
        fmt.Println(string(body))
    } else {
        fmt.Println("Failed, StatusCode =", response.StatusCode)
        fmt.Println(string(body))
    }
}

 ```

### Multipart Upload
Divide a file into a series of data chunks of specific sizes, upload these data chunks to the server separately, and then merge these data chunks into a single resource on the server.
 ```
package main

import (
    "fmt"
    "github.com/CDNetworks-Object-Storage/wcs-go-sdk/src/lib/core"
    "github.com/CDNetworks-Object-Storage/wcs-go-sdk/src/lib/utility"
    "io/ioutil"
    "net/http"
    "strconv"
    "time"
)

func main() {
    auth := utility.NewAuth("<AccessKey>", "<SecretKey>")
    config := core.NewConfig(false, "<UploadHost>", "<ManageHost>")

    bm := core.NewBucketManager(auth, config, nil)
    response, err := bm.Delete("bucket", "key")
    if nil != err {
        fmt.Sprintf("Delete() failed: %s", err)
        return
    }

    body, _ := ioutil.ReadAll(response.Body)
    if http.StatusOK == response.StatusCode {
        fmt.Println(string(body))
    } else {
        fmt.Println("Failed, StatusCode =", response.StatusCode)
        fmt.Println(string(body))
    }
}

 ```
More examples: Refer to `src/examples/test_wcslib/test_wcslib.go`
### Delete Object
 ```
package main

import (
    "fmt"
    "github.com/CDNetworks-Object-Storage/wcs-go-sdk/src/lib/core"
    "github.com/CDNetworks-Object-Storage/wcs-go-sdk/src/lib/utility"
    "io/ioutil"
    "net/http"
    "strconv"
    "time"
)

func main() {
    auth := utility.NewAuth("<AccessKey>", "<SecretKey>")
    config := core.NewConfig(false, "<UploadHost>", "<ManageHost>")

    bm := core.NewBucketManager(auth, config, nil)
    response, err := bm.Delete("bucket", "key")
    if nil != err {
        fmt.Sprintf("Delete() failed: %s", err)
        return
    }

    body, _ := ioutil.ReadAll(response.Body)
    if http.StatusOK == response.StatusCode {
        fmt.Println(string(body))
    } else {
        fmt.Println("Failed, StatusCode =", response.StatusCode)
        fmt.Println(string(body))
    }
}
 ```
More examples: Refer to `src/examples/test_wcslib/test_wcslib.go`
### Get Object Information
Get information about a file, including filename, file size, ETag, file upload time, file expiration time, file storage type, etc.
 ```
package main

import (
    "fmt"
    "github.com/CDNetworks-Object-Storage/wcs-go-sdk/src/lib/core"
    "github.com/CDNetworks-Object-Storage/wcs-go-sdk/src/lib/utility"
    "io/ioutil"
    "net/http"
    "strconv"
    "time"
)

func main() {
    auth := utility.NewAuth("<AccessKey>", "<SecretKey>")
    config := core.NewConfig(false, "<UploadHost>", "<ManageHost>")

    bm := core.NewBucketManager(auth, config, nil)
    response, err := bm.Stat(bucket, key)
    if nil != err {
        Exit(-3, fmt.Sprintf("Delete() failed: %s", err))
        return
    }

    body, _ := ioutil.ReadAll(response.Body)
    if http.StatusOK == response.StatusCode {
        fmt.Println(string(body))
    } else {
        fmt.Println("Failed, StatusCode =", response.StatusCode)
        fmt.Println(string(body))
    }
}
 ```
More examples: Refer to `src/examples/test_wcslib/test_wcslib.go`
### List Buckets
 ```
package main

import (
    "fmt"
    "github.com/CDNetworks-Object-Storage/wcs-go-sdk/src/lib/core"
    "github.com/CDNetworks-Object-Storage/wcs-go-sdk/src/lib/utility"
    "io/ioutil"
    "net/http"
    "strconv"
    "time"
)

func main() {
    auth := utility.NewAuth("<AccessKey>", "<SecretKey>")
    config := core.NewConfig(false, "<UploadHost>", "<ManageHost>")

    bm := core.NewBucketManager(auth, config, nil)
    response, err := bm.ListBucket()
    if nil != err {
        Exit(-3, fmt.Sprintf("Delete() failed: %s", err))
        return
    }

    body, _ := ioutil.ReadAll(response.Body)
    if http.StatusOK == response.StatusCode {
        fmt.Println(string(body))
    } else {
        fmt.Println("Failed, StatusCode =", response.StatusCode)
        fmt.Println(string(body))
    }
}

 ```
More examples: Refer to `src/examples/test_wcslib/test_wcslib.go`

### List Objects
List resources in a specified bucket. If there are many files, listing may need to be done in batches.
 ```
package main

import (
    "fmt"
    "github.com/CDNetworks-Object-Storage/wcs-go-sdk/src/lib/core"
    "github.com/CDNetworks-Object-Storage/wcs-go-sdk/src/lib/utility"
    "io/ioutil"
    "net/http"
    "strconv"
    "time"
)

func main() {
    auth := utility.NewAuth("<AccessKey>", "<SecretKey>")
    config := core.NewConfig(false, "<UploadHost>", "<ManageHost>")

    bm := core.NewBucketManager(auth, config, nil)

    // Specify the number of files to return in a single call, maximum is 1000
    limit := 1000

    // Specify the listing order: 0 for files first, 1 for directories first
    mode := 0

    // List files with a specific prefix
    prefix := "prefix"

    // Start listing from a specific marker, which can be obtained from the last listing result
    marker := "marker"
    // List resources, note: the last two parameters are deprecated, use "" to fill them
    response, err := bm.List("bucketName", limit, prefix, mode, marker, "", "")
    if nil != err {
        fmt.Println("List() failed:", err)
        return
    }
    body, _ := ioutil.ReadAll(response.Body)
    if http.StatusOK == response.StatusCode {
        fmt.Println(string(body))
    } else {
        fmt.Println("Failed, StatusCode =", response.StatusCode)
        fmt.Println(string(body))
    }
}
 ```
More examples: Refer to `src/examples/test_wcslib/test_wcslib.go`

### Fetch Object
Fetch a resource from a specified URL to Object Storage.
 ```
package main

import (
    "fmt"
    "github.com/CDNetworks-Object-Storage/wcs-go-sdk/src/lib/core"
    "github.com/CDNetworks-Object-Storage/wcs-go-sdk/src/lib/utility"
    "io/ioutil"
    "net/http"
    "strconv"
    "time"
)

func main() {
    auth := utility.NewAuth("<AccessKey>", "<SecretKey>")
    config := core.NewConfig(false, "<UploadHost>", "<ManageHost>")

    fm := core.NewFileManager(auth, config, nil)
    response, err := fm.Fetch(fetch_url, bucket, key, prefix, md5, decompression, notify_url, force, separate)
    if nil != err {
        Exit(-3, fmt.Sprintf("Fetch() failed: %s", err))
        return
    }

    body, _ := ioutil.ReadAll(response.Body)
    if http.StatusOK == response.StatusCode {
        fmt.Println(string(body))
    } else {
        fmt.Println("Failed, StatusCode =", response.StatusCode)
        fmt.Println(string(body))
    }
}
 ```
More examples: Refer to `src/examples/test_wcslib/test_wcslib.go`
### Audio/Video Processing
 ```
package main

import (
    "fmt"
    "github.com/CDNetworks-Object-Storage/wcs-go-sdk/src/lib/core"
    "github.com/CDNetworks-Object-Storage/wcs-go-sdk/src/lib/utility"
    "io/ioutil"
    "net/http"
    "strconv"
    "time"
)

func main() {
    auth := utility.NewAuth("<AccessKey>", "<SecretKey>")
    config := core.NewConfig(false, "<UploadHost>", "<ManageHost>")

    body := "bucket=aW1hZ2Vz&key=bGVodS5tcDQ==&fops=YXZ0aHVtYi9mbHYvcy80ODB4Mzg0fHNhdmVhcy9hVzFoWjJWek9tZHFhQzVtYkhZPQ==&force=1&separate=1"
    response, err := core.FOps(auth, config, nil, body)
    if nil != err {
        fmt.Println("FOps() failed:", err)
        return
    }
    body, _ := ioutil.ReadAll(response.Body)
    if http.StatusOK == response.StatusCode {
        fmt.Println(string(body))
    } else {
        fmt.Println("Failed, StatusCode =", response.StatusCode)
        fmt.Println(string(body))
    }
}

 ```
More examples: Refer to `src/examples/test_wcslib/test_wcslib.go`
### Image Recognition
Perform AI image recognition on a specified image resource (URL).
 ```
package main

import (
    "fmt"
    "github.com/CDNetworks-Object-Storage/wcs-go-sdk/src/lib/core"
    "github.com/CDNetworks-Object-Storage/wcs-go-sdk/src/lib/utility"
    "io/ioutil"
    "net/http"
    "strconv"
    "time"
)

func main() {
    auth := utility.NewAuth("<AccessKey>", "<SecretKey>")
    config := core.NewConfig(false, "<UploadHost>", "<ManageHost>")

    image_op := core.NewImageOp(auth, config, nil)
    response, err := image_op.ImageDetect("image", "type", "bucket")
    if nil != err {
        fmt.Println("FOps() failed:", err)
        return
    }
    body, _ := ioutil.ReadAll(response.Body)
    if http.StatusOK == response.StatusCode {
        fmt.Println(string(body))
    } else {
        fmt.Println("Failed, StatusCode =", response.StatusCode)
        fmt.Println(string(body))
    }
}
 ```
### Calculate Local File Etag
 ```
package main

import (
    "fmt"
    "github.com/Wangsu-Cloud-Storage/wcs-go-sdk/src/lib/utility"
)

func main() {
    etag := utility.ComputeEtag([]byte(data))
    fmt.Println("ETag =", etag)
}
 ```

4. More examples: Refer to `src/examples/test_wcslib/test_wcslib.go`
