# 上手 · SDWebImage · 扩展 · Nullability



### Nullability and Objective-C 官方翻译文档

原文链接： [Nullability and Objective-C](https://developer.apple.com/swift/blog/?id=25) 

更新：在 Xcode 7 中使用新的 _Nullable 语法。

Swift 可以完美的跟 Objective-C 代码进行混编。但是，在 Swift 中可选引用与非可选引用有着很大的区别（optional & non-optional）。例如：`NSView` 与 `NSView？`，但是在 Objective-C 中则将这两种类型都表示成 `NSView *`。由于 Swift 编译器无法确定 `NSView *` 到底是可选还是非可选，所以 Swift 会默认的认为这个 `NSView` 是可选的。

在之前的 Xcode 版本中，一些 Apple 框架已经经过特殊的处理，以便它的 API 可以很好的转成 Swift 中可选或不可选特性。在 Xcode 6.3 开始，也提供了新的 Objective-C 语法来支持你的代码转成合适 Swift代码。(nullability annotations)。

#### The Core: _Nullable and _Nonnull

为了支持这个转换的特性，提供了两个新的修饰符：_Nullable 和 _Nonnull。\_Nullable 代表对象可能是 NULL 和 nil，而 \_Nonnull 代表的意思则相反。在编译的时候，如果发现类型不匹配就会给出警告。

```objective-c
@interface AAPLList : NSObject <NSCoding, NSCopying>
// ...
- (AAPLListItem * _Nullable)itemWithName:(NSString * _Nonnull)name;
@property (copy, readonly) NSArray * _Nonnull allItems;
// ...
@end

// --------------

//这里参数修饰的是 _Nonnull 非空，但是我们调用的时候，传了空，这里就会编译警告
[self.list itemWithName:nil]; // warning!
```

虽然 _Nullable 和 _Nonnull 是用来修饰指针类型，但是也可以用来修饰 C const 关键字。但是，在一般情况下还有一种更好的写法：在方法申明中，如果类型是一个对象或者 block 指针，你可以直接在类型前面使用非下划线的写法，nullable 跟 nonnull。

```objective-c
- (nullable AAPLListItem *)itemWithName:(nonnull NSString *)name;
- (NSInteger)indexOfItem:(nonnull AAPLListItem *)item;
```

在属性申明中，你一样可以使用非下划线来修饰这个属性：

```objective-c
@property (copy, nullable) NSString *name;
@property (copy, readonly, nonnull) NSArray *allItems;
```

非下划线的形式会比下划线形式更好，但是你依然需要去修饰每个对象，为了使开发更加轻松和让 API 更加清晰，你可以使用 **Audited Regions**。

### Audited Regions

为了简化这种修饰，你可以在 Objective-C 一些区域把属性标记成 nonnull 。在这个区域里面，任何简单的指针类型都会修饰成 nonnull。

```objective-c
//这里会默认将对象修饰成 nonnull，如果有些对象你想设置成可以为空，你可以直接设置
NS_ASSUME_NONNULL_BEGIN
@interface AAPLList : NSObject <NSCoding, NSCopying>
// ...
- (nullable AAPLListItem *)itemWithName:(NSString *)name;
- (NSInteger)indexOfItem:(AAPLListItem *)item;

@property (copy, nullable) NSString *name;
@property (copy, readonly) NSArray *allItems;
// ...
@end
NS_ASSUME_NONNULL_END

// --------------

self.list.name = nil;   // okay

AAPLListItem *matchingItem = [self.list itemWithName:nil];  // warning!
```

为了安全起见，这个规则有些例外：

* `typedef` 类型通常是不固定，不能很好的判断是空还是非空，所以在这个宏修饰的区域内，也不会被假定用 nonnull 来修饰。
* 对于一些复杂的指针类型，如 `id *` 必须明确标注出来，例如：一个不能为空的指针指向一个可以为空的对象，请使用 `_Nullable id * _Nonnull`。
* 对于 `NSError **` 用来返回错误，默认是用 nullable 来修饰的。

#### Compatibility

在 Xcode 6.3 上面初始使用这个特性，一开始是用 \__nullable 和 __nonnull，但是可能会与第三方库存在冲突，在 Xcode 7 中就修改成了 _Nullable 和 _Nonnull。

### 扩展





### 参阅

[**Swift学习基础篇**](https://github.com/maocomen/iOS-note/blob/master/%E8%8F%9C%F0%9F%90%94%E5%AD%A6%E4%B9%A0Swift/Swift%E5%AD%A6%E4%B9%A0%E5%9F%BA%E7%A1%80%E7%AF%87.md) 

