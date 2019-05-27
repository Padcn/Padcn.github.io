---
title: JQuery validate
date: 2017-07-21
tags: js
---
# JQuery validate

> http://www.runoob.com/jquery/jquery-plugin-validate.html
> http://blog.csdn.net/xh16319/article/details/9987847

引入validate文件
```
<script src="http://static.runoob.com/assets/jquery-validation-1.14.0/lib/jquery.js"></script>
<script src="http://static.runoob.com/assets/jquery-validation-1.14.0/dist/jquery.validate.min.js"></script>
```
需要注意的是引入文件时如果有其他jquery文件可能会有冲突，导致不起效，报的错为validate不能被识别。

```javascript
$().ready(function() {
// 在键盘按下并释放及提交后验证提交表单
  $("#signupForm").validate({
    rules: {
      firstname: "required",
      lastname: "required",
      username: {
        required: true,
        minlength: 2
      },
      password: {
        required: true,
        minlength: 5
      },
      confirm_password: {
        required: true,
        minlength: 5,
        equalTo: "#password"
      },
      email: {
        required: true,
        email: true
      },
      topic: {
        required: "#newsletter:checked",
        minlength: 2
      },
      agree: "required"
    },
    messages: {
      firstname: "请输入您的名字",
      lastname: "请输入您的姓氏",
      username: {
        required: "请输入用户名",
        minlength: "用户名必需由两个字母组成"
      },
      password: {
        required: "请输入密码",
        minlength: "密码长度不能小于 5 个字母"
      },
      confirm_password: {
        required: "请输入密码",
        minlength: "密码长度不能小于 5 个字母",
        equalTo: "两次密码输入不一致"
      },
      email: "请输入一个正确的邮箱",
      agree: "请接受我们的声明",
      topic: "请选择两个主题"
    }
});
```
## 远程验证
在一些情况下需要将数据与服务器端进行对比验证，比如注册用户时一些信息不能为系统中已经存在的数据，这个时候就需要将数据发送到服务器进行验证，服务器再返回验证结果。
```js
remote:{
	url:"<IPAddress:Port>/isExists",
    type:"post",
    dataType:"json",
    data:{
    	username:function(){
        	return $("#username").val();
        }
    }
}
```
服务器端只能返回true或者false。使用方式与required等一样。


# 自定义验证规则
格式为：addMethod(name,method,message)

虽然默认提供了许多种情况的验证，但是有的时候还是不能满足我们的需求，这个时候就需要我们自己进行自定义验证
```js
JQuery.validator.addMethod("myValidate",function(value,element,param){
	//验证是否中文
	var pattern=/[^\u4e00-\u9fa5]/;
	return pattern.test(value);
},$.validator.format("error messages here..."));
```
在使用的时候就可以和系统默认的规则required,digits等一样使用。

在这里function中参数只用到了value就是组件的值，element为组件本身，param是使用规则时可以传入的参数值。如 myValidate:["1","2"]，则param[0]为1,param[1]为2

> 常用正则 http://www.cnblogs.com/zxin/archive/2013/01/26/2877765.html