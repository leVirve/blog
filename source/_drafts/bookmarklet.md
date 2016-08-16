title: bookmarklet
tags:
---

```javascript
var link = document.URL;
var result = link.match(/\/d\/(.+?)\//);
var url = 'https://drive.google.com/uc?export=download&id=' + result[1];
location.href = url;
```