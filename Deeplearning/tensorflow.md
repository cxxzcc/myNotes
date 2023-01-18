特点
1. 可移植性
2. 多语言支持 python/c++
3. 高度的灵活性与效率

TensorFlow是一个采用数据流图（data flow graphs),用于数值计算的开源软件库能够灵活进行组装图，执行图。随着开发的进展，Tensorflow的效率不段在提高



![image.png](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/20230118160524.png)








图默认已经注册，
一组表示tf.Operation计算单位的对象和tf.Tensor表示操作之间流动的数据单元的对象

  

获取调用：
1. tf.get_default_graph()
2. op、sess或者tensor的graph属性