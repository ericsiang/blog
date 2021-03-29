---
title: Laravel migrate 出現 Doctrine DBAL 報錯
date: 2022-03-29
categories:
- laravel
- error
tags:
- laravel
- error
---


# Laravel migrate 出現 Doctrine DBAL 報錯
###### tags:  `laravel` `error`    


執行php artisan migrate 顯示下方錯誤

```
requires Doctrine DBAL. Please install the doctrine/dbal package
```

錯誤顯示需安裝套套件doctrine/dbal

```
composer require doctrine/dbal
```

再次執行php artisan migrate 顯示下方錯誤

```
 Class 'Doctrine\DBAL\Driver\PDOMySql\Driver' not found
```

step 1. 查詢 composer.json檔，顯示版本為3.0，出現上方錯誤需降版為2.0，因此修改composer.json檔
```
"doctrine/dbal": "^3.0", => "doctrine/dbal": "^2.0",
```

step 2. 修改完執行
```
composer update
```

最後執行 php artisan migrate，
執行成功