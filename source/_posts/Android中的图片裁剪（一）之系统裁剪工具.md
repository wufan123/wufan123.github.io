---
title: Android中的图片裁剪（一）之系统裁剪工具
date: 2016.05.18 17:38
categories: 
    - [Android]
tags: [Android]
---

应用中图片裁剪的需求是很常见的，在android中裁剪的图片最简单的方法就是调用系统中的裁剪图片应用
``` java
Intent intent = new Intent();
intent.setAction("com.android.camera.action.CROP");
intent.setDataAndType(uri, "image/*");
startActivityForResult(intent, Constants.REQUEST_CODE_RESIZE_IMAGE);
``` 
当然在调用系统的裁剪功能时，我们还可以附加一些其他的信息：
``` java
intent.putExtra("outputX", 300);  //裁剪图片的宽
intent.putExtra("outputY", 300);  
intent.putExtra("aspectX", 1);  //裁剪方框宽的比例
intent.putExtra("aspectY", 1);  
intent.putExtra("scale", true);  //是否保持比例
intent.putExtra("return-data", false);  //是否返回bitmap

intent.putExtra(MediaStore.EXTRA_OUTPUT, uri);  //保存图片到指定uri
intent.putExtra("outputFormat", Bitmap.CompressFormat.JPEG.toString());  //输出格式
```  
上面的是一些常用的附加信息，如果你的outputX和outputY设置的比较大的话，返回的图片可能会导致OOM。最后在onActivityResult中接收图片就可以了
``` java
protected void onActivityResult(int requestCode, int resultCode, Intent data)
{
    super.onActivityResult(requestCode, resultCode, data);
    switch (requestCode)
    {
        case CROP_IMAGE_SYS:
            Bitmap bitmap = (Bitmap) data.getParcelableExtra("data");//拿到返回图片
            /*try
            {
             //取的裁剪后保存到本地的图片
              bitmap = BitmapFactory.decodeStream(getContentResolver().openInputStream(uri));
            }
            catch (FileNotFoundException e)
            {
              e.printStackTrace();
            }*/
            //TODO
            break;
//...
```  
当然因为android碎片化严重，不同厂商的系统也有所差异，所以系统的裁剪图片功能也可能有所不同，这也是坑，就像在我的项目中，发现有些手机基于内存考虑对图片缩略了很多，这样裁剪出来的图片分辨率就达不到项目要求，这时候就只能求助于第三方的开源项目或者自定义了，在第二篇文章中[Android中的图片裁剪（二）](http://www.jianshu.com/p/941bfe005e02)，我会说说如何自己实现裁剪图片的功能。
