# Image I/O Programming Guide

## Translation

Introdution

Image I/O 接口支持应用程序读写大部分图片格式。Image I/O 最开始是 Core Graphics 框架的一部分，Image I/O 框架独立出来，可以使得开发人员不依赖 Core Graphics（Quartz 2D）框架来使用它。Image I/O 非常高效，提供了明确访问图片数据的接口，可以轻松的访问元数据并提供颜色管理。

Image I/O 接口在 OS X v10.4 版本和 iOS 4 版本以后都可以使用。

### Who Should Read This Document?

本文档适用于在应用程序读取或者写入图片数据的开发人员。目前任何使用图片导入器或其他图片处理库的开发人员都应该阅读本文档，从而了解 Image I/O 框架的工作原理。

### Organization of This Document

本文档有以下章节：

* **Basics of Using Image I/O** 章节描述了支持的图片格式，如何在 Xcode 工程中使用这个框架
* **Creating and Using Image Sources** 章节展示如何创建图片源，创建图片，并提取用于在用户界面展示的属性
* **Working with Image Destinations** 章节提供创建一个图片 Destination，并为其设置属性和添加图片的信息

### See Also

[Image I/O Reference Collection](https://developer.apple.com/documentation/imageio) 详细描述了 Image I/O 框架中函数、数据类型和常量。

# Basics of Using Image I/O

Image I/O 框架提供了不透明的数据类型，用于从 CGImageSourceRef 源读取图片数据、将图片数据写入 CGImageDestination 目标对象中。它支持多种图片格式，包括标准的网页格式，<u>高动态范围图片</u>，原始相机图片格式。Image I/O 具有许多其他特性，例如：

* 图片编码&解码是 Mac 平台上面最快的
* 可以逐步加载图片（渐进式解码）
* 支持图片元数据
* 有效的缓存

你可以通过以下的方式创建图片源或者图片目标对象：

* URLs。在 Image I/O 中，URL 用 Core Foundation 数据类型 CFURLRef 表示
* Core Foundation 中的 CFDataRef 和 CFMutableDataRef 对象
* Quartz 中的数据消费者 CGDataConsumerRef，数据提供者 CGDataProviderRef 对象

### Using the Image I/O Framework in Your Application

Image I/O 存在于 OS X 中的 Application Services framework 中，iOS 中的 Image I/O 框架中。将框架导入到你的应用程序后，你可以通过下面语句来导入头文件：

```objective-c
#import <ImageIO/ImageIO.h>
```

### Supported Image Formats

Image I/O 框架支持大多数常见的图片格式，例如JPEG、JPEG2000、RAW、TIFF、BMP 和 PNG。每个平台所能支持的格式有所区别，关于 Image I/O 最新能支持的格式，你可以调用以下这些方法来查看：

```objective-c
CGImageSourceCopyTypeIdentifiers：返回 Image I/O 框架图片源能支持的统一类型标识符（UTIs）数组；
CGImageDestinationCopyTypeIdentifiers：返回 Image I/O 框架输出目标能支持的统一类型标识符(UTIs)数组
```

你可以通过调用 CFShow 函数在 Xcode 控制台将数组打印出来，代码如下；这些函数返回的数组中的字符串格式采用 `com.apple.pict`, `public.jpeg`, `public.tiff` 等形式。表 1-1 列举了常见图片文件格式的 UTI。OS X 和 iOS 为常见的图片文件格式定义了常量，这些常量被定义在 UTCoreTypes.h 这个头文件中。当需要指定图片类型的时候，你可以使用这些常量。

```objective-c
CFArrayRef mySourceTypes = CGImageSourceCopyTypeIdentifiers();
CFShow(mySourceTypes);
CFArrayRef myDestinationTypes = CGImageDestinationCopyTypeIdentifiers();
CFShow(myDestinationTypes);
```

列表 1-1：

| Uniform type identifier      | Image content type constant |
| ---------------------------- | --------------------------- |
| public.image                 | kUTTypeImage                |
| public.png                   | kUTTypePNG                  |
| public.jpeg                  | kUTTypeJPEG                 |
| public.jpeg-2000 (OS X only) | kUTTypeJPEG2000             |
| public.tiff                  | kUTTypeTIFF                 |
| com.apple.pict (OS X only)   | kUTTypePICT                 |
| com.compuserve.gif           | kUTTypeGIF                  |

# Creating and Using Image Sources

一个图片源会提取数据访问任务，并且不需要通过内存缓存区来管理数据。一个图片源可以包含多张图片、多张缩略图、每个图片的属性以及图片文件。当你使用图片数据并且你的应用程序在 OS X v10.4 或更多系统中运行时，图片源是将图片数据移动到应用程序中的首选方式。创建一个 CGImageSource 对象后，你可以通过 CGImageSource 提供的函数获取到图片、缩略图、图片属性和其他图片信息。

### Creating an Image from an Image Source

Image I/O 框架最常使用的就是利用图片源创建一张图片，如下面代码所示。下面代码展示了如何利用一个路径来创建图片源，然后提取图片。当你创建一个图片源时，你可以知道这个图片源文件的格式。

```objective-c
//利用一个图片路径来创建一张 CGImage
CGImageRef MyCreateCGImageFromFile (NSString* path)
{
    // Get the URL for the pathname passed to the function.
    NSURL *url = [NSURL fileURLWithPath:path];
    CGImageRef        myImage = NULL;
    CGImageSourceRef  myImageSource;
    CFDictionaryRef   myOptions = NULL;
    CFStringRef       myKeys[2];
    CFTypeRef         myValues[2];
 
    // Set up options if you want them. The options here are for
    // caching the image in a decoded form and for using floating-point
    // values if the image format supports them.
    myKeys[0] = kCGImageSourceShouldCache;
    myValues[0] = (CFTypeRef)kCFBooleanTrue;
    myKeys[1] = kCGImageSourceShouldAllowFloat;
    myValues[1] = (CFTypeRef)kCFBooleanTrue;
    // Create the dictionary
    myOptions = CFDictionaryCreate(NULL, (const void **) myKeys,
                   (const void **) myValues, 2,
                   &kCFTypeDictionaryKeyCallBacks,
                   & kCFTypeDictionaryValueCallBacks);
    // Create an image source from the URL.
    myImageSource = CGImageSourceCreateWithURL((CFURLRef)url, myOptions);
    CFRelease(myOptions);
    // Make sure the image source exists before continuing
    if (myImageSource == NULL){
        fprintf(stderr, "Image source is NULL.");
        return  NULL;
    }
    // Create an image from the first item in the image source.
    myImage = CGImageSourceCreateImageAtIndex(myImageSource,
                                           0,
                                           NULL);
 
    CFRelease(myImageSource);
    // Make sure the image exists before continuing
    if (myImage == NULL){
         fprintf(stderr, "Image not created from image source.");
         return NULL;
    }
 
    return myImage;
}

```

### Incrementally Loading an Image

如果你有一张非常大的图片，或者你正在通过网络加载图片数据，你可能想创建一个增量图片源，以便提前绘制图片数据，然后再慢慢累积绘制。你需要执行以下步骤来从 CFData 对象渐进式加载图片：

1. 创建用于累积图片数据的 CFData 对象
2. 通过调用 CGImageSourceCreateIncremental 函数来创建增量图片源
3. 将图片数据加入到 CFData 对象中
4. 调用 CGImageSourceUpdataData 函数，传递 CFData 对象和一个 BOOL 值，这个 BOOL 值表示数据参数是包含整个图片还是包含部分图片数据。但是，在任何情况下，这个 CFData 对象都应该是包含累积的图片文件数据
5. 如果已经累积完所有图片数据，调用 CGImageSourceCreateImageAtIndex 函数创建图片，绘制图片，然后释放它
6. 通过调用 CGImageSourceGetStatusAtIndex 函数来检查数据是否完整。如果数据完整，此函数会返回 kCGImageStatusComplete，如果图片数据不完整，请重复步骤3和步骤4直到完成
7. 释放增量图片源

### Displaying Image Properties

数码图片包含了图片的大量信息，例如图片尺寸、分辨率、方向、颜色信息、光圈、测光模式、焦距、创建日期、关键字、标题等信息。这些信息对图片处理和编辑非常有用。尽管 CGImageSourceCopyPropertiesAtIndex 函数会返回图片源中图片相关的属性字典，你仍然需要编写遍历该字典的代码来检索并显示这些信息。

## Working with Image Destinations











