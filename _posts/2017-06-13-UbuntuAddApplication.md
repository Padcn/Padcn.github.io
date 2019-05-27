---
title: Ubuntu增加程序到系统菜单
date: 2017-06-13
tags: linux
---
# Ubuntu增加程序到系统菜单
Eclipse装完之后发现在系统菜单中没有找到，每次都要进目录找比较麻烦，所以找了下方法。

在/usr/share/applications目录下都是一些程序的.desktop文件，有了这个文件就可以在系统菜单中调用程序了。

找了个类似的程序的.desktop来参考

如Atom的：

@Atom.desktop
```bash
[Desktop Entry]
Name=Atom
Comment=A hackable text editor for the 21st Century.
GenericName=Text Editor
Exec=/opt/atom/atom %F
Icon=atom
Type=Application
StartupNotify=true
Categories=GNOME;GTK;Utility;TextEditor;Development;
MimeType=text/plain;
X-Desktop-File-Install-Version=0.22
```
可以看到这里定义了名字，描述，执行程序位置，图标以及分类。分类中类似关键字，会在对于系统菜单中显示。

所以我们要创建的eclipse.desktop内容为：
```bash
[Desktop Entry]
Name=Eclipse
Comment=c project manage software
Exec=/home/pad/Software/eclipse/java-neon/eclipse/eclipse
Icon=/home/pad/Software/eclipse/java-neon/eclipse/icon.xpm
Terminal=false
Type=Application
Categories=GNOME;GTK;Development;Eclipse
```
保存后系统菜单中就有了。