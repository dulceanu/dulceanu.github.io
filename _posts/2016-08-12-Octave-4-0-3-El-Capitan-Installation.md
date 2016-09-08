---
layout: post
title: How to install Octave 4.0.3 on OS X El Capitan
author: andrei
comments: true
category: machine-learning
---

Here's a quick guide on how to install Octave 4.0.3 on OS X El Capitan and also enable plotting.

### Octave 4.0.3 installation on OS X El Capitan:

```bash
$ brew tap homebrew/science
$ brew update && brew upgrade --all
$ brew install octave
$ cat <<EOT >> ~/.bash_profile
export FONTCONFIG_PATH=/opt/X11/lib/X11/fontconfig
EOT
```
----
Now let's give it a spin:

```bash
$ octave
octave >> % let's test if it works with a simple plot
octave >>  x=1:10; y=x.^2; plot(x,y)
```

If everything is ok you should see a plot as in the image below:

![Simple plot]({{ site.url }}/assets/example-plot.png)

Note: If you don't have XQuartz, you can grab it from [XQuartz page](https://dl.bintray.com/xquartz/downloads/XQuartz-2.7.9.dmg).
