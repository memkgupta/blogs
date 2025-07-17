---
title: "Implementing Multipart Uploads to Amazon S3 using Spring Boot"
seoTitle: "Implementing Multipart upload in Amazon S3 using spring boot"
seoDescription: "Learn how to implement multipart file upload to AWS S3 using Spring Boot. This guide covers generating upload IDs, managing ETags, and securely completing f"
datePublished: Thu Jul 17 2025 20:01:01 GMT+0000 (Coordinated Universal Time)
cuid: cmd7tf78e000f02jofoy19n57
slug: implementing-multipart-uploads-to-amazon-s3-using-spring-boot
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1752782327899/cd5f9656-65d6-492d-8623-30174f10cfe7.png
tags: aws, java, springboot, file-upload, s3-bucket

---

In this tutorial we will see how to handle multi-part upload in AWS S3 with spring boot.

Simply put , multipart upload is uploading a file into chunks and finally merging it back.

Advantage of this approach is simply that we get higher throughput , we can implement pausing and resuming of uploads easily and also very large files can also be uploaded seamlessly without worrying about server load and last but not the least error recovery (i.e in case of failed upload retry that part , i know that’s awesome 😼).

Let’s get started , we will be using spring boot but you can also use any other framework or language as the flow will be same.

### Workflow

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1752775986933/93a67b7e-56a2-4b48-810d-863fa21a36e4.png align="center")

There are basically 3 steps

### Initiating upload

Client initiates the upload by sending the meta data of the file i.e fileName , fileType , fileSize etc.

Server now requests the S3 to initiate a MultiPartUpload and get the uploadId as response from the S3 and the server send that uploadId to the client because that uploadId and key will be required for the future steps.

### Uploading Chunks

Once client gets the uploadId , it starts uploading chunks of file and for each chunk the client request the server for the presigned url and sends those uploadId and key for identification of the corresponding ongoing MultiPartUpload, and on the behalf of client with those UploadId and key server requests a presigned url from the S3 for that chunk and on getting that presigned url forward it to the client

Now client uploads that chunk to the S3 directly (No role of server for now) with the help of that presigned url authorised for that upload only.

**Note : For multipart upload except the final chunk each chunk is needed to be minimum of 5MB size ,** and we can also request presigned urls for multiple chunks at once and upload parallely but that will add complexity and i will leave it for you to do.

### Finalizing Upload

Now once all the chunks are uploaded client request for finalizing the upload and merge those chunks to our backend , now the point is how the hell S3 will ensure the correct order of chunks and data is valid . AWS S3 done this with the help of Etag map , for each upload to the chunk it sends a Etag in headers which relates to the chunk-number and for finalizing upload a Map of Key value pair ( key = part-number and value = the eTag for that chunk) is sent to the the backend which is then mapped to the `PartEtag` which contains two fields , which are self understandable.

```java
private int partNumber;
private String eTag;
```

Now I hope the workflow must be clear to you , now it’s time for code , but one thing which is important is left , i.e creating the bucket in AWS.

## Creating Bucket in AWS S3

1. ### Login into AWS console and search For S3
    

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1752777857594/161bc5fd-50f7-4d11-8145-9a3eb8ef6233.png align="center")

2. ### Click on create bucket
    

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1752777968367/26a8e5be-6f63-483c-aa2e-7626893022e4.png align="center")

3. ### Choose General Purpose and enter a unique name for the bucket
    

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1752778172272/110d9a08-806f-464a-bb93-92dfa7a5c8fc.png align="center")

4. ### Leave everything as default but Allow all public access for now (as our main concern is uploading today)
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1752778306358/5c518f0a-acae-4f9b-bb76-2f995ba10e42.png align="center")
    
5. ### Bucket is created , now need to create access credentials for this bucket
    

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1752778530545/7b001336-e695-46ca-aaa3-cbd7128b8da3.png align="center")

6. ### Go to IAM and create a user
    

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1752778596121/c95c0579-22a3-4d86-a9a7-603659457e48.png align="center")

7. ### Attach these policies and finalize.
    

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1752778780086/e10923da-399d-40aa-b0c5-b9fd8ed9381a.png align="center")

8. ### Now create Access Credentials
    

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1752778991836/a0f67fef-79f1-463e-932a-74865d120a13.png align="center")

9. ### Choose Application running Outside AWS
    

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1752779216776/4b1122ac-784b-477d-846d-9d4baed8b78f.png align="center")

10. ### And Get those Credentials Access Key and Access Secret and save them
    

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1752779384189/5f2f4980-25ad-4932-98a9-7d6ac03b1618.png align="center")

Now once we have created the bucket and user we are ready for the code part

### Dependencies

These are the dependencies which we will need

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>com.amazonaws</groupId>
        <artifactId>aws-java-sdk-s3</artifactId>
        <version>1.12.707</version>
    </dependency>
    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
        <optional>true</optional>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>
    <dependency>
        <groupId>org.postgresql</groupId>
        <artifactId>postgresql</artifactId>
        <scope>runtime</scope>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-jpa</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-amqp</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.amqp</groupId>
        <artifactId>spring-rabbit-test</artifactId>
        <scope>test</scope>
    </dependency>
    <dependency>
        <groupId>org.springframework.kafka</groupId>
        <artifactId>spring-kafka</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.kafka</groupId>
        <artifactId>spring-kafka-test</artifactId>
        <scope>test</scope>
    </dependency>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
    </dependency>
</dependencies>
```

### Configuring the S3 client

We can configure our S3 client by creating the bean of `AmazonS3` and we will configure the credentials inside that bean using `AWSStaticCredentialsProvider` and `AWSCredentials`

An `AmazonS3` object can be created using `AmazonS3ClientBuilder` and will set the credentials and other configuration and region of the bucket.

```java
package com.vsnt.asset_onboarding.config;

import com.amazonaws.ClientConfiguration;
import com.amazonaws.Protocol;
import com.amazonaws.auth.AWSCredentials;
import com.amazonaws.auth.AWSStaticCredentialsProvider;
import com.amazonaws.auth.BasicAWSCredentials;
import com.amazonaws.regions.Regions;
import com.amazonaws.services.s3.AmazonS3;
import com.amazonaws.services.s3.AmazonS3Client;
import com.amazonaws.services.s3.AmazonS3ClientBuilder;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class AWSConfig {

    String accessKeyId = Secrets.AWS_ACCESS_KEY_ID;

    String secretAccessKey = Secrets.AWS_SECRET_KEY;
 @Bean
    public AmazonS3 getS3Client() {
        AWSCredentials credentials = new BasicAWSCredentials(accessKeyId, secretAccessKey);
     ClientConfiguration config = new ClientConfiguration();
     config.setProtocol(Protocol.HTTP);
        AmazonS3 s3 = AmazonS3ClientBuilder.standard()
                .withCredentials(new AWSStaticCredentialsProvider(credentials))
                .withClientConfiguration(config)
                .withRegion(Regions.AP_SOUTH_1).build();
        return s3;
    }
}
```

### S3Service

Create a service class which will contain methods to interact with S3 and have that `AmazonS3` client autowired in that class.

```java
package com.vsnt.asset_onboarding.services;

import com.amazonaws.HttpMethod;
import com.amazonaws.services.s3.AmazonS3;
import com.amazonaws.services.s3.model.*;
import com.vsnt.asset_onboarding.config.Secrets;
import com.vsnt.asset_onboarding.dtos.TranscodingJob;
import lombok.RequiredArgsConstructor;
import org.springframework.stereotype.Service;

import java.net.URL;
import java.sql.Date;
import java.util.ArrayList;
import java.util.List;
import java.util.Map;

@Service

public class S3Service {
    private final AmazonS3 s3;

    public S3Service(AmazonS3 s3) {
        this.s3 = s3;
    }

    public String startMultiPartUpload(String key) {

        InitiateMultipartUploadRequest request = new InitiateMultipartUploadRequest(Secrets.AWS_BUCKET_NAME, key);
        InitiateMultipartUploadResult result = s3.initiateMultipartUpload(request);
        return result.getUploadId();
    }
    public String getPreSignedURLForMultipartUploadChunk(String uploadId,int chunkNumber,String key) {
        GeneratePresignedUrlRequest request = new GeneratePresignedUrlRequest(Secrets.AWS_BUCKET_NAME, key)
                .withMethod(HttpMethod.PUT)
                .withContentType("application/octet-stream");


        request.addRequestParameter("uploadId", uploadId);
        request.addRequestParameter("partNumber", String.valueOf(chunkNumber));
        URL url = s3.generatePresignedUrl(request);
        return url.toString();
    }
    public void completeMultipartUpload(String uploadId, Map<Integer,String> etagMap, String key)
    {
        try{
            CompleteMultipartUploadRequest request  = new CompleteMultipartUploadRequest();
            request.setUploadId(uploadId);
            request.setBucketName(Secrets.AWS_BUCKET_NAME);
            request.setKey(key);
            List<PartETag> partETags = new ArrayList<>();
            for(Map.Entry<Integer,String> etag : etagMap.entrySet())
            {
                partETags.add(new PartETag(etag.getKey(), etag.getValue()));
            }
            request.setPartETags(partETags);
        var e = s3.completeMultipartUpload(request);

          
        }
        catch (Exception e){
            e.printStackTrace();
         
        }


    }
}
```

I will discuss each method one by one

### Initiating the upload

We initate the upload with the help of `InitiateMultipartUploadRequest` class and finally return the upload id

```java
public String startMultiPartUpload(String key) {
    InitiateMultipartUploadRequest request = new InitiateMultipartUploadRequest(Secrets.AWS_BUCKET_NAME, key);
    InitiateMultipartUploadResult result = s3.initiateMultipartUpload(request);
    return result.getUploadId();
}
```

### Getting PreSignedUrl for each chunk

Now we need to generate presigned url which we can generate by `GeneratePresignedUrlRequest` class and passing the bucket name and key along with method and content type.

For a chunk we also need to add the upload id and part number as request parameters and finally generate the presigned url with the help of S3 client’s method which we autowired `generatePresignedUrl(GeneratePresignedUrlRequest)`

```java
 public String getPreSignedURLForMultipartUploadChunk(String uploadId,int chunkNumber,String key) {
        GeneratePresignedUrlRequest request = new GeneratePresignedUrlRequest(Secrets.AWS_BUCKET_NAME, key)
                .withMethod(HttpMethod.PUT)
                .withContentType("application/octet-stream");


        request.addRequestParameter("uploadId", uploadId);
        request.addRequestParameter("partNumber", String.valueOf(chunkNumber));
        URL url = s3.generatePresignedUrl(request);
        return url.toString();
    }
```

### Finalizing upload

The `completeMultipartUpload` method finalizes a multipart upload to AWS S3. It takes the `uploadId`, a map of `partNumber → ETag`, and the file key. Using this data, it creates a request to tell S3 to stitch all uploaded parts into the final complete file. Without this step, the uploaded chunks stay incomplete and unused.

Inside the method, it first creates a `CompleteMultipartUploadRequest`, sets the bucket name, object key, and the upload ID. Then it iterates over the ETag map and creates a list of `PartETag` objects — these represent each chunk’s metadata that S3 needs to verify and stitch together the parts. Once the list is built, it's passed to the request and sent using the `completeMultipartUpload` method of the `AmazonS3` client.

```java
public void completeMultipartUpload(String uploadId, Map<Integer,String> etagMap, String key)
{
    try{
        CompleteMultipartUploadRequest request  = new CompleteMultipartUploadRequest();
        request.setUploadId(uploadId);
        request.setBucketName(Secrets.AWS_BUCKET_NAME);
        request.setKey(key);
        List<PartETag> partETags = new ArrayList<>();
        for(Map.Entry<Integer,String> etag : etagMap.entrySet())
        {
            partETags.add(new PartETag(etag.getKey(), etag.getValue()));
        }
        request.setPartETags(partETags);
    var e = s3.completeMultipartUpload(request);

       
    }
    catch (Exception e){
        e.printStackTrace();
      
    }


}
```

Now you can use these methods to implement it according to your use case.

That’s it for today guys , If you have any query you can ask below see you in next article soon till then bye.