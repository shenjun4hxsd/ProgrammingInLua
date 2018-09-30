##复杂转换

**ComplexConvert.lua.txt**

```lua
    local studentInfo = {
        name = "zs",
        age = 18,
        scores = {
            90, 80, 100
        }
    }
    
    local students = {}
    students[1] = studentInfo;
    students[1].name = "xm"
    
    students[2] = { name = "ls", age = 20 }
    students[2].scores = { 90, 50, 100 }
    
    
    -- 创建Unity1711对象
    local unity1711 = CS.shenjun.Unity1711()
    unity1711:Add(studentInfo)
    unity1711:Add(students[1])
    unity1711:Add(students[2])
    
    unity1711:ShowAll()
    
    -- 获取所有成绩都及格的学生数据
    print("show pass ======================")
    local pass = unity1711:GetPass()
    for i=1, pass.Length do
        CS.UnityEngine.Debug.Log(pass[i-1])
    end
    
    -- 获取所有某成绩不及格的学生数据
    print("show fail ======================")
    local fail = unity1711:GetFail()
    
    for i=1, fail.Length do
        CS.UnityEngine.Debug.Log(fail[i-1])
    end
```

**ComplexConvert.cs**


```csharp
    /*
     *  created by shenjun
     */
    
    using System.Collections;
    using System.Collections.Generic;
    using System.Linq;
    using UnityEngine;
    using XLua;
    
    namespace shenjun
    {
        public class ComplexConvert : MonoBehaviour
        {
            void Start()
            {
                LuaEnv luaEnv = new LuaEnv();
                luaEnv.DoString("require 'ComplexConvert'");
                luaEnv.Dispose();
            }
        }
    
        public class Unity1711
        {
            public List<StudentInfo> studentsInfo = new List<StudentInfo>();
    
            public void Add(StudentInfo s)
            {
                studentsInfo.Add(s);
            }
    
            public void ShowAll()
            {
                foreach (var item in studentsInfo)
                {
                    Debug.Log(item);
                }
            }
    
            // 获取所有成绩都及格的学生数据
            public StudentInfo[] GetPass()
            {
                var result = studentsInfo.Where(
                    n =>
                    {
                        return n.scores.All(m => m > 60);
                    }
                );
                return result.ToArray();
            }
    
            // 获取所有某成绩不及格的学生数据
            public StudentInfo[] GetFail()
            {
                var result = studentsInfo.Where(
                    n =>
                    {
                        return n.scores.Any(m => m < 60);    
                    }
                );
                return result.ToArray();
            }
        }
    
        [XLua.LuaCallCSharp]
        public struct StudentInfo
        {
            public string name;
            public int age;
            public int[] scores;
    
            public override string ToString()
            {
                //var score = from n in scores select n + "";
                var score = scores.Select(n => n + "");
                string scoreStr = string.Join("#", score.ToArray());
                return string.Format("[name:{0}, age:{1}, score:{2}]", name, age, scoreStr);
            }
        }
    }
```

🔚