在Android开发中上传图片（头像）到服务器，要先压缩图片，获取原图的长宽，然后取得压缩比例，compress到指定的质量，输出保存，然后网络上传这张图片就行了。

1，乾言

如果图片太大，上传不仅耗时，而且体验不好。即使加了loading效果，那还是挺耗流量的。so，果断要压缩图片再上传，android客户端，尤其要注意。

2，拿到原图tempPic，开撸

压缩图片并保存到临时目录，如果图片存在上传压缩的图片，没有压缩成功（不存在），就直接上原图吧。

```
// 压缩图片并上传privatevoiduploadFileInThreadByOkHttp(final Activity context, final String actionUrl, final File tempPic) {
    final String pic_path = tempPic.getPath();
    String targetPath = FileUtils.getThumbDir()+"compressPic.jpg";
    final String compressImage = PictureUtil.compressImage(pic_path, targetPath, 30);
    final File compressedPic = new File(compressImage);
    if (compressedPic.exists()) {
      LogUtils.debug(TAG,"图片压缩上传");
      uploadFileByOkHTTP(context, actionUrl, compressedPic);
   }else{//直接上传
        uploadFileByOkHTTP(context, actionUrl, tempPic);
   }
}
```

3，压缩图片，开撸

```
    public static String compressImage(String filePath, String targetPath, int quality)  {
        Bitmap bm = getSmallBitmap(filePath);
        //旋转照片角度，省略/*int degree = readPictureDegree(filePath);
        if(degree!=0){
            bm=rotateBitmap(bm,degree);
        }*/
        File outputFile=new File(targetPath);
        try {
            if (!outputFile.exists()) {
                outputFile.getParentFile().mkdirs();
                //outputFile.createNewFile();
            }else{
                outputFile.delete();
            }
            FileOutputStream out = new FileOutputStream(outputFile);
            bm.compress(Bitmap.CompressFormat.JPEG, quality, out);
        }catch (Exception e){}
        return outputFile.getPath();
    }

    /**
     * 根据路径获得突破并压缩返回bitmap用于显示
     */
     public static Bitmap getSmallBitmap(String filePath) {
        final BitmapFactory.Options options = new BitmapFactory.Options();
        options.inJustDecodeBounds = true;//只解析图片边沿，获取宽高
        BitmapFactory.decodeFile(filePath, options);
        // 计算缩放比
        options.inSampleSize = calculateInSampleSize(options, 480, 800);
        // 完整解析图片返回bitmap
        options.inJustDecodeBounds = false;
        return BitmapFactory.decodeFile(filePath, options);
    }
```



