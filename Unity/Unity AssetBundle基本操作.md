生成AB包

```
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEditor;
using System.IO;

public class ExportAB
{
    [MenuItem("AB包导出/Windows")]
    public static void ForWindows()
    {
        Export(BuildTarget.StandaloneWindows);
    }

    [MenuItem("AB包导出/Mac")]
    public static void ForMac()
    {
        Export(BuildTarget.StandaloneOSX);
    }

    [MenuItem("AB包导出/iOS")]
    public static void ForiOS()
    {
        Export(BuildTarget.iOS);
    }

    [MenuItem("AB包导出/Android")]
    public static void ForAndroid()
    {
        Export(BuildTarget.Android);
    }

    private static void Export(BuildTarget platform)
    {
        //项目的Assets目录的路径
        //老师电脑，D:/HonorZhao/Month3/Week3/Day2/Code/Assets
        string path = Application.dataPath;
        //保留，D:/HonorZhao/Month3/Week3/Day2/Code/
        //最终路径，D:/HonorZhao/Month3/Week3/Day2/Code/ab
        path = path.Substring(0, path.Length - 6) + "ab";

        //防止路径不存在
        if(!Directory.Exists(path))
        {
            Directory.CreateDirectory(path);
        }

        //导出ab包的核心代码，生成ab包文件
        //参数1：ab包文件存储路径
        //参数2：导出选项
        //参数3：平台（不同平台的ab包是不一样的）
        BuildPipeline.BuildAssetBundles(
            path,
            BuildAssetBundleOptions.ChunkBasedCompression | BuildAssetBundleOptions.ForceRebuildAssetBundle,
            platform
        );

        Debug.Log("导出ab包成功");
    }
}

```

AB包路径配置

```
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class Config
{
    //配置AB包存储的路径
    public static string ABPath = Application.dataPath.Substring(0, Application.dataPath.Length - 6) + "ab";
}

```

简单加载AB资源

```
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.UI;

public class SimpleLoad : MonoBehaviour
{
    void Start()
    {
        //第一步加载ab文件
        AssetBundle ab = AssetBundle.LoadFromFile(Config.ABPath + "/ui");

        //第二步加载资源
        Sprite sp = ab.LoadAsset<Sprite>("icon_gold");

        GameObject.Find("/Canvas/Image").GetComponent<Image>().sprite = sp;
    }

}

```

更新AB包资源

```
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.UI;

public class HotUpdate : MonoBehaviour
{
    public Image Icon;

    void Start()
    {
        AssetBundle ab = AssetBundle.LoadFromFile(Config.ABPath + "/Old/test");
        Sprite sp = ab.LoadAsset<Sprite>("btn_go_back");

        Icon.sprite = sp;
        ab.Unload(false);//释放ab包
    }

    //点击按钮后，加载新ab包里面的btn_go_back资源
    public void ChangeAB()
    {
        AssetBundle ab = AssetBundle.LoadFromFile(Config.ABPath + "/New/test");
        Sprite sp = ab.LoadAsset<Sprite>("btn_go_back");

        Icon.sprite = sp;
        ab.Unload(false);
    }
}

```

