---
title: Unity编辑器扩展
date: 2023-12-31 00:35:47
categories: [Unity,编辑器扩展]
---
### 编辑器菜单
判断菜单项是否可用函数。
```c#
    [MenuItem("GameObject/test", true, 14)]
    static bool TestisValidate()
    {
        //返回值为false，则菜单项可用，反之不可使用
        return true;
    }

//如下即可在Unity编辑器窗口的相应位置出现test菜单项，单击即可执行对应静态方法

[MenuItem("GameObject/test", false, 14)]
    static void Test(){}

//在组件上添加菜单项。

        [MenuItem("CONTEXT/PlayerHealth/InitHealthSpeed")]
//即可在相应组件位置添加菜单项

//除此之外可以在组件脚本中使用

        [ContextMenu("SetColor")]
//以及对脚本中属性添加菜单

        [ContextMenuItem("wtf","SetColor")]
            public Color Color;
//既可以在color上右键以弹出相应菜单，其中Setcolor为所执行函数名。
```
### 编辑器创建窗口(ScriptableWizard)
此窗口通过继承 **ScriptableWizard** 类进行，其中有函数
```C#
     OnWizardCreate()//在窗口中Create按钮点击后调用
     OnEnable()//在窗口创建时调用
     /*另外还有许多API

     通过此可以实现资源创建，比如Unity Shaderlab入门精要中在立方体贴图部分即使用此功能。*/

    public Transform Transform;
    public Cubemap Cubemap;

    private void OnWizardCreate()
    {
        GameObject gameObject = new GameObject();
        gameObject.transform.position = Transform.position;
        Camera camera = gameObject.AddComponent<Camera>();
        camera.RenderToCubemap(Cubemap);
        DestroyImmediate(gameObject);
    }

    //以上为窗口内部代码，在此之外调用
    [MenuItem("Create/CubeMap", false)]
    static void CreateWindow()
    {
        ScriptableWizard.DisplayWizard<PlayerEditor>("CreateCubeMap");
    }
    //其中PlayerEditor为实现窗口代码的脚本(类)名。
```
### 编辑器窗口(Window)
此窗口通过继承 **EditorWindow** 进行，与**ScriptableWizard** 不同的是，此窗口可以通过调用

    OnGUI() 
通过编写代码实现窗口布局，也通过

    Window window = EditorWindow.GetWindow<Window>();
    window.Show();
来显示窗口。

### 操作撤回
Unity的操作中会有部分修改需要撤回，比如删除某个东西或者移动某个东西，会使用到代码进行操作比如删除某个物体
```c#
    GameObject.DestroyImmediate(object);
    //或者
    DestroyImmediate(object);
```
而这种操作并不能撤回，对于编辑器而言是不合适的，所以会使用到 

**Undo**
    
    Undo.DestroyObjectImmediate(object);
这样就会保存执行的操作
### 数据保存
在编辑器扩展的窗口中可能需要保存上次修改时输入的数据，如使用**ScriptableWizard**修改数据，此处会使用到

 **EditorPrefs**
```c#
    private void OnWizardCreate()
    {
        ***

        ***
        //在结束时以关键词key保存数据value
        EditorPrefs.SetInt(key,value);
    }
    //窗口启用时调用以读取保存的数据value
    private void OnEnable()
    {
        value = EditorPrefs.GetInt(key,default_value);
    }
```
除此之外，Json也是一种保存方法
```c#
    private string json;
    List<string>=new List<string>();
    list=JsonUtility.FromJson<List<string>>(json);//加载
    json = JsonUtility.ToJson(list);//保存
```
### 编辑器中物体选择
**Selection**
编辑器扩展中，有时需要对选中的物体进行修改

    GameObject[] gameObjects = Selection.gameObjects;
---
### 其他
```c#
    //在编辑器模式下执行
    [ExecuteInEditMode]
    public class TestClass{
        //此类可在编辑模式下执行
    }
```
### 自定义组件属性界面
```c#

    [CustomEditor(typeof("组件名"),editorForChildClasses:true,isFallback = false)]

    public class TestInspector:Editor
    {
        public override void OnInspectorGUI(){
            
        }
    }
```    
继承自`Editor`添加`CustomEditor`作用与对应组件，有关键字`target`和`serializedObject`负责不同功能，具体可以查看[UnityAPI]

[UnityAPI]:https://docs.unity.cn/cn/2019.4/ScriptReference/Editor.html

