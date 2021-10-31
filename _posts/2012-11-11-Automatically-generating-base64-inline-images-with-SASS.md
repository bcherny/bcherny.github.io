---
github: https://github.com/bcherny/SASS-Base64
layout: post
title: Automatically Generating Base64 Inline Images With SASS
date: 2012-11-11
categories: CSS SASS Ruby
---

Base64 encoding has been around for years, and when applied to images (among other data) in the form of Data URI's, is a crucial tool in the performance geek's arsenal.

## Why Base64?

While base64 encoding increases the byte size of encoded content by around 1/3^[In contrast with UTF-8, which stores each character as 8 bits (an "octet"), Base64 encoding stores each character as 6 bits. Thus a 24-bit binary sequence can encode either three UTF-8 characters or four Base64 encoded characters. So, a Base64-encoded string will take up 8/6 = 4/3 &#x2248; 133% of its UTF-8 counterpart.], it has the potential to dramatically cut down on HTTP requests and latency per resource. Since many browsers allow only a few connections per host (iOS allows 4-6, IE8/9 allows 6, and IE6/7 allows just 2, as per the [HTTP 1.1 Spec](http://www.w3.org/Protocols/rfc2616/rfc2616-sec8.html)), inline images are often a good alternative to many small external images, which would be forced to be fetched in sequence rather than pipelined. For relatively small media inlined in CSS, another benefit comes in the form of no more flash of unstyled content: inline images will load at the same time as the containing CSS file.

## What?

So instead of:

```css
element {
  background-image: url('path/to/image.png');
}
```

your code will look something like:

```css
element {
  background-image: url(data:image/png;base64,TWFuIGlzIGRpc3Rpbmd1aXNoZWQsIG5vdCBvbmx5
IGJ5IGhpcyByZWFzb24sIGJ1dCBieSB0aGlzIHNpbmd1bGFyIHBhc3Npb24gZnJvbSBvdGhlciBhbmltYWxzLCB3
aGljaCBpcyBhIGx1c3Qgb2YgdGhlIG1pbmQsIHRoYXQgYnkgYSBwZXJzZXZlcmFuY2Ugb2YgZGVsaWdodCBpbiB0
aGUgY29udGludWVkIGFuZCBpbmRlZmF0aWdhYmxlIGdlbmVyYXRpb24gb2Yga25vd2xlZGdlLCBleGNlZWRzIHRo
ZSBzaG9ydCB2ZWhlbWVuY2Ugb2YgYW55IGNhcm5hbCBwbGVhc3VyZS4=);
}
```

## Browser support

Base64-encoded Data URIs are well supported for a variety of image formats in every major browser back to Internet Explorer 8. They still work in IE6/7, but need to be formatted a bit differently (see Jon Raasch's [excellent writeup](http://jonraasch.com/blog/css-data-uris-in-all-browsers)). In fact, if you don't need to support Android <3, iOS <3, or IE <9, you can even embed SVG right in your CSS^[IE9 requires your SVG elements to be Base64 or URI-encoded, in compliance with the [IETF Spec](http://www.ietf.org/rfc/rfc2397.txt).].

## The problem

Images are pretty easy to maintain. Sprite sheets are more difficult. Base64-encoded sprites are, technically speaking, a pain in the ass. [Tom Najdek](http://doppnet.com/2012/07/Ninja-SVG-sprites), among others, tackled this problem with an automated pipeline. My (minor) issue with Tom's technique is that its being written in Python requires an extra build step, something that [Compass](http://compass.handlino.com) solved with its automated SASS-based sprite creation. Unfortunately, Compass' GPL license made it unviable for the project I was working on.

## My solution

So I decided to put together a simple, self-contained SASS function:

```ruby
require "base64"
require "sass"

module Sass::Script::Functions
  def url64(image)
    assert_type image, :String

    # compute file/path/extension
    base_path = "../../.."
    root = File.expand_path(base_path, __FILE__)
    path = image.to_s[4,image.to_s.length-5]
    fullpath = File.expand_path(path, root)
    ext = File.extname(path)

    # base64 encode the file
    file = File.open(fullpath, "rb")
    text = file.read
    file.close
    text_b64 = Base64.encode64(text).gsub(/\r/,"").gsub(/\n/,"")
    contents = "url(data:image/" + ext[1,ext.length-1] + ";base64," + text_b64 + ")"

    Sass::Script::String.new(contents)

  end
  declare :url64, :args => [:string]
end
```

Basically you're defining a new SASS Function called `url64`, which loads the file referenced by the string passed into it, Base64 encodes it, strips out line endings, properly formats it as per the Data-URI Spec, and finally returns it back as a string.

## Installation

1. Copy the above code into a blank file and give it a `.rb` extension. I dropped the file into my SASS directory, but you can put it anywhere.
2. Change the `base_path` variable to the relative path from your new SASS module to your images folder.

## Usage

Usage couldn't be any easier. When starting SASS on the command line, simply invoke your new module:

```bash
sass --watch sass:css -r ./sass/functions/url64.rb
```

And that's it! The versatility of this SASS-based solution is why I prefer it to alternative systems - say you have an image in your CSS:

```css
element {
  background-image: url(path/to/image.jpg);
}
```

To automatically Base64-encode it at SASS compile time, just replace `url` with `url64`:

```css
element {
  background-image: url64(path/to/image.jpg);
}
```

SASS will then automatically encode and inline your linked image for you at compile time (eg. on save if using `sass --watch`).

## Can I get automated compression with that?

Sure, if you can live with a couple of extra dependencies:

- [Smusher](https://github.com/grosser/smusher) is a ruby package for lossless GIF/JPG/PNG compression that can be installed from the command line.
- [Scour](http://www.codedread.com/scour/) is a Python script for cleaning up SVG. Install it into a folder in the same directory as your SASS module.

Then update your module with the following:

```ruby
require "base64"
require "sass"
require "smusher"

module Sass::Script::Functions
  def url64(image)
    assert_type image, :String

    # compute file/path/extension
    base_path = "../../.."
    root = File.expand_path(base_path, __FILE__)
    path = image.to_s[4,image.to_s.length-5]
    fullpath = File.expand_path(path, root)
    absname = File.expand_path(fullpath)
    ext = File.extname(path)

    # optimize image
    if ext == "gif" || ext == "jpg" || ext == "png"
      Smusher::optimize_image(absname)
    elsif ext == "svg"
      `python scour/scour.py -i ` + absname + ` -o ` + absname
    end

    # base64 encode the file
    file = File.open(fullpath, "rb")
    text = file.read
    file.close
    text_b64 = Base64.encode64(text).gsub(/\r/,"").gsub(/\n/,"")
    contents = "url(data:image/" + ext[1,ext.length-1] + ";base64," + text_b64 + ")"

    Sass::Script::String.new(contents)

  end
  declare :url64, :args => [:string]
end
```

This pipeline really streamlined my CSS workflow, I hope you find it useful too! Be sure to grab the latest files from the github repo [here](https://github.com/bcherny/SASS-Base64).
