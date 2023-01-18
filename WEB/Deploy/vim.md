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






