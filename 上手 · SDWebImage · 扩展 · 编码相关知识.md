# 上手 · SDWebImage · 扩展 · 编码相关知识

### 问题

```objective-c
NSString *imageUrlStr = @"https://upload.jianshu.io/admin_banners/web_images/4219/05809448a1c38a25c913d9668eb6fcda272b4ab2.jpg?imageMogr2/auto-orient/strip|imageView2/1/w/1250/h/540";
NSURL *imageUrl = [[NSURL alloc] initWithString:imageUrlStr];
NSData *imageData = [NSData dataWithContentsOfURL:imageUrl];
```

在利用 NSData 请求一张网络图片时候，发现 `imageUrl` 通过 NSURL 创建出来为 nil，导致该资源无法下载。

**利用字符串生成一个 NSURL 对象为什么会为空？**

### 分析

``` objective-c
/* These methods expect their string arguments to contain any percent escape codes that are necessary. It is an error for URLString to be nil.
 */
- (nullable instancetype)initWithString:(NSString *)URLString;
- (nullable instancetype)initWithString:(NSString *)URLString relativeToURL:(nullable NSURL *)baseURL NS_DESIGNATED_INITIALIZER;
+ (nullable instancetype)URLWithString:(NSString *)URLString;
+ (nullable instancetype)URLWithString:(NSString *)URLString relativeToURL:(nullable NSURL *)baseURL;
```

这些 API 声明了入参必须将特殊字符进行必要的转义，否则将会返回 nil。`percent escape codes` 这里的百分号编码其实指的就是 URL 编码。

#### URL 编码解码

> URL 是统一资源定位符，统一资源定位符是对可以从互联网上得到的资源的位置和访问方法的一种简洁的表示，是互联网上标准资源的地址。互联网上的每个文件都有一个唯一的URL，它包含的信息指出文件的位置以及浏览器应该怎么处理它。

对于资源的唯一的 URL，但是有些字符会引起歧义，导致我们解析的跟 URL 的原意出现了偏差，例如：

```objective-c
URL:www.baidu.com?a=1&b=2
//对于这个 URL，解析可能会出现两种情况：
//1.参数a，值为1；参数b，值为2
//2.参数a，值为 1&b=2
```

对于这种情况，我们就必须对 URL 中不安全的字符进行编码，防止出现解析歧义；目前一般都是用 percent encode （百分号编码）来对 URL 进行编码，借助 % 来进行有效的编码，URL 编码的实质就是使用百分号编码来对 URL 编码。

### 解决方案

创建 URL 对象的时候，先将入参 `URLString` 先进行编码，再传入；

```objective-c
// Returns a new string made from the receiver by replacing all characters not in the allowedCharacters set with percent encoded characters. UTF-8 encoding is used to determine the correct percent encoded characters. Entire URL strings cannot be percent-encoded. This method is intended to percent-encode an URL component or subcomponent string, NOT the entire URL string. Any characters in allowedCharacters outside of the 7-bit ASCII range are ignored.
//这个函数会使用百分号对于 非allowedCharacters 进行编码
- (nullable NSString *)stringByAddingPercentEncodingWithAllowedCharacters:(NSCharacterSet *)allowedCharacters API_AVAILABLE(macos(10.9), ios(7.0), watchos(2.0), tvos(9.0));
```

代码如下：

```objective-c
NSString *imageUrlStr = @"https://upload.jianshu.io/admin_banners/web_images/4219/05809448a1c38a25c913d9668eb6fcda272b4ab2.jpg?imageMogr2/auto-orient/strip|imageView2/1/w/1250/h/540";
imageUrlStr = [imageUrlStr stringByAddingPercentEncodingWithAllowedCharacters:[NSCharacterSet URLQueryAllowedCharacterSet]];
NSURL *imageUrl = [[NSURL alloc] initWithString:imageUrlStr];
NSData *imageData = [NSData dataWithContentsOfURL:imageUrl];
```

### 参考文献

[【网络基础】为什么要对url进行encode呢？](http://blog.csdn.net/xude1985/article/details/52268533)

[URL编码与解码原理](http://blog.csdn.net/zmx729618/article/details/51381655)

[Unicode 和UTF-8 有何区别？ - 知乎](https://www.zhihu.com/question/23374078)









