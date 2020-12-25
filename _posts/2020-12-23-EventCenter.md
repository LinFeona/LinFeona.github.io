# 事件中心的处理

- 实现原则  在程序任何一个地方发送一个消息 在程序任何一个地方都能监听得到该消息

```C#

using System.Collections;
using System;
using System.Collections.Generic;

public class EventCenter{

    //委托：消息传递
    public delegate void event_delegate(object data);

    //所有的消息
    public static Dictionary<string, msg_delegate> dic_event = new Dictionary<string, msg_delegate>();

	
	//add_list
    public static void add_event_msg_list(string msg_type, event_delegate handler)
    {
        if (!dic_event.ContainsKey(msg_type))
        {
            dic_event.Add(msg_type, null);
        }
        dic_event[msg_type] += handler;
    }

	
	
	
	//移除一个监听者
    public static void delete_event_list(string msg_type, event_delegate handele)
    {
        if (dic_event.ContainsKey(msg_type))
        {
            dic_event[msg_type] -= handele;
        }

    }


	
	//发送消息
    public static void send_msg(string msg_type,object data)
    {
        event_delegate tmp_event;                     

        if (dic_event.TryGetValue(messageType, out tmp_event))
        {
            if (tmp_event != null)
            {
                
                tmp_event(data);
            }
        }
    }


	//移除所有监听消
	public static void delete_event_all()
    {
        if (dic_event != null)
        {
            dic_event.Clear();
        }
    }

}



```