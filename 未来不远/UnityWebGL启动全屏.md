# 1 修改index.html文件

```html
<div id="unity-container" style="unity-desktop">
      <canvas id="unity-canvas" width=960 height=600></canvas>
```

修改为：

```html
<div id="unity-container" style="width: 100%;height:100%">
      <canvas id="unity-canvas" width=auto height=auto></canvas>
```

---

```html
  if (/iPhone|iPad|iPod|Android/i.test(navigator.userAgent)) {
        container.className = "unity-mobile";
		config.devicePixelRatio = 1;
      } else {
        canvas.style.width = "960px";
        canvas.style.height = "600px";
      }
```

修改为：

```html
  if (/iPhone|iPad|iPod|Android/i.test(navigator.userAgent)) {
       
        var meta = document.createElement('meta');
            meta.name = 'viewport';
            meta.content = 'width=device-width, height=device-height, initial-scale=1.0, user-scalable=no, shrink-to-fit=yes';
            document.getElementsByTagName('head')[0].appendChild(meta);
            container.className = "unity-mobile";
          
            canvas.style.width = window.innerWidth + 'px';
            canvas.style.height = window.innerHeight + 'px';
 
 
        unityShowBanner('暂不支持移动端...');
      } else {
        canvas.style.width = "100%";
        canvas.style.height = "100%";
      }
```

# 2 修改style.css文件

去掉右侧滑动条，在style.css下面添加以下内容：

```css
html,body{width:100%;height:100%;margin:0;padding:0;overflow:hidden;}
.webgl-content{width: 100%; height: 100%;}
.unityContainer{width: 100%; height: 100%;}
```

