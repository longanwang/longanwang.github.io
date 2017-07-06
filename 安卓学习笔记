Failed to load F:\android-sdk\build-tools\26.0.0\lib\dx.jar  Unknown error: Unable to build: the file dx.jar was not loaded from the SDK folder!



出现这和问题，把25.0中的jar文件拷贝到出错的文件夹中，替换掉。





ReforceAPK无论该名成什么，最后安装到具体设备中都是原来这个名字。






重新给其他apk加固，需要在脱壳程序的AndroidManifest.xml中加上源程序中用到的activity，
<activity
            android:name="com.example.forceapkobj.MainActivity"
            android:label="@string/app_name" >
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />
                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>
        
        <activity
            android:name="com.example.forceapkobj.SubActivity"></activity>
        
</application>



作者：谢榭
链接：https://www.zhihu.com/question/31412585/answer/53427544
来源：知乎
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。
自问自答吧 呵呵 
第一步：打包资源文件，生成R.java文件
【输入】Resource文件（就是工程中res中的文件）、Assets文件（相当于另外一种资源，这种资源Android系统并不像对res中的文件那样优化它）、AndroidManifest.xml文件（包名就是从这里读取的，因为生成R.java文件需要包名）、Android基础类库（Android.jar文件）
【输出】打包好的资源（一般在Android工程的bin目录可以看到一个叫resources.ap_的文件就是它了）、R.java文件（在gen目录中，大家应该很熟悉了）
【工具】aapt工具，它的路径在${ANDROID_SDK_HOME}/platform-tools/aapt（如果你使用的是Windows系统，按惯例路径应该这样写：%ANDROID_SDK_HOME%\platform-tools\aapt.exe，下同）。
第二步：处理AIDL文件，生成对应的.java文件（当然，有很多工程没有用到AIDL，那这个过程就可以省了）
【输入】源码文件、aidl文件、framework.aidl文件
【输出】对应的.java文件
【工具】aidl工具
第三步：编译Java文件，生成对应的.class文件
【输入】源码文件（包括R.java和AIDL生成的.java文件）、库文件（.jar文件）
【输出】.class文件
【工具】javac工具
第四步：把.class文件转化成Davik VM支持的.dex文件
【输入】源码文件（包括R.java和AIDL生成的.java文件）、库文件（.jar文件）
【输出】.dex文件
【工具】javac工具
第五步：打包生成未签名的.apk文件
【输入】打包后的资源文件、打包后类文件（.dex文件）、libs文件（包括.so文件，当然很多工程都没有这样的文件，如果你不使用C/C++开发的话）
【输出】未签名的.apk文件
【工具】apkbuilder工具
第六步：对未签名.apk文件进行签名
【输入】未签名的.apk文件
【输出】签名的.apk文件
【工具】jarsigner
第七步：对签名后的.apk文件进行对齐处理（不进行对齐处理是不能发布到Google Market的）
【输入】签名后的.apk文件
【输出】对齐后的.apk文件
【工具】zipalign工具

 
Java高新技术第一篇：类加载器详解
2014-01-01 14:45 15893人阅读 评论(10) 收藏 举报
  分类：
Java（34）  
版权声明：本文为博主原创文章，未经博主允许不得转载。
目录(?)[+]
首先来了解一下字节码和class文件的区别：
我们知道，新建一个Java对象的时候，JVM要将这个对象对应的字节码加载到内存中，这个字节码的原始信息存放在classpath(就是我们新建Java工程的bin目录下)指定的目录下的.class文件,类加载需要将.class文件导入到硬盘中，经过一些处理之后变成字节码在加载到内存中。
下面来看一下简单的例子：
[java] view plain copy
1.	package com.loadclass.demo;  
2.	  
3.	import java.util.Date;  
4.	import java.util.List;  
5.	  
6.	/** 
7.	 * 测试类 
8.	 * @author Administrator 
9.	 */  
10.	public class ClassLoaderTest {  
11.	  
12.	    @SuppressWarnings("rawtypes")  
13.	    public static void main(String[] args){  
14.	        //输出ClassLoaderText的类加载器名称  
15.	        System.out.println("ClassLoaderText类的加载器的名称:"+ClassLoaderTest.class.getClassLoader().getClass().getName());  
16.	        System.out.println("System类的加载器的名称:"+System.class.getClassLoader());  
17.	        System.out.println("List类的加载器的名称:"+List.class.getClassLoader());  
18.	          
19.	        ClassLoader cl = ClassLoaderTest.class.getClassLoader();  
20.	        while(cl != null){  
21.	            System.out.print(cl.getClass().getName()+"->");  
22.	            cl = cl.getParent();  
23.	        }  
24.	        System.out.println(cl);  
25.	    }  
26.	      
27.	}  
输出结果：
 
可以看到，ClassLoaderTest类时由AppClassLoader类加载器加载的。下面就来了解一下JVM中的各个类加载器，同时来解释一下运行的结果。
Java虚拟机中类加载器：
Java虚拟机中可以安装多个类加载器，系统默认三个主要的类加载器，每个类负责加载特定位置的类：
BootStrap,ExtClassLoader,AppClassLoader
类加载器也是Java类，因为Java类的类加载器本身也是要被类加载器加载的，显然必须有第一个类加载器不是Java类，这个正是BootStrap,使用C/C++代码写的，已经封装到JVM内核中了，而ExtClassLoader和AppClassLoader是Java类。
看一下类加载器的属性结构图：
 
Java虚拟机中的所有类加载器采用具有父子关系的树形结构进行组织，在实例化每个类加载器对象的时候，需要为其指定一个父级类加载器对象或者默认采用系统类加载器为其父级类加载
类加载器的委托机制：
当Java虚拟机要加载第一个类的时候，到底派出哪个类加载器去加载呢？
(1). 首先当前线程的类加载器去加载线程中的第一个类(当前线程的类加载器：Thread类中有一个get/setContextClassLoader(ClassLoader cl);方法，可以获取/指定本线程中的类加载器)
(2). 如果类A中引用了类B,Java虚拟机将使用加载类A的类加载器来加载类B
(3). 还可以直接调用ClassLoader.loadClass(String className)方法来指定某个类加载器去加载某个类
每个类加载器加载类时，又先委托给其上级类加载器当所有祖宗类加载器没有加载到类，回到发起者类加载器，还加载不了，则会抛出ClassNotFoundException,不是再去找发起者类加载器的儿子，因为没有getChild()方法。例如：如上图所示： MyClassLoader->AppClassLoader->Ext->ClassLoader->BootStrap.自定定义的MyClassLoader1首先会先委托给AppClassLoader,AppClassLoader会委托给ExtClassLoader,ExtClassLoader会委托给BootStrap，这时候BootStrap就去加载，如果加载成功，就结束了。如果加载失败，就交给ExtClassLoader去加载，如果ExtClassLoader加载成功了，就结束了，如果加载失败就交给AppClassLoader加载，如果加载成功，就结束了，如果加载失败，就交给自定义的MyClassLoader1类加载器加载，如果加载失败，就报ClassNotFoundException异常，结束。

对着类加载器的层次结构图和委托加载原理，解释先前的运行的结果
因为System类，List，Map等这样的系统提供jar类都在rt.jar中，所以由BootStrap类加载器加载，因为BootStrap是祖先类，不是Java编写的，所以打印出class为null
对于ClassLoaderTest类的加载过程，打印结果也是很清楚的。

现在再来做个试验来验证上面的结论：
首先将ClassLoaderTest.java打包成.jar文件(这个步骤就不说了吧,很简单的)
然后将.jar文件拷贝到Java的安装目录中的Java/jre7/lib/ext/目录下
 
这时候你在运行ClassLoaderTest类，结果如下：
 
这时候就发现了ClassLoaderTest的类加载器变成了ExtClassLoader,这时候就说明了上面的结论是正确的，因为ExtClassLoader加载jre/ext/*.jar，首先AppClassLoader类加载器发请求给ExtClassLoader,然后ExtClassLoader发请求给BootStrap，但是BootStrap没有找到ClassLoaderTest类，所以交给ExtClassLoader处理，这时候ExtClassLoader在my_lib.jar中找到了ClassLoaderTest类，所以就把它加载了，然后结束了。
其实采用这种树形的类加载机制的好处就在于：
能够很好的统一管理类加载，首先交给上级，如果上级有了，就加载，这样如果之前已经加载过的类，这时候在来加载它的时候只要拿过来用就可以了，无需二次加载了

下面来看一下怎么定义我们自己的一个类加载器MyClassLoader:
自己可以定义类加载器，要将自己定义的类加载器挂载到系统类加载器树上，在ClassLoader的构造方法中可以指定parent,没有指定的话，就使用默认的parent
 
这里看一下默认的parent是使用getSystemClassLoader方法获取的，这个方法的源码没有找到，所以只能通过代码来测试一下了
[java] view plain copy
1.	System.out.println("默认的类加载器:"+ClassLoaderTest.class.getClassLoader().getSystemClassLoader());  
输入结果为：
 
所以默认的都是将自定义的类加载器挂载到系统类加载器的最低端AppClassLoader，这个也是很合理的。

自定义的类加载器必须继承抽象类ClassLoader然后重写findClass方法，其实他内部还有一个loadClass方法和defineClass方法，这两个方法的作用是：
loadClass方法的源代码：
[java] view plain copy
1.	public Class<?> loadClass(String name) throws ClassNotFoundException {  
2.	       return loadClass(name, false);  
3.	   }  
再来看一下loadClass(name,false)方法的源代码：
[java] view plain copy
1.	protected Class<?> loadClass(String name, boolean resolve)throws ClassNotFoundException{  
2.	         //加上锁，同步处理，因为可能是多线程在加载类  
3.	         synchronized (getClassLoadingLock(name)) {  
4.	             //检查，是否该类已经加载过了，如果加载过了，就不加载了  
5.	             Class c = findLoadedClass(name);  
6.	             if (c == null) {  
7.	                 long t0 = System.nanoTime();  
8.	                 try {  
9.	                     //如果自定义的类加载器的parent不为null,就调用parent的loadClass进行加载类  
10.	                     if (parent != null) {  
11.	                         c = parent.loadClass(name, false);  
12.	                     } else {  
13.	                         //如果自定义的类加载器的parent为null，就调用findBootstrapClass方法查找类，就是Bootstrap类加载器  
14.	                         c = findBootstrapClassOrNull(name);  
15.	                     }  
16.	                 } catch (ClassNotFoundException e) {  
17.	                     // ClassNotFoundException thrown if class not found  
18.	                     // from the non-null parent class loader  
19.	                 }  
20.	  
21.	                 if (c == null) {  
22.	                     // If still not found, then invoke findClass in order  
23.	                     // to find the class.  
24.	                     long t1 = System.nanoTime();  
25.	                     //如果parent加载类失败，就调用自己的findClass方法进行类加载  
26.	                     c = findClass(name);  
27.	  
28.	                     // this is the defining class loader; record the stats  
29.	                     sun.misc.PerfCounter.getParentDelegationTime().addTime(t1 - t0);  
30.	                     sun.misc.PerfCounter.getFindClassTime().addElapsedTimeFrom(t1);  
31.	                     sun.misc.PerfCounter.getFindClasses().increment();  
32.	                 }  
33.	             }  
34.	             if (resolve) {  
35.	                 resolveClass(c);  
36.	             }  
37.	             return c;  
38.	         }  
39.	     }  
在loadClass代码中也可以看到类加载机制的原理，这里还有这个方法findBootstrapClassOrNull,看一下源代码：
[java] view plain copy
1.	private Class findBootstrapClassOrNull(String name)  
2.	   {  
3.	       if (!checkName(name)) return null;  
4.	  
5.	       return findBootstrapClass(name);  
6.	   }  

就是检查一下name是否是否正确，然后调用findBootstrapClass方法，但是findBootstrapClass方法是个native本地方法，看不到源代码了，但是可以猜测是用Bootstrap类加载器进行加载类的，这个方法我们也不能重写，因为如果重写了这个方法的话，就会破坏这种委托机制，我们还要自己写一个委托机制，很是蛋疼的。

defineClass这个方法很简单就是将class文件的字节数组编程一个class对象，这个方法肯定不能重写，内部实现是在C/C++代码中实现的

findClass这个方法就是根据name来查找到class文件，在loadClass方法中用到，所以我们只能重写这个方法了，只要在这个方法中找到class文件，再将它用defineClass方法返回一个Class对象即可。

这三个方法的执行流程是：每个类加载器：loadClass->findClass->defineClass

前期的知识了解后现在就来实现了
首先来看一下需要加载的一个类:ClassLoaderAttachment.java:
[java] view plain copy
1.	package com.loadclass.demo;  
2.	  
3.	import java.util.Date;  
4.	/** 
5.	 * 加载类 
6.	 * @author Administrator 
7.	 */  
8.	public class ClassLoaderAttachment extends Date{  
9.	    private static final long serialVersionUID = 8627644427315706176L;  
10.	    //打印数据  
11.	    @Override  
12.	    public String toString(){  
13.	        return "Hello ClassLoader!";  
14.	    }  
15.	  
16.	}  
这个类中输出一段话即可：编译成ClassLoaderAttachment.class

再来看一下自定义的MyClassLoader.java:
[java] view plain copy
1.	package com.loadclass.demo;  
2.	  
3.	import java.io.ByteArrayOutputStream;  
4.	import java.io.FileInputStream;  
5.	import java.io.FileOutputStream;  
6.	import java.io.InputStream;  
7.	import java.io.OutputStream;  
8.	  
9.	/** 
10.	 * 自定义的类加载器 
11.	 * @author Administrator 
12.	 */  
13.	public class MyClassLoader extends ClassLoader{  
14.	      
15.	    //需要加载类.class文件的目录  
16.	    private String classDir;  
17.	      
18.	    //无参的构造方法，用于class.newInstance()构造对象使用  
19.	    public MyClassLoader(){  
20.	    }  
21.	      
22.	    public MyClassLoader(String classDir){  
23.	        this.classDir = classDir;  
24.	    }  
25.	  
26.	    @SuppressWarnings("deprecation")  
27.	    @Override  
28.	    protected Class<?> findClass(String name) throws ClassNotFoundException {  
29.	        //class文件的路径  
30.	        String classPathFile = classDir + "/" + name + ".class";  
31.	        try {  
32.	            //将class文件进行解密  
33.	            FileInputStream fis = new FileInputStream(classPathFile);  
34.	            ByteArrayOutputStream bos = new ByteArrayOutputStream();  
35.	            encodeAndDecode(fis,bos);  
36.	            byte[] classByte = bos.toByteArray();  
37.	            //将字节流变成一个class  
38.	            return defineClass(classByte,0,classByte.length);  
39.	        } catch (Exception e) {  
40.	            e.printStackTrace();  
41.	        }  
42.	        return super.findClass(name);  
43.	    }  
44.	  
45.	    //测试，先将ClassLoaderAttachment.class文件加密写到工程的class_temp目录下  
46.	    public static void main(String[] args) throws Exception{  
47.	        //配置运行参数  
48.	        String srcPath = args[0];//ClassLoaderAttachment.class原路径  
49.	        String desPath = args[1];//ClassLoaderAttachment.class输出的路径  
50.	        String desFileName = srcPath.substring(srcPath.lastIndexOf("\\")+1);  
51.	        String desPathFile = desPath + "/" + desFileName;  
52.	        FileInputStream fis = new FileInputStream(srcPath);  
53.	        FileOutputStream fos = new FileOutputStream(desPathFile);  
54.	        //将class进行加密  
55.	        encodeAndDecode(fis,fos);  
56.	        fis.close();  
57.	        fos.close();  
58.	    }  
59.	      
60.	    /** 
61.	     * 加密和解密算法 
62.	     * @param is 
63.	     * @param os 
64.	     * @throws Exception 
65.	     */  
66.	    private static void encodeAndDecode(InputStream is,OutputStream os) throws Exception{  
67.	        int bytes = -1;  
68.	        while((bytes = is.read())!= -1){  
69.	            bytes = bytes ^ 0xff;//和0xff进行异或处理  
70.	            os.write(bytes);  
71.	        }  
72.	    }  
73.	      
74.	}  
这个类中定义了一个加密和解密的算法，很简单的，就是将字节和oxff异或一下即可，而且这个算法是加密和解密的都可以用，很是神奇呀！

当然我们还要先做一个操作就是，将ClassLoaderAttachment.class加密后的文件存起来，也就是在main方法中执行的，这里我是在项目中新建一个class_temp文件夹用来皴法加密后的class文件：
 
同时采用的是参数的形式来进行赋值的，所以在运行的MyClassLoader的时候要进行输入参数的配置：右击MyClassLoader->run as -> run configurations
 
第一个参数是ClassLoaderAttachment.class文件的源路径，第二个参数是加密后存放的目录，运行MyClassLoader之后，刷新class_temp文件夹，出现了ClassLoaderAttachment.class，这个是加密后的class文件。

下面来看一下测试类：
[java] view plain copy
1.	package com.loadclass.demo;  
2.	  
3.	import java.util.Date;  
4.	import java.util.List;  
5.	  
6.	/** 
7.	 * 测试类 
8.	 * @author Administrator 
9.	 */  
10.	public class ClassLoaderTest {  
11.	  
12.	    @SuppressWarnings("rawtypes")  
13.	    public static void main(String[] args){  
14.	        //输出ClassLoaderText的类加载器名称  
15.	        System.out.println("ClassLoaderText类的加载器的名称:"+ClassLoaderTest.class.getClassLoader().getClass().getName());  
16.	        System.out.println("System类的加载器的名称:"+System.class.getClassLoader());  
17.	        System.out.println("List类的加载器的名称:"+List.class.getClassLoader());  
18.	          
19.	        System.out.println("默认的类加载器:"+ClassLoaderTest.class.getClassLoader().getSystemClassLoader());  
20.	          
21.	        ClassLoader cl = ClassLoaderTest.class.getClassLoader();  
22.	        while(cl != null){  
23.	            System.out.print(cl.getClass().getName()+"->");  
24.	            cl = cl.getParent();  
25.	        }  
26.	        System.out.println(cl);  
27.	          
28.	        try {  
29.	            Class classDate = new MyClassLoader("class_temp").loadClass("ClassLoaderAttachment");  
30.	            Date date = (Date) classDate.newInstance();  
31.	            //输出ClassLoaderAttachment类的加载器名称  
32.	            System.out.println("ClassLoader:"+date.getClass().getClassLoader().getClass().getName());  
33.	            System.out.println(date);  
34.	        } catch (Exception e1) {  
35.	            e1.printStackTrace();  
36.	        }  
37.	    }  
38.	      
39.	       
40.	  
41.	      
42.	}  
运行ClassLoaderTest类，运行结果如下：
 
ClassLoaderAttachment类的加载器是我们自己定义的类加载器MyClassLoader，同时也输出了Hello ClassLoader字段

到此不要以为结束了，这里还有很多的问题呀，看一下问题的结果是没有问题，但是这里面还有很多的东西需要去理解的，首先来看一下，按照我们之前说的类加载机制，应该是先交给父级的类加载器，AppClassLoader->ExtClassLoader->BootStrap，ExtClassLoader和BootStrap没有找到ClassLoaderAttach.class，但是AppClassLoader类加载器应该能找到呀，可以为什么也没有找到呢？这时因为loadClass方法在使用系统类加载器的时候需要传递全称(包括包名)，我们传递ClassLoaderAttachment的话，AppClassLoader也是没有找到这个ClassLoaderAttachment,所以还是MyClassLoader处理了，不信的话可以试一下：
现在将
[java] view plain copy
1.	Class classDate = new MyClassLoader("class_temp").loadClass("ClassLoaderAttachment");  
改成：
[java] view plain copy
1.	Class classDate = new MyClassLoader("class_temp").loadClass("com.loadclass.demo.ClassLoaderAttachment");  
结果运行：
 
这时候的加载器是AppClassLoader了，所以要注意loadClass方法传递的参数
到这里我们貌似还没有测试到我们加密后的class文件，我们现在将工程目录中的ClassLoaderAttachment.class删除，将class_temp中加密的ClassLoaderAttachment.class拷贝过去，然后再运行：
 
这时候就会报错了，不合适的魔数错误(class文件的前六个字节是个魔数用来标识class文件的)，这时候就对了，因为ClassLoaderAttachment.class使我们加密后的class文件,AppClassLoader是不认识的，所以报这个错误了，只有用我们自己定义的类加载器来进行解密才可以正确的访问的。到这里总算是说完了，搞了一上午，头都写大了，很是麻烦呀！

注意的两个问题：
1.可能在测试的过程中有这样的情况就是ClassLoaderTest类并没有执行，这个是因为在第一个测试的时候，将ClassLoaderTest类打成.jar放到jre目录中了，所以你后续修改ClassLoaderTest类的话，运行没有效果，因为它加载的类还是jre中的jar中的ClassLoaderTest类，所以你应该将jre中的jar删除即可。
2.就是ClassLoaderAttachment只要保存就会编译成.class文件，所以你在拷贝ClassLoaderAttachment.class文件的时候要注意了。




今天来学习一下java.lang.reflect包下有关反射的相关内容，提供类和接口，以获得关于类和对象的反射信息。在安全限制内，反射允许编程访问关于加载类的字段、方法和构造方法的信息，并允许使用反射字段、方法和构造方法对其底层对等项进行操作。
下面主要介绍如下几个类：
1.Class
    万事万物皆对象，平时我们创建的USer、Role类就是Class的实例对象,Class是对类的描述，即类类型。class类的实例表示java应用运行时的类(class ans enum)或接口(interface and annotation)。数组同样也被映射为为class 对象的一个类，所有具有相同元素类型和维数的数组都共享该 Class 对象。基本类型boolean，byte，char，short，int，long，float，double和关键字void同样表现为 class 对象。
  Class类没有公有的构造方法，它由JVM自动调用，我有如下3种方式获取Class
  （1）某个对象实例的getClass()方法，如new User().getClass()
  （2）某个类名.class属性，如User.class
  （3）通过Class.forName("类名")获取

 我们看一下下面的实例：
User对象：
[java] view plain copy
 print?
1.	package cn.slimsmart.java.demo.reflect;  
2.	  
3.	public class User {  
4.	      
5.	    public String name;  
6.	    private int age;  
7.	      
8.	    public User(){  
9.	    }  
10.	    public User(String name,int age){  
11.	        this.name=name;  
12.	        this.age=age;  
13.	    }  
14.	  
15.	    public String getName() {  
16.	        return name;  
17.	    }  
18.	  
19.	    public void setName(String name) {  
20.	        this.name = name;  
21.	    }  
22.	  
23.	    public int getAge() {  
24.	        return age;  
25.	    }  
26.	  
27.	    public void setAge(int age) {  
28.	        this.age = age;  
29.	    }  
30.	}  
ClassTest类：
[java] view plain copy
 print?
1.	package cn.slimsmart.java.demo.reflect;  
2.	  
3.	public class ClassTest {  
4.	    public static void main(String[] args) throws ClassNotFoundException, InstantiationException, IllegalAccessException {  
5.	        User user = new User();  
6.	        Class c1 = user.getClass();  
7.	        Class c2 = User.class;  
8.	        Class c3 = Class.forName("cn.slimsmart.java.demo.reflect.User");  
9.	        System.out.println(c1==c2);  
10.	        System.out.println(c2==c3);  
11.	        ///通过newInstance方法创建一个user实例对象，使用newInstance方法必须要有一个公共的构造方法  
12.	        user = (User)c3.newInstance();  
13.	    }  
14.	  
15.	}  
从上面实例可以看出，我们获取的c1,c2,c3都是同一个class，是对User类的表示。
Class有以下几个类主要方法：
（1）forName 获取给定字符串名的类或接口相关联的 Class 对象
（2）getAnnotation 获取该类的类注解
（3）getClasses 获取该类的父类或接口
（4）获取构造方法
（5）获取成员属性
（6）获取成员方法
（7）newInstance 创建此 Class 对象所表示的类的一个新实例。
方法中带Declared获取该类的所有属性、方法、注解等，不包含继承的，不带Declared的是只包含所有公共的属性、方法、注解等
2.Field
 描述类的成员属性，如user.name属性，Field 提供有关类或接口的单个字段的信息，以及对它的动态访问权限。反射的字段可能是一个类（静态）字段或实例字段。可以通过class的getDeclaredField(String name),getDeclaredFields(),getField(String name),getFields()获取，通过Field的方法可以获取、设置属性的值，并能获取属性的注解、字段的声明类型。

如下实例：
[java] view plain copy
 print?
1.	package cn.slimsmart.java.demo.reflect;  
2.	  
3.	import java.lang.reflect.Field;  
4.	  
5.	public class FieldTest {  
6.	    public static void main(String[] args) throws Exception {  
7.	        User user = new User();  
8.	        user.setName("jeey");  
9.	        user.setAge(20);  
10.	        Class c1= user.getClass();  
11.	        Field f = c1.getDeclaredField("name");  
12.	        //如下操作不能针对private属性，private只能通过get和set方法访问  
13.	        System.out.println("字段类型:"+f.getType().getName());  
14.	        System.out.println("字段值："+f.get(user));  
15.	        f.set(user, "jock");  
16.	        System.out.println("修改后的字段属性值:"+user.getName());  
17.	    }  
18.	  
19.	}  
3.Method
描述类的成员方法，Method 提供关于类或接口上单独某个方法（以及如何访问该方法）的信息。所反映的方法可能是类方法或实例方法
（包括抽象方法）。Method 允许在匹配要调用的实参与底层方法的形参时进行扩展转换
可以通过class的getDeclaredMethod(String name, Class<?>... parameterTypes) ，getDeclaredMethods() ，getMethod(String name,
 Class<?>... parameterTypes)，getMethods() 获取，通过Method的invoke方法去执行，获取返回值，也可以获取方法注解、
 返回值类型等。

如下实例：
[java] view plain copy
 print?
1.	package cn.slimsmart.java.demo.reflect;  
2.	  
3.	import java.lang.reflect.Method;  
4.	  
5.	public class MethodTest {  
6.	    public static void main(String[] args) throws Exception {  
7.	        User user = new User();  
8.	        user.setName("jeey");  
9.	        user.setAge(20);  
10.	        Class c1= user.getClass();  
11.	        Method m = c1.getDeclaredMethod("getAge");  
12.	        System.out.println("获取age属性值："+m.invoke(user));  
13.	        //通过set方法修改属性值  
14.	        m = c1.getDeclaredMethod("setAge",int.class);  
15.	        m.invoke(user, 99);  
16.	        System.out.println("修改后的值："+user.getAge());  
17.	    }  
18.	}  
4.Constructor
Constructor是对构造方法的声明描述，Constructor 提供关于类的单个构造方法的信息以及对它的访问权限。Constructor 
允许在将实参与带有底层构造方法的形参的 newInstance() 匹配时进行扩展转换。可以通过class的getConstructor(Class<?>... parameterTypes) 
，getConstructors()，getDeclaredConstructor(Class<?>... parameterTypes) ，getDeclaredConstructors() 获取。通过Constructor
可以获取注解，参数类型等，并通过newInstance(Object... initargs) 创建类实例。

如下实例：
[java] view plain copy
 print?
1.	package cn.slimsmart.java.demo.reflect;  
2.	  
3.	import java.lang.reflect.Constructor;  
4.	  
5.	public class ConstructorTest {  
6.	  
7.	    public static void main(String[] args) throws Exception {  
8.	        User user = new User();  
9.	        user.setName("jeey");  
10.	        user.setAge(20);  
11.	        Class c1= user.getClass();  
12.	        Constructor cons = c1.getDeclaredConstructor(String.class,int.class);  
13.	        //通过构造方法创建实例  
14.	        User u = (User)cons.newInstance("jack",33);  
15.	        System.out.println("创建实例:name="+u.getName()+",age="+u.getAge());  
16.	    }  
17.	  
18.	}  
5.Array
Array 类提供了动态创建和访问 Java 数组的方法。允许在执行 get 或 set 操作期间进行扩展转换。
可以通过class的isArray方法判定此 Class 对象是否表示一个数组类，getComponentType返回表示数组组件类型的 Class。通过newInstance初始化
数组。

如下实例：
[java] view plain copy
 print?
1.	package cn.slimsmart.java.demo.reflect;  
2.	  
3.	import java.lang.reflect.Array;  
4.	  
5.	public class ArrayTest {  
6.	  
7.	    public static void main(String[] args) {  
8.	        String[] aa = {"aa","bb"};  
9.	        Class c1 = aa.getClass();  
10.	        System.out.println("是否是数值："+c1.isArray());  
11.	        System.out.println("数组组件类型的 Class:"+c1.getComponentType());  
12.	        //初始化数值  
13.	        String[] arr = (String[])Array.newInstance(c1.getComponentType(), 2);  
14.	        //数组设值  
15.	        Array.set(arr, 0, "111");  
16.	        Array.set(arr, 1, "222");  
17.	        //访问  
18.	        System.out.println(arr[0]);  
19.	        System.out.println(Array.get(arr, 1));  
20.	    }  
21.	  
22.	}<strong>  
23.	</strong>  
6.Proxy
    Proxy 提供用于创建动态代理类和实例的静态方法，它还是由这些方法创建的所有动态代理类的超类。
    动态代理类（以下简称为代理类）是一个实现在创建类时在运行时指定的接口列表的类，该类具有下面描述的行为。 代理接口是代理类实现的一个接口。 代理实例 是代理类的一个实例。 每个代理实例都有一个关联的调用处理程序 对象，它可以实现接口InvocationHandler。通过其中一个代理接口的代理实例上的方法调用将被指派到实例的调用处理程序的 Invoke 方法，并传递代理实例、识别调用方法的 java.lang.reflect.Method 对象以及包含参数的 Object 类型的数组。调用处理程序以适当的方式处理编码的方法调用，并且它返回的结果将作为代理实例上方法调用的结果返回。
代理类具用以下属性：
(1)代理类是公共的、最终的，而不是抽象的。
(2)未指定代理类的非限定名称。但是，以字符串 "$Proxy" 开头的类名空间应该为代理类保留。
(3)代理类扩展 java.lang.reflect.Proxy。
(4)代理类会按同一顺序准确地实现其创建时指定的接口。
(5)如果代理类实现了非公共接口，那么它将在与该接口相同的包中定义。否则，代理类的包也是未指定的。注意，包密封将不阻止代理类在运行时在特定包中的成功定义，也不会阻止相同类加载器和带有特定签名的包所定义的类。
(6)由于代理类将实现所有在其创建时指定的接口，所以对其 Class 对象调用 getInterfaces 将返回一个包含相同接口列表的数组（按其创建时指定的顺序），对其 Class 对象调用 getMethods 将返回一个包括这些接口中所有方法的 Method 对象的数组，并且调用 getMethod 将会在代理接口中找到期望的一些方法。
(7)如果 Proxy.isProxyClass 方法传递代理类（由 Proxy.getProxyClass 返回的类，或由 Proxy.newProxyInstance 返回的对象的类），则该方法返回 true，否则返回 false。
(8)代理类的 java.security.ProtectionDomain 与由引导类加载器（如 java.lang.Object）加载的系统类相同，原因是代理类的代码由受信任的系统代码生成。此保护域通常被授予 java.security.AllPermission。
(9)每个代理类都有一个可以带一个参数（接口 InvocationHandler 的实现）的公共构造方法，用于设置代理实例的调用处理程序。并非必须使用反射 API 才能访问公共构造方法，通过调用 Proxy.newInstance 方法（将调用 Proxy.getProxyClass 的操作和调用带有调用处理程序的构造方法结合在一起）也可以创建代理实例。

静态方法newProxyInstance(ClassLoader loader, Class<?>[] interfaces, InvocationHandler h) 返回一个指定接口的代理类实例，该接口可以将方法调用指派到指定的调用处理程序。
关于反射及代理的应用可以参考：
http://blog.csdn.net/zhu_tianwei/article/details/18082045
http://blog.csdn.net/zhu_tianwei/article/details/40076391



写一个动态类接口，通过目标类或者称之为连接类使用接口的实现类（插件类）。看下面文章：
  Dalvik虚拟机如同其他Java虚拟机一样，在运行程序时首先需要将对应的类加载到内存中。而在Java标准的虚拟机中，类加载可以从class文件中读取，也可以是其他形式的二进制流，因此，我们常常利用这一点，在程序运行时手动加载Class，从而达到代码动态加载执行的目的
       然而Dalvik虚拟机毕竟不算是标准的Java虚拟机，因此在类加载机制上，它们有相同的地方，也有不同之处。我们必须区别对待

       例如，在使用标准Java虚拟机时，我们经常自定义继承自ClassLoader的类加载器。然后通过defineClass方法来从一个二进制流中加载Class。然而，这在android里是行不通的，大家就没必要走弯路了。参看源码我们知道，Android中ClassLoader的defineClass方法具体是调用VMClassLoader的defineClass本地静态方法。而这个本地方法除了抛出一个“UnsupportedOperationException”之外，什么都没做，甚至连返回值都为空
[cpp] view plain copy
1.	static void Dalvik_java_lang_VMClassLoader_defineClass(const u4* args,JValue* pResult){  
2.	    Object* loader = (Object*) args[0];  
3.	    StringObject* nameObj = (StringObject*) args[1];  
4.	    const u1* data = (const u1*) args[2];  
5.	    int offset = args[3];  
6.	    int len = args[4];  
7.	    Object* pd = (Object*) args[5];  
8.	    char* name = NULL;  
9.	    name = dvmCreateCstrFromString(nameObj);  
10.	    LOGE("ERROR: defineClass(%p, %s, %p, %d, %d, %p)\n",loader, name, data, offset, len, pd);  
11.	    dvmThrowException("Ljava/lang/UnsupportedOperationException;","can't load this type of class file");  
12.	    free(name);  
13.	    RETURN_VOID();  
14.	}  

Dalvik虚拟机类加载机制
       那如果在Dalvik虚拟机里，ClassLoader不好使，我们如何实现动态加载类呢？Android为我们从ClassLoader派生出了两个类：DexClassLoader和PathClassLoader。其中需要特别说明的是PathClassLoader中一段被注释掉的代码：
[java] view plain copy
1.	/* --this doesn't work in current version of Dalvik-- 
2.	    if (data != null) { 
3.	        System.out.println("--- Found class " + name 
4.	            + " in zip[" + i + "] '" + mZips[i].getName() + "'"); 
5.	        int dotIndex = name.lastIndexOf('.'); 
6.	        if (dotIndex != -1) { 
7.	            String packageName = name.substring(0, dotIndex); 
8.	            synchronized (this) { 
9.	                Package packageObj = getPackage(packageName); 
10.	                if (packageObj == null) { 
11.	                    definePackage(packageName, null, null, 
12.	                            null, null, null, null, null); 
13.	                } 
14.	            } 
15.	        } 
16.	        return defineClass(name, data, 0, data.length); 
17.	    } 
18.	*/  


这从另一方面佐证了defineClass函数在Dalvik虚拟机里确实是被阉割了。而在这两个继承自ClassLoader的类加载器，本质上是重载了ClassLoader的findClass方法。在执行loadClass时，我们可以参看ClassLoader部分源码：
[java] view plain copy
1.	protected Class<?> loadClass(String className, boolean resolve) throws ClassNotFoundException {  
2.	Class<?> clazz = findLoadedClass(className);  
3.	    if (clazz == null) {  
4.	        try {  
5.	            clazz = parent.loadClass(className, false);  
6.	        } catch (ClassNotFoundException e) {  
7.	            // Don't want to see this.  
8.	        }  
9.	        if (clazz == null) {  
10.	            clazz = findClass(className);  
11.	        }  
12.	    }  
13.	    return clazz;  
14.	}  

因此DexClassLoader和PathClassLoader都属于符合双亲委派模型的类加载器（因为它们没有重载loadClass方法）。也就是说，它们在加载一个类之前，回去检查自己以及自己以上的类加载器是否已经加载了这个类。如果已经加载过了，就会直接将之返回，而不会重复加载。
DexClassLoader和PathClassLoader其实都是通过DexFile这个类来实现类加载的。这里需要顺便提一下的是，Dalvik虚拟机识别的是dex文件，而不是class文件。因此，我们供类加载的文件也只能是dex文件，或者包含有dex文件的.apk或.jar文件。
也许有人想到，既然DexFile可以直接加载类，那么我们为什么还要使用ClassLoader的子类呢？DexFile在加载类时，具体是调用成员方法loadClass或者loadClassBinaryName。其中loadClassBinaryName需要将包含包名的类名中的”.”转换为”/”我们看一下loadClass代码就清楚了：
[java] view plain copy
1.	public Class loadClass(String name, ClassLoader loader) {  
2.	        String slashName = name.replace('.', '/');  
3.	        return loadClassBinaryName(slashName, loader);  
4.	}  

       在这段代码前有一段注释，截取关键一部分就是说：If you are not calling this from a class loader, this is most likely not going to do what you want. Use {@link Class#forName(String)} instead. 这就是我们需要使用ClassLoader子类的原因。至于它是如何验证是否是在ClassLoader中调用此方法的，我没有研究，大家如果有兴趣可以继续深入下去。
       有一个细节，可能大家不容易注意到。PathClassLoader是通过构造函数new DexFile(path)来产生DexFile对象的；而DexClassLoader则是通过其静态方法loadDex（path, outpath, 0）得到DexFile对象。这两者的区别在于DexClassLoader需要提供一个可写的outpath路径，用来释放.apk包或者.jar包中的dex文件。换个说法来说，就是PathClassLoader不能主动从zip包中释放出dex，因此只支持直接操作dex格式文件，或者已经安装的apk（因为已经安装的apk在cache中存在缓存的dex文件）。而DexClassLoader可以支持.apk、.jar和.dex文件，并且会在指定的outpath路径释放出dex文件。

       另外，PathClassLoader在加载类时调用的是DexFile的loadClassBinaryName，而DexClassLoader调用的是loadClass。因此，在使用PathClassLoader时类全名需要用”/”替换”.”

实际操作
使用到的工具都比较常规：javac、dx、eclipse等其中dx工具最好是指明--no-strict，因为class文件的路径可能不匹配
加载好类后，通常我们可以通过Java反射机制来使用这个类但是这样效率相对不高，而且老用反射代码也比较复杂凌乱。更好的做法是定义一个interface，并将这个interface写进容器端。待加载的类，继承自这个interface，并且有一个参数为空的构造函数，以使我们能够通过Class的newInstance方法产生对象然后将对象强制转换为interface对象，于是就可以直接调用成员方法了，下面是具体的实现步骤了:
第一步：

编写好动态代码类：
 
[java] view plain copy
1.	package com.dynamic.interfaces;  
2.	import android.app.Activity;  
3.	/** 
4.	 * 动态加载类的接口 
5.	 */  
6.	public interface IDynamic {  
7.	    /**初始化方法*/  
8.	    public void init(Activity activity);  
9.	    /**自定义方法*/  
10.	    public void showBanner();  
11.	    public void showDialog();  
12.	    public void showFullScreen();  
13.	    public void showAppWall();  
14.	    /**销毁方法*/  
15.	    public void destory();  
16.	}  
实现类代码如下：
[java] view plain copy
1.	package com.dynamic.impl;  
2.	  
3.	import android.app.Activity;  
4.	import android.widget.Toast;  
5.	  
6.	import com.dynamic.interfaces.IDynamic;  
7.	  
8.	/** 
9.	 * 动态类的实现 
10.	 * 
11.	 */  
12.	public class Dynamic implements IDynamic{  
13.	  
14.	    private Activity mActivity;  
15.	      
16.	    @Override  
17.	    public void init(Activity activity) {  
18.	        mActivity = activity;  
19.	    }  
20.	      
21.	    @Override  
22.	    public void showBanner() {  
23.	        Toast.makeText(mActivity, "我是ShowBannber方法", 1500).show();  
24.	    }  
25.	  
26.	    @Override  
27.	    public void showDialog() {  
28.	        Toast.makeText(mActivity, "我是ShowDialog方法", 1500).show();  
29.	    }  
30.	  
31.	    @Override  
32.	    public void showFullScreen() {  
33.	        Toast.makeText(mActivity, "我是ShowFullScreen方法", 1500).show();  
34.	    }  
35.	  
36.	    @Override  
37.	    public void showAppWall() {  
38.	        Toast.makeText(mActivity, "我是ShowAppWall方法", 1500).show();  
39.	    }  
40.	  
41.	    @Override  
42.	    public void destory() {  
43.	    }  
44.	  
45.	}  
这样动态类就开发好了

第二步：
将上面开发好的动态类打包成.jar，这里要注意的是只打包实现类Dynamic.java，不打包接口类IDynamic.java，
 
然后将打包好的jar文件拷贝到android的安装目录中的platform-tools目录下，使用dx命令:(我的jar文件是dynamic.jar)
dx --dex --output=dynamic_temp.jar dynamic.jar
这样就生成了dynamic_temp.jar，这个jar和dynamic.jar有什么区别呢？
其实这条命令主要做的工作是:首先将dynamic.jar编译成dynamic.dex文件(Android虚拟机认识的字节码文件)，然后再将dynamic.dex文件压缩成dynamic_temp.jar，当然你也可以压缩成.zip格式的，或者直接编译成.apk文件都可以的，这个后面会说到。
到这里还不算完事，因为你想想用什么来连接动态类和目标类呢？那就是动态类的接口了，所以这时候还要打个.jar包，这时候只需要打接口类IDynamic.java了
 
然后将这个.jar文件引用到目标类中，下面来看一下目标类的实现：
[java] view plain copy
1.	package com.jiangwei.demo;  
2.	  
3.	import java.io.File;  
4.	import java.util.List;  
5.	  
6.	import android.app.Activity;  
7.	import android.content.Intent;  
8.	import android.content.pm.ActivityInfo;  
9.	import android.content.pm.PackageManager;  
10.	import android.content.pm.ResolveInfo;  
11.	import android.os.Bundle;  
12.	import android.os.Environment;  
13.	import android.view.View;  
14.	import android.widget.Button;  
15.	import android.widget.Toast;  
16.	  
17.	import com.dynamic.interfaces.IDynamic;  
18.	  
19.	import dalvik.system.DexClassLoader;  
20.	import dalvik.system.PathClassLoader;  
21.	  
22.	public class AndroidDynamicLoadClassActivity extends Activity {  
23.	      
24.	    //动态类加载接口  
25.	    private IDynamic lib;  
26.	      
27.	    @Override  
28.	    public void onCreate(Bundle savedInstanceState) {  
29.	        super.onCreate(savedInstanceState);  
30.	        setContentView(R.layout.main);  
31.	        //初始化组件  
32.	        Button showBannerBtn = (Button) findViewById(R.id.show_banner_btn);  
33.	        Button showDialogBtn = (Button) findViewById(R.id.show_dialog_btn);  
34.	        Button showFullScreenBtn = (Button) findViewById(R.id.show_fullscreen_btn);  
35.	        Button showAppWallBtn = (Button) findViewById(R.id.show_appwall_btn);  
36.	        /**使用DexClassLoader方式加载类*/  
37.	        //dex压缩文件的路径(可以是apk,jar,zip格式)  
38.	        String dexPath = Environment.getExternalStorageDirectory().toString() + File.separator + "Dynamic.apk";  
39.	        //dex解压释放后的目录  
40.	        //String dexOutputDir = getApplicationInfo().dataDir;  
41.	        String dexOutputDirs = Environment.getExternalStorageDirectory().toString();  
42.	        //定义DexClassLoader  
43.	        //第一个参数：是dex压缩文件的路径  
44.	        //第二个参数：是dex解压缩后存放的目录  
45.	        //第三个参数：是C/C++依赖的本地库文件目录,可以为null  
46.	        //第四个参数：是上一级的类加载器  
47.	        DexClassLoader cl = new DexClassLoader(dexPath,dexOutputDirs,null,getClassLoader());  
48.	          
49.	        /**使用PathClassLoader方法加载类*/  
50.	        //创建一个意图，用来找到指定的apk：这里的"com.dynamic.impl是指定apk中在AndroidMainfest.xml文件中定义的<action name="com.dynamic.impl"/>    
51.	        Intent intent = new Intent("com.dynamic.impl", null);    
52.	        //获得包管理器    
53.	        PackageManager pm = getPackageManager();    
54.	        List<ResolveInfo> resolveinfoes =  pm.queryIntentActivities(intent, 0);    
55.	        //获得指定的activity的信息    
56.	        ActivityInfo actInfo = resolveinfoes.get(0).activityInfo;    
57.	        //获得apk的目录或者jar的目录    
58.	        String apkPath = actInfo.applicationInfo.sourceDir;    
59.	        //native代码的目录    
60.	        String libPath = actInfo.applicationInfo.nativeLibraryDir;    
61.	        //创建类加载器，把dex加载到虚拟机中    
62.	        //第一个参数：是指定apk安装的路径，这个路径要注意只能是通过actInfo.applicationInfo.sourceDir来获取  
63.	        //第二个参数：是C/C++依赖的本地库文件目录,可以为null  
64.	        //第三个参数：是上一级的类加载器  
65.	        PathClassLoader pcl = new PathClassLoader(apkPath,libPath,this.getClassLoader());  
66.	        //加载类  
67.	        try {  
68.	            //com.dynamic.impl.Dynamic是动态类名  
69.	            //使用DexClassLoader加载类  
70.	            //Class libProviderClazz = cl.loadClass("com.dynamic.impl.Dynamic");  
71.	            //使用PathClassLoader加载类  
72.	            Class libProviderClazz = pcl.loadClass("com.dynamic.impl.Dynamic");  
73.	            lib = (IDynamic)libProviderClazz.newInstance();  
74.	            if(lib != null){  
75.	                lib.init(AndroidDynamicLoadClassActivity.this);  
76.	            }  
77.	        } catch (Exception exception) {  
78.	            exception.printStackTrace();  
79.	        }  
80.	        /**下面分别调用动态类中的方法*/  
81.	        showBannerBtn.setOnClickListener(new View.OnClickListener() {  
82.	            public void onClick(View view) {  
83.	               if(lib != null){  
84.	                   lib.showBanner();  
85.	               }else{  
86.	                   Toast.makeText(getApplicationContext(), "类加载失败", 1500).show();  
87.	               }  
88.	            }  
89.	        });  
90.	        showDialogBtn.setOnClickListener(new View.OnClickListener() {  
91.	            public void onClick(View view) {  
92.	               if(lib != null){  
93.	                   lib.showDialog();  
94.	               }else{  
95.	                   Toast.makeText(getApplicationContext(), "类加载失败", 1500).show();  
96.	               }  
97.	            }  
98.	        });  
99.	        showFullScreenBtn.setOnClickListener(new View.OnClickListener() {  
100.	            public void onClick(View view) {  
101.	               if(lib != null){  
102.	                   lib.showFullScreen();  
103.	               }else{  
104.	                   Toast.makeText(getApplicationContext(), "类加载失败", 1500).show();  
105.	               }  
106.	            }  
107.	        });  
108.	        showAppWallBtn.setOnClickListener(new View.OnClickListener() {  
109.	            public void onClick(View view) {  
110.	               if(lib != null){  
111.	                   lib.showAppWall();  
112.	               }else{  
113.	                   Toast.makeText(getApplicationContext(), "类加载失败", 1500).show();  
114.	               }  
115.	            }  
116.	        });  
117.	    }  
118.	}  
这里面定义了一个IDynamic接口变量，同时使用了DexClassLoader和PathClassLoader来加载类，这里面先来说一说DexClassLoader方式加载:
[java] view plain copy
1.	//定义DexClassLoader  
2.	//第一个参数：是dex压缩文件的路径  
3.	//第二个参数：是dex解压缩后存放的目录  
4.	//第三个参数：是C/C++依赖的本地库文件目录,可以为null  
5.	//第四个参数：是上一级的类加载器  
6.	DexClassLoader cl = new DexClassLoader(dexPath,dexOutputDirs,null,getClassLoader());  
上面已经说了，DexClassLoader是继承ClassLoader类的，这里面的参数说明：
第一个参数是:dex压缩文件的路径:这个就是我们将上面编译后的dynamic_temp.jar存放的目录，当然也可以是.zip和.apk格式的
第二个参数是:dex解压后存放的目录:这个就是将.jar,.zip,.apk文件解压出的dex文件存放的目录，这个就和PathClassLoader方法有区别了，同时你也可以看到PathClassLoader方法中没有这个参数，这个也真是这两个类的区别:
PathClassLoader不能主动从zip包中释放出dex，因此只支持直接操作dex格式文件，或者已经安装的apk（因为已经安装的apk在手机的data/dalvik目录中存在缓存的dex文件）。而DexClassLoader可以支持.apk、.jar和.dex文件，并且会在指定的outpath路径释放出dex文件。
 
然而我们可以通过DexClassLoader方法指定解压后的dex文件的存放目录，但是我们一般不这么做，因为这样做无疑的暴露了dex文件，所以我们一般不会将.jar/.zip/.apk压缩文件存放到用户可以察觉到的位置，同时解压dex的目录也是不能让用户看到的。
第三个参数和第四个参数用到的不是很多，所以这里就不做太多的解释了。
这里还要注意一点就是PathClassLoader方法的时候，第一个参数是dex存放的路径，这里传递的是:
[java] view plain copy
1.	//获得apk的目录或者jar的目录    
2.	String apkPath = actInfo.applicationInfo.sourceDir;    
指定的apk安装路径,这个值只能这样获取，不然会加载类失败的

第三步：
运行目标类:
要做的工作是:
如果用的是DexClassLoader方式加载类:这时候需要将.jar或者.zip或者.apk文件放到指定的目录中，我这里为了方便就放到sd卡的根目录中
如果用的是PathClassLoader方法加载类:这时候需要先将Dynamic.apk安装到手机中，不然找不到这个activity，同时要注意的是:
[java] view plain copy
1.	//创建一个意图，用来找到指定的apk：这里的"com.dynamic.impl是指定apk中在AndroidMainfest.xml文件中定义的<action name="com.dynamic.impl"/>    
2.	Intent intent = new Intent("com.dynamic.impl", null);    
这里的com.dynamic.impl是一个action需要在指定的apk中定义，这个名称是动态apk和目标apk之间约定好的 
运行结果
 
点击showBanner显示一个Toast，成功的运行了动态类中的代码！
其实更好的办法就是将动态的.jar.zip.apk文件从网络上获取，安全可靠，同时本地的目标项目不需要改动代码就可以执行不同的逻辑了

关于代码加密的一些设想
    最初设想将dex文件加密，然后通过JNI将解密代码写在Native层。解密之后直接传上二进制流，再通过defineClass将类加载到内存中。
    现在也可以这样做，但是由于不能直接使用defineClass，而必须传文件路径给dalvik虚拟机内核，因此解密后的文件需要写到磁盘上，增加了被破解的风险。
    Dalvik虚拟机内核仅支持从dex文件加载类的方式是不灵活的，由于没有非常深入的研究内核，我不能确定是Dalvik虚拟机本身不支持还是Android在移植时将其阉割了。不过相信Dalvik或者是Android开源项目都正在向能够支持raw数据定义类方向努力。
    我们可以在文档中看到Google说：Jar or APK file with "classes.dex". (May expand this to include "raw DEX" in the future.)；在Android的Dalvik源码中我们也能看到RawDexFile的身影（不过没有具体实现）
    在RawDexFile出来之前，我们都只能使用这种存在一定风险的加密方式。需要注意释放的dex文件路径及权限管理，另外，在加载完毕类之后，除非出于其他目的否则应该马上删除临时的解密文件











 
Android中插件开发篇之----类加载器
2014-11-24 12:15 22999人阅读 评论(14) 收藏 举报
  分类：
Android（158）  
版权声明：本文为博主原创文章，未经博主允许不得转载。
目录(?)[+]
前言
关于插件，已经在各大平台上出现过很多，eclipse插件、chrome插件、3dmax插件，所有这些插件大概都为了在一个主程序中实现比较通用的功能，把业务相关或者让可以让用户自定义扩展的功能不附加在主程序中，主程序可在运行时安装和卸载。在Android如何实现插件也已经被广泛传播，实现的原理都是实现一套插件接口，把插件实现编成apk或者dex，然后在运行时使用DexClassLoader动态加载进来，不过在这个开发过程中会遇到很多的问题，所以这一片就先不介绍如何开发插件，而是先解决一下开发过程中会遇到的问题，这里主要就是介绍DexClassLoader这个类使用的过程中出现的错误

导读
Java中的类加载器：http://blog.csdn.net/jiangwei0910410003/article/details/17733153
android中的动态加载机制：http://blog.csdn.net/jiangwei0910410003/article/details/17679823
System.loadLibrary的执行过程：http://blog.csdn.net/jiangwei0910410003/article/details/41490133

一、预备知识
Android中的各种加载器介绍
插件开发的过程中DexClassLoader和PathClassLoader这两个类加载器了是很重要的，但是他们也是有区别的，而且我们也知道PathClassLoader是Android应用中的默认加载器。他们的区别是：
DexClassLoader可以加载任何路径的apk/dex/jar
PathClassLoader只能加载/data/app中的apk，也就是已经安装到手机中的apk。这个也是PathClassLoader作为默认的类加载器的原因，因为一般程序都是安装了，在打开，这时候PathClassLoader就去加载指定的apk(解压成dex，然后在优化成odex)就可以了。

我们可以看一下他们的源码：
DexClassLoader.java
[java] view plain copy
1.	/* 
2.	 * Copyright (C) 2008 The Android Open Source Project 
3.	 * 
4.	 * Licensed under the Apache License, Version 2.0 (the "License"); 
5.	 * you may not use this file except in compliance with the License. 
6.	 * You may obtain a copy of the License at 
7.	 * 
8.	 *      http://www.apache.org/licenses/LICENSE-2.0 
9.	 * 
10.	 * Unless required by applicable law or agreed to in writing, software 
11.	 * distributed under the License is distributed on an "AS IS" BASIS, 
12.	 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied. 
13.	 * See the License for the specific language governing permissions and 
14.	 * limitations under the License. 
15.	 */  
16.	  
17.	package dalvik.system;  
18.	  
19.	import java.io.File;  
20.	import java.io.IOException;  
21.	import java.net.MalformedURLException;  
22.	import java.net.URL;  
23.	import java.util.zip.ZipFile;  
24.	  
25.	/** 
26.	 * Provides a simple {@link ClassLoader} implementation that operates on a 
27.	 * list of jar/apk files with classes.dex entries.  The directory that 
28.	 * holds the optimized form of the files is specified explicitly.  This 
29.	 * can be used to execute code not installed as part of an application. 
30.	 * 
31.	 * The best place to put the optimized DEX files is in app-specific 
32.	 * storage, so that removal of the app will automatically remove the 
33.	 * optimized DEX files.  If other storage is used (e.g. /sdcard), the 
34.	 * app may not have an opportunity to remove them. 
35.	 */  
36.	public class DexClassLoader extends ClassLoader {  
37.	  
38.	    private static final boolean VERBOSE_DEBUG = false;  
39.	  
40.	    /* constructor args, held for init */  
41.	    private final String mRawDexPath;  
42.	    private final String mRawLibPath;  
43.	    private final String mDexOutputPath;  
44.	  
45.	    /* 
46.	     * Parallel arrays for jar/apk files. 
47.	     * 
48.	     * (could stuff these into an object and have a single array; 
49.	     * improves clarity but adds overhead) 
50.	     */  
51.	    private final File[] mFiles;         // source file Files, for rsrc URLs  
52.	    private final ZipFile[] mZips;       // source zip files, with resources  
53.	    private final DexFile[] mDexs;       // opened, prepped DEX files  
54.	  
55.	    /** 
56.	     * Native library path. 
57.	     */  
58.	    private final String[] mLibPaths;  
59.	  
60.	    /** 
61.	     * Creates a {@code DexClassLoader} that finds interpreted and native 
62.	     * code.  Interpreted classes are found in a set of DEX files contained 
63.	     * in Jar or APK files. 
64.	     * 
65.	     * The path lists are separated using the character specified by 
66.	     * the "path.separator" system property, which defaults to ":". 
67.	     * 
68.	     * @param dexPath 
69.	     *  the list of jar/apk files containing classes and resources 
70.	     * @param dexOutputDir 
71.	     *  directory where optimized DEX files should be written 
72.	     * @param libPath 
73.	     *  the list of directories containing native libraries; may be null 
74.	     * @param parent 
75.	     *  the parent class loader 
76.	     */  
77.	    public DexClassLoader(String dexPath, String dexOutputDir, String libPath,  
78.	        ClassLoader parent) {  
79.	  
80.	        super(parent);  
81.	......  
我们看到，他是继承了ClassLoader类的，ClassLoader是类加载器的鼻祖类。同时我们也会发现DexClassLoader只有一个构造函数，而且这个构造函数是：dexPath、dexOutDir、libPath、parent
dexPath：是加载apk/dex/jar的路径
dexOutDir：是dex的输出路径(因为加载apk/jar的时候会解压除dex文件，这个路径就是保存dex文件的)
libPath：是加载的时候需要用到的lib库，这个一般不用
parent：给DexClassLoader指定父加载器

我们在来看一下PathClassLoader的源码
PathClassLoader.java
[java] view plain copy
1.	/* 
2.	 * Copyright (C) 2007 The Android Open Source Project 
3.	 * 
4.	 * Licensed under the Apache License, Version 2.0 (the "License"); 
5.	 * you may not use this file except in compliance with the License. 
6.	 * You may obtain a copy of the License at 
7.	 * 
8.	 *      http://www.apache.org/licenses/LICENSE-2.0 
9.	 * 
10.	 * Unless required by applicable law or agreed to in writing, software 
11.	 * distributed under the License is distributed on an "AS IS" BASIS, 
12.	 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied. 
13.	 * See the License for the specific language governing permissions and 
14.	 * limitations under the License. 
15.	 */  
16.	  
17.	package dalvik.system;  
18.	  
19.	import java.io.ByteArrayOutputStream;  
20.	import java.io.File;  
21.	import java.io.FileNotFoundException;  
22.	import java.io.IOException;  
23.	import java.io.InputStream;  
24.	import java.io.RandomAccessFile;  
25.	import java.net.MalformedURLException;  
26.	import java.net.URL;  
27.	import java.util.ArrayList;  
28.	import java.util.Enumeration;  
29.	import java.util.List;  
30.	import java.util.NoSuchElementException;  
31.	import java.util.zip.ZipEntry;  
32.	import java.util.zip.ZipFile;  
33.	  
34.	/** 
35.	 * Provides a simple {@link ClassLoader} implementation that operates on a list 
36.	 * of files and directories in the local file system, but does not attempt to 
37.	 * load classes from the network. Android uses this class for its system class 
38.	 * loader and for its application class loader(s). 
39.	 */  
40.	public class PathClassLoader extends ClassLoader {  
41.	  
42.	    private final String path;  
43.	    private final String libPath;  
44.	  
45.	    /* 
46.	     * Parallel arrays for jar/apk files. 
47.	     * 
48.	     * (could stuff these into an object and have a single array; 
49.	     * improves clarity but adds overhead) 
50.	     */  
51.	    private final String[] mPaths;  
52.	    private final File[] mFiles;  
53.	    private final ZipFile[] mZips;  
54.	    private final DexFile[] mDexs;  
55.	  
56.	    /** 
57.	     * Native library path. 
58.	     */  
59.	    private final List<String> libraryPathElements;  
60.	  
61.	    /** 
62.	     * Creates a {@code PathClassLoader} that operates on a given list of files 
63.	     * and directories. This method is equivalent to calling 
64.	     * {@link #PathClassLoader(String, String, ClassLoader)} with a 
65.	     * {@code null} value for the second argument (see description there). 
66.	     * 
67.	     * @param path 
68.	     *            the list of files and directories 
69.	     * 
70.	     * @param parent 
71.	     *            the parent class loader 
72.	     */  
73.	    public PathClassLoader(String path, ClassLoader parent) {  
74.	        this(path, null, parent);  
75.	    }  
76.	  
77.	    /** 
78.	     * Creates a {@code PathClassLoader} that operates on two given 
79.	     * lists of files and directories. The entries of the first list 
80.	     * should be one of the following: 
81.	     * 
82.	     * <ul> 
83.	     * <li>Directories containing classes or resources. 
84.	     * <li>JAR/ZIP/APK files, possibly containing a "classes.dex" file. 
85.	     * <li>"classes.dex" files. 
86.	     * </ul> 
87.	     * 
88.	     * The entries of the second list should be directories containing 
89.	     * native library files. Both lists are separated using the 
90.	     * character specified by the "path.separator" system property, 
91.	     * which, on Android, defaults to ":". 
92.	     * 
93.	     * @param path 
94.	     *            the list of files and directories containing classes and 
95.	     *            resources 
96.	     * 
97.	     * @param libPath 
98.	     *            the list of directories containing native libraries 
99.	     * 
100.	     * @param parent 
101.	     *            the parent class loader 
102.	     */  
103.	    public PathClassLoader(String path, String libPath, ClassLoader parent) {  
104.	        super(parent);  
105.	....  
看到了PathClassLoader类也是继承了ClassLoader的，但是他的构造函数和DexClassLoader有点区别就是，少了一个dexOutDir，这个原因也是很简单，因为PathClassLoader是加载/data/app中的apk，而这部分的apk都会解压释放dex到指定的目录：
/data/dalvik-cache
 
这个释放解压操作是系统做的。所以PathClassLoader可以不需要这个参数的。

上面看了他们两的区别，下面在来看一下Android中的各种类加载器分别加载哪些类：
[java] view plain copy
1.	package com.example.androiddemo;  
2.	  
3.	import android.app.Activity;  
4.	import android.content.Context;  
5.	import android.os.Bundle;  
6.	import android.util.Log;  
7.	import android.widget.ListView;  
8.	  
9.	public class MainActivity extends Activity {  
10.	  
11.	    @Override  
12.	    protected void onCreate(Bundle savedInstanceState) {  
13.	        super.onCreate(savedInstanceState);  
14.	        setContentView(R.layout.activity_main);  
15.	          
16.	        Log.i("DEMO", "Context的类加载加载器:"+Context.class.getClassLoader());  
17.	        Log.i("DEMO", "ListView的类加载器:"+ListView.class.getClassLoader());  
18.	        Log.i("DEMO", "应用程序默认加载器:"+getClassLoader());  
19.	        Log.i("DEMO", "系统类加载器:"+ClassLoader.getSystemClassLoader());  
20.	        Log.i("DEMO", "系统类加载器和Context的类加载器是否相等:"+(Context.class.getClassLoader()==ClassLoader.getSystemClassLoader()));  
21.	        Log.i("DEMO", "系统类加载器和应用程序默认加载器是否相等:"+(getClassLoader()==ClassLoader.getSystemClassLoader()));  
22.	          
23.	        Log.i("DEMO","打印应用程序默认加载器的委派机制:");  
24.	        ClassLoader classLoader = getClassLoader();  
25.	        while(classLoader != null){  
26.	            Log.i("DEMO", "类加载器:"+classLoader);  
27.	            classLoader = classLoader.getParent();  
28.	        }  
29.	          
30.	        Log.i("DEMO","打印系统加载器的委派机制:");  
31.	        classLoader = ClassLoader.getSystemClassLoader();  
32.	        while(classLoader != null){  
33.	            Log.i("DEMO", "类加载器:"+classLoader);  
34.	            classLoader = classLoader.getParent();  
35.	        }  
36.	          
37.	    }  
38.	  
39.	}  
运行结果：
 
依次来看一下
1) 系统类的加载器
[java] view plain copy
1.	Log.i("DEMO", "Context的类加载加载器:"+Context.class.getClassLoader());  
2.	Log.i("DEMO", "ListView的类加载器:"+ListView.class.getClassLoader());  
从结果看到他们的加载器是：BootClassLoader，关于他源码我没有找到，只找到了class文件(用jd-gui查看)：

看到他也是继承了ClassLoader类。

2) 应用程序的默认加载器
[java] view plain copy
1.	Log.i("DEMO", "应用程序默认加载器:"+getClassLoader());  
运行结果：

默认类加载器是PathClassLoader，同时可以看到加载的apk路径，libPath(一般包括/vendor/lib和/system/lib)

3) 系统类加载器
[java] view plain copy
1.	Log.i("DEMO", "系统类加载器:"+ClassLoader.getSystemClassLoader());  
运行结果：

系统类加载器其实还是PathClassLoader，只是加载的apk路径不是/data/app/xxx.apk了，而是系统apk的路径：/system/app/xxx.apk

4) 默认加载器的委派机制关系
[java] view plain copy
1.	Log.i("DEMO","打印应用程序默认加载器的委派机制:");  
2.	ClassLoader classLoader = getClassLoader();  
3.	while(classLoader != null){  
4.	    Log.i("DEMO", "类加载器:"+classLoader);  
5.	    classLoader = classLoader.getParent();  
6.	}  
打印结果：

默认加载器PathClassLoader的父亲是BootClassLoader
5) 系统加载器的委派机制关系
[java] view plain copy
1.	Log.i("DEMO","打印系统加载器的委派机制:");  
2.	classLoader = ClassLoader.getSystemClassLoader();  
3.	while(classLoader != null){  
4.	    Log.i("DEMO", "类加载器:"+classLoader);  
5.	    classLoader = classLoader.getParent();  
6.	}  
运行结果：

可以看到系统加载器的父亲也是BootClassLoader

二、分析遇到的问题的原因和解决办法
DexClassLoader加载原理和分析在实现插件时不同操作造成错误的原因分析
这里主要用了三个工程：
PluginImpl：插件接口工程(只是接口的定义)
PluginSDK：插件工程(实现插件接口，定义具体的功能)
HostProject：宿主工程(需要引用插件接口工程，然后动态的加载插件工程)(例子项目中名字是PluginDemos)
 

第一、项目介绍
下面来看一下源代码：
1、PluginImpl工程：
1) IBean.java
[java] view plain copy
1.	package com.pluginsdk.interfaces;  
2.	  
3.	public abstract interface IBean{  
4.	  public abstract String getName();  
5.	  public abstract void setName(String paramString);  
6.	}  

2) IDynamic.java
[java] view plain copy
1.	package com.pluginsdk.interfaces;  
2.	  
3.	import android.content.Context;  
4.	  
5.	public abstract interface IDynamic{  
6.	  public abstract void methodWithCallBack(YKCallBack paramYKCallBack);  
7.	  public abstract void showPluginWindow(Context paramContext);  
8.	  public abstract void startPluginActivity(Context context,Class<?> cls);  
9.	  public abstract String getStringForResId(Context context);  
10.	}  
其他的就不列举了。

2、PluginSDK工程：
1) Dynamic.java
[java] view plain copy
1.	/** 
2.	 * Dynamic1.java 
3.	 * com.youku.pluginsdk.imp 
4.	 * 
5.	 * Function： TODO  
6.	 * 
7.	 *   ver     date           author 
8.	 * ────────────────────────────────── 
9.	 *           2014-10-20         Administrator 
10.	 * 
11.	 * Copyright (c) 2014, TNT All Rights Reserved. 
12.	*/  
13.	  
14.	package com.pluginsdk.imp;  
15.	  
16.	import android.app.AlertDialog;  
17.	import android.app.AlertDialog.Builder;  
18.	import android.app.Dialog;  
19.	import android.content.Context;  
20.	import android.content.DialogInterface;  
21.	import android.content.Intent;  
22.	  
23.	import com.pluginsdk.bean.Bean;  
24.	import com.pluginsdk.interfaces.IDynamic;  
25.	import com.pluginsdk.interfaces.YKCallBack;  
26.	import com.youku.pluginsdk.R;  
27.	  
28.	/** 
29.	 * ClassName:Dynamic1 
30.	 * 
31.	 * @author   jiangwei 
32.	 * @version   
33.	 * @since    Ver 1.1 
34.	 * @Date     2014-10-20     下午5:57:10 
35.	 */  
36.	public class Dynamic implements IDynamic{  
37.	    /** 
38.	 
39.	     */  
40.	    public void methodWithCallBack(YKCallBack callback) {  
41.	        Bean bean = new Bean();  
42.	        bean.setName("PLUGIN_SDK_USER");  
43.	        callback.callback(bean);  
44.	    }  
45.	      
46.	    public void showPluginWindow(Context context) {  
47.	         AlertDialog.Builder builder = new Builder(context);  
48.	          builder.setMessage("对话框");  
49.	          builder.setTitle(R.string.hello_world);  
50.	          builder.setNegativeButton("取消", new Dialog.OnClickListener() {  
51.	               @Override  
52.	               public void onClick(DialogInterface dialog, int which) {  
53.	                   dialog.dismiss();  
54.	               }  
55.	              });  
56.	          Dialog dialog = builder.create();//.show();  
57.	          dialog.show();  
58.	    }  
59.	      
60.	    public void startPluginActivity(Context context,Class<?> cls){  
61.	        /** 
62.	        *这里要注意几点: 
63.	        *1、如果单纯的写一个MainActivity的话，在主工程中也有一个MainActivity，开启的Activity还是主工程中的MainActivity 
64.	        *2、如果这里将MainActivity写成全名的话，还是有问题，会报找不到这个Activity的错误 
65.	        */  
66.	        Intent intent = new Intent(context,cls);  
67.	        context.startActivity(intent);  
68.	    }  
69.	      
70.	    public String getStringForResId(Context context){  
71.	        return context.getResources().getString(R.string.hello_world);  
72.	    }  
73.	  
74.	}  

2) Bean.java
[java] view plain copy
1.	/** 
2.	 * User.java 
3.	 * com.youku.pluginsdk.bean 
4.	 * 
5.	 * Function： TODO  
6.	 * 
7.	 *   ver     date           author 
8.	 * ────────────────────────────────── 
9.	 *           2014-10-20         Administrator 
10.	 * 
11.	 * Copyright (c) 2014, TNT All Rights Reserved. 
12.	*/  
13.	  
14.	package com.pluginsdk.bean;  
15.	  
16.	  
17.	/** 
18.	 * ClassName:User 
19.	 * 
20.	 * @author   jiangwei 
21.	 * @version   
22.	 * @since    Ver 1.1 
23.	 * @Date     2014-10-20     下午1:35:16 
24.	 */  
25.	public class Bean implements com.pluginsdk.interfaces.IBean{  
26.	  
27.	    /** 
28.	     * 
29.	     */  
30.	    private String name = "这是来自于插件工程中设置的初始化的名字";  
31.	  
32.	    public String getName() {  
33.	        return name;  
34.	    }  
35.	  
36.	    public void setName(String name) {  
37.	        this.name = name;  
38.	    }  
39.	  
40.	}  

3、宿主工程HostProject
1) MainActivity.java
[java] view plain copy
1.	package com.plugindemo;  
2.	import java.io.File;  
3.	import java.lang.reflect.Method;  
4.	  
5.	import android.annotation.SuppressLint;  
6.	import android.app.Activity;  
7.	import android.content.Context;  
8.	import android.content.res.AssetManager;  
9.	import android.content.res.Resources;  
10.	import android.content.res.Resources.Theme;  
11.	import android.os.Bundle;  
12.	import android.os.Environment;  
13.	import android.util.Log;  
14.	import android.view.View;  
15.	import android.widget.Button;  
16.	import android.widget.ListView;  
17.	import android.widget.Toast;  
18.	  
19.	import com.pluginsdk.interfaces.IBean;  
20.	import com.pluginsdk.interfaces.IDynamic;  
21.	import com.pluginsdk.interfaces.YKCallBack;  
22.	import com.youku.plugindemo.R;  
23.	  
24.	import dalvik.system.DexClassLoader;  
25.	  
26.	public class MainActivity extends Activity {  
27.	    private AssetManager mAssetManager;//资源管理器  
28.	    private Resources mResources;//资源  
29.	    private Theme mTheme;//主题  
30.	    private String apkFileName = "PluginSDKs.apk";  
31.	    private String dexpath = null;//apk文件地址  
32.	    private File fileRelease = null; //释放目录  
33.	    private DexClassLoader classLoader = null;  
34.	    @SuppressLint("NewApi")  
35.	    @Override  
36.	    protected void onCreate(Bundle savedInstanceState) {  
37.	        super.onCreate(savedInstanceState);  
38.	        setContentView(R.layout.activity_main);  
39.	        dexpath =  Environment.getExternalStorageDirectory() + File.separator+apkFileName;  
40.	        fileRelease = getDir("dex", 0);  
41.	          
42.	        /*初始化classloader 
43.	         * dexpath dex文件地址 
44.	         * fileRelease 文件释放地址  
45.	         *  父classLoader 
46.	         */  
47.	          
48.	        Log.d("DEMO", (getClassLoader()==ListView.class.getClassLoader())+"");  
49.	        Log.d("DEMO",ListView.class.getClassLoader()+"");  
50.	        Log.d("DEMO", Context.class.getClassLoader()+"");  
51.	        Log.d("DEMO", Context.class.getClassLoader().getSystemClassLoader()+"");  
52.	        Log.d("DEMO",Activity.class.getClassLoader()+"");  
53.	        Log.d("DEMO", (Context.class.getClassLoader().getSystemClassLoader() == ClassLoader.getSystemClassLoader())+"");  
54.	        Log.d("DEMO",ClassLoader.getSystemClassLoader()+"");  
55.	          
56.	        classLoader = new DexClassLoader(dexpath, fileRelease.getAbsolutePath(),null,getClassLoader());  
57.	          
58.	        Button btn_1 = (Button)findViewById(R.id.btn_1);  
59.	        Button btn_2 = (Button)findViewById(R.id.btn_2);  
60.	        Button btn_3 = (Button)findViewById(R.id.btn_3);  
61.	        Button btn_4 = (Button)findViewById(R.id.btn_4);  
62.	        Button btn_5 = (Button)findViewById(R.id.btn_5);  
63.	        Button btn_6 = (Button)findViewById(R.id.btn_6);  
64.	          
65.	        btn_1.setOnClickListener(new View.OnClickListener() {//普通调用  反射的方式  
66.	            @Override  
67.	            public void onClick(View arg0) {  
68.	                Class mLoadClassBean;  
69.	                try {  
70.	                    mLoadClassBean = classLoader.loadClass("com.pluginsdk.bean.Bean");  
71.	                    Object beanObject = mLoadClassBean.newInstance();  
72.	                    Log.d("DEMO", "ClassLoader:"+mLoadClassBean.getClassLoader());  
73.	                    Log.d("DEMO", "ClassLoader:"+mLoadClassBean.getClassLoader().getParent());  
74.	                    Method getNameMethod = mLoadClassBean.getMethod("getName");  
75.	                    getNameMethod.setAccessible(true);  
76.	                    String name = (String) getNameMethod.invoke(beanObject);  
77.	                    Toast.makeText(MainActivity.this, name, Toast.LENGTH_SHORT).show();  
78.	                } catch (Exception e) {  
79.	                    Log.e("DEMO", "msg:"+e.getMessage());  
80.	                }   
81.	            }  
82.	        });  
83.	        btn_2.setOnClickListener(new View.OnClickListener() {//带参数调用  
84.	            @Override  
85.	            public void onClick(View arg0) {  
86.	                Class mLoadClassBean;  
87.	                try {  
88.	                    mLoadClassBean = classLoader.loadClass("com.pluginsdk.bean.Bean");  
89.	                    Object beanObject = mLoadClassBean.newInstance();  
90.	                    //接口形式调用  
91.	                    Log.d("DEMO", beanObject.getClass().getClassLoader()+"");  
92.	                    Log.d("DEMO",IBean.class.getClassLoader()+"");  
93.	                    Log.d("DEMO",ClassLoader.getSystemClassLoader()+"");  
94.	                    IBean bean = (IBean)beanObject;  
95.	                    bean.setName("宿主程序设置的新名字");  
96.	                    Toast.makeText(MainActivity.this, bean.getName(), Toast.LENGTH_SHORT).show();  
97.	                }catch (Exception e) {  
98.	                    Log.e("DEMO", "msg:"+e.getMessage());  
99.	                }  
100.	                 
101.	            }  
102.	        });  
103.	        btn_3.setOnClickListener(new View.OnClickListener() {//带回调函数的调用  
104.	            @Override  
105.	            public void onClick(View arg0) {  
106.	                Class mLoadClassDynamic;  
107.	                try {  
108.	                    mLoadClassDynamic = classLoader.loadClass("com.pluginsdk.imp.Dynamic");  
109.	                     Object dynamicObject = mLoadClassDynamic.newInstance();  
110.	                      //接口形式调用  
111.	                    IDynamic dynamic = (IDynamic)dynamicObject;  
112.	                    //回调函数调用  
113.	                    YKCallBack callback = new YKCallBack() {//回调接口的定义  
114.	                        public void callback(IBean arg0) {  
115.	                            Toast.makeText(MainActivity.this, arg0.getName(), Toast.LENGTH_SHORT).show();  
116.	                        };  
117.	                    };  
118.	                    dynamic.methodWithCallBack(callback);  
119.	                } catch (Exception e) {  
120.	                    Log.e("DEMO", "msg:"+e.getMessage());  
121.	                }  
122.	                 
123.	            }  
124.	        });  
125.	        btn_4.setOnClickListener(new View.OnClickListener() {//带资源文件的调用  
126.	            @Override  
127.	            public void onClick(View arg0) {  
128.	                loadResources();  
129.	                Class mLoadClassDynamic;  
130.	                try {  
131.	                    mLoadClassDynamic = classLoader.loadClass("com.pluginsdk.imp.Dynamic");  
132.	                    Object dynamicObject = mLoadClassDynamic.newInstance();  
133.	                    //接口形式调用  
134.	                    IDynamic dynamic = (IDynamic)dynamicObject;  
135.	                    dynamic.showPluginWindow(MainActivity.this);  
136.	                } catch (Exception e) {  
137.	                    Log.e("DEMO", "msg:"+e.getMessage());  
138.	                }  
139.	            }  
140.	        });  
141.	        btn_5.setOnClickListener(new View.OnClickListener() {//带资源文件的调用  
142.	            @Override  
143.	            public void onClick(View arg0) {  
144.	                loadResources();  
145.	                Class mLoadClassDynamic;  
146.	                try {  
147.	                    mLoadClassDynamic = classLoader.loadClass("com.pluginsdk.imp.Dynamic");  
148.	                    Object dynamicObject = mLoadClassDynamic.newInstance();  
149.	                    //接口形式调用  
150.	                    IDynamic dynamic = (IDynamic)dynamicObject;  
151.	                    dynamic.startPluginActivity(MainActivity.this,  
152.	                            classLoader.loadClass("com.plugindemo.MainActivity"));  
153.	                } catch (Exception e) {  
154.	                    Log.e("DEMO", "msg:"+e.getMessage());  
155.	                }  
156.	            }  
157.	        });  
158.	        btn_6.setOnClickListener(new View.OnClickListener() {//带资源文件的调用  
159.	            @Override  
160.	            public void onClick(View arg0) {  
161.	                loadResources();  
162.	                Class mLoadClassDynamic;  
163.	                try {  
164.	                    mLoadClassDynamic = classLoader.loadClass("com.pluginsdk.imp.Dynamic");  
165.	                    Object dynamicObject = mLoadClassDynamic.newInstance();  
166.	                    //接口形式调用  
167.	                    IDynamic dynamic = (IDynamic)dynamicObject;  
168.	                    String content = dynamic.getStringForResId(MainActivity.this);  
169.	                    Toast.makeText(getApplicationContext(), content+"", Toast.LENGTH_LONG).show();  
170.	                } catch (Exception e) {  
171.	                    Log.e("DEMO", "msg:"+e.getMessage());  
172.	                }  
173.	            }  
174.	        });  
175.	          
176.	    }  
177.	  
178.	     protected void loadResources() {  
179.	            try {  
180.	                AssetManager assetManager = AssetManager.class.newInstance();  
181.	                Method addAssetPath = assetManager.getClass().getMethod("addAssetPath", String.class);  
182.	                addAssetPath.invoke(assetManager, dexpath);  
183.	                mAssetManager = assetManager;  
184.	            } catch (Exception e) {  
185.	                e.printStackTrace();  
186.	            }  
187.	            Resources superRes = super.getResources();  
188.	            superRes.getDisplayMetrics();  
189.	            superRes.getConfiguration();  
190.	            mResources = new Resources(mAssetManager, superRes.getDisplayMetrics(),superRes.getConfiguration());  
191.	            mTheme = mResources.newTheme();  
192.	            mTheme.setTo(super.getTheme());  
193.	        }  
194.	      
195.	    @Override  
196.	    public AssetManager getAssets() {  
197.	        return mAssetManager == null ? super.getAssets() : mAssetManager;  
198.	    }  
199.	  
200.	    @Override  
201.	    public Resources getResources() {  
202.	        return mResources == null ? super.getResources() : mResources;  
203.	    }  
204.	  
205.	    @Override  
206.	    public Theme getTheme() {  
207.	        return mTheme == null ? super.getTheme() : mTheme;  
208.	    }  
209.	}  
三个工程的下载地址：http://download.csdn.net/detail/jiangwei0910410003/8188011
第二、项目引用关系
工程文件现在大致看完了，我们看一下他们的引用关系吧：
1、将接口工程PluginImpl设置成一个Library
 
2、插件工程PluginSDKs引用插件的jar
注意是lib文件夹，不是libs，这个是有区别的，后面会说道
 

3、HostProject项目引用PluginImpl这个library
 

项目引用完成之后，我们编译PluginSDKs项目，生成PluginSDKs.apk放到手机的sdcard的根目录(因为我代码中是从这个目录进行加载apk的，当然这个目录是可以修改的)，然后运行HostProject
 
看到效果了吧。运行成功，其实这个对话框是在插件中定义的，但是我们知道定义对话框是需要context变量的，所以这个变量就是通过参数从宿主工程中传递到插件工程即可，成功了就不能这么了事，因为我还没有说道我遇到的问题，下面就来看一下遇到的几个问题

三、问题分析
问题一：Could not find class...(找不到指定的类)
 
这个问题产生的操作：
插件工程PluginSDKs的引用方式不变，宿主工程PluginDemos的引用方式改变
 
在说这个原因之前先来了解一下Eclipse中引用工程的不同方式和区别：
第一种：最常用的将引用工程打成jar放到需要引用工程的libs下面(这里是将PluginImpl打成jar,放到HostProject工程的libs中)
这种方式是Eclipse推荐使用的，当我们在建立一个项目的时候也会自动产生这个文件夹，当我们将我们需要引用的工程打成jar，然后放到这个文件夹之后，Eclipse就自动导入了(这个功能是Eclipse3.7之后有的)。
第二种：和第一种的区别是，我们可以从新新建一个文件夹比如是lib,然后将引用的jar放到这个文件夹中，但是此时Eclipse是不会自动导入的，需要我们手动的导入(add build path...)，但是这个是一个区别，还有一个区别，也是到这个这个报错原因的区别，就是libs文件夹中的jar，在运行的时候是会将这个jar集成到程序中的，而我们新建的文件夹(名字非libs即可)，及时我们手动的导入，编译是没有问题的，但是运行的时候，是不会将jar集成到程序中。
第三种：和前两种的区别是不需要将引用工程打成jar，直接引用这个工程
 
这种方式其实效果和第一种差不多，唯一的区别就是不需要打成jar,但是运行的时候是不会将引用工程集成到程序中的。
第四种：和第三种的方式是一样的，也是不需要将引用工程打成jar,直接引用工程：
 
这个前提是需要设置PluginImpl项目为Library,同时引用的项目和被引用的项目必须在一个工作空间中，不然会报错，这种的效果和第二种是一样的，在运行的时候是会将引用工程集成到程序中的。
第五种：和第一种、第二种差不多，导入jar：
 
这里有很多种方式选择jar的位置，但是这些操作的效果和第一种是一样的，运行的时候是不会将引用的jar集成到程序中的。

总结上面的五种方式，我们可以看到，第二种和第四种的效果是一样的，也是最普遍的导入引用工程的方式，因为其他三种方式的话，其实在编译的时候是不会有问题的，但是在运行的时候会报错(找不到指定的类，可以依次尝试一下)，不过这三种方式只要一步就可以和那两种方式实现的效果一样了

只要设置导出的时候勾选上这个jar就可以了。那么其实这五种方式都是可以的，性质和效果是一样的。

说完了Eclipse中引用工程的各种方式以及区别之后，我们在回过头来看一下，上面遇到的问题：Could not find class...
其实这个问题就简单了，原因是：插件工程PluginSDKs使用的是lib文件夹导入的jar(这个jar是不会集成到程序中的)，而宿主工程PluginDemos的引用工程的方式也变成了lib文件夹(jar也是不会集成到程序中的)。那么程序运行的时候就会出现错误：
Could not find class 'com.pluginsdk.interfaces.IBean' 

问题二：Class ref in pre-verified class resolved to unexpected implementation(相同的类加载了两次)
 
这个问题产生的操作：
插件工程PluginSDKs和宿主工程PluginDemos引用工程的方式都变成library(或者是都用libs文件夹导入jar)
 
这个错误的原因也是很多做插件的开发者第一次都会遇到的问题，其实这个问题的本质是PluginImpl中的接口被加载了两次，因为插件工程和宿主工程在运行的时候都会把PluginImpl集成到程序中。对于这个问题，我们来分析一下，首先对于宿主apk,他的类加载器是PathClassLoader(这个对于每个应用来说是默认的加载器，原因很简单，PathClassLoader只能加载/data/app目录下的apk,就是已经安装的apk,一般我们的apk都是安装之后在运行，所以用这个加载器也是理所当然的)。这个加载器开始加载插件接口工程(宿主工程中引入的PluginImpl)中的IBean。当使用DexClassLoader加载PluginSDKs.apk的时候，首先会让宿主apk的PathClassLoader加载器去加载，这个好多人有点迷糊了，为什么会先让PathClassLoader加载器去加载呢？
这个就是Java中的类加载机制的双亲委派机制：http://blog.csdn.net/jiangwei0910410003/article/details/17733153
Android中的加载机制也是类似的，我们这里的代码设置了DexClassLoader的父加载器为当前类加载器(宿主apk的PathClassLoader),不行的话，可以打印一下getClassLoader()方法的返回结果看一下。
[java] view plain copy
1.	classLoader = new DexClassLoader(dexpath, fileRelease.getAbsolutePath(),null,getClassLoader());  
那么加载器就是一样的了(宿主apk的PathClassLoader)，那么就奇怪了，都是一个为什么还有错误呢？查看系统源码可以了解：
Resolve.c源码(这个是在虚拟机dalvik中的)：源码下载地址为：http://blog.csdn.net/jiangwei0910410003/article/details/37988637
我们来看一下他的一个主要函数：
[cpp] view plain copy
1.	/* 
2.	 * Find the class corresponding to "classIdx", which maps to a class name 
3.	 * string.  It might be in the same DEX file as "referrer", in a different 
4.	 * DEX file, generated by a class loader, or generated by the VM (e.g. 
5.	 * array classes). 
6.	 * 
7.	 * Because the DexTypeId is associated with the referring class' DEX file, 
8.	 * we may have to resolve the same class more than once if it's referred 
9.	 * to from classes in multiple DEX files.  This is a necessary property for 
10.	 * DEX files associated with different class loaders. 
11.	 * 
12.	 * We cache a copy of the lookup in the DexFile's "resolved class" table, 
13.	 * so future references to "classIdx" are faster. 
14.	 * 
15.	 * Note that "referrer" may be in the process of being linked. 
16.	 * 
17.	 * Traditional VMs might do access checks here, but in Dalvik the class 
18.	 * "constant pool" is shared between all classes in the DEX file.  We rely 
19.	 * on the verifier to do the checks for us. 
20.	 * 
21.	 * Does not initialize the class. 
22.	 * 
23.	 * "fromUnverifiedConstant" should only be set if this call is the direct 
24.	 * result of executing a "const-class" or "instance-of" instruction, which 
25.	 * use class constants not resolved by the bytecode verifier. 
26.	 * 
27.	 * Returns NULL with an exception raised on failure. 
28.	 */  
29.	ClassObject* dvmResolveClass(const ClassObject* referrer, u4 classIdx,  
30.	    bool fromUnverifiedConstant)  
31.	{  
32.	    DvmDex* pDvmDex = referrer->pDvmDex;  
33.	    ClassObject* resClass;  
34.	    const char* className;  
35.	  
36.	    /* 
37.	     * Check the table first -- this gets called from the other "resolve" 
38.	     * methods. 
39.	     */  
40.	    resClass = dvmDexGetResolvedClass(pDvmDex, classIdx);  
41.	    if (resClass != NULL)  
42.	        return resClass;  
43.	  
44.	    LOGVV("--- resolving class %u (referrer=%s cl=%p)\n",  
45.	        classIdx, referrer->descriptor, referrer->classLoader);  
46.	  
47.	    /* 
48.	     * Class hasn't been loaded yet, or is in the process of being loaded 
49.	     * and initialized now.  Try to get a copy.  If we find one, put the 
50.	     * pointer in the DexTypeId.  There isn't a race condition here -- 
51.	     * 32-bit writes are guaranteed atomic on all target platforms.  Worst 
52.	     * case we have two threads storing the same value. 
53.	     * 
54.	     * If this is an array class, we'll generate it here. 
55.	     */  
56.	    className = dexStringByTypeIdx(pDvmDex->pDexFile, classIdx);  
57.	    if (className[0] != '\0' && className[1] == '\0') {  
58.	        /* primitive type */  
59.	        resClass = dvmFindPrimitiveClass(className[0]);  
60.	    } else {  
61.	        resClass = dvmFindClassNoInit(className, referrer->classLoader);  
62.	    }  
63.	  
64.	    if (resClass != NULL) {  
65.	        /* 
66.	         * If the referrer was pre-verified, the resolved class must come 
67.	         * from the same DEX or from a bootstrap class.  The pre-verifier 
68.	         * makes assumptions that could be invalidated by a wacky class 
69.	         * loader.  (See the notes at the top of oo/Class.c.) 
70.	         * 
71.	         * The verifier does *not* fail a class for using a const-class 
72.	         * or instance-of instruction referring to an unresolveable class, 
73.	         * because the result of the instruction is simply a Class object 
74.	         * or boolean -- there's no need to resolve the class object during 
75.	         * verification.  Instance field and virtual method accesses can 
76.	         * break dangerously if we get the wrong class, but const-class and 
77.	         * instance-of are only interesting at execution time.  So, if we 
78.	         * we got here as part of executing one of the "unverified class" 
79.	         * instructions, we skip the additional check. 
80.	         * 
81.	         * Ditto for class references from annotations and exception 
82.	         * handler lists. 
83.	         */  
84.	        if (!fromUnverifiedConstant &&  
85.	            IS_CLASS_FLAG_SET(referrer, CLASS_ISPREVERIFIED))  
86.	        {  
87.	            ClassObject* resClassCheck = resClass;  
88.	            if (dvmIsArrayClass(resClassCheck))  
89.	                resClassCheck = resClassCheck->elementClass;  
90.	  
91.	            if (referrer->pDvmDex != resClassCheck->pDvmDex &&  
92.	                resClassCheck->classLoader != NULL)  
93.	            {  
94.	                LOGW("Class resolved by unexpected DEX:"  
95.	                     " %s(%p):%p ref [%s] %s(%p):%p\n",  
96.	                    referrer->descriptor, referrer->classLoader,  
97.	                    referrer->pDvmDex,  
98.	                    resClass->descriptor, resClassCheck->descriptor,  
99.	                    resClassCheck->classLoader, resClassCheck->pDvmDex);  
100.	                LOGW("(%s had used a different %s during pre-verification)\n",  
101.	                    referrer->descriptor, resClass->descriptor);  
102.	                dvmThrowException("Ljava/lang/IllegalAccessError;",  
103.	                    "Class ref in pre-verified class resolved to unexpected "  
104.	                    "implementation");  
105.	                return NULL;  
106.	            }  
107.	        }  
108.	  
109.	        LOGVV("##### +ResolveClass(%s): referrer=%s dex=%p ldr=%p ref=%d\n",  
110.	            resClass->descriptor, referrer->descriptor, referrer->pDvmDex,  
111.	            referrer->classLoader, classIdx);  
112.	  
113.	        /* 
114.	         * Add what we found to the list so we can skip the class search 
115.	         * next time through. 
116.	         * 
117.	         * TODO: should we be doing this when fromUnverifiedConstant==true? 
118.	         * (see comments at top of oo/Class.c) 
119.	         */  
120.	        dvmDexSetResolvedClass(pDvmDex, classIdx, resClass);  
121.	    } else {  
122.	        /* not found, exception should be raised */  
123.	        LOGVV("Class not found: %s\n",  
124.	            dexStringByTypeIdx(pDvmDex->pDexFile, classIdx));  
125.	        assert(dvmCheckException(dvmThreadSelf()));  
126.	    }  
127.	  
128.	    return resClass;  
129.	}  
我们看下面的判断可以得到，就是在这里抛出的异常，代码逻辑我们就不看了，因为太多的头文件相互引用，看起来很费劲，直接看一下函数的说明：
 
红色部分内容，他的意思是我们需要解决从不同的dex文件中加载相同的class,需要使用不同的类加载器。
说白了就是，同一个类加载器从不同的dex文件中加载相同的class。所以上面是同一个类加载器PathClassLoader去加载(宿主apk和插件apk)来自不同的dex中的相同的类IBean。所以我们在做动态加载的时候都说过：不要把接口的jar一起打包成jar/dex/apk

问题三：Connot be cast to....(类型转化异常)
 
这个问题产生的操作：
插件工程PluginSDKs和宿主工程都是用Library方式引用工程(或者是libs)，同时将上面的一行代码
[java] view plain copy
1.	classLoader = new DexClassLoader(dexpath, fileRelease.getAbsolutePath(),null,getClassLoader());  
修改成：
[java] view plain copy
1.	classLoader = new DexClassLoader(dexpath, fileRelease.getAbsolutePath(),null,ClassLoader.getSystemClassLoader());  
就是将DexClassLoader的父加载器修改了一下：我们知道getClassLoader()获取到的是应用的默认加载器PathClassLoader，而ClassLoader.getSystemClassLoader()是获取系统类加载器，这样修改之后会出现这样的错误的原因是：插件工程和宿主工程都集成了PluginImpl，所以DexClassLoader在加载Bean的时候，首先会让ClassLoader.getSystemClassLoader()类加载器(DexClassLoader的父加载器)去查找，因为Bean是实现了IBean接口，这时候ClassLoader.getSystemClassLoader就会从插件工程的apk中查找这个接口，结果没找到，没找到的话就让DexClassLoader去找，结果在PluginSDKs.apk中找到了，就加载进来，同时宿主工程中也集成了插件接口PluginImpl，他使用PathClassLoader去宿主工程中去查找，结果也是查找到了，也加载进来了，但是在进行类型转化的时候出现了错误：
[java] view plain copy
1.	IBean bean = (IBean)beanObject;  
原因说白了就是：同一个类，用不同的类加载器进行加载产生出来的对象是不同的，不能进行相互赋值，负责就会出现转化异常。

总结
上面就说到了一些开发插件的过程中会遇到的一些问题，当我们知道这些问题之后，解决方法自然就会有了，
1) 为了避免Could not find class...，我们必须要集成PluginImpl，方式是使用Library或者是libs文件夹导入jar
(这里要注意，因为我们运行的其实是宿主工程apk，所以宿主工程一定要集成PluginImpl，如果他不集成的话，即使插件工程apk集成了也还是没有效果的)
2) 为了避免Class ref in pre-verified class resolved to unexpected implementation，我们在宿主工程和插件工程中只能集成一份PluginImpl，在结合上面的错误避免方式，可以得到正确的方式：
一定是宿主工程集成PluginImpl，插件工程一定不能集成PluginImpl。
(以后再制作插件的时候记住一句话就可以了，插件工程打包不能集成接口jar，宿主工程打包一定要集成接口jar)
关于第三个问题，其实在开发的过程中一般不会碰到，这里说一下主要是为了马上介绍Android中的类加载器的相关只是来做铺垫的

(PS:问题都解决了，后续就要介绍插件的制作了~~)







Android中插件开发篇之----动态加载Activity(免安装运行程序)
2015-08-30 14:09 25952人阅读 评论(27) 收藏 举报
  分类：
Android（158）  
版权声明：本文为博主原创文章，未经博主允许不得转载。
目录(?)[+]
一、前言
又到周末了，时间过的很快，今天我们来看一下Android中插件开发篇的最后一篇文章的内容：动态加载Activity(免安装运行程序)，在上一篇文章中说道了，如何动态加载资源(应用换肤原理解析)，没看过的同学，可以转战：
http://blog.csdn.net/jiangwei0910410003/article/details/47679843
当然，今天说道的内容还这这篇文章有关系。关于动态加载Activity的内容，网上也是有很多文章介绍了。但是他们可能大部分都是介绍通过代理的方式去实现的，所以今天我要说的加载会有两种方式：
1、使用反射机制修改类加载器
2、使用代理的方式
这两种方式都有各自的优缺点，我会在后面的文章详细解说。

二、技术介绍
1、第一种方式：使用反射机制修改类加载器来实现动态加载Activity
首先来看一个例子：360安全卫士
         
在主界面有一个添加更多工具的菜单，点进去之后，可以看到有很多功能选项。我们添加一个手机防盗的功能：有一个进度条开始添加。那么我们如何知道他是使用动态加载的呢？我们可以去查看他的数据文件目录：

我们可以看到有两个目录，比较见名知意：
app_plugins_v3
app_plugins_v3_odex
第一个目录是存放需要动态加载的功能插件，第二个目录是存放加载之后释放的dex目录。

上面分析了360的动态加载Activity功能，下面我们就来实现以下这个功能吧：
不过我们还得了解一下android中的类加载器的相关知识，这里就不做介绍了：我在这篇文章中详细介绍了类加载器：
http://blog.csdn.net/jiangwei0910410003/article/details/41384667
我们知道PathClassLoader是一个应用的默认加载器(而且他只能加载data/app/xxx.apk的文件)，但是我们加载插件一般使用DexClassLoader加载器，所以这里就有问题了，其实如果对于开始的时候，每个人都会认为很简单，很容易想到使用DexClassLoader来加载Activity获取到class对象，在使用Intent启动，这个很简单呀？但是实际上并不是想象的这么简单。原因很简单，因为Android中的四大组件都有一个特点就是他们有自己的启动流程和生命周期，我们使用DexClassLoader加载进来的Activity是不会涉及到任何启动流程和生命周期的概念，说白了，他就是一个普普通通的类。所以启动肯定会出错。

所以我们知道了问题所在，解决起来也就简单了，但是我们这里还有两种思路去解决这个问题：

1) 第一个思路： 替换LoadedApk中的mClassLoader
我们只要让加载进来的Activity有启动流程和生命周期就OK了，所以这里我们需要看一下一个Activity的启动过程。当然这里不会详细介绍一个Activity启动过程的，因为那个太复杂了，而且我也说不清楚，我们知道我们可以将我们使用的DexClassLoader加载器绑定到系统加载Activity的类加载器上就可以了，这个是我们的思路。也是最重要的突破点。下面我们就来通过源码看看如何找到加载Activity的类加载器。
加载Activity的时候，有一个很重要的类：LoadedApk.Java，这个类是负责加载一个Apk程序的，我们可以看一下他的源码：

我们可以看到他内部有一个mClassLoader变量，他就是负责加载一个Apk程序的，那么我们只要获取到这个类加载器就可以了。他不是static的，所以我们还得获取一个LoadedApk对象。我们在去看一下另外一个类：ActivityThread.java的源码

这里我们可以看到ActivityThread类中有一个自己的static对象，然后还有一个ArrayMap存放Apk包名和LoadedApk映射关系的数据结构，那么我们分析清楚了，下面就来通过反射来获取mClassLoader对象吧。
友情提示：这里可能有些同学会困惑，怎么能够找到这个mClassLoader呢。我在这里因为是为了讲解内容，所以反过来找这个东西了，其实正常情况下，我们在找关于一个Apk或者是Activity的相关信息的时候，特别是启动流程的时候，我们肯定会去找：ActivityThread.java这个类，这个类是很重要很重要的，也是关键的突破口，它内部其实有很多信息的，所以，我们应该先去找这个ActivityThread,然后从这个类中发现信息，然后会找到了LoadedApk这个类。关于ActivityThread这个类，为何如此重要，我们可以在看看他的源码：

他有这个方法？这个方法看到很熟悉呀？他不是Java程序运行的入口main方法吗？是的，没错，所有的app程序的执行入口就是这里，所以以后有人问你Android中程序运行的入口是哪里？不要再说是Application的onCreate方法了，其实是ActivityThread中的main方法。

好的，回到主干上来，我们现在开始编写代码来实现反射获取mClassLoader类，然后将其替换成我们的DexClassLoader类，不多说了，看一下Demo工程结构：
PluginActivity1 ==》插件工程
DynamicActivityForClassLoader ==》宿主工程
其中PluginActivity1工程很简单啦，就一个Activiity:
[java] view plain copy
1.	package com.example.dynamicactivityapk;  
2.	  
3.	import android.app.Activity;  
4.	import android.os.Bundle;  
5.	import android.view.View;  
6.	import android.view.View.OnClickListener;  
7.	import android.widget.Toast;  
8.	  
9.	public class MainActivity extends Activity {  
10.	      
11.	    private static View parentView;  
12.	  
13.	    @Override  
14.	    protected void onCreate(Bundle savedInstanceState) {  
15.	        super.onCreate(savedInstanceState);  
16.	        if(parentView == null){  
17.	            setContentView(R.layout.activity_main);  
18.	        }else{  
19.	            setContentView(parentView);  
20.	        }  
21.	          
22.	        findViewById(R.id.btn).setOnClickListener(new OnClickListener(){  
23.	  
24.	            @Override  
25.	            public void onClick(View arg0) {  
26.	                Toast.makeText(getApplicationContext(), "I came from 插件~~", Toast.LENGTH_LONG).show();  
27.	            }});  
28.	          
29.	    }  
30.	      
31.	    public static void setLayoutView(View view){  
32.	        parentView = view;  
33.	    }  
34.	  
35.	}  

我们看到其实这里有一个问题，为何要定义一个setLayoutView的方法，这个我们后面会说道。
我们编译这个工程，得到PluginActivity1.apk程序：


下面来看一下宿主工程
宿主工程其实最大的功能就是加载上面的PluginActivity1.apk，然后启动内部的MainActivity就可以了，这里的核心代码就是如何通过反射替换系统的mClassLoader类：
[java] view plain copy
1.	@SuppressLint("NewApi")  
2.	private void loadApkClassLoader(DexClassLoader dLoader){  
3.	    try{  
4.	  
5.	        String filesDir = this.getCacheDir().getAbsolutePath();  
6.	        String libPath = filesDir + File.separator +"PluginActivity1.apk";  
7.	  
8.	        // 配置动态加载环境  
9.	        Object currentActivityThread = RefInvoke.invokeStaticMethod(  
10.	                "android.app.ActivityThread", "currentActivityThread",  
11.	                new Class[] {}, new Object[] {});//获取主线程对象  
12.	        String packageName = this.getPackageName();//当前apk的包名  
13.	        ArrayMap mPackages = (ArrayMap) RefInvoke.getFieldOjbect(  
14.	                "android.app.ActivityThread", currentActivityThread,  
15.	                "mPackages");  
16.	        WeakReference wr = (WeakReference) mPackages.get(packageName);  
17.	        RefInvoke.setFieldOjbect("android.app.LoadedApk", "mClassLoader",  
18.	                wr.get(), dLoader);  
19.	  
20.	        Log.i("demo", "classloader:"+dLoader);  
21.	  
22.	    }catch(Exception e){  
23.	        Log.i("demo", "load apk classloader error:"+Log.getStackTraceString(e));  
24.	    }  
25.	  
26.	}  

这里有一个参数就是需要替换的DexClassLoader的，从外部传递过来，然后进行替换。我们看看外部定义的DexClassLoader类：
[java] view plain copy
1.	String filesDir = this.getCacheDir().getAbsolutePath();  
2.	String libPath = filesDir + File.separator +"PluginActivity1.apk";  
3.	Log.i("inject", "fileexist:"+new File(libPath).exists());  
4.	  
5.	//loadResources(libPath);  
6.	  
7.	DexClassLoader loader = new DexClassLoader(libPath, filesDir,filesDir, getClassLoader());  


这里，需要注意的是，DexClassLoader的最后一个参数，是DexClassLoader的parent，这里需要设置成PathClassLoader类，因为我们上面虽然说是替换PathClassLoader为DexClassLoader。但是PathClassLoader是系统本身默认的类加载器(也就是mClassLoader变量的值，我们如果单独的将DexClassLoader设置为mClassLoader的值的话，就会出错的)，所以一定要讲DexClassLoader的父加载器设置成PathClassLoader，因为类加载器是符合双亲委派机制的。

下面我们来运行一下这个程序，首先我们将PluginActivity1.apk放到宿主工程的data/data/cache目录下：

运行程序，点击加载：


运行发现失败，说是插件中的那个MainActivity没有在AndroidManifest.xml中声明？不对呀，我们在插件工程中明明声明了呀，为何他还是提示没有声明呢？

哎，其实仔细想想原因很简单的，因为DexClassLoader加载插件Apk，不会将其xml中的内容加载进来，所以在插件中声明是没有任何用途的，必须在宿主工程中声明：
[java] view plain copy
1.	<activity   
2.	    android:name="com.example.dynamicactivityapk.MainActivity">  
3.	</activity>  

我们在运行程序：
 
我们点击按钮，看效果啦。果然可以加载成功了啦啦啦。很开心了。
不过这里还是需要注意两个问题：
1>、因为要加载插件中的资源，所以需要调用loadResources方法
2>、在测试的过程中，发现插件工程中setContentView方法没有效果了。所以就在插件工程中定义一个static的方法，用来提前设置视图的。

2) 第二思路：合并PathClassLoader和DexClassLoader中的dexElements数组
好了，这里就介绍了一个如何使用反射机制来动态加载一个Activity了，但是到这里还没有结束呢？因为还要介绍另外一种方式来设置类加载器。
我们首先来看一下PathClassLoader和DexClassLoader类加载器的父类BaseDexClassloader的源码：
(这里需要注意的是PathClassLoader和DexClassLoader类的父加载器是BootClassLoader,他们的父类是BaseDexClassLoader)

这里有一个DexPathList对象，在来看一下DexPathList.java源码：

首先看一下这个类的描述，还有一个Elements数组，我们看到这个变量他是专门存放加载的dex文件的路径的，系统默认的类加载器是PathClassLoader，本身一个程序加载之后会释放一个dex出来，这时候会将dex路径放到里面，当然DexClassLoader也是一样的，那么我们会想到，我们是否可以将DexClassLoader中的dexElements和PathClassLoader中的dexElements进行合并，然后在设置给PathClassLoader中呢？这也是一个思路。我们来看代码：
[java] view plain copy
1.	/** 
2.	 * 以下是一种方式实现的 
3.	 * @param loader 
4.	 */  
5.	private void inject(DexClassLoader loader){  
6.	    PathClassLoader pathLoader = (PathClassLoader) getClassLoader();  
7.	  
8.	    try {  
9.	        Object dexElements = combineArray(  
10.	                getDexElements(getPathList(pathLoader)),  
11.	                getDexElements(getPathList(loader)));  
12.	        Object pathList = getPathList(pathLoader);  
13.	        setField(pathList, pathList.getClass(), "dexElements", dexElements);  
14.	    } catch (IllegalArgumentException e) {  
15.	        e.printStackTrace();  
16.	    } catch (NoSuchFieldException e) {  
17.	        e.printStackTrace();  
18.	    } catch (IllegalAccessException e) {  
19.	        e.printStackTrace();  
20.	    } catch (ClassNotFoundException e) {  
21.	        e.printStackTrace();  
22.	    }  
23.	}  
24.	  
25.	private static Object getPathList(Object baseDexClassLoader)  
26.	        throws IllegalArgumentException, NoSuchFieldException, IllegalAccessException, ClassNotFoundException {  
27.	    ClassLoader bc = (ClassLoader)baseDexClassLoader;  
28.	    return getField(baseDexClassLoader, Class.forName("dalvik.system.BaseDexClassLoader"), "pathList");  
29.	}  
30.	  
31.	private static Object getField(Object obj, Class<?> cl, String field)  
32.	        throws NoSuchFieldException, IllegalArgumentException, IllegalAccessException {  
33.	    Field localField = cl.getDeclaredField(field);  
34.	    localField.setAccessible(true);  
35.	    return localField.get(obj);  
36.	}  
37.	  
38.	private static Object getDexElements(Object paramObject)  
39.	        throws IllegalArgumentException, NoSuchFieldException, IllegalAccessException {  
40.	    return getField(paramObject, paramObject.getClass(), "dexElements");  
41.	}  
42.	private static void setField(Object obj, Class<?> cl, String field,  
43.	        Object value) throws NoSuchFieldException,  
44.	        IllegalArgumentException, IllegalAccessException {  
45.	  
46.	    Field localField = cl.getDeclaredField(field);  
47.	    localField.setAccessible(true);  
48.	    localField.set(obj, value);  
49.	}  
50.	  
51.	private static Object combineArray(Object arrayLhs, Object arrayRhs) {  
52.	    Class<?> localClass = arrayLhs.getClass().getComponentType();  
53.	    int i = Array.getLength(arrayLhs);  
54.	    int j = i + Array.getLength(arrayRhs);  
55.	    Object result = Array.newInstance(localClass, j);  
56.	    for (int k = 0; k < j; ++k) {  
57.	        if (k < i) {  
58.	            Array.set(result, k, Array.get(arrayLhs, k));  
59.	        } else {  
60.	            Array.set(result, k, Array.get(arrayRhs, k - i));  
61.	        }  
62.	    }  
63.	    return result;  
64.	}  
我们在运行宿主程序，发现发现也是可以的，这里就不演示了，效果都是一样的。
这里总结一下：
我们在使用反射机制来动态加载Activity的时候，有两个思路：
1>、替换LoadApk类中的mClassLoader变量的值，将我们动态加载类DexClassLoader设置为mClassLoader的值
2>、合并系统默认加载器PathClassLoader和动态加载器DexClassLoader中的dexElements数组
这两个的思路原理都是一样的：就是让我们动态加载进来的Activity能够具备正常的启动流程和生命周期。

项目下载地址：http://download.csdn.net/detail/jiangwei0910410003/9063377

2、第二种方式来动态加载Activity：静态代理的方式
首先我们也是先来看一个例子：23Code
这个应用的功能就是实时的展示一些开源的UI控件。他是在线下载，然后动态加载进行展示的：
            
我们看到，点击运行Demo的时候，他会去下载apk,我们看看他的数据目录结构：


这里我们看到了，他把下载之后的apk都用每个插件的功能包名存起来的。
好了，上面分析了23Code的加载机制，我们来看看如何使用代理的方式来动态加载Activity
先来看看原理：

所以说，这种方式来加载Activity的话，其实真正意义上每个插件的Activity都不再是想方式一中的那样，没有生命周期，没有启动流程了，他们就是一个普通的Activity类，然后将其生命周期的所有任务都交给代理Activity去执行就可以了。
下面我们来看一下项目工程：
1、DynamicActivityForProxy ==》宿主工程
2、PluginActivity2 ==》插件工程
看一下插件工程
BaseActivity.java
[java] view plain copy
1.	package com.example.dynamicactivity;  
2.	  
3.	import com.example.dynamicactivity.R;  
4.	  
5.	import android.app.Activity;  
6.	import android.os.Bundle;  
7.	import android.view.View;  
8.	import android.view.View.OnClickListener;  
9.	import android.widget.Toast;  
10.	  
11.	public class BaseActivity extends Activity {  
12.	    protected Activity mProxyActivity;  
13.	  
14.	    public void setProxy(Activity proxyActivity) {   
15.	        mProxyActivity = proxyActivity;  
16.	    }  
17.	  
18.	    @Override  
19.	    protected void onCreate(Bundle savedInstanceState) {  
20.	    }  
21.	  
22.	    @Override  
23.	    public void setContentView(int layoutResID) {   
24.	        if (mProxyActivity != null && mProxyActivity instanceof Activity) {  
25.	            mProxyActivity.setContentView(layoutResID);  
26.	            mProxyActivity.findViewById(R.id.btn).setOnClickListener(  
27.	                    new OnClickListener() {  
28.	                        @Override  
29.	                        public void onClick(View v) {  
30.	                            Toast.makeText(mProxyActivity, "我是插件，你是谁!",Toast.LENGTH_LONG).show();  
31.	                        }  
32.	                    });  
33.	        }  
34.	    }  
35.	  
36.	}  
这里会重写setContentView的方法，同时有一个setProxy方法。
在来看一下MainActivity.java
[java] view plain copy
1.	package com.example.dynamicactivity;  
2.	  
3.	import com.example.dynamicactivity.R;  
4.	  
5.	import android.os.Bundle;  
6.	import android.util.Log;  
7.	  
8.	public class MainActivity extends BaseActivity {  
9.	      
10.	    @Override  
11.	    protected void onCreate(Bundle savedInstanceState) {  
12.	        super.onCreate(savedInstanceState);  
13.	        setContentView(R.layout.activity_main);  
14.	    }  
15.	  
16.	    @Override  
17.	    protected void onDestroy() {  
18.	        //这里需要注意的是，不能调用super.onDestroy的方法了，不然报错，原因也很简单，这个Activity实质上不是真正的Activity了，没有生命周期的概念了，调用super的方法肯定报错  
19.	        Log.i("demo", "onDestory");  
20.	    }  
21.	  
22.	    @Override  
23.	    protected void onPause() {  
24.	        Log.i("demo", "onPause");  
25.	    }  
26.	  
27.	    @Override  
28.	    protected void onResume() {  
29.	        Log.i("demo", "onResume");  
30.	    }  
31.	  
32.	    @Override  
33.	    protected void onStart() {  
34.	        Log.i("demo", "onStart");  
35.	    }  
36.	  
37.	    @Override  
38.	    protected void onStop() {  
39.	        Log.i("demo", "onStop");  
40.	    }  
41.	      
42.	      
43.	  
44.	}  
这里打印一下生命周期中的每个方法，待会需要验证。
注意：这里的生命周期方法不能再调用super.XXX方法了，否则会报错的，原因很简单啦。插件Activity不在是真正意义上的Activity了，就是一个空壳的Activity。所以调用的话，肯定会出错
运行插件工程，得到一个PluginActivity2.apk

下面来看一下宿主工程：
首先我们来看一下重要的代理对象ProxyActiviity
[java] view plain copy
1.	package com.example.dynamic.activity;  
2.	  
3.	import java.io.File;  
4.	import java.lang.reflect.Constructor;  
5.	import java.lang.reflect.Method;  
6.	import java.util.HashMap;  
7.	  
8.	import android.annotation.SuppressLint;  
9.	import android.app.Activity;  
10.	import android.os.Bundle;  
11.	import android.util.Log;  
12.	import dalvik.system.DexClassLoader;  
13.	  
14.	public class ProxyActivity extends BaseActivity {  
15.	      
16.	    private Object pluginActivity;  
17.	    private Class<?> pluginClass;  
18.	      
19.	    private HashMap<String, Method> methodMap = new HashMap<String,Method>();  
20.	  
21.	    @SuppressLint("NewApi")  
22.	    @Override  
23.	    protected void onCreate(Bundle savedInstanceState) {  
24.	        super.onCreate(savedInstanceState);  
25.	        try {  
26.	            DexClassLoader loader = initClassLoader();  
27.	              
28.	            //动态加载插件Activity  
29.	            pluginClass = loader.loadClass("com.example.dynamicactivity.MainActivity");  
30.	            Constructor<?> localConstructor = pluginClass.getConstructor(new Class[] {});  
31.	            pluginActivity = localConstructor.newInstance(new Object[] {});  
32.	  
33.	            //将代理对象设置给插件Activity  
34.	            Method setProxy = pluginClass.getMethod("setProxy",new Class[] { Activity.class });   
35.	            setProxy.setAccessible(true);  
36.	            setProxy.invoke(pluginActivity, new Object[] { this });  
37.	              
38.	            initMethodMap();  
39.	  
40.	            //调用它的onCreate方法  
41.	            Method onCreate = pluginClass.getDeclaredMethod("onCreate",  
42.	                    new Class[] { Bundle.class });  
43.	            onCreate.setAccessible(true);  
44.	            onCreate.invoke(pluginActivity, new Object[] { new Bundle() });  
45.	              
46.	        } catch (Exception e) {  
47.	            Log.i("demo", "load activity error:"+Log.getStackTraceString(e));  
48.	        }  
49.	    }  
50.	      
51.	    /** 
52.	     * 存储每个生命周期的方法 
53.	     */  
54.	    private void initMethodMap(){  
55.	        methodMap.put("onPause", null);  
56.	        methodMap.put("onResume", null);  
57.	        methodMap.put("onStart", null);  
58.	        methodMap.put("onStop", null);  
59.	        methodMap.put("onDestroy", null);  
60.	          
61.	        for(String key : methodMap.keySet()){  
62.	            try{  
63.	                Method method = pluginClass.getDeclaredMethod(key);  
64.	                method.setAccessible(true);  
65.	                methodMap.put(key, method);  
66.	            }catch(Exception e){  
67.	                Log.i("demo", "get method error:"+Log.getStackTraceString(e));  
68.	            }  
69.	        }  
70.	          
71.	    }  
72.	      
73.	    @SuppressLint("NewApi")  
74.	    private DexClassLoader initClassLoader(){  
75.	        String filesDir = this.getCacheDir().getAbsolutePath();  
76.	        String libPath = filesDir + File.separator +"PluginActivity2.apk";  
77.	        Log.i("inject", "fileexist:"+new File(libPath).exists());  
78.	        loadResources(libPath);  
79.	        DexClassLoader loader = new DexClassLoader(libPath, filesDir,null , getClass().getClassLoader());  
80.	        return loader;  
81.	    }  
82.	      
83.	    @Override  
84.	    protected void onDestroy() {  
85.	        super.onDestroy();  
86.	        Log.i("demo", "proxy onDestroy");  
87.	        try{  
88.	            methodMap.get("onDestroy").invoke(pluginActivity, new Object[]{});  
89.	        }catch(Exception e){  
90.	            Log.i("demo", "run destroy error:"+Log.getStackTraceString(e));  
91.	        }  
92.	    }  
93.	  
94.	    @Override  
95.	    protected void onPause() {  
96.	        super.onPause();  
97.	        Log.i("demo", "proxy onPause");  
98.	        try{  
99.	            methodMap.get("onPause").invoke(pluginActivity, new Object[]{});  
100.	        }catch(Exception e){  
101.	            Log.i("demo", "run pause error:"+Log.getStackTraceString(e));  
102.	        }  
103.	    }  
104.	  
105.	    @Override  
106.	    protected void onResume() {  
107.	        super.onResume();  
108.	        Log.i("demo", "proxy onResume");  
109.	        try{  
110.	            methodMap.get("onResume").invoke(pluginActivity, new Object[]{});  
111.	        }catch(Exception e){  
112.	            Log.i("demo", "run resume error:"+Log.getStackTraceString(e));  
113.	        }  
114.	    }  
115.	  
116.	    @Override  
117.	    protected void onStart() {  
118.	        super.onStart();  
119.	        Log.i("demo", "proxy onStart");  
120.	        try{  
121.	            methodMap.get("onStart").invoke(pluginActivity, new Object[]{});  
122.	        }catch(Exception e){  
123.	            Log.i("demo", "run start error:"+Log.getStackTraceString(e));  
124.	        }  
125.	    }  
126.	  
127.	    @Override  
128.	    protected void onStop() {  
129.	        super.onStop();  
130.	        Log.i("demo", "proxy onStop");  
131.	        try{  
132.	            methodMap.get("onStop").invoke(pluginActivity, new Object[]{});  
133.	        }catch(Exception e){  
134.	            Log.i("demo", "run stop error:"+Log.getStackTraceString(e));  
135.	        }  
136.	    }  
137.	  
138.	}  
这里主要就是：
1、加载插件Activity
2、使用反射将代理对象设置给插件Activity
3、测试插件Activity中的生命周期方法
运行程序，我们将上面的PluginActivity2.apk放到宿主程序的cache目录下：


运行：
 
也是成功了，看到效果啦啦，同时我们打印一下Log日志：

插件Activity的每个生命周期的方法也是都执行了。
到这里我们就讲解完了使用代理的方式来实现动态加载Activity。这种方式其实还是很简单的。

项目下载地址：http://download.csdn.net/detail/jiangwei0910410003/9063483

总算是讲完了，累死了，两种方式各有千秋，各有各的好处。

三、案例分析
上面讲解的两种方式的时候，介绍了两个例子：一个是360卫士，一个是23Code
为什么要用这两个例子呢？原因下载说明一下啦：
1、首先来看一下360卫士，我们打开它的一个辅助功能：
使用：adb shell dumpsys activity top

我们再来看一下他的AndroidManifest.xml内容：

我们可以看到，他在宿主工程中声明了，而且我们会看到有很多个辅助功能都声明了。所以我们断定他使用的是第一种方式去实现动态加载的，所以我们可以看到这种方式有一个不好就是：需要在宿主工程中声明很多个插件Activity。

2、在来看一下23Code应用
运行一个例子之后，我们看到他的Activity是TestActivity

我们再去切换另外一个例子运行之后，也是发现还是这个TestActivity。看看他的AndroidManifest.xml

那么我们断定，这个TestActivity就是一个代理的Activity,他使用的是第二种方式来实现动态加载的。
现在知道为何我开始的时候为什么用这两个例子来说明了吧。

四、两种方式的比较
第一种方式：使用反射机制来实现
优点：可以不用太多的关心插件中的Activity的生命周期方法，因为他加载进来之后就是一个真正意义上的Activity了
缺点：需要在宿主工程中进行声明，如果插件中的Activity多的话，那么就不灵活了。
第二种方式：使用代理机制来实现
优点：不需要在宿主工程中进行声明太多的Activity了，只需要有一个代理Activity的声明就可以了，很灵活
缺点：需要管理手动的去管理插件中Activity的生命周期方法，难度复杂。

五、存在的问题
我们看到上面讲到的两种方式去动态加载Activity,其实两种方式都还存在很多问题：
1、其他组件的动态加载问题(服务，广播，ContentProvier)
2、跨进程访问的问题

六、总结
这篇文章讲完之后，那么插件开发篇的三部曲就算结束了，Android中的插件开发也算是有一个好的总结了，也是讲解了现在主流市场中加载的原理和机制。当然我们在真正的使用过程中还会存在很多问题，当然这个就需要我们自己去探索和解决了。
(PS：两种方式的项目下载地址都在上面给出了，如果在运行的过程中有什么问题的话，请留言。能帮就尽量帮助解决一下~~)


