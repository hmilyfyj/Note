title: CI源码分析-Image_lib.php
date: 2016-02-20 15:51
tags: [CI,PHP]
categories: CI
---

对图像的处理较生疏，特地研究一下CI的图像类库。

[官方文档](http://codeigniter.org.cn/user_guide/libraries/image_lib.html)

<!-- more -->


# 知识点

    bool property_exists ( mixed $class , string $property )

本函数检查给出的 property 是否存在于指定的类中（以及是否能在当前范围内访问）。


## 扩展载入

## 图片处理

## 数组处理

    end() 

> end() 将 array 的内部指针移动到最后一个单元并返回其值。

# 流程

## 初始化

1. 格式化颜色参数 #123 => #112233
2. 检测 必要参数与函数是否存在：`source_image`、`getimagesize()`
3. 获取路径、文件名：
3.1 图片绝对路劲`full_source_path`、`source_folder`、图片名`source_image`
3.2 利用getimagesize() 函数获取图片的基本信息：尺寸（`orig_width、orig_height`）、大小(`size_str`)、后缀、mime类型（`mime_type`）
4. 检测参数
4.1 目标图片路径`new_image`，`dest_folder`、`dest_image`、 `full_dest_path`
4.2 检测缩略图参数 `create_thumb`
4.3 获取目标图片的 名字、后缀
4.4 `image_reproportion()`  在设定新的宽高时可能出现不协调的问题。按照比例修复设置错误的宽高比
4.4 修复未配置的`width`、`height`
4.5 设置图片品质`quality`，剪裁尺寸：`x_axis`、`y_axis`
5. 水印参数的配置


### 添加水印的方式

> **Text**：水印信息将以文字方式生成，要么使用你所指定的 TrueType 字体， 要么使用 GD 库所支持的内部字体。如果你要使用
> TrueType 版本， 那么你安装的 GD 库必须是以支持 TrueType 的形式编译的（大多数都是，但不是所有）。
> **Overlay**：水印信息将以图像方式生成，新生成的水印图像 （通常是透明的 PNG 或者 GIF）将覆盖在原图像上。

# 源码

待添加

```php
```

