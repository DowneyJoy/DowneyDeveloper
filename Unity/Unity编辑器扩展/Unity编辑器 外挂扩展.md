```
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

//单选我们使用十进制理解，即不同数代表不同选项
public enum PlayerProfression
{
    Warrior = 0,
    Wizard = 1,
}

//多选使用二进制理解，即不同位代表不同选项
public enum PlayerLoveColor
{
    Green = 1,
    Red = 2,
    Pink = 4,
}

//将Player组件添加到AddComponent上
//第一个参数：分类名/组件名
//第二个参数：列表中显示的顺序
[AddComponentMenu("自定义控制器/玩家控制器", 1)]
//使生命周期函数，在编辑器状态下可以执行，游戏中也可以正常使用
//Update()在场景中对象发生变化或项目组织发生变化时会在编辑器下执行
[ExecuteInEditMode]
//关于类型和类名
//BoxCollider：是类名，适用于函数提供泛型方法
//typeof(BoxCollider)：System.Type，C#的类型，适用于函数需要System.Type参数
//当前组件依赖于盒子碰撞体
//当前组件挂载在对象上时，盒子碰撞体会一起被添加上去
//当Player组件没有被移除时，盒子碰撞体不能被删除
[RequireComponent(typeof(BoxCollider))]
public class Player : MonoBehaviour
{
    public int ID;

    public string Name;

    public float Atk;

    public bool isMan;

    public Vector3 HeadDir;

    public Color Hair;

    public GameObject Weapon;

    public Texture Cloth;

    public PlayerProfression Pro;

    public PlayerLoveColor LoveColor;

    public List<string> Items;

    void Update()
    {
        //Debug.Log("Update");
    }
}

```

```
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
//步骤1：引入编辑器的命名空间，检视器属于编辑器开发范畴
using UnityEditor;

[CustomEditor(typeof(Player))]//步骤3：将编辑器开发脚本与需要编辑的组件脚本建立外挂关联关系
//外挂脚本因为存储在Editor目录下，所以不会被打入最终的游戏包
//不继承自Mono，而是继承自Editor
public class PlayerEditor : Editor //步骤2：继承Editor类，使用编辑器相关的成员变量和生命周期函数
{
    //获得到需要编辑显示的组件
    private Player _Component;

    //步骤4：需要在当前的外挂脚本中，获得需要被扩展的Player组件对象
    //当关联组件所在对象被选中或组件被添加时，调用
    private void OnEnable()
    {
        //Debug.Log("enable");
        //步骤5：获取Player组件对象
        _Component = target as Player;

    }

    //当关联组件所在对象被取消或组件被移除时，调用
    private void OnDisable()
    {
        //Debug.Log("disable");

        _Component = null;
    }
    
    //用于绘制检视面板的生命周期函数
    public override void OnInspectorGUI()
    {
        //标题显示
        EditorGUILayout.LabelField("人物相关属性");

        _Component.ID = EditorGUILayout.IntField("玩家ID", _Component.ID);
        //文本
        _Component.Name = EditorGUILayout.TextField("玩家名称", _Component.Name);
        //浮点数
        _Component.Atk = EditorGUILayout.FloatField("玩家攻击力", _Component.Atk);
        //布尔
        _Component.isMan = EditorGUILayout.Toggle("是否为男性", _Component.isMan);
        //向量
        _Component.HeadDir = EditorGUILayout.Vector3Field("头部方向", _Component.HeadDir);
        //颜色
        _Component.Hair = EditorGUILayout.ColorField("头发颜色", _Component.Hair);

        ////对象数据类型绘制////////////////////////////////////////////////////////
        //参数1：标题
        //参数2：原始组件的值
        //参数3：成员变量的类型
        //参数4：是否可以将场景中的对象拖给这个成员变量
        _Component.Weapon = EditorGUILayout.ObjectField("持有武器", _Component.Weapon, typeof(GameObject), true) as GameObject;
        //纹理
        _Component.Cloth = EditorGUILayout.ObjectField("衣服材质贴图", _Component.Cloth, typeof(Texture), false) as Texture;

        ////枚举数据类型绘制////////////////////////////////////////////////////////
        //整数转枚举
        //int id = 0;
        //PLAYER_PROFESSION p = (PLAYER_PROFESSION)id;

        //单选枚举（标题, 组件上的原始值）
        _Component.Pro = (PlayerProfression)EditorGUILayout.EnumPopup("玩家职业", _Component.Pro);

        //多选枚举（标题, 组件上的原始值）
        _Component.LoveColor = (PlayerLoveColor)EditorGUILayout.EnumFlagsField("玩家喜欢的颜色", _Component.LoveColor);

        ////终极数据类型绘制////////////////////////////////////////////////////////
        //更新可序列化数据
        serializedObject.Update();
        //通过成员变量名找到组件上的成员变量
        SerializedProperty sp = serializedObject.FindProperty("Items");
        //可序列化数据绘制（取到的数据，标题，是否将所有获得的序列化数据显示出来）
        EditorGUILayout.PropertyField(sp, new GUIContent("道具信息"), true);
        //将修改的数据，写入到可序列化的原始数据中
        serializedObject.ApplyModifiedProperties();

        ////滑动条绘制////////////////////////////////////////////////////////
        //滑动条显示（1.标题，2.原始变量，最小值，最大值）
        _Component.Atk = EditorGUILayout.Slider(new GUIContent("玩家攻击力"), _Component.Atk, 0, 100);

        if (_Component.Atk > 80)
        {
            //显示消息框（红色）
            EditorGUILayout.HelpBox("攻击力太高了", MessageType.Error);
        }

        if (_Component.Atk < 20)
        {
            //显示消息框（黄色）
            EditorGUILayout.HelpBox("攻击力太低了", MessageType.Warning);
        }

        //按钮显示和元素排列
        //（按钮是否被按下）显示按钮（按钮名称）
        GUILayout.Button("来个按钮");
        GUILayout.Button("来个按钮");

        if (GUILayout.Button("测试点击"))
        {
            Debug.Log("测试点击");
        }

        //开始横向排列绘制
        EditorGUILayout.BeginHorizontal();

        GUILayout.Button("再来个按钮");
        GUILayout.Button("再来个按钮");

        //结束横向排列绘制
        EditorGUILayout.EndHorizontal();
    }
}

```

