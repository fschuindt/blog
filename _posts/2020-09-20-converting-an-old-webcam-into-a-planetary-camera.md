---
layout: post
title: "Converting an old webcam into a planetary camera"
categories: [Astronomy, IT]
---

*This post was originally published on my [old blog](https://boredprogrammer.postach.io/post/converting-an-old-webcam-into-a-planetary-camera) dedicated to amateur astronomy.*

Willing to test [EQAlign](http://eqalign.net/e_eqalign.html) I decided to grab an old webcam I had here (Logitech C270 HD) and convert it to a so called "planetary camera". It consists of removing the built-in lens and replacing it with a telescope.  So I removed the lens and glued a 1.25" ocular tube to the plastic case, then I could easily attach/detach it to/from telescope focusers.

Starting to disassembly:

![1]({{ site.url }}/images/2020-09-20-converting-an-old-webcam-into-a-planetary-camera/1.jpg)

This black dusty thing is the lens, I need to remove it:

![2]({{ site.url }}/images/2020-09-20-converting-an-old-webcam-into-a-planetary-camera/2.jpg)

Two screws in the back held it in place:

![3]({{ site.url }}/images/2020-09-20-converting-an-old-webcam-into-a-planetary-camera/3.jpg)

Sensor exposed (I forgot to photograph, but this component marked as D1 is a green LED that lights up when the camera is being used. I covered it with black electrical tape so it won't mess with the sensor):

![4]({{ site.url }}/images/2020-09-20-converting-an-old-webcam-into-a-planetary-camera/4.jpg)

Reassembly without the lens and gluing the 1.25" ocular tube. It's really important to center the sensor on the tube. (Used super glue here):

![5]({{ site.url }}/images/2020-09-20-converting-an-old-webcam-into-a-planetary-camera/5.jpg)

I got this tube from a crappy ocular I had here, so I used the ocular lens itself as a dust cover for the sensor:

![6]({{ site.url }}/images/2020-09-20-converting-an-old-webcam-into-a-planetary-camera/6.jpg)

Cut off the webcam support:

![7]({{ site.url }}/images/2020-09-20-converting-an-old-webcam-into-a-planetary-camera/7.jpg)

Added black tape all around it to prevent light from entering:

![8]({{ site.url }}/images/2020-09-20-converting-an-old-webcam-into-a-planetary-camera/8.jpg)

And just out of curiosity, you can use this on a DSLR lens:

![9]({{ site.url }}/images/2020-09-20-converting-an-old-webcam-into-a-planetary-camera/9.jpg)

![10]({{ site.url }}/images/2020-09-20-converting-an-old-webcam-into-a-planetary-camera/10.jpg)

I was able to take this picture using this lens, kinda funny:

![11]({{ site.url }}/images/2020-09-20-converting-an-old-webcam-into-a-planetary-camera/11.png)

Now it's time to test it on a telescope:

![12]({{ site.url }}/images/2020-09-20-converting-an-old-webcam-into-a-planetary-camera/12.png)

Pointing it to the top of a far away building:

![13]({{ site.url }}/images/2020-09-20-converting-an-old-webcam-into-a-planetary-camera/13.png)

Used the notebook on Google Hangouts for a proof of concept and it worked great, just a little bit of dust on the sensor but that is easily cleaned (The low quality is due to atmospheric perturbations):

![14]({{ site.url }}/images/2020-09-20-converting-an-old-webcam-into-a-planetary-camera/14.png)

So I decided to connect my control station with Ekos / INDI:

![15]({{ site.url }}/images/2020-09-20-converting-an-old-webcam-into-a-planetary-camera/15.png)

And got it streaming wireless to my workstation, where I can also control the mount using a X Box joystick:

![16]({{ site.url }}/images/2020-09-20-converting-an-old-webcam-into-a-planetary-camera/16.png)

It was fun to do this, now I'll manage to clean the sensor and proceed to try polar alignment using EQAlign.
