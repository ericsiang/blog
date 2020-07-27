---
title: Laravel Socialate google 登入出現403 Forbidden
date: 2020-07-27
categories:
- Laravel
- Socialate
- Google login
tags:
- Laravel
- Socialate
- Google login
---
# Laravel 使用 Socialate google 登入出現403 Forbidden
>製作者：Ericsiang
>
###### tags: `Laravel` `Socialate` `Google login`

### 本機測試正常，但移到正式站時會出現此錯誤

在google大神上搜尋，終於找到

參考網址:https://stackoverflow.com/questions/53348777/socialite-laravel-authantication-error-with-google-api/53349266#53349266

主要問題是當google callback URL內帶的參數有.profile，就會出現此錯誤

因此要修改Socialate套件，下面是檔案位置
``` php
core/vendor/laravel/socialite/src/Two/GoogleProvider.php
``` 

將profile隱藏
``` php
protected $scopes = [
    'openid',
    //'profile',
    'email',
];
``` 

再次測試  就不會報錯了
記錄一下   下次發生就知道了