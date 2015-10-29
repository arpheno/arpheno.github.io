---
layout: post
title:  "The interesting bugs only happen in production - SSL is difficult"
date:   2015-10-29 16:37:35
categories: backendfail
---
It turned out that getting SSL to agree with my application was harder than expected.

This was caused by the `DynProxyView` that is used to proxy URIs that match `https://backend.fail/\d+/result/.*$`.
The author of the `HTTPProxyView` which I am inheriting from, rightly assumed that it would be good to
copy all headers from the client request to the new `urllib2.Request` object that is generated to access the
proxied content, however for my case that also copied the `is_secure` headers and the `https://` scheme.

As the docker containers do not have certificates, they did not understand the incoming request and the requests would
constantly time out.

This would only happen in production when operating behind nginx and it was driving me insane.

----

In the process of debugging this problem I also installed and configured [`rabbitmq`](https://www.rabbitmq.com/)
 and [`celery`](http://docs.celeryproject.org/en/latest/whatsnew-3.1.html), which is going to be handy when we figure
 out a better way to launch the containers asynchronously.
