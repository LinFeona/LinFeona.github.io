# Unity 2D帧动画实现


- 播放器 公开出去了一个设置状态的接口供给玩家使用
- 播放器需要定义玩家状态 每个状态对应一个动画
- 需要定义两个状态 当前状态 和新的状态
- 监听当前状态和新的状态 检测是否相同 若不相同即状态需要改变
- 公开出去的切换状态接口 先检测传递过来的状态和当前状态是相同 相同则返回 因为现在正在播放这个动画的状态 若不相同 即把新的状态 设为传递过来的状态
- 监听状态发生改变  就把当前状态设为新的状态 即启动协同程序 循环切换图片 
- 一定要定义一个协同程序的字段 用来存储当前播放动画的协成
- 每次需要播放新的动画的时候都需要把上一个播放动画协成给停止掉

- 原则是 状态改变 动画改变 传递过来是同样的动画 就不处理 只处理不同的动画


- 注意：玩家传递过来的状态 一定要是帧动画里面实现的状态




## code

- 帧动画脚本

```C#

//帧动画

// 全由状态实现 每个状态都有一套动画 玩家层只需要修改当前状态即可改变动画

//绑定在动画玩家身上



using System;
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class FrameAnimation : MonoBehaviour
{

    public Dictionary<AllState, Sprite[]> Anim = new Dictionary<AllState, Sprite[]>();

	//播放动画使用的协同函数 需要切换动画的时候需要把上一次停止掉
    IEnumerator PlayAnim = null;

    //序列化的结构体
    [Serializable]
    public struct Info
    {
        public AllState key;
        public Sprite[] value;
    }
    //公开在编辑器中的结构体数组
    public Info[] info;

	//玩家的2D渲染器
    SpriteRenderer player;

 //所有状态
    public enum AllState
    {
        IDLE_UP, //前 站
        IDLE_DOWN, //后 站
        IDLE_LEFT, //左 站
        IDLE_RIGHT, //右 站
        NORTHEAST_RUN, //东北 跑
        SOUTHEAST_RUN, //东南 跑 
        SOUTHWEST_RUN, //西南 跑
        NORTHWEST_RUN, //西北 跑
        UP_RUN, //前跑
        DOWN_RUN, //后跑
        LEFT_RUN, //左跑
        RIGHT_RUN,//右跑
        IDLE, //啥也不干
    }

    //当前状态
    public AllState currentState = AllState.IDLE;
	
	//新的状态
    public AllState newState = AllState.IDLE_UP;



    private void Start()
    {
         player = gameObject.GetComponent<SpriteRenderer>();
        
        foreach (var item in info)
        {
            Anim.Add(item.key, item.value);
            
          
        }
		
		//开局需要玩家是idle动画
        SetState(newState);
    }


   




    //设置状态
    public void SetState(AllState state)
    {

        //已经在播放新的状态
        if(state== currentState)
        {
            
            return;
        }

        newState = state;

    }

	//监听状态
    private void Update()
    {
        //状态有变动
        if (currentState != newState)
        {
           
            currentState = newState;
            play();

        }


    }


    void play()
    {

        //player.AddClip(Anim[currentState], currentState.ToString());
        //Debug.Log(currentState.ToString());
        //player.clip = Anim[currentState];
        //player.Play(player.GetClip(currentState.ToString()).name);
        //Debug.Log(player.GetClip(currentState.ToString()).name);


        if (PlayAnim != null)
        {
            StopCoroutine(PlayAnim);
        }

         PlayAnim = playAnim(Anim[currentState]);

        StartCoroutine(PlayAnim);

    }


    
    IEnumerator playAnim(Sprite[] anim)
    {

        while (true)
        {
            for (int i = 0; i < anim.Length; i++)
            {

                player.sprite = anim[i];
                yield return new WaitForSeconds(0.1f);
            }

            yield return null;
        }


        

    }


}

```


- 玩家脚本 
- 玩家脚本无需实现成这样 我写成这样是因为我还要考虑到做其他内容
- 遥感的可以去阅读我的遥感实现文章

```C#

using System;
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class Player : MonoBehaviour
{
   
  

    public float sppes = 5; //移动速度

	//遥感对象 范围[0,1  0,-1 1,0 -1,0 0.5,0.5 -0.5,-0.5 0.5,-0.5 -0.5,0.5]
    public EEasyTouch eEasyTouch; 

    public FrameAnimation frame;//玩家的帧动画播放器
 

  	//Idle 动画需要四个方向 所以这里定义一个四个方向的IDLE 枚举结构使用的帧动画的枚举
    public FrameAnimation.AllState Currentdirection = FrameAnimation.AllState.IDLE_UP;



    private void Start()
    {
        //eEasyTouch = UI.GetComponent<EEasyTouch>();

        frame = GetComponent<FrameAnimation>();
    }
    // Update is called once per frame
    void Update()
    {
		//动画状态控制
        direction();
		//移动
        transform.Translate(new Vector3(eEasyTouch.Horizontal, eEasyTouch.Vertical, 0) * sppes * Time.deltaTime);


    }

     void direction()
     {

        Debug.Log(eEasyTouch.name);
       
        if (eEasyTouch.state == EEasyTouch.DIRECTION.UP)
        {
            frame.SetState(FrameAnimation.AllState.UP_RUN);
            UP();
        }
        else if (eEasyTouch.state == EEasyTouch.DIRECTION.DOWN)
        {
            frame.SetState(FrameAnimation.AllState.DOWN_RUN);
            down();
        }
        else if (eEasyTouch.state == EEasyTouch.DIRECTION.LEFT)
        {
            frame.SetState(FrameAnimation.AllState.LEFT_RUN);
            Left();
        }
        else if (eEasyTouch.state == EEasyTouch.DIRECTION.RIGHT)
        {
            frame.SetState(FrameAnimation.AllState.RIGHT_RUN);
            right();
        }
        else if (eEasyTouch.state == EEasyTouch.DIRECTION.NORTHEAST)
        {
            frame.SetState(FrameAnimation.AllState.UP_RUN);
            northeast();
        }
        else if (eEasyTouch.state == EEasyTouch.DIRECTION.SOUTHEAST)
        {

            frame.SetState(FrameAnimation.AllState.DOWN_RUN);
            southeast();
        }
        else if (eEasyTouch.state == EEasyTouch.DIRECTION.SOUTHWEST)
        {
            frame.SetState(FrameAnimation.AllState.DOWN_RUN);
            southwest();
        }
        else if (eEasyTouch.state == EEasyTouch.DIRECTION.NORTHWEST)
        {
            frame.SetState(FrameAnimation.AllState.UP_RUN);
            northwest();
        }
        else if (eEasyTouch.state == EEasyTouch.DIRECTION.NULL)
        {
            frame.SetState(Currentdirection);
        }
    }


	//下面正在处理 四个方向的IDLE动画
	
    //上
    void UP()
    {
        //transform.Translate(Vector3.up * sppes * Time.deltaTime);

        Debug.Log("UP");
        Currentdirection = FrameAnimation.AllState.IDLE_UP;


    }

    //下
    void down()
    {
        //transform.Translate(Vector3.down * sppes * Time.deltaTime);

        Currentdirection = FrameAnimation.AllState.IDLE_DOWN;
    }
    //左
    void Left()
    {

        //transform.Translate(Vector3.left * sppes * Time.deltaTime);
        Currentdirection = FrameAnimation.AllState.IDLE_LEFT;
    }

    //右
    void right()
    {

        //transform.Translate(Vector3.right * sppes * Time.deltaTime);
        Currentdirection = FrameAnimation.AllState.IDLE_RIGHT;
    }


    //东北
    void northeast()
    {

        //transform.Translate(Vector3.up * sppes * Time.deltaTime);
        Currentdirection = FrameAnimation.AllState.IDLE_UP;
    }

    //东南
    void southeast()
    {

        //transform.Translate(Vector3.down * sppes * Time.deltaTime);
        Currentdirection = FrameAnimation.AllState.IDLE_DOWN;
    }
    //西南
    void southwest()
    {

        //transform.Translate(Vector3.left * sppes * Time.deltaTime);
        Currentdirection = FrameAnimation.AllState.IDLE_LEFT;
    }

    //西北
    void northwest()
    {

        //transform.Translate(Vector3.right * sppes * Time.deltaTime);
        Currentdirection = FrameAnimation.AllState.IDLE_UP;
    }


}




```
