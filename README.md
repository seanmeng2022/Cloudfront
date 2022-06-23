# 基于Cloudfront的时间戳防盗链方案

## 使用标准策略创建签名url
签名url示例：
http://d111111abcdef8.cloudfront.net/image.jpg?color=red&size=medium&Expires=1357034400&Signature=nitfHRCrtziwO2HwPfWw~yYDhUF5EwRunQA-j19DzZrvDh6hQ73lDx~-ar3UocvvRQVw6EkC~GdpGQyyOSKQim-TxAnW7d8F5Kkai9HVx0FIu-5jcQb0UEmatEXAMPLE3ReXySpLSMj0yCd3ZAB4UcBCAqEijkytL6f3fVYNGQI6&Key-Pair-Id=K2JCJMDEHXQW5F

签名url由以下参数组成：
* 文件基本url：
  * ：http://d111111abcdef8.cloudfront.net/images/image.jpg
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
```
{
    "Statement": [
        {
            "Resource": "base URL or stream name",
            "Condition": {
                "DateLessThan": {
                    "AWS:EpochTime": ending date and time in Unix time format and UTC
                }
            }
        }
    ]
}
```

