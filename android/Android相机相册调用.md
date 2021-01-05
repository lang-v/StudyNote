# Android 调用相机拍照  & 调用系统裁剪 & 打开系统相册获取图片Uri 



### Android 调用系统相机拍照

1. 首先是申请权限

   ```kotlin
   Manifest.permission.CAMERA,
   Manifest.permission.WRITE_EXTERNAL_STORAGE,
   Manifest.permission.READ_EXTERNAL_STORAGE
   ```

2. 创建图片文件用来保存拍摄的照片

   - 获得存储目录

   ```kotlin
   val path = File(this.getExternalFilesDir(Environment.DIRECTORY_PICTURES)!!.absolutePath + File.separator + "picture_temp" + File.separator)
   ```

   **这里的picture_temp需要在内容提供器配置xml中写出来**

   - 注册内容提供器

   ```xml
   <provider
       android:name="androidx.core.content.FileProvider"
       android:authorities="cn.cqupt.cameraapplication.fileprovider"
       android:exported="false"
       android:enabled="true"
       android:grantUriPermissions="true">
       <meta-data
           android:name="android.support.FILE_PROVIDER_PATHS"
           android:resource="@xml/file_paths" />
   </provider>
   ```

   其中name为固定值 *androidx.core.content.FileProvider* ，authorities的值自己定义，后面会使用到；

   meta-data中的name也是固定值

   resource就是他的配置文件，我们需要在这里面把要分享的目录写出来

   - file_paths.xml

     ```xml
     <paths xmlns:android="http://schemas.android.com/apk/res/android">
         <external-files-path
             name="img_path"
             path="Pictures/picture_temp/"/>
     </paths>
     ```

     根目录使用paths ，子目录为 external-files-path（外置存储路径） , name任意,**path = "Pictures/picture_temp/" **代表的是

     **/storage/emulated/0/Android/data/你的包名/files/Pictures/picture_temp/**  

     通过上面的this.getExternalFilesDir(Environment.DIRECTORY_PICTURES).absolutePath得到的就是**/storage/emulated/0/Android/data/你的包名/files/Pictures/**  区分在Environment.DIRECTORY_PICTURES

     除此外还有

     ```java
     android.os.Environment#DIRECTORY_MUSIC
     android.os.Environment#DIRECTORY_PODCASTS
     android.os.Environment#DIRECTORY_RINGTONES
     android.os.Environment#DIRECTORY_ALARMS
     android.os.Environment#DIRECTORY_NOTIFICATIONS
     android.os.Environment#DIRECTORY_MOVIES
     ```

     ![img](https://img-blog.csdnimg.cn/20190103200323674.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzE3Njc4MjE3,size_16,color_FFFFFF,t_70)

     

   - 在此目录创建文件

     ```kotlin
     val file = File(path, "${System.currentTimeMillis()}.jpg")
     //存在则删除
     if (file.exists()) {
         deleteFile(file.absolutePath)
     }
     file.createNewFile()
     ```

   - 获取此文件的uri

     ```kotlin
     //在23 以下直接从文件获取
     imgUri = if (Build.VERSION.SDK_INT > Build.VERSION_CODES.M) {
         FileProvider.getUriForFile(this, "cn.cqupt.cameraapplication.fileprovider", file);
     } else {
         Uri.fromFile(file)
     }
     ```

3. 打开摄像头

   ```kotlin
   val intent = Intent(MediaStore.ACTION_IMAGE_CAPTURE)
   intent.putExtra(MediaStore.EXTRA_OUTPUT, imgUri)
   startActivityForResult(intent, 100)
   ```

4. 处理返回结果

   ```kotlin
   override fun onActivityResult(requestCode: Int, resultCode: Int, data: Intent?) {
       super.onActivityResult(requestCode, resultCode, data)
       if (requestCode == 100 && resultCode == RESULT_OK) {
           //发送广播刷新图库
           noticeSystemScanPicture()
           //显示拍摄的照片
           runOnUiThread {
               Glide.with(this)
                   .load(imgUri)
                   .override(400, 400)
                   .into(img)
           }
       }
   }
   ```



### 调用系统裁剪

```kotlin
private fun startCrop(uri: Uri) {
    //隐式跳转
    val intent = Intent("com.android.camera.action.CROP")
    //必须设置类型和将要裁剪的uri
    intent.setDataAndType(uri, "image/*")
    //下面这些没搞懂是什么作用，修改过后功能没有什么变化
    intent.putExtra("crop", true)
    intent.putExtra("scale", true)
    intent.putExtra("return-data", false)
    
    //        裁剪比例
    intent.putExtra("aspectX",1)
    intent.putExtra("aspectY",1)
    //是否有返回值，但是无论我开不开，都能在result中获取到图片
    intent.putExtra("return-data", false)
    //设置输出格式
    intent.putExtra("outputFormat", Bitmap.CompressFormat.JPEG.toString())
    startActivityForResult(intent, REQUTEST_CROP)//裁剪后的结果保存在onActivityResult的data：Intent中
}
```

#### 注意：

- 在摄像头拍完照片后无法直接进行裁剪，需要提前保存此图片在调用裁剪
- 每次裁剪都会将裁剪过后的图片自动保存在本地



### 打开系统相册选择图片

```kotlin
val intent = Intent(Intent.ACTION_PICK)
intent.type = "image/*"
startActivityForResult(intent, 200)
```

在onActivityResult中获取图片URI

```kotlin
if (requestCode == 200 && resultCode == RESULT_OK) {
     val temp = data?.data
    //调用系统裁剪  此处temp即为图片URI
     temp?.let { startCrop(it) }
}
```









