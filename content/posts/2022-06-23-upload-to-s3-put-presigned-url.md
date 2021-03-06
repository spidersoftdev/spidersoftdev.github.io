---
title: How to upload to S3 with presigned URL
author: admin
type: post
date: 2022-06-23T08:30:00+00:00
url: /2022/upload-with-presigned-url-to-bucket
lead: Dealing with presigned URL
categories:
- Technologies
- Go Lang
tags:
- aws

---
Object storage is getting more and more popular. AWS started, but nowadays, almost every provider has it. Recently [Cloudflare](https://blog.cloudflare.com/introducing-r2-object-storage/) announced its availability. [DigitalOcean](https://www.digitalocean.com/products/spaces) call them Spaces.

So today, we are trying to upload a file to an S3 type of storage with a presigned URL.

TL;DR: how to generate the correct URL and upload data into it.

<!--more-->

## Overview
A presigned URL is generated by an AWS user who has access to the object. The generated URL is then given to the unauthorized user. The presigned URL can be entered in a browser or used by a program or HTML webpage. The credentials used by the presigned URL are those of the AWS user who generated the URL.

We do have two types of requests GET and PUT. The first one is for data retrieval and the second one is for data upload.

## Connection

We are going to use AWS SDK for Go v2 because it's our favourite.

The first step is to create a connection to AWS.

The assumption is that we have credentials in our environment.

`EndPoint` it's the endpoint of the AWS service or other. For example, for Cloudflare it's `https://%s.r2.cloudflarestorage.com` where `%s` should be replaced with account id. For DigitialOcean it looks similar to that `https://ams3.digitaloceanspaces.com` where `ams3`
is a region name.

`AccessKeyID`, `SecretAccessKey` also will be living in our environment.

Because we are not working with generic AWS, we have to create a custom resolver

```go
    customResolver := aws.EndpointResolverWithOptionsFunc(func(service, region string, options ...interface{}) (aws.Endpoint, error) {
        return aws.Endpoint{
            //URL: fmt.Sprintf("https://%s.r2.cloudflarestorage.com", accountId),
            URL: os.Getenv("EndPoint"),
        }, nil
    })

    // aws default config
    cfg, err = config.LoadDefaultConfig(ctx, // Hard coded credentials.
        config.WithCredentialsProvider(credentials.StaticCredentialsProvider{
            Value: aws.Credentials{
                AccessKeyID: os.Getenv("AccessKeyID"), SecretAccessKey: os.Getenv("SecretAccessKey"), SessionToken: "",
                Source: "example hard coded credentials",
            },
        }),
        config.WithEndpointResolverWithOptions(customResolver))

    if err != nil {
        log.Fatalf("unable to load SDK config, %v", err)
    }
```

in case of "regular" AWS approach we can skip `config.WithEndpointResolverWithOptions(customResolver))`


and finally, we have working code:

```go
    client := s3.NewFromConfig(cfg)
    
    input := &s3.PutObjectInput{
        Bucket: aws.String(os.Getenv("Bucket")),
        Key:    aws.String(key),
    }
    
    psClient := s3.NewPresignClient(client)

    resp, err := psClient.PresignPutObject(ctx, input)
    if err != nil {
        fmt.Println("Got an error retrieving pre-signed object:")
        return
    }
   
   fmt.Printf("Presigned URL: %s\n", resp.URL)
```

to test it, we can use curl:
```
    curl "%s" --upload-file cli.go
```
where `%s` is a presigned URL.

complete code with GET and PUT requests:

{{< gist slav123 910f2485d1ed74bf684917e67699d9b3 >}}
