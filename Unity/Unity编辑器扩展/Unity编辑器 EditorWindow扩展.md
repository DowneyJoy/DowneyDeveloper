```
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

[ExecuteInEditMode]//在编辑模式下可以执行一些生命周期函数
public class NodeManager : MonoBehaviour {

    //存储了所有编辑器下点击生成的点，并使用预制体显示
    public List<GameObject> nodes;

	// Use this for initialization
	void Start () {
		
	}

	void Update () {
        //if (nodes.Count <= 0)
        //{
        //    return;
        //}
        for (int i = 0; i < nodes.Count - 1; i++)
        {
            Debug.DrawLine(nodes[i].transform.position, 
                nodes[i + 1].transform.position, 
                Color.red,
                Time.deltaTime);
        }
	}
}

```

```
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEditor;

public class NodeWindow : EditorWindow
{
    static NodeWindow window;

    static GameObject nodeManager;

    public static void OpenWindow(GameObject manager)
    {
        nodeManager = manager;
        //真正开启了一个窗口
        window = EditorWindow.GetWindow<NodeWindow>();
    }

    void Update()
    {
        //通过窗口的Update，每帧执行一次，当前被选中的对象为板子
        Selection.activeGameObject = nodeManager;
    }

    public static void CloseWindow()
    {
        window.Close();
    }
}

//外挂式关联NodeManager
[CustomEditor(typeof(NodeManager))]
public class NodeManagerEditor : Editor
{

    NodeManager manager;

    bool isEditor = false;//是否是编辑的状态

    //当选中带有NodeManager组件对象的时候，获得组件
    void OnEnable()
    {
        manager = (NodeManager)target;
    }

    //绘制组件的生命周期函数
    public override void OnInspectorGUI()
    {
        //通过终极的数据获取方法，显示列表中的数据
        serializedObject.Update();
        SerializedProperty nodes = serializedObject.FindProperty("nodes");
        EditorGUILayout.PropertyField(nodes, new GUIContent("路径"), true);
        serializedObject.ApplyModifiedProperties();

        //开始编辑的开关
        if (!isEditor && GUILayout.Button("开始编辑"))
        {
            NodeWindow.OpenWindow(manager.gameObject);//调用打开界面的方法
            isEditor = true;//改变状态变成编辑模式
        }
        //结束编辑的开关
        else if (isEditor && GUILayout.Button("结束编辑"))
        {
            NodeWindow.CloseWindow();//调用关闭界面的方法
            isEditor = false;//改变状态变成非编辑模式
        }

        //删除按钮
        if (GUILayout.Button("删除最后一个节点"))
        {
            RemoveAtLast();
        }
        //删除所有按钮
        else if (GUILayout.Button("删除所有节点"))
        {
            RemoveAll();
        }
    }

    RaycastHit hit;
   
    //有点类似前期Update函数，发送射线
    //当选中关联的脚本挂载的物体
    //当鼠标在Scene视图下发生变化时，执行该方法，比如鼠标移动，比如鼠标的点击
    void OnSceneGUI()
    {

        if (!isEditor)//非编辑状态下不能生成路点
        {
            return;
        }


		//Scene试图的Gizmos选中，OnSceneGUI()才有反应。
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
                InstancePathNode(hit.point + Vector3.up * 0.1f);
            }

        }
    }

    /// <summary>
    /// 生成节点
    /// </summary>
    /// <param name="position"></param>
    void InstancePathNode(Vector3 position)
    {
        //点预制体
        GameObject prefab = Resources.Load<GameObject>("PathNode");
        //点对象，生成到Plane的子物体下
        GameObject pathNode= Instantiate<GameObject>(prefab, position, Quaternion.identity, manager.transform);
        //把生成的路点添加到列表里
        manager.nodes.Add(pathNode);
    }

    /// <summary>
    /// 删除最后一个节点
    /// </summary>
    void RemoveAtLast()
    {
        //保证有节点才能删节点
        if (manager.nodes.Count > 0)
        {
            //从场景中删除游戏物体
            DestroyImmediate(manager.nodes[manager.nodes.Count - 1]);
            //把该节点从列表中移除
            manager.nodes.RemoveAt(manager.nodes.Count - 1);
        }

    }


    /// <summary>
    /// 删除所有的节点
    /// </summary>
    void RemoveAll()
    {
        ///遍历删除所有的节点物体
        for (int i = 0; i < manager.nodes.Count; i++)
        {
            if (manager.nodes[i] != null)
            {
                DestroyImmediate(manager.nodes[i]);
            }
        }

        manager.nodes.Clear();//清空列表
    }


}

```

