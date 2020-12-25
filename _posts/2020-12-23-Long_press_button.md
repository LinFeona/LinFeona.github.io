# Unity 实现长按


```C#

using UnityEngine;
using System.Collections;
using UnityEngine.EventSystems;
public class EventTriggerListener : UnityEngine.EventSystems.EventTrigger
{
    public delegate void VoidDelegate(GameObject go);
    public delegate void BoolDelegate(GameObject go, bool state);
    public delegate void FloatDelegate(GameObject go, float delta);
    public delegate void VectorDelegate(GameObject go, Vector2 delta);
    public delegate void ObjectDelegate(GameObject go, GameObject obj);
    public delegate void KeyCodeDelegate(GameObject go, KeyCode key);

    //点击
    public VoidDelegate onClick;
    //按下
    public VoidDelegate onDown;

    //进入时
    public VoidDelegate onEnter;
    //出去时
    public VoidDelegate onExit;
    //弹起
    public VoidDelegate onUp;

    public VoidDelegate onSelect;

    public VoidDelegate onUpdateSelect;

    static public EventTriggerListener Get(GameObject go)
    {
        EventTriggerListener listener = go.GetComponent<EventTriggerListener>();
        if (listener == null) listener = go.AddComponent<EventTriggerListener>();
        return listener;
    }

    static public EventTriggerListener Get(Transform transform)
    {
        EventTriggerListener listener = transform.GetComponent<EventTriggerListener>();
        if (listener == null) listener = transform.gameObject.AddComponent<EventTriggerListener>();
        return listener;
    }
    public override void OnPointerClick(PointerEventData eventData)
    {
        if (onClick != null) onClick(gameObject);
    }
    public override void OnPointerDown(PointerEventData eventData)
    {
        if (onDown != null) onDown(gameObject);
    }
    public override void OnPointerEnter(PointerEventData eventData)
    {
        if (onEnter != null) onEnter(gameObject);
    }
    public override void OnPointerExit(PointerEventData eventData)
    {
        if (onExit != null) onExit(gameObject);
    }
    public override void OnPointerUp(PointerEventData eventData)
    {
        if (onUp != null) onUp(gameObject);
    }
    public override void OnSelect(BaseEventData eventData)
    {
        if (onSelect != null) onSelect(gameObject);
    }
    public override void OnUpdateSelected(BaseEventData eventData)
    {
        if (onUpdateSelect != null) onUpdateSelect(gameObject);
    }
}

```





```C#

using UnityEngine;
using System.Collections;
using UnityEngine.UI;
using System;
using UnityEngine.Events;

public class bnt_event : MonoBehaviour
{

    private bool isUp;




    #region 事件
    [Tooltip("长按中")]
    public OnPressing onPressing;
    [Tooltip("按下")]
    public OnPressBegin onPressBegin;
    [Tooltip("长按借宿")]
    public OnEndPress onEndPress;
    [Tooltip("开始长按前")]
    public OnPressing StartonPressing;

    #endregion


    void Start()
    {
        box_itme = transform.parent.gameObject;
        //EventTriggerListener.Get(gameObject).onDown += (go) => { Debug.Log("按下！"); };
        //EventTriggerListener.Get(gameObject).onUp += (go) => { Debug.Log("抬起！"); };
        //EventTriggerListener.Get(gameObject).onSelect += (go) => { Debug.Log("选中！"); };
        //EventTriggerListener.Get(gameObject).onEnter += (go) => { Debug.Log("进入！"); };
        //EventTriggerListener.Get(gameObject).onExit += (go) => { Debug.Log("退出！"); };

        //Debug.Log(GameObject.FindGameObjectWithTag("knapsack").name);

        //knapsackHelp.GetComp<knapsack>(GameObject.FindGameObjectWithTag("knapsack")).LongPress(gameObject);
        onPressing.AddListener(knapsackHelp.GetComp<knapsack>(GameObject.FindGameObjectWithTag("knapsack")).LongPress);
        onPressBegin.AddListener(knapsackHelp.GetComp<knapsack>(GameObject.FindGameObjectWithTag("knapsack")).ClickItem);
        onEndPress.AddListener(knapsackHelp.GetComp<knapsack>(GameObject.FindGameObjectWithTag("knapsack")).BntUp);
        StartonPressing.AddListener(knapsackHelp.GetComp<knapsack>(GameObject.FindGameObjectWithTag("knapsack")).StartLongPress);
        //onPressBegin.Invoke(gameObject);


        // img = transform.Find("Image").GetComponent<Image>();
        EventTriggerListener.Get(gameObject).onDown += OnClickDown;
        EventTriggerListener.Get(gameObject).onUp += OnClickUp;

    }

    //按下
    void OnClickDown(GameObject go)
    {
        isUp = false;
        onPressBegin.Invoke(gameObject);
        StartCoroutine(grow());
    }

    //抬起
    void OnClickUp(GameObject go)
    {
        isUp = true;
        onEndPress.Invoke(gameObject);
    }



    //长按携程
    private IEnumerator grow()
    {
      
        yield return new WaitForSecondsRealtime(1);
      
        if (!isUp)
        {

            StartonPressing.Invoke(gameObject);
            while (true)
            {
           
                 if (isUp)
                    {
                        break;
                    }
                onPressing.Invoke(gameObject);
                 yield return null;
              }
        }

        yield return null;
    }

}

[Serializable]
public class OnPressing : UnityEvent<GameObject> { }

[Serializable]
public class OnEndPress : UnityEvent<GameObject> { }

[Serializable]
public class OnPressBegin : UnityEvent<GameObject> { }

```