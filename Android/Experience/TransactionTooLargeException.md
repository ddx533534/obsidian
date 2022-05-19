 [How to send an object from one Android Activity to another using Intents?](https://stackoverflow.com/questions/2139134/how-to-send-an-object-from-one-android-activity-to-another-using-intents)
## 一、前言

作为 Android 开发，日常 Coding 时，最频繁的操作应该就是操作 App 内的一系列 Activity。而在 Activity 间传递数据，就需要借助 Intent。不少资料中写到，Intent 在 Activity 间传递基础类型数据或者可序列化的对象数据。但是 Intent 对数据大小是有限制的，当超过这个限制后，就会触发 TransactionTooLargeException 异常。


## 二、为什么会出现异常？

### 2.1 异常原因

Intent 传递大数据，会出现 TransactionTooLargeException 的场景，这本身也是一道面试题，经常在面试中被问到。其实这个问题，如果遇到了，查查文档就知道了。
在 [TransactionTooLargeException](https://developer.android.com/reference/android/os/TransactionTooLargeException.html) 的文档中，其实已经将触发原因详细说明了。

简单来说，Intent 传输数据的机制中，用到了 Binder。Intent 中的数据，会作为 Parcel 被存储在 Binder 的事务缓冲区(Binder transaction buffer)中的对象进行传输。而这个 **Binder 事务缓冲区具有一个有限的固定大小，当前为 1MB**。你可别以为传递 1MB 以下的数据就安全了，这里的 1MB 空间并不是当前操作独享的，而是由当前进程所共享。也就是说 Intent 在 Activity 间传输数据，本身也不适合传递太大的数据。

### 2.2 Bundle 的锅？

这里再补充一些细节，Intent 使用 Bundle 存储数据，到底是值传递(深拷贝)还是引用传递？

Intent 传输的数据，都存放在一个 Bundle 类型的对象 mExtras 中，Bundle 要求所有存储的数据，都是可被序列化的。在 Android 中，序列化数据需要实现 Serializable 或者 Parcelable。对于基础数据类型的包装类，本身就是实现了 Serializable，而我们自定义的对象，按需实现这两个序列化接口的其中一个即可。

**那是不是只要通过 Bundle 传递数据，就会面临序列化的问题？**

并不是，Activity 之间传递数据，首先要考虑跨进程的问题，而 Android 中又是通过 Binder 机制来解决跨进程通信的问题。涉及到跨进程，对于复杂数据就要涉及到序列化和反序列化的过程，这就注定是一次值传递(深拷贝)的过程。

这个问题用反证法也可以解释，如果是引用传递，那传递过去的只是对象的引用，指向了对象的存储地址，就只相当于一个 Int 的大小，也就根本不会出现 TransactionTooLargeException 异常。

**传输数据序列化和 Bundle 没有关系，只与 Binder 的跨进程通信有关。**

为什么要强调这个呢？在 Android 中，使用 Bundle 传输数据，并非 Intent 独有的。例如使用弹窗时，DialogFragment 中也可以通过 `setArguments(Bundle)` 传递一个 Bundle 对象给对话框。Fragment 本身是不涉及跨进程的，这里虽然使用了 Bundle 传输数据，但是并没有通过 Binder，也就是不存在序列化和反序列化。和 Fragment 数据传递相关的 Bundle，其实传递的是原对象的引用。

有兴趣可以做个试验，弹出 Dialog 时传递一个对象，Dialog 中修改数据后，在 Activity 中检查数据是否被修改了。

## 三、如何解决这个异常？
1. 可以从数据源上来考虑。
   例如 Bitmap，本身就已经实现了 Parcelable 是可以支持序列化的。用 Intent 传输，稍微大一点的图一定会出现 TransactionTooLargeException。当然真是业务场景，肯定不存在传递 Bitmap 的情况。那就先看看这个图片的数据源。Drawable？本地文件？线上图片？无论数据源在哪里，我们只需要传递一个 drawable_id、路径、URL，就可以还原这张图片，无需将这个 Bitmap 对象传递过去。大数据总有数据源，从数据源还原数据，对我们而言只是调用一个方法而已。
2. 将需要传递的数据写在临时文件或者数据库中，再跳转到另外一个组件的时候再去读取这些数据信息，这种处理方式会由于读写文件较为耗时导致程序运行效率较低
3. 将需要传递的数据信息封装在一个静态的类中。
   例如自定义一个静态类DataHolder，并且设置setData和getData方法，而且考虑到极端的情况，有可能传递的对象的内存是极其大的，所以为了不造成内存泄漏，我们将要传递的对象构造成一个弱引用保存到该静态类。下面是DataHolder的代码：
   
``` java
public class DataHolder {
 
    private Map dataList = new HashMap<>();
 
    private static volatile DataHolder instance;
 
    public static DataHolder getInstance() {
 
       if(instance==null) {
 
            synchronized(DataHolder.class) {
 
                if(instance==null) {
 
                     instance = new DataHolder();
                }
           }
       }
 
       return instance;
 
    }
 
    public void setData(String key, Object o) {
 
        WeakReference value =new WeakReference<>(o);
 
        dataList.put(key, value);
 
    }
 
    public Object getData(String key) {
 
          WeakReference reference =dataList.get(key);
 
          if(reference !=null) {
              Object o = reference.get();
              return o;
          }
          return null;
      }
 
}
```


