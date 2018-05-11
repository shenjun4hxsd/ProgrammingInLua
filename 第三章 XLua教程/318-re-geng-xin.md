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