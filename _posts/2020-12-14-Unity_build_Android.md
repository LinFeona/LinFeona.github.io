


# Unity 打包Android环境配置

- 资源准备
- [windows-Android-SDK 密码:1111](https://pan.baidu.com/s/1xaeF8gSlp-RqsEWEP6Doxw)
- [Java-jdk 密码:1111](https://pan.baidu.com/s/1EwK8UiulgJM8Uaexa_MReg )

---

## java-jdk安装
 
-  先查看 Unity 是否有jdk 
- Unity 打开首选项    edit -> preferences -> External Tools -> jdk 如果是勾选上的 并且是这样子的

![0-1](build/jdk_path_1-4.png "ddd")

- 那么恭喜你 这一步可以跳过了 可往下拉到sdk的环节了

- 1.安装jdk

![0-1](build/jdk_0-1.png "ddd")

- 选择一个路径 最好默认的路径 记住这个路径 环境变量会用到

![0-1](build/jdk_0-2.png"ddd")

- 直接下一步 默认下一步 直到完成安装

---

## java环境配置

- 复制jdk的路径

![0-1](build/jdk_path_1-0.png "ddd")

- 右键我的电脑-属性-高级系统设置-环境变量-系统变量

![0-1](build/jdk_path_1-1.png "ddd")

- 新建系统变量 变量名为 `JAVA_HOME` 变量值为 `C:\Program Files\Java\jdk1.8.0_152`  就是你jdk的路径

![0-1](build/jdk_path_1-5.png "ddd")

- 找到系统变量的 `PATH` 双击它 新建一个变量值 `%JAVA_HOME%/bin`  若是没有 `PATH` 变量则新建

![0-1](build/jdk_path_1-2.png "ddd")

- 新建系统变量 变量名为 `CLASSPATH` 变量值为 `.;%JAVA_HOME%\lib;%JAVA_HOME%\lib\tools.jar;` 前面还有一个· 后面还有一个分号

![0-1](build/jdk_path_1-6.png "ddd")

---

### 测试java环境

- 按住 `WIN + R` 键 输入 `cmd` 回车

- 在命令行窗口 输入 `java -version` 回车

![0-1](build/jdk_path_1-3.png "ddd")

- 如果正常显示 这样 表示环境已经搭建成功了

---

## sdk环境安装 `要记住安装或解压的路径`
- 直接使用现成的 Windows-Android-sdk  在下载链接中的 `android-sdk_r24.4.1-windows.zip` 解压即用 非常的银杏
- 或者 [外力援助](https://blog.csdn.net/chinarcsdn/article/details/79562567)  使用Android Studio  在下载链接中的 `Android-Studio.exe` 安装非常的头疼

---

## Unity 与SDK关联 

- 打开首选项    edit -> preferences


-  External Tools -> jdk   如果是如图`3-1`酱紫  那这步也跳过 否则那就 取消勾选 点击 `Browse`  选择jdk的位置 

![tools](jdk_path_1-4.png)
![0-1](unity_build_0-9.png "ddd")

-  External Tools -> sdk 将勾选取消掉 点击  `Browse` 选择sdk的路径

![0-1](unity_build_0-8.png "ddd")

---

## 打包设置

- 检查一下你的场景是否加载到这里 如果没有的话 就点击 `Add Open Scenes ` 将场景加载进来
- 若你有多个场景 请依次打开 并且依次 `Add Open Scenes ` 进来
- 全部加载进来后再次打开你游戏的初始化场景 再点击 `Add Open Scenes ` 保证它是打开游戏后运行的第一个场景

- 再将平台切换到安卓

![0-1](build/unity_build_0-10.png "ddd")


- 打开 `Player Settings` 进行打包设置

![0-1](build/unity_build_0-4.png "ddd")

- 注意看包名格式 xxx.xxx.xxx 公司名.包名.游戏名

![0-1](build/unity_build_0-3.png "ddd")

- 最后 开始打包 build 

![0-1](build/unity_build_0-6.png "ddd")


