---
layout: post
title:  "使用微信jssdk上传和下载图片"
date:   2015-08-08
keywords: ["weixin", "js-sdk"]
description: "使用微信jssdk上传和下载图片"
category: "weixin"
tags: ["weixin"]
---

前段时间在工作中需要实现图片的上传和下载的功能，今天做一下总结。
微信发布了js-sdk工具包，极大地方便了开发人员基于微信api开发一些微信相关的功能。
基于该api，开发人员可以限定用户上传图片的方式是基于手机的拍照功能还是直接上传手机中原有的图片，
这项功能对于那些保证上传的图片的真实性有一定的帮助。

### JS-SDK引入
要使用该工具包，需要在用到weixin API的页面引入js-sdk，引入的方式如下：

    <script  type="text/javascript" src="http://res.wx.qq.com/open/js/jweixin-1.0.0.js" ></script>
### 微信appId和微信appSecret

关于如何获取微信appId和Secret，微信js-sdk官方文档有比较详细的说明，这里就不赘述了。
微信appId和appSecret的引入可以直接在需要用到的地方直接引入，也可以写入ENV变量中，
用到的时候直接使用ENV["appId"]和ENV["appSecret"]直接取出来。这样做的好处是可以集中管理，
防止在写入的时候犯下拼写错误的低级错误，同时也有利于发布开源项目时隐藏私有的敏感信息。

首先在config目录下创建一个local_env.yml的文件，用来存放appId和appSecret，该文件也可一用来存放一些其
他比较敏感的数据。yml文件接受键值对的写法，非常适合保存配置信息。

    Ruby Code:
    appId: "your appId"
    appSecret: "your appSecret"

然后在`application.rb`文件中加入如下代码，用来读取'local_env.yml'中的配置信息。

    Ruby Code:
    class Application < Rails::Application
      config.before_configuration do
        env_file = File.join(Rails.root, "config", "local_env.yml")
        YAML.load(File.open(env_file)).each do |k, v|
          ENV[k.to_s] = v
        end if File.exists?(env_file)
      end
    end

### appToken的获取

这里appToken的获取需要用到第三方的Gem--weixin_authorize,在gem中加入:

    gem 'wenxin_authrize'

然后执行'bundle isntall', 最后获取client的实例：

    $client = WeinAuthrize::Client.new(ENV['appId'], ENV['appSecret'])

### 从微信端下载图片到本地

这一步的关键是要获取到上传到微信服务器的`serverId`，这个参数可以在微信的上传接口`uploadImage`中获取，然后
通过`carrierWave`插件的`remote_xxx_url`方法下载到本地。其中xxx代表的是通过`carrierWave`插件mount过的字段。

### 微信配置项
这里主要讲一下jsApiList这个选项，这里是你接下来在该页面中将要用到的微信接口。
本例中主要用到`chooseImage`和`uploadImage`两个接口。

    Ruby Code:
    <% sign_package = $client.get_jssign_package(request.url) %>
    wx.config({
      debug: false,
      appId: "<%= sign_package['appId'] %>",
      timestamp: "<%= sign_package['timestamp'] %>",
      nonceStr: "<%= sign_package['nonceStr'] %>",
      signature: "<%= sign_package['signature'] %>",
      jsApiList: [
        'chooseImage',
        'uploadImage'
      ]
    });

### 主要代码

    Ruby Code:
    <script  type="text/javascript" src="http://res.wx.qq.com/open/js/jweixin-1.0.0.js" ></script>
    <% sign_package = $client.get_jssign_package(request.url) %>
    <script>
      wx.config({
        debug: false,
        appId: "<%= sign_package['appId'] %>",
        timestamp: "<%= sign_package['timestamp'] %>",
        nonceStr: "<%= sign_package['nonceStr'] %>",
        signature: "<%= sign_package['signature'] %>",
        jsApiList: [
          'chooseImage',
          'uploadImage'
        ]
      });
      wx.ready(function () {

        var images = {
        localIdFront: [],
        localIdBack:[],
        combinedId: [],
        serverId: []
        };

        $('#frontImageList').click(function () {

            wx.chooseImage({
              count: 1,
              sizeType: ['compressed'],
              sourceType: ['camera'],
              success: function (res) {
                images.localIdFront = res.localIds;
                var image = document.createElement("img");
                image.src = res.localIds;
                image.id  = "front_img_uploaded";
                var front = document.querySelector("#front_img_uploaded");
                front.parentNode.replaceChild(image, front);
              }
            });
        });


        $('#backImageList').click(function () {

            wx.chooseImage({
              count: 1,
              sizeType: ['compressed'],
              sourceType: ['camera'],
              success: function (res) {
                images.localIdBack = res.localIds;
                var image = document.createElement("img");
                image.src = res.localIds;
                image.id  = "back_img_uploaded";
                var back = document.querySelector("#back_img_uploaded");
                back.parentNode.replaceChild(image, back);
              }
            });
        });

      $("#submit_and_upload_image").click( function() {
        if (images.localIdFront.length === 0 || images.localIdBack.length === 0 ) {
          alert('请先拍摄身份证正面和反面');
          return;
        }
        var i = 0, length = images.localIdFront.length + images.localIdBack.length;
        images.serverId = [];
        images.combinedId.push(images.localIdFront);
        images.combinedId.push(images.localIdBack);
        function upload() {
          wx.uploadImage({
            localId: images.combinedId[i][0],
            success: function (res) {
              i++;
              images.serverId.push(res.serverId);
              if (i < length) {
                upload();
              }
              if (i == 2 ){
                downloadBackImages(images.serverId[0], images.serverId[1]);
              }
            },
            fail: function (res) {
              alert(JSON.stringify(res));
            }
          });
        }
        upload();

      });

      function downloadBackImages(mediaIdOne, mediaIdTwo) {
        $.ajax({
          url: '<%= download_identity_image_user_path %>',
          data: { mediaIdOne: mediaIdOne, mediaIdTwo: mediaIdTwo }
          }).done(function(html ) {
            window.location.href = "<%= redirect_to_path %>";
          }).fail(function(jqXHR, textStatus) {
            alert("提交失败!");
          });
      }

      });
    </script>
