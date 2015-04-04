# Image Loader - Android-Universal-Image-Loader, picasso, glide, fresco

背景: Android-Universal-Image-Loader, picasso, glide, fresco

為什麼要 Image Loader ，什麼是 Image Loader ？

1. 顯示快取 - 首先，為了再次顯示一樣的圖片時，可以快速顯示，所以我們會把 bitmap 暫存在記憶體 memory cache 。那在有限的 memory cache ，我們要如何管理 ? 常見的是 LRU cache 策略，memory cache 有限，最近看過的優先留下來，其他捨去。以及依據畫布大小的快取。
2. 儲存快取 - 再來，網路來的圖片、網址圖片，下載儲存快取管理 - Disk Cache。
3. 連線快取 - okhttp SPDY

在 fresco 出現之前，我會比較推崇 AUIL(Android Universal Image Loader)，接著 glide 、picasso 。

追求小 code size 可以選 picasso ，輕簡。

AUIL 是因為在記憶體、儲存空間的快取策略還有其它有的沒有的都可以訂製。彈性比較大一點(所以 code size 大一點)。

只有 fresco 需要更換 layout class 原因是因為它為了效能，操作較低階的畫布。(Layout Class 都換了，所以順便支援 Gif 、影片? 誤)

對於 Image Loader 常見的需求：

* 更換 OkHttp 下載器
* 潤角、圓圖、顯示動畫
* 支援 Exif 轉正，支援影片快照轉正

## Fresco - facebook

* ImagePipeLine - 更換 OkHttp 下載器 (有 bug ，[PR#21](https://github.com/facebook/fresco/pull/21) 中)

## picasso - square

* Factory -> 更換 OkHttp 下載器
* Transformer -> 潤角、圓圖、顯示動畫

## glide

glide 目前看起來 google 有些演講有提過


## Android-Universal-Image-Loader

* ImageDownloader -> 更換 OkHttp 下載器
* ImageDisplayer -> 潤角、圓圖、顯示動畫
* ImageDecoder -> 支援 Exif 轉正，支援影片轉正

## Fresco - facebook

* ImagePipeLine - 更換 OkHttp 下載器 (有 bug ，[PR#21](https://github.com/facebook/fresco/pull/21) 中)

## picasso - square

* Factory -> 更換 OkHttp 下載器
* Transformer -> 潤角、圓圖、顯示動畫

## glide

glide 目前看起來 google 有些演講有提過


## 附錄 - 虛擬網址 - NetworkPipeContentProvider

`https://graph.facebook.com/{id}/picture` 雖然有轉址能力，不過這裡為了教學所需，還是寫了一個 FacebookPictureProvider

ImageLoader 大多支援 ContentProvider 網址，`content://` 所以我們可以利用它來做虛擬網址轉址。

例如:

GET `https://graph.facebook.com/601234567/picture`:

```json
{
  "data": {
    "is_silhouette": false, 
    "url": "https://fbcdn-profile-a.akamaihd.net/hprofile-ak-xpf1/v/t1.0-1/p50x50/1234567"
  }
}
```

data.url 才是真實的圖片網址。

所以我們註冊一個內部網域： 

`content://com.facebook.provider.PictureProvider/601234567`

提供轉址到真實圖片網址：

`https://fbcdn-profile-a.akamaihd.net/hprofile-ak-xpf1/v/t1.0-1/p50x50/1234567`

```java
public class FacebookPictureProvider extends NetworkPipeContentProvider {
    public static final String AUTHORITY = "com.facebook.provider.PictureProvider";
    public static final Uri CONTENT_URI = Uri.parse("content://" + AUTHORITY + "/");
    private Facebook facebook;

    // content://com.facebook.provider.PictureProvider/601234567
    @Override
    public String getUri(Uri uri) { // 轉址 - 改寫網址
       Picture picture = facebook.picture(uri.getPathSegments().get(0)).toBlocking().first();
       
       return picture.data.url;
    }
    
    interface Facebook {
        // https://graph.facebook.com/{id}/picture
        // https://graph.facebook.com/601234567/picture
        @GET("/{id}/picture")
        Observable<Picture> picture(@Path("id") String id);
    }
    
    static class Picture {
        Data data;
        static class Data {
            // "https://fbcdn-profile-a.akamaihd.net/hprofile-ak-xpf1/v/t1.0-1/p50x50/1234567"
            String url; 
        }
    }

    public FacebookPictureProvider() {
        RestAdapter restAdapter = new RestAdapter.Builder()
            .setEndpoint("https://graph.facebook.com")
            .build();
        facebook = restAdapter.create(Facebook.class);
    }
}
```


* https://gist.github.com/yongjhih/2175ec83be13bd7d9740#file-networkpipecontentprovider-java

## See Also

* Using okhttp backed for Volley - https://gist.github.com/bryanstern/4e8f1cb5a8e14c202750
* https://github.com/facebook/fresco/pull/21