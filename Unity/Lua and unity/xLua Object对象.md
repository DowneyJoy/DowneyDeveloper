C#

```
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class Npc
{
    public string Name;
    public int HP
    {
        get;
        set;
    }

    public Npc()
    {

    }

    public Npc(string name)
    {
        Name = name;
    }

    public string Output()
    {
        return this.Name;
    }
}

public class LuaCallObject : MonoBehaviour
{
    void Start()
    {
        xLuaEnv.Instance.DoString("require('C2L/LuaCallObject')");
    }

    void OnDestroy()
    {
        xLuaEnv.Instance.Free();
    }
}

```

Lua

```
-- Lua实例化类
-- C# Npc obj = new Npc()
-- 通过调用构造函数创建对象
local obj = CS.Npc()
obj.HP = 100

print(obj.HP)

local obj1 = CS.Npc("admin")
print(obj1.Name)

-- 表方法希望调用表成员变量（表:函数()）
-- 为什么是冒号，对象引用成员变量时，会隐性调用this，等同于Lua中的self
print(obj1:Output())

-- Lua实例化GameObject
-- C# GameObject obj = new GameObject("LuaCreateGO")
local go = CS.UnityEngine.GameObject("LuaCreateGO")

```

