# messagesign

A rust library to sign requests to mehal services based on the [AWS S3V4 approach](https://docs.aws.amazon.com/AmazonS3/latest/API/sigv4-query-string-auth.html). 
 
The implementation is based on the [s3v4](https://github.com/uv-rust/s3v4) library. 

 This crate provides a `signature` function that can be used to sign a request to an mehal services.

 Both functions return an `Error` generated by the `error_chain` crate which can be 
 converted to a `String` or accessed through the `description` method or `display_chain` 
 and `backtrace` methods in case a full backtrace is needed.

[![Build](https://github.com/mehal-tech/messagesign/actions/workflows/build-test.yaml/badge.svg)](https://github.com/mehal-tech/messagesign/actions/workflows/build-test.yaml)
[![codecov](https://codecov.io/gh/mehal-tech/messagesign/graph/badge.svg?token=S1X67D6K6I)](https://codecov.io/gh/mehal-tech/messagesign)
[![OpenSSF Best Practices](https://www.bestpractices.dev/projects/10183/badge)](https://www.bestpractices.dev/projects/10183)

 # Examples
 
 ## Signing a request
 ```rust
    let url = url::Url::parse("https://mehal.tech/endpoint").unwrap();
    let signature: messagesign::Signature = messagesign::signature(
        &url, // The endpoint of the mehel services
        "GET",   // The http Method  
        "ivegotthekey",  // the access key provided with your secret
        "ivegotthesecret", // The secret provided for your project
        "global", // A supported region See mehal.tech docs
        &"brog",
        "machineid", // The data in /etc/machine-id
        "hostname", // The data in /etc/machine-id
        "UNSIGNED-PAYLOAD", //payload hash, or "UNSIGNED-PAYLOAD"
        "", // An empty string or a random u32
    ).map_err(|err| format!("Signature error: {}", err.display_chain()))?;
``` 
 
 ### Using the signature data to make a request 

 #### Hyper 
 ```rust
        let req = Request::builder()
        .method(Method::GET)
        .header("x-mhl-content-sha256", "UNSIGNED-PAYLOAD")
        .header("x-mhl-date", &signature.date_time)
        .header("x-mhl-mid", &machineid)
        .header("authorization", &signature.auth_header)
 ```
 #### Ureq
 ```rust
    let agent = AgentBuilder::new().build();
    let response = agent
        .put(&uri)
        .set("x-mhl-content-sha256", "UNSIGNED-PAYLOAD")
        .set("x-mhl-date", &signature.date_time)
        .set("x-mhl-mid", &machineid)
        .set("authorization", &signature.auth_header)
 ```