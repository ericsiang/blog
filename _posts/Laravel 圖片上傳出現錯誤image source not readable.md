---
title: Laravel 圖片上傳出現錯誤image source not readable
date: 2021-03-05
categories:
- Laravel
- Image

tags:
- Laravel
- Image
---

# Laravel 圖片上傳出現錯誤image source not readable
>製作者：Ericsiang
>
###### tags: `Laravel` `Image` 

### 本機測試正常，但移到正式站時會出現此錯誤


在google大神上搜尋，終於找到

參考網址: https://medium.com/@hosomikai/laravel-php-artisan-storage-link-on-shared-hosting%E4%BB%A5cpanel-%E7%82%BA%E4%BE%8B-7c1217b8e96e

主要問題是在本地端有執行php artisan storage:link
但在正式Server沒有執行過，因此沒有關聯public/storage

### 解決方法
1. 把server上的public/storage，將storage刪除或重新命名
2. 在你的專案下增加一個簡單的 route 
``` php
// routes/web.php
<?php 
Route::get('create-symlink', function () {
  Artisan::call('storage:link');
});

```
3. 一樣直接瀏覽器瀏覽執行：
``` php
https://yourdomain/create-symlink
```
 產生完storage的連結在移除程式即可