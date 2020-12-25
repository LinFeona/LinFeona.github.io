# Animation AddClip 无效的问题

- 在Unity 5.x中，如果还需要使用Animation组件,则需要把AnimationClip的egacy设置成true.

- clip.legacy = true;

- 才能够痛过Animation的AddClip方法添加进去和播放。
- 否则就算是手动拖到Animation中,也是不能播放的。
