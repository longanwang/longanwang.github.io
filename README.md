# longanwang.github.io
王大年的学习笔记
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
