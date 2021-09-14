---
title: How to secure S3 bucket with username and password
author: admin
type: post
date: 2018-09-12T08:00:16+00:00
url: /2018/how-to-secure-s3-bucket-with-username-and-password/
thumbnail: images/uploads/2018/09/data-1590455_1280.jpg
nkweb_code_in_head:
  - default
nkweb_Use_Custom_js:
  - default
nkweb_Use_Custom_Values:
  - default
nkweb_Use_Custom:
  - 'false'
categories:
  - Open Source
  - Software
tags:
  - caddy
  - password
  - s3

---
When you start looking for a solution you will find answers like: use Cloudfront, use lambda, use middleware 3rd party service. And basically you don&#8217;t have any other choice. There is no simple solution to your problem. But &#8211; as long as you are using [Caddy Server][1] solution is super easy.

<!--more-->

All we need is to have [basicauth](https://en.wikipedia.org/wiki/Basic_access_authentication), and then proxy server to redirect queries. The only problem is a matter of Authorization header. If you pass it &#8220;as is as&#8221; to S3 &#8211; it will cause a problem because expects valid header when data is protected. But if you pass &#8220;empty&#8221; header it will pass.&nbsp;


```
example.com.au {
	basicauth / username password
	proxy / http://s3-ap-southeast-2.amazonaws.com/example.com.au/ {
		header_upstream Authorization " "
	}
}
```

 [1]: https://caddyserver.com/