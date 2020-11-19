# Unity   序列化 Dictionary 以及 常用的特性 

- 在Unity中Dictionary类型不能直接序列化到编辑器上，但可以利用结构体来模拟对泛型的序列化
-  [Serializable] 特性 将一个类型序列化到编辑器中
-  思路：序列化一个结构体 结构体属性分别对应 Dictionary的key value 最后定义结构体数组绑定到编辑器上，在初始化的时候再将结构体的数据赋值给Dictionary

> 代码

``` c#

       public Dictionary<string, string> Dic = new Dictionary<string, string>();
       
       //序列化的结构体
        [Serializable]
        public struct Info
        {
           public  string key;
           public string value;
        }
        //公开在编辑器中的结构体数组
        public Info[] info;
        
        //初始化
        public void Init()
        {
             foreach (var item in info)
            {
                Dic.Add(item.key, item.value);
            }
        }
```

- 常用特性

| 特性 |  作用  |
|-|-|
|Serializable | 使自定义的结构体等等类型序列化 |
|SerializeField|使非public类型(如private)也能序列化 |
|HideInInspector|将原本显示在面板上的序列化值隐藏起来 |
|ContextMenu| 将一个方法绑定到编辑器中|
|RequireComponent | 必须要有相应的组件([RequireComponent(typeof (Rigidbody))]) |
|NonSerialized |不被序列化|

- 常用特性借鉴于[unity中的c# Attribute特性的使用记录](http://blog.sina.com.cn/s/blog_6ba9a5300102wdfi.html)

$\color{#FFB6C1}{Feona 编辑}$
