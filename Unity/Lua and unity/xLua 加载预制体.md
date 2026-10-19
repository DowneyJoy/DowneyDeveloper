C#

```
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.Events;

public static class Prefabs
{
    //根据预制体路径，加载UI页面
    public static GameObject Load(string path)
    {
        //加载预制体
        //资源的第一次加载后，会产生缓存，所以后面的加载会变快
        Object prefab = Resources.Load(path);
        //实例化预制体
        GameObject page = Object.Instantiate(prefab) as GameObject;
        //将页面放在Canvas下
        page.transform.SetParent(GameObject.Find("/Canvas").transform);
        page.transform.localPosition = Vector3.zero;
        page.transform.localRotation = Quaternion.identity;
        page.transform.localScale = Vector3.one;

        //四锚点边距归零
        RectTransform rt = page.transform as RectTransform;
        rt.offsetMin = Vector2.zero;
        rt.offsetMax = Vector2.zero;

        //去掉初始化的Clone
        page.name = prefab.name;

        return page;
    }

    public static void Alert(string message, UnityAction callback = null)
    {
        /*GameObject go = Prefabs.Load("Prefabs/UI/Alert");
        UIAlert script = go.GetComponent<UIAlert>();
        script.Message.text = message;

        //如果有附加的回调函数，则添加到确定按钮上
        if(callback != null)
        {
            script.Confirm.onClick.AddListener(callback);
        }*/
    }
}

```

Lua

```
Prefabs = {}

-- 预制体存储路径
function Prefabs:Instantiate(prefab)
	local go = CS.UnityEngine.Object.Instantiate(prefab)
	go.name = prefab.name
	local canvas = CS.UnityEngine.GameObject.Find("/Canvas").transform

	local trs = go.transform
	trs:SetParent(canvas)
	trs.localPosition = CS.UnityEngine.Vector3.zero
	trs.localRotation = CS.UnityEngine.Quaternion.identity
	trs.localScale = CS.UnityEngine.Vector3.one

	trs.offsetMin = CS.UnityEngine.Vector2.zero
	trs.offsetMax = CS.UnityEngine.Vector2.zero

	return go
end

```

