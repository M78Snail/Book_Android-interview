在Android开发中上传图片（头像）到服务器，要先压缩图片，获取原图的长宽，然后取得压缩比例，compress到指定的质量，输出保存，然后网络上传这张图片就行了。

1，乾言

如果图片太大，上传不仅耗时，而且体验不好。即使加了loading效果，那还是挺耗流量的。so，果断要压缩图片再上传，android客户端，尤其要注意。

2，拿到原图tempPic，开撸

压缩图片并保存到临时目录，如果图片存在上传压缩的图片，没有压缩成功（不存在），就直接上原图吧。

```
// 压缩图片并上传
private void uploadFileInThreadByOkHttp(final Activity context, final String actionUrl, final File tempPic) {
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

4，上传图片，发射

使用okhttp，如果不太熟悉okhttp的话，可以看我的另一博文:  
Android网络请求：OkHttp实战  
本文上传头像，封装uploadFile方法如下：

```
/**
     * 上传文件
     * @param url  接口地址
     * @param file 上传的文件
     * @param mediaType  资源mediaType类型:比如 MediaType.parse("image/png");
     * @param responseCallback 回调方法,在子线程,更新UI要post到主线程
     * @return
     */
     public boolean uploadFile(String url, File file, MediaType mediaType, Callback responseCallback) {
        MultipartBody.Builder builder = new MultipartBody.Builder().setType(MediaType.parse("multipart/form-data"));
        if (!file.exists()|| TextUtils.isEmpty(url)){
            returnfalse;
        }
        //addFormDataPart视项目自身的情况而定//builder.addFormDataPart("description","2.jpg");
        builder.addFormDataPart("file", file.getName(), RequestBody.create(mediaType, file));
        //构建请求体
        RequestBody requestBody = builder.build();
        Request request = new Request.Builder()
                .url(url)
                .post(requestBody)
                .build();
        enqueue(request,responseCallback);
        returntrue;
    }
```



