# 网站轮播管理

主体思路与导航菜单管理相同，唯一的区别是这里有文件上传需求，所以我们这里重点关注文件上传问题

## 快速调整除上传图片以外的功能

<!-- JSON 解析失败是因为转义字符问题 -->

> 🚩 源代码: step-80

## 异步图片上传

### 核心问题

目前遇到的问题是，我们需要在点击保存按钮之前，将图片上传到服务器，并且可以获取到图片的访问路径。

由于我们这里的轮播管理采用的是 AJAX 方式维护，使用之前的表单方式肯定没法在这里使用了（同步方式），这里必须有一种异步上传文件的方式：

明确问题：在选择完图片过后，异步将文件上传到服务端，并获取这张图片的访问路径。

### 解决方案

1. 第三方 flash 库（Deprecated）
1. iframe（Deprecated）
2. HTML5 提供的 [XMLHttpRequest Level 2](https://developer.mozilla.org/cn/docs/Web/API/XMLHttpRequest/Using_XMLHttpRequest#提交表单和上传文件)（Modern）

> https://developer.mozilla.org/zh-CN/docs/Web/API/FormData

### 异步上传文件接口

添加一个支持上传文件的接口 `upload.php`

```php
// 如果选择了文件 $_FILES['file']['error'] => 0
if (empty($_FILES['file']['error'])) {
  // PHP 在会自动接收客户端上传的文件到一个临时的目录
  $temp_file = $_FILES['file']['tmp_name'];
  // 我们只需要把文件保存到我们指定上传目录
  $target_file = '../static/uploads/' . $_FILES['file']['name'];
  if (move_uploaded_file($temp_file, $target_file)) {
    $image_file = '/static/uploads/' . $_FILES['file']['name'];
  }
}

// 设置响应类型为 JSON
header('Content-Type: application/json');

if (empty($image_file)) {
  echo json_encode(array(
    'success' => false
  ));
} else {
  echo json_encode(array(
    'success' => true,
    'data' => $image_file
  ));
}
```

> 🚩 源代码: step-81

### XHR2 调用接口上传文件

```js
/**
 * 异步上传文件
 */
$('#upload').on('change', function () {
  // 准备要上传的数据
  var formData = new FormData()
  formData.append('file', this.files[0])

  // 发送 AJAX 请求，上传文件
  var xhr = new XMLHttpRequest()
  xhr.open('POST', '/admin/upload.php')
  xhr.addEventListener('load', function () {
    var res = JSON.parse(xhr.response)
    console.log(res)
  })
  xhr.send(formData)
})
```

> 🚩 源代码: step-82

jQuery 也是可以的（内部仍然是使用的 XMLHttpRequest level 2）

```js
/**
 * 异步上传文件
 */
$('#upload').on('change', function () {
  // 准备要上传的数据
  var formData = new FormData()
  formData.append('file', this.files[0])

  // 发送 AJAX 请求，上传文件
  $.ajax({
    url: '/admin/upload.php',
    cache: false,
    contentType: false,
    processData: false,
    data: formData,
    type: 'post',
    success: function (res) {
      if (res.success) {
        $('#image').val(res.data).siblings('.thumbnail').attr('src', res.data).fadeIn()
      } else {
        $('#image').val('').siblings('.thumbnail').fadeOut()
        notify('上传文件失败')
      }
    }
  })
})
```

> 🚩 源代码: step-83
