LayerMask实际上是一个位码操作，在Unity3D中一共有32个[Layer](https://so.csdn.net/so/search?q=Layer&spm=1001.2101.3001.7020)层，并且不可增加。

在Unity官方中给出的描述是：

------

```
Layers通常被摄像机用来渲染部分场景，和灯光照射部分场景使用。但是它们也可以用来做射线检测时忽略一些collider或Collision使用。
```

------

在代码中使用时如何开启某个Layers？

LayerMask mask = 1 << 你需要开启的Layers层。
LayerMask mask = 0 << 你需要关闭的Layers层。

比如：
LayerMask mask = 1 << 2; 表示开启Layer2。
LayerMask mask = 0 << 5;表示关闭Layer5。
LayerMask mask = 1<<2|1<<8;表示开启Layer2和Layer8。
LayerMask mask = 0<<3|0<<7;表示关闭Layer3和Layer7。

上面也可以写成：
LayerMask mask = ~（1<<3|1<<7）;表示关闭Layer3和Layer7。
LayerMask mask = 1<<2|0<<4;表示开启Layer2并且同时关闭Layer4.

------

Layer一共有32个层，分别按索引表示：Layer 0到Layer 31,对应int32的32位：

以下分别按int32类型的十进制数、<<左移位操作符、二进制表示选择对应的Layer层：

0层 1层 2层 3层 4层 5层 。。。 31层

int32 1 2 4 8 16 32 ... 2147483648

<< 1<<0 1<<1 1<<2 1<<3 1<<4 1<<5 ... 1<<31

2进制 1 10 100 1000 10000 100000 ... 100(共31个0)...

取反：表示该层之外的所有层

int32 ~1 ~2 ~4 ~8 ~16 ~32 ... ~2147483648

<< ~(1<<0) ~(1<<1) ~(1<<2) ~(1<<3) ~(1<<4) ~(1<<5) ... ~(1<<31)

2进制 0 01 011 0111 01111 011111 ... 011(共31个1)...

LayerMask是图层的一种表示类型：是 位掩码 (数据类型是struct，int其实也是struct)
可表示单层，也可以表示多层

选择Layer层：选择a层(a是Layer索引0到31)：

LayerMask mask = 1<<a;

关闭Layer层：关闭a层(a层是选中的情况下)：

LayerMask mask = 0<<a;

选择所有Layer层：

LayerMask mask = ~0;

关闭所有Layer层（Nothing,不选择任何Layer）：

LayerMask mask = 0;

选择a和b层：

LayerMask mask = 1<<a|1<<b;

选择a和b之外的层：

LayerMask mask = ~(1<<a|1<<b);

&与、|或操作（Layer均以索引表示）：

|添加层：

eg：默认相机cullingMask渲染了4、5层，给相机添加一个渲染层2

camera.cullingMask |= 1<<2;

camera.cullingMask最终值为：

位符号表示：1<<2|1<<4|1<<5

二进制表示：10|100|1000=1110;

&删除层：

eg：默认相机cullingMask渲染了2、3、4层，此时删除3层，不渲染3层

camera.cullingMask &= ~8; 或者 camera.cullingMask &= ~(1<<3)

camera.cullingMask = 100|1000|10000&0111=11100&0111=10100=100|10000；

camera.cullingMask最终值为：

位符号表示：1<<2|1<<4

二进制表示：100|10000

静态函数：

1.GetMask(params string[] layerNames)，根据层名称获取 位掩码 
int layerMask = LayerMask.GetMask("Player");  获取层 Player 的位掩码
int layerMask2 = LayerMask.GetMask("Player","Enemy"); 获取层 Player、Enemy 的位掩码

2.LayerToName(int layer)，根据层索引，获取层名称
string layerName = LayerMask.LayerToName(7); 获取索引为7的层的名称

3.NameToLayer(string layerName)，根据层名称，获取层索引
int layer = LayerMask,NameToLayer("Player"); 获取层名称为 Player 的索引

变量：

1.value 获取LayerMask的value值，该值为LayerMask的 位掩码 值

运算符：

LayerMask 整数值 int 可以直接隐式转换为 LayerMask
int layerIndex = 7;   层索引7名称为 Player
LayerMask layerMask;
layerMask = layerIndex;
LayerMask.LayerToName(layerMask); 结果为 Player

注意点：
像物理检测Physics.Raycast、Physics.OverlapSphereNonAlloc等函数中的layerMask参数均为 位掩码，而非 层索引，layerMask参数 即可以直接填 LayerMask数据类型，也可以填 int 类型的 位掩码，所以也可以填数据类型为LayerMask的value值，因为value是位掩码

------

Unity LayerMask一般用在碰撞射线检测。

+++++++++++++++++++++更新++++++++++++++++++++++++++++
射线碰撞检测相关API：

```
1、Ray Camera.main.ScreenPointToRay(Vector3 pos) 
2、Ray Camera.main.ViewportPointToRay(Vector3 pos) 
//从鼠标的位置需要发射射线了
//因为是从Scene视图下发射射线，跟场景中的摄像机并没有关系，所以不能使用相机发射射线的方法
//从编辑器GUI中的一个点向世界定义一条射线, 参数一般都是鼠标的坐标
Ray ray = HandleUtility.GUIPointToWorldRay(Event.current.mousePosition);
3、Ray 
4、RaycastHit 
5、bool Physics.Raycast(Vector3 origin, Vector3 direction, float distance, int layerMask)
```

示例代码

```
using UnityEngine;
public class LayerMaskTest : MonoBehaviour
{
    private LayerMask mask;
    private Camera mainCamera;
    private RaycastHit GbjHit;
    private Ray myRay;
    void Start()
    {
        mainCamera = Camera.main;
        mask = 1 << (LayerMask.NameToLayer("MaskName"));
    }
    void Update()
    {
        if (Input.GetMouseButtonDown(0))
        {
            myRay = mainCamera.ScreenPointToRay(Input.mousePosition);
            Debug.DrawLine(myRay.origin, GbjHit.point, Color.red, 2);
            if (Physics.Raycast(myRay, out GbjHit, 100, mask.value))
            {
                GameObject Gbj = GbjHit.collider.gameObject;
                Debug.Log("HitGameObjectName:" + Gbj.name + "--HitLayoutName:" + LayerMask.LayerToName(10));
            }
        }
    }
}

```

Editor scene检测



```
void OnSceneGUI()
    {

        if (!isEditor)//非编辑状态下不能生成路点
        {
            return;
        }


        //当鼠标按下左键时发射一条射线 
        //非运行时，使用Event类
        //Event.current.button 判断鼠标是哪个按键的（0是鼠标左键）
        //Event.current.type 判断鼠标的事件方式的（鼠标按下）
        if (Event.current.button == 0 && Event.current.type == EventType.MouseDown)
        {

            Debug.Log("hit");
            //从鼠标的位置需要发射射线了
            //因为是从Scene视图下发射射线，跟场景中的摄像机并没有关系，所以不能使用相机发射射线的方法
            //从编辑器GUI中的一个点向世界定义一条射线, 参数一般都是鼠标的坐标
            Ray ray = HandleUtility.GUIPointToWorldRay(Event.current.mousePosition);
            if (Physics.Raycast(ray, out hit, 300, 1 << 31))
            {
                //需要在检测到的点实例化，路点
                //InstancePathNode(hit.point + Vector3.up * 0.1f);
            }

        }
    }
```

