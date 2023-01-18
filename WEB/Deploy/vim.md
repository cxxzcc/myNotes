![image.png](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/20230118163635.png)


查询软连接命令
```shell
#查找 vi 的运行文件 
which vi 
ls -l /usr/bin/vi 
ls -l /etc/alternatives/vi 
ls -l /usr/bin/vim.basic 

# 查找 vim 的运行文件 
which vim 
ls -l /usr/bin/vim 
ls -l /etc/alternatives/vim 
ls -l /usr/bin/vim.basic

vi 文件名 存在打开 不存在新建 
vi 文件名 + 行数 
不指定行号，会直接定位到文件末尾 如果 vi 异常退出，在磁盘上可能会保存有 交换文件 下次再使用 vi 编辑该文件时，会看到以下屏幕信息，按下字母 d 可以 删除交换文件 即可

```

# 三种工作模式

## 命令模式

打开文件首先进入命令模式，是使用 `vi` 的入口 
通过命令对文件进行常规的编辑操作，例如：定位、翻页、复制、粘贴、删除…… 
在其他图形编辑器下，通过快捷键或者鼠标 实现的操作，都在命令模式下实现

## 末行模式 
执行 保存、退出 等操作 

要退出 `vi` 返回到控制台，需要在末行模式下输入命令 
末行模式是 `vi` 的出口 
```vim
w  保存 
q  退出，如果没有保存，不允许退出 
q! 强行退出，不保存退出 
wq 保存并退出 
x  保存并退出
```

## 编辑模式

