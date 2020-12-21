<div align="center">

# LARAVEL HASIN

<p>
    <a href="https://github.com/biiiiiigmonster/hasin/blob/master/LICENSE"><img src="https://img.shields.io/badge/license-MIT-7389D8.svg?style=flat" ></a>
    <a href="https://github.com/biiiiiigmonster/hasin/releases" ><img src="https://img.shields.io/github/release/biiiiiigmonster/hasin.svg?color=4099DE" /></a> 
    <a href="https://packagist.org/packages/biiiiiigmonster/hasin"><img src="https://img.shields.io/packagist/dt/biiiiiigmonster/hasin.svg?color=" /></a> 
    <a><img src="https://img.shields.io/badge/php-7+-59a9f8.svg?style=flat" /></a> 
</p>

</div>

`hasin(hasMorphIn)`是一个基于`where in`语法实现的`Laravel ORM`关联关系查询的扩展包，部分业务场景下可以替代`Laravel ORM`中基于`where exists`语法实现的`has(hasMorphIn)`，以获取更高的性能。


## 环境

- PHP >= 7
- laravel >= 5.5


## 安装

```bash
composer require biiiiiigmonster/hasin
```

### 简介

`Laravel ORM`的关联关系非常强大，基于关联关系的查询`has`也给我们提供了诸多灵活的使用方式，但是在**主表**数据量比较多的时候会有比较严重的性能问题，主要是因为`whereHas`用了`where exists (select * ...)`这种方式去查询关联数据。


通过这个扩展包提供的`whereHasIn`方法，可以把语句转化为`where id in (select xxx.id ...)`的形式，从而提高查询性能，下面我们来做一个简单的对比：


> 当主表数据量较多的情况下，`where id in`会有明显的性能提升；当主表数据量较少的时候，两者性能相差无几。

```php
<?php
/**
 * SQL:
 * 
 * select * from `product` 
 * where exists 
 *   ( 
 *      select * from `product_skus` 
 *      where `product`.`id` = `product_skus`.`p_id` 
 *      and `product_skus`.`deleted_at` is null 
 *   ) 
 * and `product`.`deleted_at` is null 
 * limit 10 offset 0
 */
$products = Product::has('skus')->paginate(10);

/**
 * SQL:
 * 
 * select * from `product` 
 * where `product`.`id` IN  
 *   ( 
 *      select `product_skus`.`p_id` from `product_skus` 
 *      where `product`.`id` = `product_skus`.`p_id` 
 *      and `product_skus`.`deleted_at` is null 
 *   ) 
 * and `product`.`deleted_at` is null 
 * limit 10 offset 0
 */
$products = Product::hasIn('skus')->paginate(10);
```

> `Laravel ORM`十种关联关系详细案例sql输出可查看[有道云笔记](https://note.youdao.com/noteshare?id=882bfd7ccdf1370c55326a33333c6f62)

### 使用

在配置文件app.php添加配置，自动注册服务
```php
<?php
    // ...
    
    'providers' => [
        // ...
        
        BiiiiiigMonster\Hasin\HasinServiceProvider::class,// hasin扩展包引入
    ],
```
此扩展`hasIn(hasMorphIn)`支持`Laravel ORM`中的所有关联关系，入参及使用方式与`has(hasMorph)`完全一致，可安全替换

> hasIn

```php
// hasIn
Product::hasIn('skus')->get();

// orHasIn
Product::where('name', 'like', '%拌饭酱%')->orHasIn('skus')->get();

// doesntHaveIn
Product::doesntHaveIn('skus')->get();

// orDoesntHaveIn
Product::where('name', 'like', '%拌饭酱%')->orDoesntHaveIn('skus')->get();
```

> whereHasIn

```php
// whereHasIn
Product::whereHasIn('skus', function ($query) {
    $query->where('sales', '>', 10);
})->get();

// orWhereHasIn
Product::where('name', 'like', '%拌饭酱%')->orWhereHasIn('skus', function ($query) {
    $query->where('sales', '>', 10);
})->get();

// whereDoesntHaveIn
Product::whereDoesntHaveIn('skus', function ($query) {
    $query->where('sales', '>', 10);
})->get();

// orWhereDoesntHaveIn
Product::where('name', 'like', '%拌饭酱%')->orWhereDoesntHaveIn('skus', function ($query) {
    $query->where('sales', '>', 10);
})->get();
```

> hasMorphIn

```php
Image::hasMorphIn('imageable', [Product::class, Brand::class])->get();
```

#### 嵌套关联

```php
Product::hasIn('attrs.values')->get();
```

## 联系交流
wx：biiiiiigmonster(备注：hasin)

## License
[MIT 协议](LICENSE)
