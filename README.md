# 基于Cloudfront的时间戳防盗链方案

## 使用标准策略创建签名url
签名url由以下参数组成：
* 文件url：
  * 比如：http://d111111abcdef8.cloudfront.net/images/image.jpg
* ？
  * 问号参数位于文件url和签名token之间
* Expires=Unix时间格式（以秒为单位）
  * 您希望的URL不再允许访问文件的日期和时间，也就是该条url的过期时间
* &Signature=经过哈希处理和签名的策略声明版本
  * JSON 策略声明经过哈希处理、签署和 Base64 编码的版本
* &Key-Pair-Id=您在cloudfront上配置的公有密钥ID
