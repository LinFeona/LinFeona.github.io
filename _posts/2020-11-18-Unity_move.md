# Unity-------键盘移动 鼠标旋转 空格跳跃


- 将脚本绑定到需要移动旋转操作的物体节点上（图0-1）
- 物体节点需要有刚体组件，因为在跳跃的时候需要物体特性才能掉下来，否则会一直停留在空中(图0-1)
- 可以将用户摄像机拖动到可移动旋转的物体节点下 从而实现 第一人称漫游

![](../../../../images/Unity_move/rig.png "0-1")



> 代码块


```C#
//移动速度
 public float speed = 4f;
 //视角灵敏度
 public float move_offes = 4f;
 //跳跃力度
 public float jump_speed = 0.1f;
 
 //旋转值的叠加量
 private float H = 0F;
 private float V = 0F;

	

void Update () {

        if (Input.GetKey(KeyCode.W))
        {
            transform.Translate(Vector3.forward*speed * Time.deltaTime);
        }else if (Input.GetKey(KeyCode.S))
        {
            transform.Translate(Vector3.back * speed * Time.deltaTime);
        }else if (Input.GetKey(KeyCode.A))
        {
            transform.Translate(Vector3.left * speed * Time.deltaTime);
        }else if (Input.GetKey(KeyCode.D))
        {
            transform.Translate(Vector3.right * speed * Time.deltaTime);
        }
        //跳跃
        if (Input.GetKey(KeyCode.Space))
        {
            transform.Translate(Vector3.up * jump_speed * Time.deltaTime);
        }

        //旋转 角度叠加量
         H += (-Input.GetAxis("Mouse Y") * move_offes);
         V += Input.GetAxis("Mouse X") * move_offes;

        //将Vector3转为Quaternion 再赋值给用户角度
        Quaternion rotation = Quaternion.Euler(new Vector3(H, V, 0));
        transform.rotation = rotation;
        
    }

```




$\color{#FFB6C1}{Feona 编辑}$ 
