---
layout: post
title:  "Flickr API + PHP 简单示例"
date:   2010-07-15 14:17:59
categories: 
- Notes 
tags:
- Flickr
- PHP

---

[Flickr](http://www.flickr.com/) 是互联网上最好的相册。如果不是当年被封锁过，可能在国内的普及会更广一点。现在虽然解封了（听说部分地方还是不能访问），但以后怎样还是不好说。

不过这不影响我使用它。Flickr 是第一个让我明白和学会使用 API 的网站。如果你懂得 PHP，这篇文章可以让你学会如何创建一个简单的程序。文章假设你已经注册过 Flickr 帐号并且使用过一段时间（上传了一些照片）。

目标：创建一个 PHP 页面显示自己 Flickr 帐号的最新图片。类似[这个页面](http://dannyli.net/photos)，不过没有预览大图的功能。

## 申请 API Key，建立应用程序
---

使用 Flickr 的 API，需要在官网申请一个 API Key

1.  进入 API Key 申请页面 [http://www.flickr.com/services/apps/create/apply/](http://www.flickr.com/services/apps/create/apply/)
2.  点 "APPLY FOR NON-COMMERCIAL KEY" 按钮。
3.  在下个页面填入应用程序信息，如名称和描述。自己用的话这个不是很重要。并且把下面两个条款打上勾，然后点 Submit 提交。
4.  下个页面会得到你的 Key 和一个被称做 Secret 的字串。记下这两个字符串，之后我们要用。
<!--more-->

## 查找自己的 User ID
---

进入[这个](http://www.flickr.com/services/api/explore/?method=flickr.people.getInfo)页面。右边有个 Your user ID，类似于 27769101@N03 的形式。记下这个 User ID。

## 建立 PHP 程序
---

1.  创建一个 PHP 程序，例如命名为 flickr.php
2.  把以下代码粘贴进去，修改里面的 API KEY 和 User ID 为你自己的，保存。 
{% highlight php %}
<?php
	# build the API URL to call
	$params = array(
		'api_key' => '这里替换成你的 API KEY',
		'method' => 'flickr.people.getPublicPhotos',
		'user_id' => '这里替换成你的 User ID',
		'format' => 'php_serial',
	);

	$encoded_params = array();
	foreach ($params as $k => $v){
		$encoded_params[] = urlencode($k).'='.urlencode($v);
	}

    // create the url string...

	# call the API and decode the response

    $url = "http://api.flickr.com/services/rest/?".implode('&', $encoded_params);

    //if you enter above url in browser, you will get a string with relevant data.

    $rsp = file_get_contents($url);

    $rsp_obj = unserialize($rsp); //create an multi-dimension array with data

	# display the photo title (or an error if it failed)

    if ($rsp_obj['stat'] == 'ok'){
    	foreach ($rsp_obj['photos']['photo'] as $photo){
			$photo['m_url'] = 'http://farm'.$photo['farm'].'.static.flickr.com/'.$photo['server'].'/'.$photo['id'].'_'.$photo['secret'].'_s.jpg';
			echo '<img src="'.$photo['m_url'].'" alt="" />';
		}
    }else{
    	echo "Call failed!";
	}

?>
{% endhighlight %} 
