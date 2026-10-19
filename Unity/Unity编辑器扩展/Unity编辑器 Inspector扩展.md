```
using System;
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

//对象如果不标记为可序列化，则Unity在存储的时候，会认为他不可被序列化，那么也就无法被显示
//Unity的内置JSON工具运行原理与之类似
[Serializable]
public class Numerical
{
    public float Atk;
    public float Def;
}

//职业枚举
public enum Profression
{
    Warrior = 0,
    Wizard,
}

public class SimpleInspector : MonoBehaviour
{
    //隐藏公共成员变量，防止Inspector的值影响到它
    //同时保证脚本中变量的可访问度
    [HideInInspector]
    public int ID = 99;

    //私有变量，检视面板可见
    //Unity会将对象进行序列化存储，所以即使是私有的，那么标记为可序列化后，就会显示，共有默认是可序列化的
    [SerializeField]
    private string Name;

    //监视面板显示对象
    public Numerical Num;

    //把当前成员变量上方留50像素空白区域
    [Space(50)]
    //当前成员变量上方加入一个标题文字
    [Header("年龄")]
    //添加变量悬浮提示文字
    //一个成员变量可以添加多个特性
    [Tooltip("不要填写大于150岁的年龄")]
    //给数值设定范围（最小0，最大150）
    [Range(0, 150)]
    public int Age;

    //指定输入框，拥有五行
    [Multiline(5)]
    public string NickName;

    //默认显示5行，最多显示10行内容，再多用滚动条控制显示区域
    [TextArea(5, 10)]
    public string Description;

    public Color Flag;

    public Texture Tex;

    public List<string> Tags;

    public Profression Pro;

    //给小齿轮增加一个回调函数
    [ContextMenu("输出攻防比")]
    public void PrintADProportion()
    {
        Debug.Log("当前角色的攻防比：" + (Num.Atk / Num.Def));
    }

    //给一个成员变量添加右键菜单
    //第一个参数是菜单的名称
    //第二个参数是右键点击的回调函数
    [ContextMenuItem("输出国家", "OutCountry")]
    [Tooltip("右键看看")]
    public string Country;

    public void OutCountry()
    {
        Debug.Log(Country);
    }
}

```



```

```

