# 上手 · SDWebImage · 扩展 · NSCache

### 前言

SDWebImage 对于图片数据的处理使用了常规的三级缓存，即内存缓存、磁盘缓存、网络缓存；对于内存缓存，SDWebImage 使用的是 NSCache 来缓存数据。这里我们来简单聊聊 NSCache 这个东西。

### NSCache 相关

咱们在这里就不讨论如何使用 NSCache ，NSCache 在调用方面还是比较简单的，想了解 NSCache 是什么及其基本用法的可以看看文末的参考链接，看完基本可以入门 NSCache 。

在这里我们聊聊在做内存缓存的时候为什么选择 NSCache 以及 NSCache 一些特性：

#### 为什么选择使用 NSCache 来做内存缓存

在 App 运行过程中，有一些频繁使用的对象（创建这些对象比较昂贵），我们可能会去考虑全局共享这些对象，这样子就不用频繁的去创建这些对象，可以为用户更快的展示数据；一般会创建单例来全局共享，然后用这个单例的属性来保存我们所需要缓存的对象；

一般缓存就是利用键值对这种数据格式来缓存所需的对象，在 Objective-C 中，我们最常用的就是利用 NSDictionary & NSMutableDictionary 来保存键值对，这种做法在一定程度上可以的；但是我们可以思考下：缓存的对象越来越多，遇到内存警告的时候如何处理呢？是全部清除呢？还是清除一部分呢？

很庆幸，NSCache 已经帮我们处理好了这些场景，不用我们再去关心如何实现缓存淘汰这些算法；NSCache 会自动删减缓存，而且还会优先删除 “最久未经使用” 的对象。

而且 NSCache 里面方法都是线程安全的，不用我们再去考虑多线程调度时如何保证线程安全问题；

相比 NSMutableDictionary 的 `setObject:forKey:` 方法，不要求 Key 实现 NSCoping 协议，这可以让我们很灵活设置键值；

这里列举下使用 NSCache 的优势：

* NSCache 具有自动删减缓存，不用我们自己再去实现复杂的缓存淘汰算法
* NSCache 是线程安全的，不用我们再去考虑多线程安全问题
* NSCache 对于 Key-Value 中的 Key 没有要求，不像 NSDictionary 中要求 Key 需要实现 NSCoping 协议

#### 使用 NSCache 中有什么需要注意的点

- `-(void)setObject:(ObjectType)obj forKey:(KeyType)key;  `  Object 不能为空
- `-(void)setObject:(ObjectType)obj forKey:(KeyType)key cost:(NSUInteger)g;`  不能根据 cost 来完成特殊的行为，因为缓存的移除顺序并不是根据这个 cost 来移除的
- NSCache 会强引用对象

#### SDWebImage 中使用 NSCache 缓存中一个有趣的设计

前面我们说过 SDWebImage 做图片内存缓存是用 NSCache 来实现的，但是 SDWebImage 在利用 NSCache 进行缓存的的同时，用了一个 NSMapTable 保存了缓存到 NSCache 中的键值对；

> Create a subclass of NSCache using a weak cache. Only remove the cache when memory warning and sync back the alive instance from weak cache into cache.
>
> 这个是 SDWebImage 作者的提交日志

这个设计是对以下这种场景进行了优化，不得不说很细节：

1. 首先 NSCache 会强引用缓存对象，然后我们的 NSCache 监听了内存警告的通知，当发生内存警告的时候，NSCache 会 RemoveAllObjects，移除所有缓存对象
2. 前面说过，NSCache 会强引用缓存对象，在NSCache 调用了 RemoveAllObjects 方法之后，就会将对象的引用计数 -1
3. 调用完 RemoveAllObjects 方法之后，这里就会有以下两种情况：
   * 该对象已经没有被其他对象所强引用了，此时，这个对象会被完全的销毁
   * 该对象还被其他对象所强引用，在 NSCache RemoveAllObjects 将引用计数 -1 之后，它的引用计数还是会大于1，此时，这个对象并不会真正的从内存中移除
4. 在面对第二种情况的时候，NSCache 虽然移除了缓存对象，但是这个对象依然被其他对象强引用了，所以它并不会销毁；换句话说，就是下次我们重用该对象，并不需要重新创建，我们应该继续使用该对象
5. 所以面对这种情况，SDWebImage 会重新将该对象放回缓存中，就没有必要再去磁盘中查找照片

##### SDWebImage 为这个设计做了什么

1. 为什么使用 NSMapTable 来记录键值对

   相对于 NSMutableDictionary 来说，NSMapTable 可以配置对 Key & Value 的策略，使得 NSMapTable 不会强引用对象，这里就不会影响对象本身的生命周期；并且这里不会要求对 Key 需要实现 NSCoping 协议，完美的跟 NSCache 配合

2. NSCache 方法是线程安全的，NSMapTable 是线程不安全的，这里使用信号量对 NSMapTable 方法进行了加锁，保证了多线程调用方法时不会出现非原子性操作

##### 对于 NSCache 如何可以一次性获取所有 allValues

这个问题我是在 YYCache 中的 issues 看到的，觉得挺有意思的：[用setObjeck:forKey: 多次存一个类型的对象，怎么可以一次获取所有的allValues??](https://github.com/ibireme/YYCache/issues/44)

恰巧 SDWebImage 这个设计，可以实现这个功能，因为 NSMapTable 已经记录好了缓存到 Cache 中的键值对，可以很方便的获取所有的 Keys & Values

```objective-c
- (NSEnumerator<KeyType> *)keyEnumerator;

- (nullable NSEnumerator<ObjectType> *)objectEnumerator;
```

然后这里需要注意的是：如果我们为外界提供这种获取所有键值对的接口，要注意到 NSArray、NSMutableDictionary 会强引用对象，影响该对象的生命周期，这个是我们所需要避免的。在这里，我们可以实现一个不强引用对象的 NSArray 和 NSDictionary 以便达到需求

```objective-c
//获取缓存中的所有 Keys
- (NSArray <KeyType>*)fetchAllCacheKeys;

//获取缓存中的所有 Values
- (NSArray <ObjectType>*)fetchAllCacheValues;

//获取缓存中的所有 Key-Values
- (NSDictionary <KeyType, ObjectType>*)fetchAllKeyValues;
```

###  扩展

#### 如何实现一个不拷贝 “Key”  & 不强引用 “Object” 的 NSDictionary

#### 如何实现一个不强引用 “Object” 的 NSArray

上面两个需求，其实是一样的，都是改变 NSDictionary & NSArray 对于 Key 或者 Value 的引用策略；在这里我们可以使用 CoreFoundation 来创建 CF 对象，然后桥接给 Foundation 对象，因为在 CoreFoundation 可以更加自由的对 CF 对象进行设置，下面是对 CFMutableArrayRef 桥接成 NSMutableArray 的一个例子，其他的都差不多是一个原理。

```objective-c
+ (NSMutableArray *)creatWeakMutableArray;

+ (NSMutableArray *)creatWeakMutableArray {
    CFMutableArrayRef cfMutableArray = CFArrayCreateMutable(NULL, 0, &arrayCallbacks);
    NSMutableArray *mutableArray = (__bridge_transfer NSMutableArray *) cfMutableArray;
    return mutableArray;
}

const void * PXYArrayRetainCallBack(CFAllocatorRef allocator, const void *value) {
    return value;
}

void PXYAArrayReleaseCallBack(CFAllocatorRef allocator, const void *value) {
    
}

CFArrayCallBacks arrayCallbacks = {
    0,
    PXYArrayRetainCallBack,
    PXYAArrayReleaseCallBack,
    NULL,
    CFEqual
};
```

### 参考文献

[NSCache](https://www.jianshu.com/p/5e69e211b161)

[NSCache - NSHipster](http://nshipster.cn/nscache/)

[Foundation: NSCache | 南峰子的技术博客](http://southpeak.github.io/2015/02/11/cocoa-foundation-nscache/)

[聊聊NSCache - iOS - 掘金](https://juejin.im/entry/5948bd53fe88c2006a93744e)



### 