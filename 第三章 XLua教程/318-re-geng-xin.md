##热更新

**Editor目录下CreateAssetBundle.cs**

```csharp
    /*
     *  created by shenjun
     */
    
    using System.Collections;
    using System.Collections.Generic;
    using UnityEngine;
    using UnityEditor;
    using UnityEngine.Networking;
    
    namespace shenjun
    {
        public class CreateAssetBundle : EditorWindow
        {
    
            public static string createAssetPath{ get { return Application.dataPath + "/LessonXLua_shenjun/04_Examples/02_HotUpdate/AssetBundle/Build/"; }}
    
    
            static CreateAssetBundle window;
    
            int platformSelect = 0;
            string[] platformName = { "Windows", "Mac OS", "IOS", "Android", "All" };
    
            string assetbundlePath = "";
            string Path 
            {
                set {
                    assetbundlePath = value;
                }
                get
                {
                    if(string.IsNullOrEmpty(assetbundlePath))
                    {
                        assetbundlePath = createAssetPath;
                    }
                    return assetbundlePath;
                }
            }
    
    
            [MenuItem("AssetBundle/CreateAssetBundleTools")]
            static void ShowWindow()
            {
                if (window == null)
                {
                    window = EditorWindow.GetWindow<CreateAssetBundle>(true, "AssetBundle Tools");
                    window.minSize = new Vector2(800, 600);
                }
                window.Show();
            }
    
            void OnGUI()
            {
                platformSelect = GUILayout.SelectionGrid(platformSelect, platformName, platformName.Length);
    
                GUILayout.BeginHorizontal();
    
                GUILayout.Label("打包路径：");
                GUILayout.Label(Path);
    
                if(GUILayout.Button("选择路径", GUILayout.MaxWidth(100)))
                {
                    Path = EditorUtility.OpenFolderPanel("选择文件夹", Application.dataPath, Path);
                }
    
                GUILayout.EndHorizontal();
                if (GUILayout.Button("打包"))
                {
                    CreateBundle(platformSelect);
                    EditorUtility.DisplayDialog("资源打包信息提示", "打包完成", "确定");
                }
            }
    
            void CreateBundle(int plat)
            {
                string combinePath = "";
                BuildTarget target = BuildTarget.NoTarget;
    
                switch(plat)
                {
                    case 0:
                        combinePath = Path + "Windows";
                        target = BuildTarget.StandaloneWindows;
                        break;
                    case 1:
                        combinePath = Path + "Mac OS";
    					target = BuildTarget.StandaloneOSX;
                        break;
                    case 2:
                        combinePath = Path + "IOS";
                        target = BuildTarget.iOS;
                        break;
                    case 3:
                        combinePath = Path + "Android";
                        target = BuildTarget.Android;
                        break;
                    case 4:
                        for (int i = 0; i < 4; i++)
                        {
                            CreateBundle(i);
                        }
                        return;
                }
    
                if (!System.IO.Directory.Exists(combinePath))
                {
                    System.IO.Directory.CreateDirectory(combinePath);
                }
                CreateBundle(combinePath, target);
            }
    
            void CreateBundle(string path ,BuildTarget target)
            {
                BuildPipeline.BuildAssetBundles(path, BuildAssetBundleOptions.ForceRebuildAssetBundle, target);
                AssetDatabase.Refresh();
            }
    
            [MenuItem("AssetBundle/资源清理(删除本地及远程数据)")]
            public static void ClearDirectory()
            {
                string[] clearPath = { LoadHotUpdate.localAssetPath, LoadHotUpdate.remoteAssetPath };
                for (int i = 0; i < clearPath.Length; i++)
                {
                    string[] fileNames = System.IO.Directory.GetFiles(clearPath[i]);
                    for (int j = 0; j < fileNames.Length; j++)
                    {
                        System.IO.File.Delete(fileNames[j]);
                    }
                }
                AssetDatabase.Refresh();
            }
        }
    }
```

**HotUpdateLua.cs**

```csharp
    /*
     *  created by shenjun
     */
    
    using System.Collections;
    using System.Collections.Generic;
    using UnityEngine;
    using System;
    using UnityEngine.EventSystems;
    using XLua;
    
    namespace shenjun
    {
        public class HotUpdateLua : MonoBehaviour, IPointerDownHandler {
    
            [CSharpCallLua]
            public delegate void AwakeSet(HotUpdateLua self);
    
            [CSharpCallLua]
            public delegate void OnPanelClick(PointerEventData eventData);
    
            LuaEnv luaEnv = new LuaEnv();
    
            AwakeSet awake;
            Action start;
            Action update;
            Action destroy;
    
            OnPanelClick panelClick;
    
            void Awake()
            {
                AssetBundle bundle = AssetBundle.LoadFromFile(LoadHotUpdate.localAssetPath + "luahotupdate_lua.ab");
                TextAsset luaAsset = bundle.LoadAsset<TextAsset>("LuaHotUpdate.lua.txt");
    
                luaEnv.DoString(luaAsset.text);
    
                awake = luaEnv.Global.Get<AwakeSet>("Awake");
                start = luaEnv.Global.Get<Action>("Start");
                update = luaEnv.Global.Get<Action>("Update");
                destroy = luaEnv.Global.Get<Action>("OnDestroy");
    
                panelClick = luaEnv.Global.Get<OnPanelClick>("OnPanelClick");
    
                if (awake != null) awake(this);
            }
    
            void Start () {
    
                if (start != null) start();
            }
    
            void Update () {
                if (update != null) update();
            }
    
            void OnDestroy()
            {
                awake = null;
                start = null;
                update = null;
    
                if (destroy != null)
                {
                    destroy();
                    destroy = null;
                }
    
                panelClick = null;
                luaEnv.Dispose();
            }
    
    
            public void OnPointerDown(PointerEventData eventData)
            {
                if (panelClick != null) panelClick(eventData);
            }
        }
    }
```

**LoadHotUpdate.cs**

```csharp
    /*
     *  created by shenjun
     */
    
    using System.Collections;
    using System.Collections.Generic;
    using UnityEngine;
    using System;
    using System.IO;
    using UnityEngine.Networking;
    using UnityEngine.UI;
    
    namespace shenjun
    {
        public class LoadHotUpdate : MonoBehaviour
        {
            static Dictionary<string, AssetBundle> bundleCaches = new Dictionary<string, AssetBundle>();
            static AssetBundleManifest manifest = null;
    
    
            public static string localAssetPath
            {
                get
                {
                    // FIXME : 移动平台的路径不一样
                    // 移动平台的话 请使用路径（具有可读写权限）： Application.persistentDataPath
                    return Application.dataPath + "/LessonXLua_shenjun/04_Examples/02_HotUpdate/AssetBundle/Local/";
    //#if UNITY_STANDALONE_WIN || UNITY_STANDALONE_OSX
    //#elif UNITY_IOS
    //#elif UNITY_ANDROID
    //#endif
                }
            }
    
            public static string remoteAssetPath
            {
                get
                {
                    // FIXME : 移动平台的路径不一样 同 localAssetPath
                    return Application.dataPath + "/LessonXLua_shenjun/04_Examples/02_HotUpdate/AssetBundle/Remote/";
                }
            }
    
            void Start()
            {
                // 从服务器 下载配置文件列表  配置文件格式：txt json xml cvs
                // 和本地配置文件对比  版本号、资源校验码等 不同的则更新
                // 更新
                // 运行程序
    
                StartCoroutine(LoadRemoteProfile(LoadAsset));
            }
    
            void Update()
            {
            }
    
            private void OnDestroy()
            {
                bundleCaches.Clear();
                AssetBundle.UnloadAllAssetBundles(true);
            }
    
            IEnumerator LoadRemoteProfile(Action<string> onFinished)
            {
                //Caching.ClearCache();
                //while (!Caching.ready)
                //yield return null;
    
                string url =
    #if UNITY_STANDALONE_WIN
                "file:///" +
    #elif UNITY_STANDALONE_OSX
                "file://" +
    #endif
                    remoteAssetPath + "luahotupdate_profile.ab";
    
                AssetBundle bundle = null;
    
                // 1、WWW.LoadFromCacheOrDownload 加载
                //using(WWW www = WWW.LoadFromCacheOrDownload(url, 0))
                //{
                //    yield return www;
                //    if (!string.IsNullOrEmpty(www.error))
                //    {
                //        Debug.Log(www.error);
                //        yield break;
                //    }
                //    bundle = www.assetBundle;
                //}
    
                // 2、UnityWebRequest.GetAssetBundle 加载
                UnityWebRequest webRequest = UnityWebRequest.GetAssetBundle(url);
    			yield return webRequest.SendWebRequest();
                if (!string.IsNullOrEmpty(webRequest.error))
                {
                    Debug.Log(webRequest.error);
                    yield break;
                }
                bundle = DownloadHandlerAssetBundle.GetContent(webRequest);
                // 3、AssetBundle.LoadFromFile 加载
                //bundle = AssetBundle.LoadFromFile(remoteAssetPath + "luahotupdate_profile.ab");
    
                // 4、AssetBundle.LoadFromFileAsync 加载
                //AssetBundleCreateRequest request = AssetBundle.LoadFromMemoryAsync(File.ReadAllBytes(remoteAssetPath + "luahotupdate_profile.ab"));
                //yield return request;
                //bundle = request.assetBundle;
    
                TextAsset profile = bundle.LoadAsset<TextAsset>("file");
                onFinished(profile.text);
            }
    
            void LoadAsset(string remoteVersion)
            {
                AssetBundle.UnloadAllAssetBundles(true);
    
                // 加载本地配置文件
                string profilePath = localAssetPath + "luahotupdate_profile.ab";
                if (!File.Exists(profilePath))
                {
                    // 加载服务器文件列表
                    LoadRemoteAsset();
                }
                else
                {
                    // 加载本地列表并比较
                    AssetBundle bundle = AssetBundle.LoadFromFile(profilePath);
                    TextAsset profile = bundle.LoadAsset<TextAsset>("file");
    
                    float localVersion, removeVersion;
                    if (float.TryParse(profile.text, out localVersion) && float.TryParse(remoteVersion, out removeVersion))
                    {
                        if (removeVersion > localVersion)
                        {
                            // 加载服务器文件列表
                            LoadRemoteAsset();
                        }
                    }
                }
    
                LoadPanel();
            }
    
            // 加载服务器文件列表
            void LoadRemoteAsset()
            {
                if (Directory.Exists(localAssetPath))
                {
                    string[] oldFiles = Directory.GetFiles(localAssetPath);
                    for (int i = 0; i < oldFiles.Length; i++)
                    {
                        File.Delete(oldFiles[i]);
                    }
    
                    Directory.Delete(localAssetPath);
                    Directory.CreateDirectory(localAssetPath);
                }
                string[] files = Directory.GetFiles(remoteAssetPath);
                for (int i = 0; i < files.Length; i++)
                {
                    string fileName = files[i].Substring(files[i].LastIndexOf("/", StringComparison.Ordinal) + 1);
                    File.Copy(remoteAssetPath + fileName, localAssetPath + fileName);
                }
            }
    
            // 加载UI
            void LoadPanel()
            {
                LoadAssetBundleInstance("luahotupdate_ui.ab", "Panel", transform);
            }
    
            public static GameObject LoadAssetBundleInstance(string bundleName, string objName, Transform parent = null)
            {
                AssetBundle bundle = LoadAssetBundle(bundleName);
                GameObject obj = bundle.LoadAsset<GameObject>(objName);
                return Instantiate(obj, parent);
            }
    
            // 加载AssetBundle包以及依赖文件
            public static AssetBundle LoadAssetBundle(string bundleName)
            {
                if (manifest == null)
                {
                    string platStr = "";
    #if UNITY_STANDALONE_WIN
                    platStr = "Windows";
    #elif UNITY_STANDALONE_OSX
                    platStr = "Mac OS";
    #elif UNITY_IOS
                    platStr = "IOS";
    #elif UNITY_ANDROID
                    platStr = "Android";
    #endif
    
                    AssetBundle platBundle = AssetBundle.LoadFromFile(localAssetPath + platStr);
                    manifest = platBundle.LoadAsset<AssetBundleManifest>("AssetBundleManifest");
                }
    
                if (bundleCaches.ContainsKey(bundleName))
                {
                    return bundleCaches[bundleName];
                }
                else
                {
                    AssetBundle bundle = AssetBundle.LoadFromFile(localAssetPath + bundleName);
                    bundleCaches[bundleName] = bundle;
    
                    // 注意：GetAllDependencies会返回直接和间接关联的AssetBundle
                    string[] depends = manifest.GetAllDependencies(bundleName);
                    for (int i = 0; i < depends.Length; i++)
                    {
                        if (!bundleCaches.ContainsKey(depends[i]))
                        {
                            AssetBundle depAB = AssetBundle.LoadFromFile(localAssetPath + depends[i]);
                            bundleCaches[depends[i]] = depAB;
                        }
                    }
                    return bundle;
                }
            }
        }
    }
```

🔚