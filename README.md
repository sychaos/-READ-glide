Glide
=====

### 解析
  如果你是在子线程调用with方法,或者传入的Context是Application的话,请求是跟你的Application的生命周期同步，如果有指定的activity或者fragment会创建一个无界面的Fragment
  1.Util.assertMainThread();这里会检查是否主线程,不是的话会抛出异常,所以into方法必须在主线程中调用.
  2.当你没有调用transform方法,并且你的ImageView设置了ScaleType,那么他会根据你的设置,对图片做处理(具体处理可以查看DrawableRequestBuilder的applyCenterCrop或者applyFitCenter方法,我们自己自定义BitmapTransformation也可以参考这里的处理).
  3.这里可以看到控件封装成的Target能够获取自身绑定的请求,当发现之前的请求还在的时候,会把旧的请求清除掉,绑定新的请求,这也就是为什么控件复用时不会出现图片错位的问题(这点跟我在Picasso源码中看到的处理方式很相像)
  #### load
  1.首先会尝试从cache里面取,这里cache就是Glide的构造函数里面的MemoryCache(是一个LruResourceCache),如果取到了,就从cache里面删掉,然后加入activeResources中
  2.如果cache里面没取到,就会从activeResources中取,activeResources是一个以弱引用为值的map,他是用于存储使用中的资源.之所以在内存缓存的基础上又多了这层缓存,是为了当内存不足而清除cache中的资源中,不会影响使用中的资源.


[![Maven Central](https://maven-badges.herokuapp.com/maven-central/com.github.bumptech.glide/glide/badge.svg)](https://maven-badges.herokuapp.com/maven-central/com.github.bumptech.glide/glide) [![Build Status](https://travis-ci.org/bumptech/glide.svg?branch=master)](https://travis-ci.org/bumptech/glide)
| [View Glide's documentation][20] | [简体中文文档][22] | [Report an issue with Glide][5]

Glide is a fast and efficient open source media management and image loading framework for Android that wraps media
decoding, memory and disk caching, and resource pooling into a simple and easy to use interface.

![](static/glide_logo.png)

Glide supports fetching, decoding, and displaying video stills, images, and animated GIFs. Glide includes a flexible API
that allows developers to plug in to almost any network stack. By default Glide uses a custom `HttpUrlConnection` based
stack, but also includes utility libraries plug in to Google's Volley project or Square's OkHttp library instead.

Glide's primary focus is on making scrolling any kind of a list of images as smooth and fast as possible, but Glide is
also effective for almost any case where you need to fetch, resize, and display a remote image.

Download
--------
You can download a jar from GitHub's [releases page][1].

Or use Gradle:

```gradle
repositories {
  mavenCentral()
  google()
}

dependencies {
  implementation 'com.github.bumptech.glide:glide:4.4.0'
  annotationProcessor 'com.github.bumptech.glide:compiler:4.4.0'
}
```

Or Maven:

```xml
<dependency>
  <groupId>com.github.bumptech.glide</groupId>
  <artifactId>glide</artifactId>
  <version>4.4.0</version>
</dependency>
<dependency>
  <groupId>com.google.android</groupId>
  <artifactId>support-v4</artifactId>
  <version>r7</version>
</dependency>
<dependency>
  <groupId>com.github.bumptech.glide</groupId>
  <artifactId>compiler</artifactId>
  <version>4.4.0</version>
  <optional>true</optional>
</dependency>
```

For info on using the bleeding edge, see the [Snapshots][17] docs page.

ProGuard
--------
Depending on your ProGuard (DexGuard) config and usage, you may need to include the following lines in your proguard.cfg (see the [Download and Setup docs page][25] for more details):

```pro
-keep public class * implements com.bumptech.glide.module.GlideModule
-keep public class * extends com.bumptech.glide.module.AppGlideModule
-keep public enum com.bumptech.glide.load.resource.bitmap.ImageHeaderParser$** {
  **[] $VALUES;
  public *;
}

# for DexGuard only
-keepresourcexmlelements manifest/application/meta-data@value=GlideModule
```

How do I use Glide?
-------------------
Check out the [documentation][20] for pages on a variety of topics, and see the [javadocs][3].

For Glide v3, see the [wiki][2].

Simple use cases with Glide's [generated API][21] will look something like this:

```java
// For a simple view:
@Override public void onCreate(Bundle savedInstanceState) {
  ...
  ImageView imageView = (ImageView) findViewById(R.id.my_image_view);

  GlideApp.with(this).load("http://goo.gl/gEgYUd").into(imageView);
}

// For a simple image list:
@Override public View getView(int position, View recycled, ViewGroup container) {
  final ImageView myImageView;
  if (recycled == null) {
    myImageView = (ImageView) inflater.inflate(R.layout.my_image_view, container, false);
  } else {
    myImageView = (ImageView) recycled;
  }

  String url = myUrls.get(position);

  GlideApp
    .with(myFragment)
    .load(url)
    .centerCrop()
    .placeholder(R.drawable.loading_spinner)
    .into(myImageView);

  return myImageView;
}
```

Status
------
Version 4 is now released and stable. Updates are currently released at least monthly with new features and bug fixes.

Comments/bugs/questions/pull requests are always welcome! Please read [CONTRIBUTING.md][5] on how to report issues.

Compatibility
-------------

 * **Minimum Android SDK**: Glide v4 requires a minimum API level of 14.
 * **Compile Android SDK**: Glide v4 requires you to compile against API 26 or later.

 If you need to support older versions of Android, consider staying on [Glide v3][14], which works on API 10, but is not actively maintained.

 * **OkHttp 3.x**: There is an optional dependency available called `okhttp3-integration`, see the [docs page][23].
 * **Volley**: There is an optional dependency available called `volley-integration`, see the [docs page][24].
 * **Round Pictures**: `CircleImageView`/`CircularImageView`/`RoundedImageView` are known to have [issues][18] with `TransitionDrawable` (`.crossFade()` with `.thumbnail()` or `.placeholder()`) and animated GIFs, use a [`BitmapTransformation`][19] (`.circleCrop()` will be available in v4) or `.dontAnimate()` to fix the issue.
 * **Huge Images** (maps, comic strips): Glide can load huge images by downsampling them, but does not support zooming and panning `ImageView`s as they require special resource optimizations (such as tiling) to work without `OutOfMemoryError`s.

Build
-----
Building Glide with gradle is fairly straight forward:

```shell
git clone https://github.com/bumptech/glide.git 
cd glide
./gradlew jar
```

**Note**: Make sure your *Android SDK* has the *Android Support Repository* installed, and that your `$ANDROID_HOME` environment
variable is pointing at the SDK or add a `local.properties` file in the root project with a `sdk.dir=...` line.

Samples
-------
Follow the steps in the [Build](#build) section to set up the project and then:

```shell
./gradlew :samples:flickr:run
./gradlew :samples:giphy:run
./gradlew :samples:svg:run
./gradlew :samples:contacturi:run
```
You may also find precompiled APKs on the [releases page][1].

Development
-----------
Follow the steps in the [Build](#build) section to setup the project and then edit the files however you wish.
[Android Studio][26] cleanly imports both Glide's source and tests and is the recommended way to work with Glide.

To open the project in Android Studio:

1. Go to *File* menu or the *Welcome Screen*
2. Click on *Open...*
3. Navigate to Glide's root directory.
4. Select `setting.gradle`

For more details, see the [Contributing docs page][27].

Getting Help
------------
To report a specific problem or feature request, [open a new issue on Github][5]. For questions, suggestions, or
anything else, email [Glide's discussion group][6], or join our IRC channel: [irc.freenode.net#glide-library][13].

Contributing
------------
Before submitting pull requests, contributors must sign Google's [individual contributor license agreement][7].

Thanks
------
* The **Android team** and **Jake Wharton** for the [disk cache implementation][8] Glide's disk cache is based on.
* **Dave Smith** for the [GIF decoder gist][9] Glide's GIF decoder is based on.
* **Chris Banes** for his [gradle-mvn-push][10] script.
* **Corey Hall** for Glide's [amazing logo][11].
* Everyone who has contributed code and reported issues!

Author
------
Sam Judd - @sjudd on GitHub, @samajudd on Twitter

License
-------
BSD, part MIT and Apache 2.0. See the [LICENSE][16] file for details.

Disclaimer
---------
This is not an official Google product.

[1]: https://github.com/bumptech/glide/releases
[2]: https://github.com/bumptech/glide/wiki
[3]: https://bumptech.github.io/glide/ref/javadocs.html
[4]: https://www.jetbrains.com/idea/download/
[5]: https://github.com/bumptech/glide/blob/master/CONTRIBUTING.md
[6]: https://groups.google.com/forum/#!forum/glidelibrary
[7]: https://developers.google.com/open-source/cla/individual
[8]: https://github.com/JakeWharton/DiskLruCache
[9]: https://gist.github.com/devunwired/4479231
[10]: https://github.com/chrisbanes/gradle-mvn-push
[11]: static/glide_logo.png
[12]: https://github.com/bumptech/glide/wiki/Integration-Libraries
[13]: http://webchat.freenode.net/?channels=glide-library
[14]: https://github.com/bumptech/glide/tree/3.0
[15]: https://github.com/bumptech/glide/tree/master
[16]: https://github.com/bumptech/glide/blob/master/LICENSE
[17]: http://bumptech.github.io/glide/dev/snapshots.html
[18]: https://github.com/bumptech/glide/issues?q=is%3Aissue+CircleImageView+OR+CircularImageView+OR+RoundedImageView
[19]: https://github.com/wasabeef/glide-transformations
[20]: https://bumptech.github.io/glide/
[21]: https://bumptech.github.io/glide/doc/generatedapi.html
[22]: https://muyangmin.github.io/glide-docs-cn/
[23]: http://bumptech.github.io/glide/int/okhttp3.html
[24]: http://bumptech.github.io/glide/int/volley.html
[25]: http://bumptech.github.io/glide/doc/download-setup.html#proguard
[26]: https://developer.android.com/studio/index.html
[27]: http://bumptech.github.io/glide/dev/contributing.html
