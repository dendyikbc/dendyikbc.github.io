---
layout: post
title: 'TensorFlow模型保存 测试篇'
date: 2019-04-19
author: Dave
cover: 'http://on2171g4d.bkt.clouddn.com/jekyll-banner.png'
tags: TensorFlow
---

![](https://raw.githubusercontent.com/dendyikbc/PicGoBed/master/imgTF-CNNmodelsaver0.jpg)


博客迁移，原文链接[TensorFlow模型保存 测试篇](https://blog.csdn.net/ddpiccolo/article/details/89392108)

### 模型保存——saver
```python
  model_path='E:/model/kdd/model.ckpt' ##绝对路径
  model_path='scnn_model/model.ckpt'##相对路径
  tf.train.Saver.save(sess,model_path)
  saver=tf.train.Saver()
```
### 调用模型进行预测
可以先看模型保存了哪些文件
![](https://raw.githubusercontent.com/dendyikbc/PicGoBed/master/imgTensorFlow-model-saver-and-test-.jpg)
>  .meta文件保存了当前图结构
> 
>  .data文件保存了当前参数名和值
> 
>  .index文件保存了辅助索引信息
> 
> .data文件可以查询到参数名和参数值，使用下面的命令可以查询保存在文件中的全部变量{名：值}对，

```tf.train.get_checkpoint_state( )  ```函数可以通过检查点文件锁定最新的模型
```tf.train.import_meta_graph( )```函数给出```model.ckpt-n.meta```的路径后会加载图结构，并返回```saver```对象
```saver.restore(sess, model.model_checkpoint_path)```用于模型的恢复
```python
ckpt = tf.train.get_checkpoint_state('ckpt-5/') # 通过检查点文件锁定最新的模型
saver = tf.train.import_meta_graph(ckpt.model_checkpoint_path + '.meta')  # 载入图结构，保存在.meta文件中
saver.restore(sess, ckpt.model_checkpoint_path)
```
```tf.train.Saver```函数会返回加载默认图的```saver```对象，```saver```对象初始化时可以指定变量映射方式，根据名字映射变量
下面是最近一个完整的加载过程
```python
# 加载模型
saver = tf.train.Saver()
    with tf.Session() as sess:
        ckpt = tf.train.get_checkpoint_state('ckpt-5/')  # 通过检查点文件锁定最新的模型
        saver = tf.train.import_meta_graph(ckpt.model_checkpoint_path + '.meta')  # 载入图结构，保存在.meta文件中
        if ckpt and ckpt.model_checkpoint_path:
            global_step = ckpt.model_checkpoint_path.split('/')[-1].split('-')[-1]
            saver.restore(sess, ckpt.model_checkpoint_path)
            print('CNN Model Loading Success')
        else:
            print('No Checkpoint')
            #ii=0
        graph = tf.get_default_graph()
        xs = graph.get_tensor_by_name("inputs/pic_data:0")
        keep_prob = graph.get_tensor_by_name("inputs/keep_prob:0")
        logits = graph.get_tensor_by_name("prediction_eval:0")
        prediction = sess.run(logits, feed_dict={xs: feature,keep_prob: 1.0})  ##?
        print('Prediction Matrix of Test Data Set:')
        print(prediction)
        max_index = np.argmax(prediction, 1)
        print('Prediction Vector of Test Data Set:')
        print(max_index)
        m0 = max_index
        print('Size of Test Data Set:   ',m0.shape)

```
上述过程，调用了模型中```graph```中的```tensor```，注意大小要保持一致
```python
xs = graph.get_tensor_by_name("inputs/pic_data:0")#我的命名空间input中有一个名为pic_data的tensor
```
如果不知道原图中有哪些```tensor```，可以加载模型后按照下面语句去查看
```python
#获得几乎所有的operations相关的tensor
ops = [o for o in sess.graph.get_operations()]
    for o in ops:
        print(o.name)
```
最后奉上
#### **模型的干货加载**
```python
# 连同图结构一同加载
ckpt = tf.train.get_checkpoint_state('./model/')
saver = tf.train.import_meta_graph(ckpt.model_checkpoint_path +'.meta')
with tf.Session() as sess:
    saver.restore(sess,ckpt.model_checkpoint_path)
             
# 只加载数据，不加载图结构，可以在新图中改变batch_size等的值
# 不过需要注意，Saver对象实例化之前需要定义好新的图结构，否则会报错
saver = tf.train.Saver()
with tf.Session() as sess:
    ckpt = tf.train.get_checkpoint_state('./model/')
    saver.restore(sess,ckpt.model_checkpoint_path)

```
