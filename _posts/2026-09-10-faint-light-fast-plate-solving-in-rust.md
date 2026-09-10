---
layout: post
title: "Faint Light, fast plate-solving in Rust"
categories: [IT]
image: images/2026-09-10-faint-light-fast-plate-solving-in-rust/preview.png
excerpt: "Publishing a new open source software for plate-solving astronomy images up to 100x faster."
---

![Cover]({{ site.baseurl }}/images/2026-09-10-faint-light-fast-plate-solving-in-rust/Medieval_knight_puppets_from_Hortus_Deliciarum.png)

It's been a while since I've been wanting to publish this software I'm working on. It's been helping me a lot with my polar alignment routines, and I think it has great potential to keep growing and help others too. During my benchmarks, I noticed it can be 10-100x faster than other plate-solving or astrometry solutions out there.

For those wondering, plate-solving is the process of taking any random picture of the night sky and figuring out exactly where in the sky that image is pointed. The software starts with nothing but the pattern of stars in the image and no prior pointing information, matches those patterns against a star catalog, and then computes the image's exact position, orientation, and scale. From there, every pixel can be mapped to real celestial coordinates.

**Input:**

![Before plate-solving]({{ site.baseurl }}/images/2026-09-10-faint-light-fast-plate-solving-in-rust/plate_solving_example_before.jpg)

**Output:**

![After plate-solving]({{ site.baseurl }}/images/2026-09-10-faint-light-fast-plate-solving-in-rust/plate_solving_example_after.jpg)

And Faint Light is solving these images quite fast when compared against other implementations:

![Median wall time per blind solve over 256 real ZTF and TESS frames: Faint Light 0.32 s, offline astrometry.net (modified by us) 1.91 s, ansvr 7.11 s, PlateSolve 3 7.22 s, offline astrometry.net 10.1 s, ASTAP 19.9 s]({{ site.baseurl }}/images/2026-09-10-faint-light-fast-plate-solving-in-rust/blind-light.svg)

Check its website [here](https://nightsky.observer/faint-light/).  
Or check it on GitHub: [https://github.com/fschuindt/faint_light](https://github.com/fschuindt/faint_light)

<div style="height: 25px;">
</div>

<p style="text-align: center; margin: 2rem 0;"><a href="https://nightsky.observer/faint-light/" title="Faint Light on nightsky.observer"><img src="{{ site.baseurl }}/images/2026-09-10-faint-light-fast-plate-solving-in-rust/nightsky-observer-logo.png" alt="nightsky.observer" width="550" style="display: inline-block;"></a></p>

* * *

+ Cover picture: *Fighting Knight-Puppets, from Hortus Deliciarum, Herrad of Landsberg; c.1167 - c.1185* (Public Domain)
