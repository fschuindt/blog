---
layout: post
title: Ruby Gem To Verify Firebase Id Token Signatures
---

![Article screenshot](https://s27.postimage.org/w2oxaldn7/firebase_banner.png)

<div style="height: 25px;"></div>

This is a off blog's topic, but I just released a new Ruby gem.

The gem [firebase_id_token](https://github.com/fschuindt/firebase_id_token/) was developed to easily verify Firebase ID Token signatures in Ruby back-end environments. It uses Redis to store Google's x509 certificates, which helps other processes in your application to access it really fast.

The Firebase ID Token is really a [JWT](https://tools.ietf.org/html/rfc7519). What the gem does is to check if the token was made for your application and if it's valid, both in it's parameters and in it's RSA signature.

Check out the [gem's Github](https://github.com/fschuindt/firebase_id_token/) for more info. :)
