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
