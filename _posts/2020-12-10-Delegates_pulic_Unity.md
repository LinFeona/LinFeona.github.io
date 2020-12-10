
# unity 序列化一个事件到编辑器

## code

- 先序列化一个类 

```C#
[Serializable]
public class OnClick : UnityEvent<GameObject> { }
```

- 再脚本中 使用类当类型 公开到编辑器中 

```C#
    public OnClick Click;
```

- 调用

```C#
Click.Invoke(GameObject);
```

- 参数

```C#
序列化的泛型类型 就是参数
```
