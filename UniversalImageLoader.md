####UniversalImageLoader图片加载库源码分析总结：

#####1.支持bitmap预处理;

  预处理接口为__BitmapProcessor.java__ ，源码如下：
				
	public interface BitmapProcessor {
	/**
	 * Makes some processing of incoming bitmap.<br />
	 * This method is executing on additional thread (not on UI thread).<br />
	 * <b>Note:</b> If this processor is used as {@linkplain DisplayImageOptions.Builder#preProcessor(BitmapProcessor)
	 * pre-processor} then don't forget {@linkplain Bitmap#recycle() to recycle} incoming bitmap if you return a new
	 * created one.
	 *
	 * @param bitmap Original {@linkplain Bitmap bitmap}
	 * @return Processed {@linkplain Bitmap bitmap}
	 */
	Bitmap process(Bitmap bitmap);
	}
			
代码只有process一个方法，传入一个bitmap参数 ，返回一个bitmap ,具体实现接口时，可以对传入的bitmap进行处理，然后返回一个新的bitmap对象，当然，如果是新的对象的话，记得要调用传入的bitmap的recycle方法；

源码中定义了**preProcessor**和**postProcessor**2个对象，

**preProcessor**：前处理，当bitmap没有在内存缓存中时，会从资源中加载，加载完成后，会调用preProcessor返回处理后的bitmap对象，并保存到内存缓存中，这一过程主要在LoadAndDisplayImageTask类的run方法里面处理；

**postProcessor**：后处理，当bitmap对象从内存中取出来，需要使用时，会调用postProcessor进行预处理，后处理这个过程在ProcessAndDisplayImageTask类和LoadAndDisplayImageTask类中都都有处理。
  
**preProcessor**和**postProcessor**都是通过builder外部传入，源码默认没有实现，一般可能用来处理图片的水印，滤镜等；

#####2.支持自字义discCache的文件名称；

自定义的方法：public ImageLoaderConfiguration.Builder **diskCacheFileNameGenerator**(FileNameGenerator fileNameGenerator)；

	public interface FileNameGenerator {

	/** Generates unique file name for image defined by URI */
	String generate(String imageUri);
	}

通过imageUri生成文件名称，默认实现为获取imageUri的哈希码做为文件名：

	public class HashCodeFileNameGenerator implements FileNameGenerator {
	@Override
	public String generate(String imageUri) {
		return String.valueOf(imageUri.hashCode());
	}
	}


#####3.支持自定义BitmapDisplayer；

 **BitmapDisplayer**用来表示bitmap在UI上的显示方式，比如渐显动画显示，圆角等，内置5种类型的Displayer；
 
 <img src="UIL_bitmap_display.png" width=500px height=350px/>
 
需要自定义显示图片的同学，可以实现**BitmapDisplayer**接口，比如需要圆角显示 ，则创建一个圆角半径为20像素的RoundedBitmapDisplayer对象：

	BitmapDisplayer roundDisplayer = new RoundedBitmapDisplayer(20);
然后在加载具体的图片时，通过：

	DisplayImageOptions options = new DisplayImageOptions.Builder().displayer(roundDisplayer).build();
	
显示时直接调用：

	ImageLoader.getInstance().displayImage(imageUrl, imageView, options);


#####4.支持各种URI：file,http/s,res等;

#####5.支持同步加载和异步加载；

#####6.支持view滑动时暂停加载，在view的onScrollListener里面处理回调，通过pauseLock等控制notify；相关方法pause,resume;

#####7.支持自定义线程池，默认3个线程池，
1个distributorExecutor用做任务分发，3个核心线程，FIFO ; 
1个executor用做非内存缓存加载任务，1个cachedExecutor用做内存缓存加载任务，不限线程数，60S生存周期。