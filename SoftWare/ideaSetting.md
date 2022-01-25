# IDEA 设置

- [IDEA 设置](#idea-设置)
  - [idea设置](#idea设置)
  - [template](#template)
  - [插件配置](#插件配置)

## idea设置

- extend selection : alt + w
- close : ctrl + w
- up : alt + k ; down : alt + j ; left : alt + h ; right : alt + l;

## template

- test

    ```java
    @Test
    public void test$var1$(){
        $var2$
    }
    ```

- 获取线程名 ``mythread Thread.currentThread().getName()``

- 线程睡眠 tsleep

    ```java
    try {TimeUnit.SECONDS.sleep($var1$);} catch (Exception e) {e.printStackTrace();}
        $var2$
    ```

## 插件配置

1. ideaVim
   - Vim Emulator : ctrl + windows
   - 这个就不做解释了，有了它敲代码的时候就基本不用鼠标和方向键了,在设置项里面,plugin找到ideaVim,Vim Emulator设置快捷键,启动关闭vim模式
2. AceJump: 它是一款能够代替鼠标的软件，只要安装了这款插件，可以在代码中跳转到任意位置。按快捷键进入 AceJump 模式后（可以在 ``.ideavimrc`` 中添加 ``noremap f :action AceAction<CR>`` , 设置普通模式下的 f 为快捷键，默认是 Ctrl+;），再按任一个字符，插件就会在屏幕中这个字符的所有出现位置都打上标签，你只要再按一下标签的字符，就能把光标移到该位置上。换言之，你要移动光标时，眼睛一直看着目标位置就行了，根本不用管光标的当前位置。
3. Key promoter: Key promoter 快捷键提示的，会统计你鼠标点击某个功能的次数，提示你应该用什么快捷键，帮助记忆快捷键，等熟悉了之后可以关闭掉这个插件。



