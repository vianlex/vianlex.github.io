# SecurityManager 笔记

## 简介

Java SecurityManager 安全管理器，配合安全策略文件使用，能够控制代码对敏感或关键资源的访问，例如文件系统，网络服务，系统属性访问等，加强代码的安全性。注意读取程序类路径中的文件是不需要进行显式授权，默认已有读取执行删除的权限。

简单使用例子：

```java
import java.io.BufferedReader;
import java.io.File;
import java.io.FileReader;
import java.util.Objects;

public class SecurityManagerTest {

    public static void main(String[] args) {

        // 代码中启动安全管理器，或者通过指定参数启动安全管理  java -Djava.security.manager xxx.jar
	    SecurityManager manager = new SecurityManager();
	    if(Objects.isNull(System.getSecurityManager())) {
            // 开启安全管理器，安全管理器默认的安全策略配置文件是 JAVA_HOME/jre/lib/security/java.policy
	    	System.setSecurityManager(manager);
	    }
	    
	    try {
	    	// 检查是否有访问属性权限，没有的话，抛出异常
	    	manager.checkPropertyAccess("java.home");
	    	System.out.println(System.getProperty("java.home"));
	    }catch (SecurityException e) {
	    	System.out.println("没有读取 java.home 属性的权限");
		}
	    	
		try (FileReader fileReader = new FileReader(new File("D:\\test-security.txt"));
				BufferedReader reader = new BufferedReader(fileReader)) {

			for (String text = reader.readLine(); Objects.nonNull(text); text = reader.readLine()) {
				System.out.println(text);
			}
		} catch (Exception e) {
			e.printStackTrace();
		}

	}

}
```

上述代码设置安全管理，因为默认的 java.policy 安全策略文件未配置读取该文件的权限，则会报如下错误：

```java
java.security.AccessControlException: access denied ("java.io.FilePermission" "D:\test-security.txt" "read")
	at java.security.AccessControlContext.checkPermission(AccessControlContext.java:457)
	at java.security.AccessController.checkPermission(AccessController.java:884)
	at java.lang.SecurityManager.checkPermission(SecurityManager.java:549)
	at java.lang.SecurityManager.checkRead(SecurityManager.java:888)
	at java.io.FileInputStream.<init>(FileInputStream.java:127)
	at java.io.FileReader.<init>(FileReader.java:72)
	at com.ils.securityManager.SecurityManagerTest.main(SecurityManagerTest.java:16)
```

开启安全管理方式如下：

1. 通过编码方式

```java
System.setSecurityManager(new SecurityManager());
```
2. 通过设置 Java 程序启动参数方式

```bash
java -Djava.security.manager xxx.jar
```

安全策略文件默认使用的是 JAVA_HOME/jre/lib/security/java.policy 和哟用户目录下的 .jara.policy 文件，当然可以另外指定，支持以下两种方式指定：

1. 通过编码方式

```java
// 指定类路径下的文件
System.setProperty("java.security.policy", SecurityManagerTest.class.getResource("/")+"testFile.policy");
// 指定全路径
System.setProperty("java.security.policy","D:/testFile.policy");
```
2. 通过 Java 程序启动参数方式

```bash
# 表示优先查找当前策略文件权限，如果查找不到，则去查找默认的 JAVA_HOME/jre/lib/security/java.policy 安全策略文件
-Djava.security.policy=D:/testFile.policy
# == 符号表示查找权限只查找当前指定策略文件
-Djava.security.policy==D:/testFile.policy
```

上述例子，配置安全策略权限有两种如下：

1. 在默认的安全策略文件中添加

```
grant { 
    // 在 grant 语句处添加如下权限，表示所有资源的可以访问，一般不建议这么做
    permission java.security.AllPermission;
    // 或者添加单独的文件访问权限
    permission java.io.FilePermission "D:\\test-security.txt", read;
}
```
2. 在项目的资源根路径下下新建一个安全策略配置文件，文件名 testFile.policy，内容如下：

```
grant { 
    permission java.io.FilePermission "D:\\test-security.txt", "read";
}
```
然后将代码修改如下：

```java
public class SecurityManagerTest {

    public static void main(String[] args) {

        // 代码中启动安全管理器，或者通过指定参数启动安全管理  java -Djava.security.manager -Djava.security.policy=xxx/xx/my.policy   xxx.jar
        System.setProperty("java.security.policy", SecurityManagerTest.class.getResource("/")+"testFile.policy");
        // 注意安全管理器会优先使用命令行或者 system.setPropety 设置的 policy 文件，如果权限不匹配在去查找默认的策略文件
	    SecurityManager manager = new SecurityManager();
	    if(Objects.isNull(System.getSecurityManager())) {
            // 开启安全管理器，安全管理器默认的安全策略配置文件是 JAVA_HOME/jre/lib/security/java.policy
	    	System.setSecurityManager(manager);
	    }
	    try {
	    	// 检查是否有访问属性权限，没有的话，抛出异常
	    	manager.checkPropertyAccess("java.home");
	    	System.out.println(System.getProperty("java.home"));
	    }catch (SecurityException e) {
	    	System.out.println("没有读取 java.home 属性的权限");
		}
	    	
		try (FileReader fileReader = new FileReader(new File("D:\\test-security.txt"));
				BufferedReader reader = new BufferedReader(fileReader)) {

			for (String text = reader.readLine(); Objects.nonNull(text); text = reader.readLine()) {
				System.out.println(text);
			}
		} catch (Exception e) {
			e.printStackTrace();
		}

	}

}
```


## 安全管理策略文件

以下是安全策略文件的简单语法说明，具体详细说明可以查看[官方文档](https://docs.oracle.com/javase/8/docs/technotes/guides/security/PolicyFiles.html)

1. 语法格式说明

```java
// 表示签名秘钥文件路径
keystore "http://foo.example.com/blah/.keystore";

// 授权条目，其中 signedBy、principal、codeBase 语句都是可选的，它们是与的关系。
grant signedBy "signer_names", codeBase "URL", principal principal_class_name "principal_name", 
    principal principal_class_name "principal_name" {
    
      // 通过 signedBy 指定签名人
      permission permission_class_name "target_name", "action", signedBy "signer_names";;
      permission permission_class_name "target_name", "action";
  }

grant codeBase "URL" {
      permission permission_class_name "target_name", "action";
      permission permission_class_name "target_name", "action";
  }

grant {
      permission permission_class_name "target_name", "action";
      permission permission_class_name "target_name", "action";
}

// 表示允许任何以 X500Principal，"cn=Alice " 的身份执行的代码，有权限读写 "/home/Alice" 资源
grant principal javax.security.auth.x500.X500Principal "cn=Alice" {
      permission java.io.FilePermission "/home/Alice", "read, write";
};

// 表示允许任何以 X500Principal 的任何身份执行的代码，都有权限读写 "/home/Alice" 资源
grant principal javax.security.auth.x500.X500Principal * {
      permission java.io.FilePermission "/home/Alice", "read, write";
};

```
每一个 policy 文件可以包含多个 grant 授权条目，其中 signedBy、principal、codeBase 语句都是可选的。

signedBy 表示针对某个签名者赋予权限，签名例子如下：

```
jarsigner -verbose -keystore [私钥存放路径] -signedjar [签名后文件存放路径] [未签名的文件路径] [证书别名]

jarsigner -verbose -keystore /xx/path/keystore -signedjar ./signed-xxx.jar ./xxx.jar signer_names

```
codeBase 用于指定运行 URL 目录下的代码，才能赋予配置的 permission 权限。注意如果 URL 以 / 结尾的匹配指定目录下的所有类文件（ 非 JAR 文件 ），以  /*  结尾的匹配该目录下的所有文件（类文件和 JAR 文件），以 /- 结尾的匹配该目录下的所有文件（类文件和 JAR 文件）及该目录下子目录中的所有文件，直接以目录名结尾的等同于 / 结尾的。 

principal 表示针对证书或者主体认证的 class_name/principal_name 对，授予资源的读取权限。

2. 例子说明

```
// 表示扩展包目录的下的代码，使用资源时，授予所有资源权限
grant codeBase "file:${{java.ext.dirs}}/*" {
        // 开放所以权限
        permission java.security.AllPermission;
};

// 表示该 URL 目录中的代码执行时，能获取所有资源的权限
grant codeBase "file:C:/Users/xxpath/learn-test/target/classes" { 
    permission java.security.AllPermission;
};

grant {
   
    // java.security.AllPermission  所有权限的集合
    permission java.security.AllPermission;  // 表示开发所有权限

    // java.util.PropertyPermission 系统/环境属性权限 
    permission java.util.PropertyPermission "java.home", "read";
    permission java.util.PropertyPermission "java.version", "read";
    permission java.util.PropertyPermission "java.vendor", "read";
    permission java.util.PropertyPermission "java.vendor.url", "read";

    // ... 等等其他权限

};

```


## 参考链接
1. https://docs.oracle.com/javase/8/docs/technotes/guides/security/permissions.html
2. https://docs.oracle.com/javase/8/docs/technotes/guides/security/PolicyFiles.html