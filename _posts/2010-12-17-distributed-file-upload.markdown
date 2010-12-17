---
layout: post
title: Distributed File Upload
tags: file upload multiple distributed
---

In a project I had the problem to upload multiple files through multiple http servers. But the files should have been processed at the same time in a predefined order. The files are not necessarily persisted. After all uploads and some other editing I want a list of all uploads. I wanted the file upload to be stateless and without a I central store, that I have to poll separately.

To accomplish this, the client has to send the edited meta-data and the files (or locations to them) at once, so that the responding http server has the full list. Sending all files at once wasn't a possible choice, because the customer wanted to upload files without leaving the current page.

One can easily upload files to another iframe to omit the page reload, but you want to keep the location information of the upload. Saving such data at the client side opens the door for security issues (if the client manipulates the file location on the server). I decided to use a little cryptographically signed and maybe encrypted *JSON* package containing information such as name, type, size and location. Be sure to use something like an [HMAC](http://en.wikipedia.org/wiki/HMAC) which is proven to be secure.

Because each request can end up on a separate system i use the hostname of the current web server and created a full qualified URI for the uploaded file. So that, when the final request to process the files was send, the server can then download the files from the other http servers. Because of security reasons you want these download URIs for these files to only be accessible from the local network.

If you combine the internal file delivery with an [X-Sendfile](https://tn123.org/mod_xsendfile/) (*apache*) or [X-Accel-Redirect](http://wiki.nginx.org/XSendfile) (*nginx*) file upload accelerator it is also a very performant solution. Bonus, you don't have to administrate further systems. Also there is no configuration beside the normal *DNS* load balancing necessary.

The downside of downloading the files are present in all solutions i can imagine. Either if you use a database, *NFS*, *NAS*, key-value stores or something else. You have to download all files if that is part of your problem. 

If you like or dislike the solution let me know.