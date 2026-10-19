C#

```
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class Father
{
    public string Name = "father";

    public void Talk()
    {
        Debug.Log("这是父类中的方法");
    }

    public virtual void Overide()
    {
        Debug.Log("这是父类中的虚方法");
    }
}

public class Child : Father
{
    public override void Overide()
    {
        Debug.Log("这是子类中的重写方法");
    }
}

public class LuaCallBase : MonoBehaviour
{
    void Start()
    {
        xLuaEnv.Instance.DoString("require('C2L/LuaCallBase')");
    }

    void OnDestroy()
    {
        xLuaEnv.Instance.Free();
    }
}

```

Lua

```
-- 调用Father
local father = CS.Father()
print(father.Name)
father:Overide()

-- 调用Child
local child = CS.Child()
print(child.Name)
child:Talk()
child:Overide()

```

