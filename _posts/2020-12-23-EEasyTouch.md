# Unity遥感的实现

- 注意 范围设为105是因为 需要控制遥感值的控制 可以把105看成100  这个多出来的5是为了四舍五入
- 因为需要把值控制在 -1->1 之间 设为100 四舍五入就不能得到1
- 需要 一个遥感的背景圆
- 需要一个遥感的圆 将脚本放在遥感的圆上 把遥感的圆放在背景下

```C#

using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.EventSystems;
public class EEasyTouch : MonoBehaviour, IDragHandler, IEndDragHandler
{
    //图标移动最大半径
    public float maxRadius = 105;
    public float DivisorValue = 100;
    public float deviation = 0.3f;
    //初始化背景图标位置
    private Vector2 moveBackPos;

    //hor,ver的属性访问器
    private float horizontal = 0;
    private float vertical = 0;

    public float Horizontal
    {
        get { return horizontal; }
    }

    public float Vertical
    {
        get { return vertical; }
    }


    public  enum DIRECTION
    {
        UP, //前
        DOWN, //后
        LEFT, //左
        RIGHT, //右
        NORTHEAST, //东北
        SOUTHEAST, //东南
        NORTHWEST, //西北
        SOUTHWEST, //西南
        NULL, //空
    }

    private DIRECTION STATE = DIRECTION.NULL;

    public DIRECTION state
    {
        get { return STATE; }
    }

    // Use this for initialization
    void Start()
    {
        //初始化背景图标位置
        moveBackPos = transform.parent.transform.position;
    }

    // Update is called once per frame
    void Update()
    {
        //horizontal = (int)(transform.localPosition.x/ 100);
        //vertical = (int)(transform.localPosition.y/ 100);
        //Debug.Log(horizontal + "  " + vertical);
        if((transform.localPosition.x / DivisorValue) > deviation && (transform.localPosition.y / DivisorValue) > deviation)
        {
            //Debug.Log("东北方向");
            STATE = DIRECTION.NORTHEAST;
            horizontal = 0.5f;
            vertical = 0.5f;
        }
        else if ((transform.localPosition.x / DivisorValue) < -deviation && (transform.localPosition.y / DivisorValue) > deviation)
        {
            //Debug.Log("西北方向");
            STATE = DIRECTION.NORTHWEST;
            horizontal = -0.5f;
            vertical = 0.5f;
        }
        else if ((transform.localPosition.x / DivisorValue) < -deviation && (transform.localPosition.y / DivisorValue) < -deviation)
        {
            //Debug.Log("西南方向");
            STATE = DIRECTION.SOUTHWEST;
            horizontal = -0.5f;
            vertical = -0.5f;
        }
        else if ((transform.localPosition.x / DivisorValue) > deviation && (transform.localPosition.y / DivisorValue) < -deviation)
        {
            //Debug.Log("东南方向");
            STATE = DIRECTION.SOUTHEAST;
            horizontal = 0.5f;
            vertical = -0.5f;
        }
        else if((int)(transform.localPosition.x / DivisorValue) == 0 && (int)(transform.localPosition.y / DivisorValue) ==1)
        {
            //Debug.Log("正前");
            STATE = DIRECTION.UP;
            horizontal =0f;
            vertical = 1f;
        }
        else if ((int)(transform.localPosition.x / DivisorValue) == 0 && (int)(transform.localPosition.y / DivisorValue) == -1)
        {
            //Debug.Log("正后");
            STATE = DIRECTION.DOWN;
            horizontal = 0f;
            vertical = -1f;
        }
        else if ((int)(transform.localPosition.x / DivisorValue) == 1 && (int)(transform.localPosition.y / DivisorValue) == 0)
        {
            //Debug.Log("正右");
            STATE = DIRECTION.RIGHT;
            horizontal = 1;
            vertical = 0f;
        }
        else if ((int)(transform.localPosition.x / DivisorValue) == -1 && (int)(transform.localPosition.y / DivisorValue) == 0)
        {
            //Debug.Log("正左");
            STATE = DIRECTION.LEFT;
            horizontal = -1f;
            vertical = 0f;
        }
        else
        {
            STATE = DIRECTION.NULL;
            horizontal = 0f;
            vertical = 0f;
        }
        //Debug.Log(horizontal + "  " + vertical);
    }

    /// <summary>
    /// 当鼠标开始拖拽时
    /// </summary>
    /// <param name="eventData"></param>
    public void OnDrag(PointerEventData eventData)
    {
        //获取鼠标位置与初始位置之间的向量
        Vector2 oppsitionVec = eventData.position - moveBackPos;
        //获取向量的长度
        float distance = Vector3.Magnitude(oppsitionVec);

        //最小值与最大值之间取半径
        float radius = Mathf.Clamp(distance, 0, maxRadius);
        //限制半径长度
        transform.position = moveBackPos + oppsitionVec.normalized * radius;

    }

    /// <summary>
    /// 当鼠标停止拖拽时
    /// </summary>
    /// <param name="eventData"></param>
    public void OnEndDrag(PointerEventData eventData)
    {
        transform.position = moveBackPos;
        transform.localPosition = Vector3.zero;
    }
}
```