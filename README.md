# 基于Cloudfront的时间戳防盗链方案

## 使用标准策略创建签名url
签名url示例：
http://d111111abcdef8.cloudfront.net/image.jpg?color=red&size=medium&Expires=1357034400&Signature=nitfHRCrtziwO2HwPfWw~yYDhUF5EwRunQA-j19DzZrvDh6hQ73lDx~-ar3UocvvRQVw6EkC~GdpGQyyOSKQim-TxAnW7d8F5Kkai9HVx0FIu-5jcQb0UEmatEXAMPLE3ReXySpLSMj0yCd3ZAB4UcBCAqEijkytL6f3fVYNGQI6&Key-Pair-Id=K2JCJMDEHXQW5F

签名url由以下参数组成：
* 文件基本url：
  * http://d111111abcdef8.cloudfront.net/image.jpg?color=red&size=medium
* ？
  * 问号参数，用于查询字符串和文件基本url之间
* &（如果有）
  * 查询字符串，color=red&size=medium
* Expires=Unix时间格式（以秒为单位）
  * 您希望的URL不再允许访问文件的日期和时间，也就是该条url的过期时间
* &Signature=经过哈希处理和签名的策略声明版本
  * JSON 策略声明经过哈希处理、签署和 Base64 编码的版本
* &Key-Pair-Id
  * 您在cloudfront上配置的公有密钥ID

其中，文件基本url，问号参数，查询字符串，过期时间，key-paid-id均为指定的固定参数，根据具体url情况直接指定即可。Signature是需要进行哈希处理的签名参数。所以接下来重点介绍下签名的生成规则。


## Signigure生成规则

首先，需生成json格式的策略声明，策略中需指定对应的基本url及过期时间：

```
{
    "Statement": [
        {
            "Resource": "文件基本url，如上示例即为http://d111111abcdef8.cloudfront.net/image.jpg?color=red&size=medium",
            "Condition": {
                "DateLessThan": {
                    "AWS:EpochTime": "url过期时间，如上示例即为1357034400"
                }
            }
        }
    ]
}
```


其次，对该json文件进行SHA-1哈希编码。对于哈希函数所需的私有密钥，请使用之前配置在cloudfront上的私有密钥。示例代码如下：
```
// Signed URLs for a private distribution
// Note that Java only supports SSL certificates in DER format, 
// so you will need to convert your PEM-formatted file to DER format. 
// To do this, you can use openssl:
// openssl pkcs8 -topk8 -nocrypt -in origin.pem -inform PEM -out new.der 
//    -outform DER 
// So the encoder works correctly, you should also add the bouncy castle jar
// to your project and then add the provider.

Security.addProvider(new org.bouncycastle.jce.provider.BouncyCastleProvider());

String distributionDomain = "a1b2c3d4e5f6g7.cloudfront.net";
String privateKeyFilePath = "/path/to/rsa-private-key.der";
String s3ObjectKey = "s3/object/key.txt";
String policyResourcePath = "https://" + distributionDomain + "/" + s3ObjectKey;

// Convert your DER file into a byte array.

byte[] derPrivateKey = ServiceUtils.readInputStreamToBytes(new
    FileInputStream(privateKeyFilePath));

// Generate a "canned" signed URL to allow access to a 
// specific distribution and file

String signedUrlCanned = CloudFrontService.signUrlCanned(
    "https://" + distributionDomain + "/" + s3ObjectKey, // Resource URL or Path
    keyPairId,     // Certificate identifier, 
                   // an active trusted signer for the distribution
    derPrivateKey, // DER Private key data
    ServiceUtils.parseIso8601Date("2011-11-14T22:20:00.000Z") // DateLessThan
    );
System.out.println(signedUrlCanned);

// Build a policy document to define custom restrictions for a signed URL.

String policy = CloudFrontService.buildPolicyForSignedUrl(
    // Resource path (optional, can include '*' and '?' wildcards)
    policyResourcePath, 
    // DateLessThan
    ServiceUtils.parseIso8601Date("2011-11-14T22:20:00.000Z"), 
    // CIDR IP address restriction (optional, 0.0.0.0/0 means everyone)
    "0.0.0.0/0", 
    // DateGreaterThan (optional)
    ServiceUtils.parseIso8601Date("2011-10-16T06:31:56.000Z")
    );

// Generate a signed URL using a custom policy document.

String signedUrl = CloudFrontService.signUrl(
    // Resource URL or Path
    "https://" + distributionDomain + "/" + s3ObjectKey, 
    // Certificate identifier, an active trusted signer for the distribution
    keyPairId,     
    // DER Private key data
    derPrivateKey, 
    // Access control policy
    policy 
    );
System.out.println(signedUrl);
```

得到签名url后，需对url再次进行字符串进行 Base64 编码









