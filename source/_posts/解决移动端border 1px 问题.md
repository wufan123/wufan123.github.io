---
title: 解决移动端border 1px 问题
date: 2022-03-22 15:45:22
categories: 
    - [前端]
tags: [css]
---
由于某些机型分辨率过高,会导致1px变成2+px的问题,样式解决的话，通过媒体选择器和`transform`方式,参考代码如下

```css
.list li:after{
  content:'';
  float:left;
  width:100%;
  border-bottom: #eeeeee solid 1px;
  height:1px;
}
@media only screen and (-webkit-min-device-pixel-ratio: 2.0),
only screen and (min--moz-device-pixel-ratio: 2.0),
only screen and (-o-min-device-pixel-ratio: 200/100),
only screen and (min-device-pixel-ratio: 2.0) {
  .list li:after{
    -webkit-transform: scaleY(0.5);
    transform: scaleY(0.5);
  }
}
@media only screen and (-webkit-min-device-pixel-ratio: 2.5),
only screen and (min--moz-device-pixel-ratio: 2.5),
only screen and (-o-min-device-pixel-ratio: 250/100),
only screen and (min-device-pixel-ratio: 2.5) {
  .list li:after{
    -webkit-transform: scaleY(0.33333334);
    transform: scaleY(0.33333334);
  }
}
```