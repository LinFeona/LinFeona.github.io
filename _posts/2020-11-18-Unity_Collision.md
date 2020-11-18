# Unity-------碰撞事件 触发事件
$\color{#0000FF}{*2020-11-18 Feona 编辑*}$ 
[![Build Status](https://travis-ci.org/joemccann/dillinger.svg?branch=master)](https://www.baidu.com)

- 碰撞和被碰撞的物体都需要加有 Collider 碰撞组件
- 碰撞的物体身上需带有刚体，被碰撞则不需要


> 代码块

>>　触发器事件
```C#
//触发开始
public void OnTriggerEnter(Collider collider){
    
}

//触发中
public void OnTriggerStay(Collider collider){
    
}

//触发结束
public void OnTriggerExit(Collider collider){
    
}

```

>> 碰撞器事件

``` c#
//碰撞开始
public void OnCollisionEnter(Collision collision){
    
}
//碰撞中
public void OnCollisionStay(Collision collision){
    
}

//碰撞结束
public void OnCollisionExit(Collision collision){
    
}

```
