# Bitmap 的加载和 Cache
## Bitmap 的高效加载
1. Bitmap 在 Android 中指的是一张图片。
2. BitmapFactory 类提供了四类方法：decodeFile、decodeResource、decodeStream 和 decodeByteArray，分别用于支持从文件系统、资源、输入流以及字节数组中加载一个 Bitmap 对象，其中 decodeFile 和 decodeResource 又间接调用了 decodeStream 方法。
3. 高效地加载 Bitmap 的核心思想就是采用 BitmapFactory.Options 来加载所需尺寸的图片。主要用到了 inSampleSize 参数，即采样率。inSampleSize 的取值应该总是为2的指数，如果外界传递给系统的 inSampleSize 不为2的指数，那么系统会向下取整并选择一个最接近2的指数来代替。
4. 获取采样率的流程：
    - 将 BitmapFactory.Options 的 inJustDecodeBounds 参数设为 true并加载图片；
    - 从 BitmapFactory.Options 中取出图片的原始宽高信息，它们对应于 outWidth 和 outHeight 参数；
    - 根据采样率的规则并结合目标 View 的所需大小计算出采样率 inSampleSize；
    - 将 BitmapFactory.Options 的 inJustDecodeBounds 参数设为 false，然后重新加载图片。

5. 代码实现：
    ````java
        public static Bitmap decodeSampleBitmapFromResource(Resource res,int resId,int reqWidth,int reqHeight){
            //First decode with inJustDecodeBounds = true to check dimensions

            final BitmapFactory.Options options = new BitmapFactory.Options();
            options.inJustDecodeBounds = true;
            BitmapFactory.decodeResource(res,redId,options);

            // Calculate inSampleSize
            options.inSampleSize = calculateInSampleSize(options,reqWidth,reqHeight);
            return BitmapFactory.decodeResource(res,resId,options);
        }


        public  static int calculateInSampleSize(BitmapFactory.Options options,int reqWidth,int reqHeight){
            //Raw height and width of image
            final int height = options.outHeight;
            final int width = options.outWidth;
            int inSampleSize = 1;

            if(height > reqHeigth || width > reqWidth){
                final int halfHeight = height / 2;
                final int halfWidth = width / 2;
                //Calculate the largest inSampleSize value that is a power of 2 and keeps both height and width larger than the requested height and width.

                while ((halfHeight / inSampleSize) >= reqHeight && (halfWidth / inSampleSize) >= reqWidth){
                    inSampleSize *= 2;
                }
            }
            return inSampleSize;
        }
    ````
加载并显示图片
````java
    mImageView.setImageBitmap(
        decodeSampleBitmapFromResource(getResource(),R.id.myimage,100,100));
````

## Android 中的缓存策略
1. 一般来说，缓存策略主要包含缓存的添加、获取和删除这三类操作。
2. 目前常用的一种算法是 LRU，是近期最少使用算法，核心思想是当缓存满时，会优先淘汰那些近期最少使用的缓存对象。采用 LRU 算法的缓存有两种：LruCache(用于实现内存缓存) 和 DiskLruCache(充当存储设备缓存)，
3. LruCache
    LruCache 是一个泛型类，内部采用一个 LinkedHashMap 以强引用的方式存储外界的缓存对象，提供了 get 和 put 方法来完成缓存的获取和添加操作，当缓存满时，LruCache 会移除较早使用的缓存对象，然后再添加新的缓存对象。

    - 强引用：直接的对象引用；
    - 软引用：当一个对象只有软引用存在时，系统内存不足时此对象会被 gc 回收；
    - 弱引用: 当一个对象只有弱引用存在时，此对象会随时被 gc 回收。

    LruCache 是线程安全的。
    LruCache 的典型的初始化过程：
    ````java
        int maxMemory = (int) (Runtime.getRuntime().maxMemory() / 1024);
        int cacheSize = maxMemory / 8;
        mMenoryCache = new LruCache<String, Bitmap>(cacheSize){
            @Override
            // sizeOf() 方法的作用是计算缓存对象的大小
            protected int sizeOf(String key, Bitmap bitmap){
                return bitmap.getRowBytes() * bitmap.getHeight() / 1024;
            }
        };
    ````

    从 LruCache 中获取一个缓存对象
    ````java
        mMenoryCache.put(key,bitmap);
    ````

4. DiskLruCache
    DiskLruCache 用于实现存储设备缓存，即磁盘缓存，通过将缓存对象写入文件系统从而实现缓存的效果。

    DiskLruCache 的创建
    DiskLruCache 并不能通过构造方法来创建，提供了 open 方法用于创建自身。
    ````java
        public static DiskLruCache open(File directory, int appVersion, int valueCount, long maxSize)
    ````
    open 方法有四个参数：

    - 第一个参数表示磁盘缓存在文件系统中的存储路径；
    - 第二个参数表示应用的版本号，一般设为1即可；
    - 第三个参数表示单节点所对应的数据的个数，一般设为1即可。
    - 第四个参数表示缓存的总大小，当缓存大小超出这个设定值之后， DiskLruCache 会清楚一些缓存从而保证大小不大于这个设定值。

    DiskLruCache 的缓存添加
    DiskLruCache 的缓存添加操作是通过 Editor 完成的，Editor 表示一个缓存对象的编辑对象。

    DiskLruCache 的缓存查找
    缓存查找过程需要将 url 转换为 key，然后通过 DiskLruCache 的 get 方法得到一个 Snapshot 对象，接着再通过 Snapshot 对象即可得到缓存的文件输入流，有了文件输出流，自然就可以得到 Bitmap 对象。

5. ImageLoader 的实现
    一般来说，ImageLoader 应该具备如下功能：
    - 图片的同步加载；
    - 图片的异步加载；
    - 图片的压缩；
    - 内存缓存；
    - 磁盘缓存；
    - 网络垃圾。
    



                                                                     2017.8.29
                                                                        W.Z.H