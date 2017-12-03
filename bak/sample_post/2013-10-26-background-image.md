---
layout: post
title: 과연 한글이 온다면 어떻게 될까?!
description: "Sample post with a background image CSS override."
tags: [sample post]
image:
  background: triangular.png
---

뭐라고?! 여기에 한글이 나온다고?! 으아아아 유니코드는 지원이 되는걸까? 한번 테스트 해 보자! 이건 과연 어디까지 보이게 될지 시험해보자구!

```yaml
image:
  background: filename.png
```

This little bit of YAML makes the assumption that your background image asset is in the `/images` folder. If you place it somewhere else or are hotlinking from the web, just include the full http(s):// URL. Either way you should have a background image that is tiled.

If you want to set a background image for the entire site just add `background: filename.png` to your `_config.yml` and BOOM --- background images on every page!

<div xmlns:cc="http://creativecommons.org/ns#" xmlns:dct="http://purl.org/dc/terms/" about="http://subtlepatterns.com" class="notice">Background images from <span property="dct:title">Subtle Patterns</span> (<a rel="cc:attributionURL" property="cc:attributionName" href="http://subtlepatterns.com">Subtle Patterns</a>) / <a rel="license" href="http://creativecommons.org/licenses/by-sa/3.0/">CC BY-SA 3.0</a></div>
