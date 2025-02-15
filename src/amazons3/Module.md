Connects to Amazon S3 from Ballerina. 

# Module Overview

The Amazon S3 client allows you to access Amazon S3 REST API through Ballerina. The following section provide you the 
details on connector operations.

**Buckets Operations**

The `wso2/amazons3` module contains operations that work with buckets. You can list the existing buckets, create a bucket, 
delete a bucket, and list objects in a bucket.

**Objects Operations**

The `wso2/amazons3` module contains operations that create an object, delete an object and retrieve an object.

## Compatibility
|                    |    Version     |  
|:------------------:|:--------------:|
| Amazon S3 API      |   2006-03-01   |

## Running Sample

Let's get started with a simple program in Ballerina to create a new bucket.

Use the following command to search for modules where the module name, description, or org name contains the word "amazons3".

```ballerina
$ ballerina search amazons3
```

This results in a list of available modules. You can pull the one you want from Ballerina Central.

```ballerina
$ ballerina pull wso2/amazons3
```

You can use the `wso2/amazons3` module to integrate with Amazon S3 back-end. Import the `wso2/amazons3` module into the Ballerina project.

Now you can use Ballerina to integrate with Amazon S3.

#### Before you Begin

You need to get credentials such as Access Key, Secret Access Key (API Secret) from Amazon S3.

**Obtaining Access Keys**

 1. Create an amazon account by visiting <https://aws.amazon.com/s3/>
 2. Create a new access key, which includes a new secret access key.
    - To create a new secret access key for your root account, use the [security credentials](https://console.aws.amazon.com/iam/home?#security_credential) page. Expand the Access Keys section, and then click Create New Root Key.
    - To create a new secret access key for an IAM user, open the [IAM console](https://console.aws.amazon.com/iam/home?region=us-east-1#home). Click Users in the Details pane, click the appropriate IAM user, and then click Create Access Key on the Security Credentials tab.
3. Download the newly created credentials, when prompted to do so in the key creation wizard.

In the directory where you have your sample, create a `ballerina.conf` file and add the details you obtained above within the quotes.

# Ballerina config file
ACCESS_KEY_ID = ""

SECRET_ACCESS_KEY = ""

REGION = ""

TRUST_STORE_PATH=""

TRUST_STORE_PASSWORD=""
```

#### Ballerina Program to Create a new Bucket
Create a new ballerina project and import the `ballerina/config` and `ballerina/http` module.

```ballerina
import ballerina/config;
import ballerina/http;
```

Add this code after the import statement to create base/parent Amazon S3 client.

```ballerina
amazons3:ClientConfiguration amazonS3Config = {
    accessKeyId: config:getAsString("ACCESS_KEY_ID"),
    secretAccessKey: config:getAsString("SECRET_ACCESS_KEY"),
    region: config:getAsString("REGION")
};

amazons3:AmazonS3Client|amazons3:ConnectorError amazonS3Client = new(amazonS3Config);
```
Here, we are creating a client object with the above configuration to connect with the Amazon S3 service.

Now you can create a new bucket in Amazon S3 by invoking the `createBucket` remote function.

```ballerina
string bucketName = "sample-amazon-bucket";
// Invoke createBucket remote function using base/parent Amazon S3 client.
amazons3:ConnectorError? createBucketResponse = s3Client->createBucket(bucketName);
```

If the creation was unsuccessful, the response from the `createBucket` function is an `amazons3:ConnectorError`.

The complete source code look similar to the following:
```ballerina
import wso2/amazons3;
import ballerina/config;
import ballerina/http;
import ballerina/io;

 // Create the ClientConfiguration that can be used to connect with the Amazon S3 service..
amazons3:ClientConfiguration amazonS3Config = {
    accessKeyId: config:getAsString("ACCESS_KEY_ID"),
    secretAccessKey: config:getAsString("SECRET_ACCESS_KEY"),
    region: config:getAsString("REGION"),
    clientConfig: {
        http1Settings: {chunking: http:CHUNKING_NEVER},
        secureSocket:{
            trustStore:{
                path: config:getAsString("TRUST_STORE_PATH"),
                password: config:getAsString("TRUST_STORE_PASSWORD")
            }
        }
    }
};

amazons3:AmazonS3Client|amazons3:ConnectorError amazonS3Client = new(amazonS3Config);

public function main() {
    // Create the AmazonS3 client with amazonS3Config. 
    amazons3:AmazonS3Client|amazons3:ConnectorError amazonS3Client = new(amazonS3Config);
    if (amazonS3Client is amazons3:AmazonS3Client) {
        string bucketName = "sample-amazon-bucket";
        amazons3:CannedACL cannedACL = amazons3:ACL_PRIVATE;
        // Invoke createBucket remote function using base/parent Amazon S3 client.
        amazons3:ConnectorError? createBucketResponse = amazonS3Client->createBucket(bucketName, cannedACL);
        if (createBucketResponse is amazons3:ConnectorError) {
            // If unsuccessful, print the error returned.
            io:println("Error: ", createBucketResponse.reason());
        } else {
            // If successful, print the status of the operation.
            io:println("Bucket Creation Status: Success");
        }
    }
}
```
Now you can run the sample using the following command:
```ballerina
$ ballerina run <module_name>
```

## Sample
```ballerina
import ballerina/config;
import ballerina/http;
import ballerina/io;
import ballerina/lang.'string as strings;

import wso2/amazons3;

amazons3:ClientConfiguration amazonS3Config = {
    accessKeyId: config:getAsString("ACCESS_KEY_ID"),
    secretAccessKey: config:getAsString("SECRET_ACCESS_KEY"),
    region: config:getAsString("REGION"),
    clientConfig: {
        http1Settings: {chunking: http:CHUNKING_NEVER},
        secureSocket:{
            trustStore:{
                path: config:getAsString("TRUST_STORE_PATH"),
                password: config:getAsString("TRUST_STORE_PASSWORD")
            }
        }
    }
};
public function main(string... args) {
    amazons3:AmazonS3Client|error amazonS3Client = new(amazonS3Config);
    if (amazonS3Client is amazons3:AmazonS3Client) {
        string bucketName = "sample-amazon-bucket";
        io:println("-----------------Calling createBucket() ------------------");
        amazons3:CannedACL cannedACL = amazons3:ACL_PRIVATE;
        amazons3:ConnectorError? createBucketResponse = amazonS3Client->createBucket(bucketName, cannedACL);
        if (createBucketResponse is amazons3:ConnectorError) {
            // If unsuccessful, print the error returned.
            io:println("Error: ", createBucketResponse.reason());
        } else {
            // If successful, print the status of the operation.
            io:println("Bucket Creation Status: Success");
        }

        io:println("-----------------Calling listBuckets() ------------------");
        var listBucketResponse = amazonS3Client->listBuckets();
        if (listBucketResponse is amazons3:Bucket[]) {
            io:println("Listing all buckets: ");
            foreach var bucket in listBucketResponse {
                io:println("Bucket Name: ", bucket.name);
            }
        } else {
            io:println("Error: ", listBucketResponse);
        }

        io:println("-----------------Calling createObject() ------------------");
        amazons3:ConnectorError? createObjectResponse = amazonS3Client->createObject(bucketName, "test.txt", "Sample content");
        if (createObjectResponse is amazons3:ConnectorError) {
            io:println("Error: ", createObjectResponse.reason());
        } else {
            io:println("Object created successfully");
        }

        io:println("-----------------Calling getObject() ------------------");
        var getObjectResponse = amazonS3Client->getObject(bucketName, "test.txt");
        if (getObjectResponse is amazons3:S3Object) {
            io:println(getObjectResponse);
            byte[]? byteArray = getObjectResponse["content"];
            if (byteArray is byte[]) {
                string content = <string>strings:fromBytes(byteArray);
                io:println("Object content: ", content);
            }
        } else {
            io:println("Error: ", getObjectResponse);
        }

        io:println("-----------------Calling listObjects() ------------------");
        var listObjectsResponse = amazonS3Client->listObjects(bucketName);
        if (listObjectsResponse is amazons3:S3Object[]) {
            io:println("Listing all object: ");
            foreach var s3Object in listObjectsResponse {
                io:println("---------------------------------");
                io:println("Object Name: ", s3Object["objectName"]);
                io:println("Object Size: ", s3Object["objectSize"]);
            }
        } else {
            io:println("Error: ", listObjectsResponse);
        }

        io:println("-----------------Calling deleteObject() ------------------");
        amazons3:ConnectorError? deleteObjectResponse = amazonS3Client->deleteObject(bucketName, "test.txt");
        if (deleteObjectResponse is amazons3:ConnectorError) {
            io:println("Error: ", deleteObjectResponse.reason());
        } else {
            io:println("Successfully deleted object");
        }

        io:println("-----------------Calling deleteBucket() ------------------");
        amazons3:ConnectorError? deleteBucketResponse = amazonS3Client->deleteBucket(bucketName);
        if (deleteBucketResponse is amazons3:ConnectorError) {
            io:println("Error: ", deleteBucketResponse.reason());
        } else {
            io:println("Successfully deleted bucket");
        }
    } else {
        io:println("Error: ", <string>amazonS3Client.detail()?.message);
    }
}
```