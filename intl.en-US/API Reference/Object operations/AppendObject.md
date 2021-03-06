# AppendObject {#reference_fvf_xld_5db .reference}

AppendObject is used to upload a file by appending the file to an existing object.

An object created with the AppendObject operation is an appendable object, and an object uploaded with the PutObject operation is a normal object.

**Note:** 

-   You cannot use AppendObject to upload a file to an object protected by the WORM policy.
-   You cannot use KMS to encrypt appendable objects on the server by specifying CMK IDs for them.

## Association with other operations {#section_psd_yw2_5gb .section}

|Operations|Relationship|
|:---------|:-----------|
|[PutObject](intl.en-US/API Reference/Object operations/PutObject.md)|If you perform a PutObject operation on an existing appendable object, the appendable object is overwritten by a new normal object.|
|[HeadObject](intl.en-US/API Reference/Object operations/HeadObject.md)|If you perform a HeadObject operation on an existing appendable object, then x-oss-next-append-position, x-oss-hash-crc64ecma, and x-oss-object-type are returned. The x-oss-object-type of the appendable object is Appendable.|
|[GetBucket](intl.en-US/API Reference/Object operations/GetObject.md)|In the response to a GetBucket request, the x-oss-object-type of the appendable object is set to Appendable.|
|[CopyObject](intl.en-US/API Reference/Object operations/CopyObject.md)|You can neither use CopyObject to copy an appendable object, nor change the server-side encryption method of this object. However, you can use CopyObject to modify the custom metadata of an object.|

## Request syntax {#section_n23_kpw_bz .section}

```
POST /ObjectName? append&position=Position HTTP/1.1
Content-Length：ContentLength
Content-Type: ContentType
Host: BucketName.oss.aliyuncs.com
Date: GMT Date
Authorization: SignatureValue
```

## Parameters in an AppendObject request {#section_crp_5zm_qgb .section}

An AppendObject request must include the append and position parameters, which are both CanonicalizedResource and must be included in the signature.

-   append

    This parameter indicates that the request is sent to perform an AppendObject operation.

-   position

    This parameter specifies the position from where the append operation starts. The value of position in the first AppendObject operation must be 0, and the value of position in the subsequent operation is the current object length. For example, if the value of position specified in the first AppendObject request is 0, and the value of content-length is 65536, the value of position specified in the second AppendObject request must be set to 65536.

    Each time after an AppendObject operation succeeds, x-oss-next-append-position in the response header specifies the position of the next AppendObject request.

    Note the following when setting position:

    -   If the value of position is 0 and an object with the same name does not exist, you can set headers \(such as x-oss-server-side-encryption\) in the AppendObject request in the same way as you do in a PutObject request. If you add a correct x-oss-server-side-encryption header in an AppendObject request in which the value of position is 0, the x-oss-server-side-encryption header is also included in the response header. You can initiate a CopyObject request to modify the metadata of the object in subsequent operations.
    -   If the value of position is 0 and an appendable object with the same name does not exist, or if the length of an appendable object with the same name is 0, the AppendObject operation is successful. Otherwise, the system determines that the position and object length do not match and returns a PositionNotEqualToLength error code.
    -   The length limit of an object generated by an AppendObject operation is the same as that of an object generated by a PutObject operation. Each time after an AppendObject operation is performed, the last modification time of this object is updated.
    -   If the position value is correct and content with a length of 0 is appended to an existing appendable object, the status of the object does not change.

## Request headers {#section_x4j_lpw_bz .section}

|Header|Type|Description|
|------|----|-----------|
|Cache-Control|String|Specifies the Web page caching behavior for the object. For more information, see [RFC2616](https://www.ietf.org/rfc/rfc2616.txt). Default value: none

|
|Content-Disposition|String|Specifies the name of the object when the object is downloaded. For more information, see [RFC2616](https://www.ietf.org/rfc/rfc2616.txt). Default value: none

|
|Content-Encoding|String|Specifies the content encoding format of the object. For more information, see [RFC2616](https://www.ietf.org/rfc/rfc2616.txt). Default value: none

|
|Content-MD5|String|Content-MD5 is a string calculated by the MD5 algorithm. This header is used to check whether the message content is consistent with the sent content. The value of Content-MD5 can be obtained as follows: Calculate a 128-bit number based on the message content, rather than the header, and then base64-encode the number.

Default value: none

Restriction: none

|
|Expires|Integer|Specifies the expiration time. For more information, see [RFC2616](https://www.ietf.org/rfc/rfc2616.txt). Default value: none

|
|x-oss-server-side-encryption|String|Specifies the server-side encryption algorithm.Valid values: AES256 or KMS

**Note:** You must enable KMS \(Key Management Service\) in the console before you can use the KMS encryption algorithm. Otherwise, a KmsServiceNotEnabled error is returned.

|
|x-oss-object-acl|String|Specifies the ACL for the object. Valid values: public-read, private, and public-read-write

|
|x-oss-storage-class|String|Specifies the storage class of the object.Values:

-   Standard
-   IA
-   Archive

Supported interfaces: PutObject, InitMultipartUpload, AppendObject, PutObjectSymlink, and CopyObject

**Note:** 

-   The status code 400 Bad Request is returned if the value of StorageClass is invalid. Error description: InvalidArgument.
-   If you specify the value of x-oss-storage-class when uploading an object to a bucket, the storage class of the uploaded object is the specified value of x-oss-storage-class regardless of the storage class of the bucket. For example, if you specify the value of x-oss-storage-class to Standard when uploading an object to a bucket of the IA storage class, the storage class of the object is Standard.
-   This header takes effect only if you specify it when you perform the AppendObject operation for the first time.

|

## Response headers {#section_xbv_rpw_bz .section}

|Header|Type|Description|
|------|----|-----------|
|x-oss-next-append-position|64-bit integer|Specifies the position that must be provided in the next request, that is, the current object length. This header is returned when a successful message is returned for an AppendObject request, or when a 409 error occurs because the position and the object length do not match.|
|x-oss-hash-crc64ecma|64-bit integer|Specifies the 64-bit CRC value of the object. This value is calculated according to the [ECMA-182](http://www.ecma-international.org/publications/standards/Ecma-182.htm).|

## CRC64 calculation method {#section_xb5_xpw_bz .section}

The CRC value of an appendable object is calculated according to [ECMA-182](http://www.ecma-international.org/publications/standards/Ecma-182.htm). You can calculate the CRC64 in the following methods:

-   Calculate using boost CRC module:

    ```
    typedef boost::crc_optimal<64, 0x42F0E1EBA9EA3693ULL, 0xffffffffffffffffULL, 0xffffffffffffffffULL, true, true> boost_ecma;
    
    uint64_t do_boost_crc(const char* buffer, int length)
    {
        boost_ecma crc;
        crc.process_bytes(buffer, length);
        return crc.checksum();
    }
    ```

-   Calculate using the Python crcmod:

    ```
    do_crc64 = crcmod.mkCrcFun(0x142F0E1EBA9EA3693L, initCrc=0L, xorOut=0xffffffffffffffffL, rev=True)
    
    print do_crc64(“123456789”)
    ```


## Example {#section_phh_cqw_bz .section}

Request example:

```
POST /oss.jpg? append&position=0 HTTP/1.1 
Host: oss-example.oss.aliyuncs.com 
Cache-control: no-cache 
Expires: Wed, 08 Jul 2015 16:57:01 GMT 
Content-Encoding: utf-8 
x-oss-storage-class: Archive
Content-Disposition: attachment;filename=oss_download.jpg 
Date: Wed, 08 Jul 2015 06:57:01 GMT 
Content-Type: image/jpg 
Content-Length: 1717 
Authorization: OSS qn6qrrqxo2oawuk53otfjbyc:kZoYNv66bsmc10+dcGKw5x2PRrk=  
[1717 bytes of object data]
```

Response example:

```
HTTP/1.1 200 OK
Date: Wed, 08 Jul 2015 06:57:01 GMT
ETag: "0F7230CAA4BE94CCBDC99C5500000000"
Connection: keep-alive
Content-Length: 0  
Server: AliyunOSS
x-oss-hash-crc64ecma: 14741617095266562575
x-oss-next-append-position: 1717
x-oss-request-id: 559CC9BDC755F95A64485981
```

## Error messages {#section_oj4_zfn_qgb .section}

|Error message|HTTP status code|Description|
|:------------|:---------------|:----------|
|ObjectNotAppendable|409|You cannot perform AppendObject operations on a non-appendable object.|
|PositionNotEqualToLength|409|The value of position does not match the current object length. You can obtain the position for the next operation from the response header x-oss-next-append-position and initiate a request again.**Note:** 

-   Although multiple requests may be sent concurrently, even if you set the value of x-oss-next-append-position in one request, the request may still fail because the value is not updated immediately.
-   The PositionNotEqualToLength error message is returned if the value of position is 0 and the length of an appendable object with the same name is not 0.

|

