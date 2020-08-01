```javascript
function lightboxReady() {
  var promise = new Promise(function(resolve,reject) {
    var inc = 40;
    var lightboxListener = function(interval) {
      var lightbox = document.getElementsByClassName(LIGHTBOX_SELECTOR);
      if (lightbox.length != 0) {
        resolve('true');
```
