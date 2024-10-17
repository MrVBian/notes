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

# 3 nginx 部署

```nginx
server {
    listen       10000;
    server_name  172.24.97.3;

    root /home/zme/project/webgl/zmebot;

    location / {
        index  index.html;
        try_files $uri $uri/ index.html;
        default_type application/wasm;
    }

    # On-disk Brotli-precompressed data files should be served with compression enabled:
    location ~ .+\.(data|symbols\.json)\.br$ {
        # Because this file is already pre-compressed on disk, disable the on-demand compression on it.
        # Otherwise nginx would attempt double compression.
        gzip off;
        add_header Content-Encoding br;
        default_type application/octet-stream;
    }

    # On-disk Brotli-precompressed JavaScript code files:
    location ~ .+\.js\.br$ {
        gzip off; # Do not attempt dynamic gzip compression on an already compressed file
        add_header Content-Encoding br;
        default_type application/javascript;
    }

    # On-disk Brotli-precompressed WebAssembly files:
    location ~ .+\.wasm\.br$ {
        gzip off; # Do not attempt dynamic gzip compression on an already compressed file
        add_header Content-Encoding br;
        # Enable streaming WebAssembly compilation by specifying the correct MIME type for
        # Wasm files.
        default_type application/wasm;
    }

    # On-disk gzip-precompressed data files should be served with compression enabled:
    location ~ .+\.(data|symbols\.json)\.gz$ {
        gzip off; # Do not attempt dynamic gzip compression on an already compressed file
        add_header Content-Encoding gzip;
        default_type application/gzip;
    }

    # On-disk gzip-precompressed JavaScript code files:
    location ~ .+\.js\.gz$ {
        gzip off; # Do not attempt dynamic gzip compression on an already compressed file
        add_header Content-Encoding gzip; # The correct MIME type here would be application/octet-stream, but due to Safari bug https://bugs.webkit.org/show_bug.cgi?id=247421, it's preferable to use MIME Type application/gzip instead.
        default_type application/javascript;
    }

    # On-disk gzip-precompressed WebAssembly files:
    location ~ .+\.wasm\.gz$ {
        gzip off; # Do not attempt dynamic gzip compression on an already compressed file
        add_header Content-Encoding gzip;
        # Enable streaming WebAssembly compilation by specifying the correct MIME type for
        # Wasm files.
        default_type application/wasm;
    }
}
```

