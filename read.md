
--------

### Jumping around

按键|效果
:-: |:-:
wget| 从命令行轻松地从各种不同协议下载文件
> 使用Wget非常简单，需要获取要下载的URL，并在命令后添加它，比如下载云网牛站的logo.png：
> 
> wget https://ywnz.com/images/logo.png
> 
> 它还可以通过在文件列表中指定多个文件来下载它们，例如，首先使用touch创建一个新的wget下载文件：
> 
> touch ~/Downloads/wget-downloads.txt
> 
> 然后打开文件管理器，双击文本文件，将要下载的内容的URL添加到文本文件中并保存，从那里，可以使用“i”选项：
> 
> wget -i ~/Downmloads/wget-downloads.txt
