---
layout: post
title: How to | Set a Website as Linux Mint Wallpaper
date: 2017-11-08T10:48:14.000Z
categories: linux
excerpt: Display a website as your lockscreen
github: Gmentsik/linux-mint-website-screensaver
image: /images/posts/04/grafana.png
---

Since I'm monitoring everything I can with grafana and really like the graphs,
I was looking for a way to display grafana as my lock screen / screensaver.

Linux mint uses its own screensaver, you can install xscreensaver and download
the additional "webscreensaver" python script, but this is unneccesary.

The cinnamon screensaver provides a webkit screensaver plugin. We will use this
to load a html file that has an iframe to display a website.

## 01: Install cinnamon webkit screensaver

```
sudo apt install cinnamon-screensaver-webkit-plugin
```

## 02: Copy the template
* goto: `cd /usr/share/cinnamon-screensaver/screensavers/webkit@cinnamon.org`
* copy: `cp -r webkit-stars@cinnamon.org webkit-web@cinnamon.org`
* goto: `cd webkit-web@cinnamon.org`

## 03: Specify the target

* open editor `index.html`
`nano index.html`

Replace content with (also replace GOOGLE-URL with your URL):

``` index.html
<!DOCTYPE HTML>
<html>
  <head>

  </head>
  <body style="margin: 0">
<iframe style="position: absolute;width: 100%; height: 100%; border: none"
        src="http://google.com"></iframe>
  </body>
</html>
```

* open `metadata.json`

Replace content with:

```
{
    "uuid": "webkit-web@cinnamon.org",
    "name": "Website",
    "description": "Display a website"
}
```

## 04: Apply settings
Save everything and head to: `Start Menu > Preferences > Screensaver`
(or just Screensaver into the search)

It should look like this:

![grafana](/images/posts/04/screenshot1.png)

***Note: If you cannot enable auth.anonymous for your grafana instance, you have
 to figure out how to login automatically. This should be rather easy because it
 is possible to just include jQuery and use JavaScript.***
