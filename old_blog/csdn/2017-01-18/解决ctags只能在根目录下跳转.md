原文链接:http://blog.chinaunix.net/uid-26557245-id-3594974.html
 <!--more-->  
使用 ctags -R,再用vim打开其目录下的源文件时，出现了

cstag:tag not found

但是已经在顶层目录下已经有了tags;

这是因为源文件在当前目录下没有找到tags文件，

解决办法是在在vim的配置文件~/vimrc添加set tags=tags;

使如果源文件在当前文件夹下没有找到tags,可以到它的上层目录下继续寻找； 
