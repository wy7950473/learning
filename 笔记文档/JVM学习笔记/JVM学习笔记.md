## 一.类加载

#### 1.类加载过程

------

1.在Java代码中，类型(class、interface、enum...)的**加载**、**连接**与**初始化**过程都是在程序**==运行期间==**完成的（好处：提供了更大的灵活性，增加了更多的功能）

- 加载：查找并加载类的二进制数据
- 连接
   - -验证：确保被加载的类的正确性
   - -准备：为类的==静态变量==分配内存，并将其初始化为==默认值==
   - -解析：==把类的符号引用转换为直接引用==

- 初始化：为类的静态变量赋予正确的初始值（clinit）
- 使用
- 卸载

![屏幕快照 2020-01-06 下午8.56.35](%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202020-01-06%20%E4%B8%8B%E5%8D%888.56.35.png)

2.Java程序对类的使用方式分为两种

- 主动使用(==七种==)
   - -创建类的实例
   - -访问某个类或接口的静态变量(getstatic)，或者对该静态变量赋值(putstatic)
   - -调用类的静态方法(invokestatic)
   - -反射(如Class.forName("com.example.Test"))
   - -初始化一个类的子类
   - -Java虚拟机启动时被标明为启动类的类(Java Test)
   - -JDK1.7开始提供的动态语言支持：java.lang.invoke.MethodHandle实例的解析结果REF_getStatic,REF_putStatic,REF_invokeStatic句柄对应的类没有初始化，则初始化
- 被动使用
   - 除了以上七种情况，其他使用Java类的方式都被看作对类的==被动使用==，都不会导致类的==初始化==

3.所有Java虚拟机实现必须在每个类或接口被Java程序“首次主动使用”时才初始化他们

````java
/**
 * 对于静态字段来说，只有直接定义了该字段的类才会被初始化
 * 当一个类在初始化时，要求其父类全部都已经初始化完毕了
 * -XX:+TraceClassLoading,用于追踪类的加载信息并打印出来
 * -XX:+TraceClassUnloading,用于追踪类的卸载信息并打印出来
 *
 * -XX:+<option>,表示开启option选项
 * -XX:-<option>,表示关闭option选项
 * -XX:<option>=<value>,表示将option选项的值设置为value
 */

public class MyTest1 {
    public static void main(String[] args) {
        System.out.println(MyChild1.str);
        System.out.println(MyChild1.str2);
    }
}

class MyParent1 {
   public static String str = "hello world";

   static {
       System.out.println("MyParent1 static block");
   }
}

class MyChild1 extends MyParent1{
    public static String str2 = "Welcome";

    static {
        System.out.println("MyChild1 static block");
    }
}
````

````java
/**
 * 常量在编译阶段会存入到调用这个常量的方法所在的类的常量池中，
 * 本质上，调用类并没用直接引用定义常量的类，因此不会触发定义
 * 常量的类的初始化。
 * 注意⚠️：这里指的是将常量存放到了MyTest2的常量池中，之后MyTest2与
 * MyParent2就没有任何关系了。甚至，我们可以将MyParent2的class文件删除
 */

/**
 * javap -c <class文件> ：反编译class文件
 */

/**
 * 助记符：
 * ldc表示将int,float或是String类型的常量值从常量池中推送到栈顶
 * bipush表示将单字节（-128 ～ 127）的常量指推送至栈顶
 * sipush表示将一个短整型常量值（-32768 ～ 32767）推送至栈顶
 * iconst_1表示将int类型1推送至栈顶（iconst_m1(-1) ~ iconst_5）
 */
public class MyTest2 {
    public static void main(String[] args) {
        System.out.println(MyParent2.str);
        System.out.println(MyParent2.a);
        System.out.println(MyParent2.i);
        System.out.println(MyParent2.m);
    }
}

class MyParent2 {
    public static final String str = "hello world";

    public static final short a = 127;

    public static final int i = 128;

    public static final int m = 2;

    static {
        System.out.println("MyParent2 static block");
    }
}
````

````java
/**
 * 当一个常量的值并非编译期可以确定的，那么其值就不会放到调用类的常量池中，
 * 这时在程序，会导致主动使用这个常量所在的类，显然会导致这个类被初始化。
 */

public class MyTest3 {
    public static void main(String[] args) {
        System.out.println(MyPrent3.str);
    }
}

class MyPrent3 {
    public static final String str = UUID.randomUUID().toString();

    static {
        System.out.println("MyParents static block");
    }
}
````

````java
/**
 * 对于数组实例来说，其类型是由JVM在运行期动态生成的，表示为[Lcom.wanyi.jvm.classloader.Myparent4
 * 这种形式，动态生成的类型，其父类型就是Object
 * 对于数组来说，JavaDoc经常将构成数组的元素为Component,实际上就是将数组降低一个维度后的类型
 */

/**
 * 助记符
 * anewarray：表示创建一个引用类型的（如类、接口、数组）数组，并将其引用值压入栈顶
 * newarray：表示创建一个指定的原始类型（如int、float、char等）的数组，并将其引用值压入栈顶
 *
 */
public class MyTest4 {
    public static void main(String[] args) {
        MyParent4 myParent4 = new MyParent4();
        System.out.println("=====");
        MyParent4 myParent5 = new MyParent4();

        //---------------------------------

        MyParent4[] myParent4s = new MyParent4[1];
        System.out.println(myParent4s.getClass());

        MyParent4[][] myParent4s1 = new MyParent4[1][1];
        System.out.println(myParent4s1.getClass());

        System.out.println(myParent4s.getClass().getSuperclass());
        System.out.println(myParent4s1.getClass().getSuperclass());

        //-----------------------------

        int[] ints = new int[1];
        System.out.println(ints.getClass());
        System.out.println(ints.getClass().getSuperclass());

        char[] chars = new char[1];
        System.out.println(chars.getClass());

        boolean[] booleans = new boolean[1];
        System.out.println(booleans.getClass());

        short[] shorts = new short[1];
        System.out.println(shorts.getClass());

        byte[] bytes = new byte[1];
        System.out.println(bytes.getClass());
    }
}

class MyParent4 {

    static {
        System.out.println("MyParent4 static block");
    }
}
````

````java
/**
 * 当一个接口在初始化时，并不要求其父接口都完成了初始化
 * 只有在真正使用到父接口的时候（如引用接口中定义的常量时），才会初始化
 */

public class MyTest5 {
    public static void main(String[] args) {
        System.out.println(MyChild5.b);
    }
}

interface MyParent5 {
    public static final int a = 5;
}

interface MyChild5 extends MyParent5 {
    public static final int b = new Random().nextInt(6);
}
````

````java
public class MyTest6 {
    public static void main(String[] args) {
        Singleton singleton = Singleton.getInstance();
        System.out.println("counter1===>"+Singleton.counter1);
        System.out.println("counter2===>"+Singleton.counter2);
    }
}

class Singleton {
    public static int counter1 = 1;

    //public static int counter2 = 0;

    private static Singleton singleton = new Singleton();

    private Singleton(){
        counter1++;
        counter2++; // 准备阶段的重要意义
    }

    public static int counter2 = 0;

    public static Singleton getInstance(){
        return singleton;
    }
}
````

4.类的加载：类的加载是指将类的.class文件中的二进制数据读入内存中，将其放在运行时数据区的方法区内然后创建一个java.lang.Class对象(规范并未说明Class对象位于哪里，HotSpot虚拟机将其放在方法区中)用来封装类在方法区中的数据结构(一个类不管有多少个实例，但是它对应的Class对象只有一个)

5.加载.class文件的方式

- -从本地系统中直接加载
- -通过网络下载.class文件
- -从zip，jar等归档文件中加载.class文件
- 从专有数据库中提取.class文件
- ==将Java源文件动态编译为.class文件==（动态代理、将jsp转换为servlet然后转换为.class文件）

6.==类的加载的最终产品是位于内存中的Class对象==

7.Class对象封装了类在方法区的数据结构，并且向Java程序员提供了访问方法区内的数据结构的接口

8.==有两种类型的类加载器==

- Java虚拟机自带的加载器
   - 根类加载器（Bootstrap ClassLoader）：$JAVA_HOME中jre/lib/rt.jar里面所有的class，由C++实现，不是ClassLoader子类
   - 扩展类加载器（Extension ClassLoader）: 负责加载java平台中扩展功能的一些jar包，包括$JAVA_HOME中jre/lib/*.jar或-Djava.ext.dirs指定目录下的jar包
   - 系统（应用）类加载器（System、App ClassLoader）: 负责加载classpath中指定的jar包及目录中class

- 用户自定义的类加载器（自定义类加载器都应该继承ClassLoader类）
   - java.lang.ClassLoader的子类
   - 用户可以定制类的加载方式

![屏幕快照 2020-01-06 下午10.12.44](%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202020-01-06%20%E4%B8%8B%E5%8D%8810.12.44.png)

![屏幕快照 2020-01-06 下午10.13.11](%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202020-01-06%20%E4%B8%8B%E5%8D%8810.13.11.png)

![屏幕快照 2020-01-06 下午10.13.35](%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202020-01-06%20%E4%B8%8B%E5%8D%8810.13.35.png)

````java
public class MyTest14 {
    public static void main(String[] args) {
        ClassLoader classLoader = ClassLoader.getSystemClassLoader();
        System.out.println(classLoader);
        while (classLoader != null){
            classLoader = classLoader.getParent();

            System.out.println(classLoader);
        }
    }
}
````

````java
public class MyTest15 {
    public static void main(String[] args) throws IOException {
        ClassLoader classLoader = Thread.currentThread().getContextClassLoader();
        String resourceName = "com/wanyi/jvm/classloader/MyTest14.class";
        Enumeration<URL> urls = classLoader.getResources(resourceName);
        while (urls.hasMoreElements()){
            URL url = urls.nextElement();
            System.out.println(url);
        }
       
        Class<?> clazz = MyTest15.class;
        System.out.println(clazz.getClassLoader());

        clazz = String.class;
        System.out.println(clazz.getClassLoader());
    }
}
````

9.类加载器并不需要等到某个类被“首次主动使用”时再加载它

10.JVM规范允许类加载器在预料某个类将要被使用时就预先加载它，如果在预先加载的过程中遇到了.class文件缺失或存在错误，类加载器必须在==程序首次主动使用==该类时才报告错误（==LinkageError==错误）

11.如果这个类一直没有被程序主动使用，那么==类加载器就不会报告错误==

12.类被加载后，就进入连接阶段。连接就是将已经读入到内存的类的二进制数据合并到虚拟机的运行时环境中去

13.类的验证的内容

- -类文件的结构检查
- -语义检查
- -字节码验证
- -二进制兼容性验证

![屏幕快照 2020-01-06 下午9.41.12](%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202020-01-06%20%E4%B8%8B%E5%8D%889.41.12.png)

![屏幕快照 2020-01-06 下午9.45.59](%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202020-01-06%20%E4%B8%8B%E5%8D%889.45.59.png)

![屏幕快照 2020-01-06 下午9.47.03](%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202020-01-06%20%E4%B8%8B%E5%8D%889.47.03.png)

14.类的初始化步骤

- 假如这类还没有被加载和连接，那就先进行加载和连接
- 假如这个类存在直接的父类，并且这个父类还没有被初始化，那就先初始化直接父类
- 假如类中存在初始化语句，那就依次执行这些初始化语句

![屏幕快照 2020-01-06 下午9.56.22](%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202020-01-06%20%E4%B8%8B%E5%8D%889.56.22.png)

````java
public class MyTest7 {
    public static void main(String[] args) {
        System.out.println(MyChild7.b);
        //-----------------------------
        System.out.println(MyParent7.thead);
    }
}

interface MyGrandpa {
    public static Thread thead = new Thread() {
        {
            System.out.println("MyGrandpa invoked");
        }
    };
}


interface MyParent7 extends MyGrandpa{
    public static Thread thead = new Thread() {
        {
            System.out.println("MyParent7 invoked");
        }
    };
}

class MyChild7 implements MyParent7 {
    public static int b = 5;
}

class C {
    {
        System.out.println("Hello");
    }

    public C(){
        System.out.println("C");
    }
}
````

15.只有当程序访问的静态变量或静态方法确实在当前类或接口中定义时，才可以认为是对当前类或接口的主动使用

16.调用ClassLoader类的loadClass方法加载一个类，并不是对类的主动使用，不会导致类的初始化

17.在父亲委托机制中，各个加载器按照父子关系形成==树形结构==，除了根类加载器之外，其余的类加载器都有且只有一个父加载器。

18.若有一个类加载器能够成功加载Test类，那么这个类加载被称为==定义类加载器==，所有能够返回Class对象引用的类加载器（包括定义类加载器）都被称为==初始类加载器==

![屏幕快照 2020-01-07 下午10.06.01](%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202020-01-07%20%E4%B8%8B%E5%8D%8810.06.01.png)

````java
public class MyTest8 {
    public static void main(String[] args) throws ClassNotFoundException {
        Class<?> clazz = Class.forName("java.lang.String");
        System.out.println(clazz.getClassLoader());
        //------------------------------------------------
        Class<?> clazz2 = Class.forName("com.wanyi.jvm.classloader.D");
        System.out.println(clazz2.getClassLoader());
    }
}

class D {

}
````

````java
/**
 * 调用ClassLoader类的loadCLass方法加载一个类，并不是对类的主动使用，不会导致类的初始化
 * 反射是对类的主动使用，会导致类的初始化
 */
public class MyTest13 {
    public static void main(String[] args) throws ClassNotFoundException {
        ClassLoader loader = ClassLoader.getSystemClassLoader();
        Class<?> clazz = loader.loadClass("com.wanyi.jvm.classloader.CL");
        System.out.println(clazz);
        System.out.println("-------------->");
        clazz = Class.forName("com.wanyi.jvm.classloader.CL");
        System.out.println(clazz);
    }
}

class CL {
    static {
        System.out.println("Class CL");
    }
}
````

19.获取ClassLoader的途径

````java
获取当前类的ClassLoader
clazz.getClassLoader();

获取当前线程上下文的ClassLoader
Thread.currentThread().getContextClassLoader();
   
获取系统的ClassLoader
ClassLoader.getSystemClassLoader();

获取调用者的ClassLoader
DriverManager.getCallerClassLoader();
````

20.命名空间

- 每个类加载器都有自己的命名空间，==命名空间由该加载器及所有父加载器所加载的类组成==
- 在同一个命名空间中，不会出现类的完整名字（包括类的包名）相同的两个类
- 在不同的命名空间中，可能会出现类的完整名字（包括类的包名）相同的两个类

![屏幕快照 2020-01-12 下午8.36.53](%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202020-01-12%20%E4%B8%8B%E5%8D%888.36.53.png)

21.类的卸载

- 当某个类被加载、连接和初始化后，它的生命周期就开始了。当代表这个类的Class对象不在被引用，即不可触及时，Class对象就会结束生命周期，该类在方法区的数据也会被卸载，从而结束这个类的生命周期。
- ==一个类何时结束生命周期，取决于代表它的Class对象何时结束生命周期==

![屏幕快照 2020-01-11 下午3.12.29](%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202020-01-11%20%E4%B8%8B%E5%8D%883.12.29.png)

![屏幕快照 2020-01-11 下午3.18.04](%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202020-01-11%20%E4%B8%8B%E5%8D%883.18.04.png)

````java
package com.wanyi.jvm.classloader;

import java.io.ByteArrayOutputStream;
import java.io.File;
import java.io.FileInputStream;
import java.io.InputStream;

public class MyTest17 extends ClassLoader {

    private String classLoaderName;

    private String path;

    private String fileExtension = ".class";

    public MyTest17(String classLoaderName){
        // 将系统类加载器当做该类加载器的父类加载器
        super();
        this.classLoaderName = classLoaderName;
    }

    public MyTest17(ClassLoader parent,String classLoaderName){
        // 显示指定该类加载器的父类加载器
        super(parent);
        this.classLoaderName = classLoaderName;
    }

    public void setPath(String path) {
        this.path = path;
    }

    @Override
    public String toString() {
        return "[" + "classLoaderName=" + this.classLoaderName + "]";
    }

    @Override
    protected Class<?> findClass(String name) throws ClassNotFoundException {
        System.out.println("findClass invoked:"+name);
        System.out.println("class loader name:"+this.classLoaderName);
        byte[] data = this.loadClassData(name);
        return this.defineClass(name,data,0,data.length);
    }

    private byte[] loadClassData(String name){
        InputStream in = null;
        byte[] data = null;
        ByteArrayOutputStream bos = null;
        name = name.replace(".","/");
        try{
            //this.classLoaderName = this.classLoaderName.replaceAll(".","/");
            in = new FileInputStream(new File(path + name + this.fileExtension));
            bos = new ByteArrayOutputStream();
            int ch;
            while ((ch = in.read()) != -1){
                bos.write(ch);
            }
            data = bos.toByteArray();
        }catch (Exception e){
            e.printStackTrace();
        } finally {
            try{
                in.close();
                bos.close();
            } catch (Exception e){
                e.printStackTrace();
            }
        }
        return data;
    }

    public static void main(String[] args) throws Exception {
        MyTest17 loader = new MyTest17("load1");
        //loader.setPath("/Users/wanyi/project/jvm_lecture/build/classes/");
        loader.setPath("/Users/wanyi/Desktop/");
        Class<?> clazz = loader.loadClass("com.wanyi.jvm.classloader.MyTest1");
        Object object = clazz.newInstance();
        System.out.println("class:"+clazz.hashCode());
        System.out.println(object);
        System.out.println(object.getClass().getClassLoader());

        loader = null;
        object = null;
        clazz = null;
        // 类的卸载
        System.gc();

        Thread.sleep(2000000);

        System.out.println("====================================>");

        loader = new MyTest17("loader1");
        loader.setPath("/Users/wanyi/Desktop/");
        clazz = loader.loadClass("com.wanyi.jvm.classloader.MyTest1");
        object = clazz.newInstance();
        System.out.println("class:"+clazz.hashCode());
        System.out.println(object);
        System.out.println(object.getClass().getClassLoader());

//        System.out.println("----------------------------------------->");
//
//        MyTest17 loader2 = new MyTest17(loader,"loader2");
//        loader2.setPath("/Users/wanyi/Desktop/");
//        Class<?> clazz2 = loader2.loadClass("com.wanyi.jvm.classloader.MyTest1");
//        Object object2 = clazz2.newInstance();
//        System.out.println("class2:"+clazz2.hashCode());
//        System.out.println(object2);
//        System.out.println(object2.getClass().getClassLoader());
//
//        System.out.println("----------------------------------------->");
//
//        MyTest17 loader3 = new MyTest17(loader2,"loader3");
//        loader3.setPath("/Users/wanyi/Desktop/");
//        Class<?> clazz3 = loader3.loadClass("com.wanyi.jvm.classloader.MyTest1");
//        Object object3 = clazz3.newInstance();
//        System.out.println("class2:"+clazz3.hashCode());
//        System.out.println(object3);
//        System.out.println(object3.getClass().getClassLoader());
    }
}
````

````java
public class MyCat {

    public MyCat(){
        System.out.println("MyCat is loaded by:"+this.getClass().getClassLoader());

        //System.out.println("from MyCat:"+MySample.class);
    }
}

public class MySample {
    public MySample(){
        System.out.println("MySampl is loaded by:"+this.getClass().getClassLoader());

        new MyCat();
    }
}

public class MyTest18 {
    public static void main(String[] args) throws Exception {
        MyTest17 loader = new MyTest17("loader");
        Class<?> clazz = loader.loadClass("com.wanyi.jvm.classloader.MySample");
        System.out.println("class:"+clazz.hashCode());

        // 如果注释掉改行，那么并不会实例化MySmaple对象，即MySample掉构造方法不会被调用
        // 因此不会实例化MyCat对象，即没有对MyCat进行主动使用，这里就不会加载MyCat Class
        Object object = clazz.newInstance();
    }
}

/**
 * 命名空间的重要说明：
 * 1.子加载器所加载的类能够访问到父加载器所加载器所加载的类
 * 2.父加载器所加载的类无法访问到子加载器所加载器所加载的类
 */
public class MyTest18_1 {
    public static void main(String[] args) throws Exception {
        MyTest17 loader = new MyTest17("loader");
        loader.setPath("/Users/wanyi/Desktop/");
        Class<?> clazz = loader.loadClass("com.wanyi.jvm.classloader.MySample");
        System.out.println("class:"+clazz.hashCode());

        // 如果注释掉改行，那么并不会实例化MySmaple对象，即MySample掉构造方法不会被调用
        // 因此不会实例化MyCat对象，即没有对MyCat进行主动使用，这里就不会加载MyCat Class
        Object object = clazz.newInstance();
    }
}
````

````java
public class MyTest19 {
    public static void main(String[] args) {
        // 根类加载器加载.class文件的路径
        System.out.println(System.getProperty("sun.boot.class.path"));
        // 扩展类加载器加载.class文件的路径
        System.out.println(System.getProperty("java.ext.dirs"));
        // 系统（应用）类加载器加载类的路径
        System.out.println(System.getProperty("java.class.path"));
    }
}
````

````java
/**
 *  java -Djava.ext.dirs=./ com.wanyi.jvm.classloader.MyTest20
 * -Djava.ext.dirs=./  把扩展类加载器加载的目录换成当前目录
 */

public class MyTest20 {
    public static void main(String[] args) {
        AESKeyGenerator aesKeyGenerator = new AESKeyGenerator();
        // 扩展类加载器加载
        System.out.println(aesKeyGenerator.getClass().getClassLoader());
        // 系统（应用）类加载器加载
        System.out.println(MyTest20.class.getClassLoader());
    }
}
````

````java
public class MyPerson {
    private MyPerson myPerson;

    public void setMyPerson(Object object){
        this.myPerson = (MyPerson) object;
    }
}

public class MyTest21 {
    public static void main(String[] args) throws Exception {
        MyTest17 loader1 = new MyTest17("loader1");
        MyTest17 loader2 = new MyTest17("loader2");

        Class<?> clazz1 = loader1.loadClass("com.wanyi.jvm.classloader.MyPerson");
        Class<?> clazz2 = loader2.loadClass("com.wanyi.jvm.classloader.MyPerson");

        System.out.println(clazz1 == clazz2);

        Object object1 = clazz1.newInstance();
        Object object2 = clazz2.newInstance();

        Method method = clazz1.getMethod("setMyPerson",Object.class);
        method.invoke(object1,object2);
    }
}

/**
 * 类加载器的双亲委托模型的好处：
 * 1、可以确保Java核心库的类型安全：所有的Java应用都至少会引用java.lang.Object类，
 * 也就是说在运行期，java.lang.Object这个类库会被加载到Java虚拟机中；如果这个加载过程
 * 是由Java应用自己的类加载所完成，那么很可能就会在JVM中存在多个版本的java.lang.Object类，
 * 而且这些类之间还是不兼容的，相互不可见的（正是命名空间在发挥着作用）。借助于双亲委托机制，
 * Java核心类库中的类的加载工作都是由启动类加载器来统一完成，从而确保了Java应用所使用的都是
 * 同一版本的。
 * 2、可以确保Java核心类库所提供的类不会被自定义的类所替代。
 * 3、不同的类加载器可以为相同名称（binary name）的类创建额外的命名空间。相同名称的类可以并存
 * 在Java虚拟机中，只需要用不同的类加载器来加载即可。不同类加载器所加载的类之间是不兼容的，这就相当于
 * 在Java虚拟机内部创建了一个又一个相互隔离的Java类空间，这类技术在很多框架中都得到了实际应用。
 */
public class MyTest22 {
    public static void main(String[] args) throws Exception{
        MyTest17 loader1 = new MyTest17("loader1");
        MyTest17 loader2 = new MyTest17("loader2");

        loader1.setPath("/Users/wanyi/Desktop/");
        loader2.setPath("/Users/wanyi/Desktop/");

        Class<?> clazz1 = loader1.loadClass("com.wanyi.jvm.classloader.MyPerson");
        Class<?> clazz2 = loader2.loadClass("com.wanyi.jvm.classloader.MyPerson");

        System.out.println(clazz1 == clazz2);

        Object object1 = clazz1.newInstance();
        Object object2 = clazz2.newInstance();

        Method method = clazz1.getMethod("setMyPerson",Object.class);
        method.invoke(object1,object2);
    }
}
````

````java
/**
 *  jar cvf test.jar com/wanyi/jvm/classloader/MyTest1.class : 把.class文件打成jar包
 *  扩展类加载器不能直接加载特定路径下打.class文件，需要把文件打成jar包，然后加载jar包中打文件
 */

public class MyTest23 {
    static {
        System.out.println("MyTest23 initializer");
    }

    public static void main(String[] args) {
        System.out.println(MyTest23.class.getClassLoader());

        System.out.println(MyTest1.class.getClassLoader());
    }
}
````

````java
/**
 * 在运行期，一个Java类是由该类的完全限定名(binary name，二进制名)和用于加载该类的定义类加载器(defining name)
 * 所共同决定的。如果同样名字(即相同的完全限定名)的类是由两个不同的加载器所加载，那么这些类是不同的，即便.class文件
 * 的字节码完全一样，并从相同的位置加载亦如此。
 */

/**
 * 在Oracle的Hotspot实现中，系统属性sun.boot.class.path如果修改错了，则运行会出错，提示如下错误：
 *
 * Error occurred during initialization of VM
 * java/lang/NoClassDefFoundError: java/lang/Object
 */

import sun.misc.Launcher;

/**
 * 内建于JVM中的启动类加载器会加载java.lang.ClassLoader以及其他的Java平台类，
 * 当JVM启动时，一块特殊的机器码会运行，它会加载扩展类加载器及系统类加载器，这块特殊
 * 的机器码叫做启动类加载器(Bootstrap ClassLoader)
 *
 * 启动类加载器并不是Java类，而其他的加载器则都是Java类
 * 启动类加载器是特定于平台的机器指令，它负责开启整个加载过程
 *
 * 所有类加载器（除了启动类加载器）都是实现为Java类，不过，总归要有一个组件来加载第一个Java类加载器，
 * 从而让整个加载过程都能够顺利进行下去，加载第一个纯Java类加载器就是启动类加载的职责
 *
 * 启动类加载器还会负责加载JRE正常运行所需的基本组件，这包括java.util于java.lang包中的类等等
 */
public class MyTest24 {
    public static void main(String[] args) {
        // 根类加载器加载.class文件的路径
        System.out.println(System.getProperty("sun.boot.class.path"));
        // 扩展类加载器加载.class文件的路径
        System.out.println(System.getProperty("java.ext.dirs"));
        // 系统（应用）类加载器加载类的路径
        System.out.println(System.getProperty("java.class.path"));

        System.out.println(ClassLoader.class.getClassLoader());

        // 扩展类加载器于系统类加载器也是由启动类加载器进行加载的
        System.out.println(Launcher.class.getClassLoader());

        System.out.println(System.getProperty("java.system.class.loader"));

        System.out.println(MyTest24.class.getClassLoader());

        System.out.println(MyTest17.class.getClassLoader());

        System.out.println(ClassLoader.getSystemClassLoader());
    }
}
````

Launcher、ClassLoader、Class、Thread、ServiceLoader类的源码理解

源码中重要代码

````java
package sun.misc;

public Launcher() {
        Launcher.ExtClassLoader var1;
        try {
            var1 = Launcher.ExtClassLoader.getExtClassLoader();
        } catch (IOException var10) {
            throw new InternalError("Could not create extension class loader", var10);
        }

        try {
            this.loader = Launcher.AppClassLoader.getAppClassLoader(var1);
        } catch (IOException var9) {
            throw new InternalError("Could not create application class loader", var9);
        }

   	  // 重要代码
        Thread.currentThread().setContextClassLoader(this.loader);
        String var2 = System.getProperty("java.security.manager");
        if (var2 != null) {
            SecurityManager var3 = null;
            if (!"".equals(var2) && !"default".equals(var2)) {
                try {
                    var3 = (SecurityManager)this.loader.loadClass(var2).newInstance();
                } catch (IllegalAccessException var5) {
                } catch (InstantiationException var6) {
                } catch (ClassNotFoundException var7) {
                } catch (ClassCastException var8) {
                }
            } else {
                var3 = new SecurityManager();
            }

            if (var3 == null) {
                throw new InternalError("Could not create SecurityManager: " + var2);
            }

            System.setSecurityManager(var3);
        }
}

@CallerSensitive
    public static ClassLoader getSystemClassLoader() {
       // 重要代码
        initSystemClassLoader();
        if (scl == null) {
            return null;
        }
        SecurityManager sm = System.getSecurityManager();
        if (sm != null) {
            checkClassLoaderPermission(scl, Reflection.getCallerClass());
        }
        return scl;
    }

    private static synchronized void initSystemClassLoader() {
        if (!sclSet) {
            if (scl != null)
                throw new IllegalStateException("recursive invocation");
           // 重要代码
            sun.misc.Launcher l = sun.misc.Launcher.getLauncher();
            if (l != null) {
                Throwable oops = null;
                scl = l.getClassLoader();
                try {
                   // 重要代码
                    scl = AccessController.doPrivileged(
                        new SystemClassLoaderAction(scl));
                } catch (PrivilegedActionException pae) {
                    oops = pae.getCause();
                    if (oops instanceof InvocationTargetException) {
                        oops = oops.getCause();
                    }
                }
                if (oops != null) {
                    if (oops instanceof Error) {
                        throw (Error) oops;
                    } else {
                        // wrap the exception
                        throw new Error(oops);
                    }
                }
            }
            sclSet = true;
        }
    }

class SystemClassLoaderAction
    implements PrivilegedExceptionAction<ClassLoader> {
    private ClassLoader parent;

    SystemClassLoaderAction(ClassLoader parent) {
        this.parent = parent;
    }

    public ClassLoader run() throws Exception {
        String cls = System.getProperty("java.system.class.loader");
        if (cls == null) {
            return parent;
        }

        Constructor<?> ctor = Class.forName(cls, true, parent)
            .getDeclaredConstructor(new Class<?>[] { ClassLoader.class });
        ClassLoader sys = (ClassLoader) ctor.newInstance(
            new Object[] { parent });
       // 重要代码
        Thread.currentThread().setContextClassLoader(sys);
        return sys;
    }
}
````

````java
/**
 * 当前类加载器（Current classloader）
 * 每个类都会使用自己的类加载器（即加载自身的类加载器）来去加载其他类（指的是所依赖的类）
 * 如果ClassX引用了ClassY，那么ClassX的类加载器就会去加载ClassY（前提是ClassY未被加载）
 *
 * 线程上下文类加载器(Context ClassLoader)
 * 线程上下文加载器是从JDK1.2开始引入的，类Thread中的getContextClassLoader()与
 * setContextClassLoader(ClassLoader c1)分别用来获取和设置上下文类加载器。
 *
 * 如果没有通过setContextClassLoader(ClassLoader c1)进行设置的话，线程将继承其父线程
 * 的上下文类加载器。Java应用运行时的初始线程的上下文类加载器是系统类加载器。在线程中运行的
 * 代码可以通过该类加载器来加载类与资源。
 *
 * 线程上下文类加载器的重要性：
 *
 * SPI(Service Provider Interface)
 *
 * 父ClassLoader可以使用当前线程Thread.currentThread().getContextLoader()所指定
 * 的classloader加载的类，这就改变了父ClassLoader不能使用子ClassLoader或者是其他没有
 * 直接父子关系的classloader加载的类的情况。即改变了双亲委托模型。
 *
 * 线程上下文类加载器就是当前线程的Current ClassLoader
 *
 * 在双亲委托模型下，类加载是由下至上的，即下层的类加载器会委托上层进行加载，但是对于SPI来说，
 * 有些接口是Java核心库所提供的，而Java核心库是由启动类加载器来加载的，而这些接口的实现却来自
 * 不同的jar包（厂商提供），Java的启动类加载器是不会加载其他来源的jar包，这样传统的双亲委托模型
 * 就无法满足SPI的要求，而通过给当前线程设置上下文类加载器，就可以由设置的上下文类加载器来实现对于
 * 接口实现类的加载。
 */

public class MyTest5 {
    public static void main(String[] args) {
        System.out.println(Thread.currentThread().getContextClassLoader());
        // 在java.lang包中，由启动类加载器加载
        System.out.println(Thread.class.getClassLoader());
    }
}
````

````java
public class MyTest25 implements Runnable{

    private Thread thread;

    public MyTest25(){
        thread = new Thread(this);
        thread.start();
    }

    public void run() {
        ClassLoader classLoader = this.thread.getContextClassLoader();

        this.thread.setContextClassLoader(classLoader.getParent());

        System.out.println("Class :"+classLoader.getClass());

        System.out.println("Parent class :"+classLoader.getParent().getClass());
    }

    public static void main(String[] args) {
        new MyTest25();
    }
}
````

````java
import java.sql.Driver;
import java.util.Iterator;
import java.util.ServiceLoader;

/**
 * 线程上下文类加载器的一般使用模式(获取-使用-还原)
 *
 *  ClassLoader classloader = Thread.currentThread().getContextClassLoader()；
 *  try{
 *      Thread.currentThread().setContextClassLoader(targetTccl);
 *      myMethod();
 *  } finally {
 *      Thread.currentThread().setContextClassLoader(classloader);
 *  }
 *
 * myMethod()里面则调用了Thread.currentThread().getContextClassLoader()，获取当前线程
 * 的上下文类加载器做某些事情。
 *
 * 如果一个类由类加载器A加载，那么这个类的依赖类也是由相同的类加载器加载的（如果该类加载器之前没有被加载过）
 * ContextClassLoader的作用就是为了破坏Java的类加载委托机制。
 *
 * 当高层提供了统一的接口让低层去实现，同时又要在高层加载（或实例化）低层时，那么必须要通过线程上下文类加载器
 * 来帮助高层的ClassLoader找到并加载该类。
 */

public class MyTest26 {
    public static void main(String[] args) {
       //Thread.currentThread().setContextClassLoader(MyTest26.class.getClassLoader().getParent());
       
        ServiceLoader<Driver> loader = ServiceLoader.load(Driver.class);
        Iterator<Driver> iterator = loader.iterator();

        while (iterator.hasNext()){
            Driver driver = iterator.next();
            System.out.println("driver:"+driver.getClass()+"loader:"+driver.getClass().getClassLoader());
        }

        System.out.println("当前上下文类加载器："+Thread.currentThread().getContextClassLoader());

        System.out.println("ServiceLoader的类加载器:"+ServiceLoader.class.getClassLoader());
    }
}
````

````java
实现这几句代码的源代码非常重要

public class MyTest27 {
    public static void main(String[] args) throws Exception {
        Class.forName("com.mysql.cj.jdbc.Driver");
        Connection conn = DriverManager.getConnection("jdbc:mysql://localhost:3306/test",
                "root","Wy7950473!");
    }
}
````



------



#### 2.Java虚拟机与程序的生命周期

------

1.如下几种情况下，Java虚拟机将结束生命周期

- 执行了System.exit()方法
- 程序正常结束
- 程序在执行过程中遇到了异常或错误而异常终止
- 由于操作系统出现错误而导致Java虚拟机进程终止

------





## 二.字节码

1.javap -verbose .class文件：查看一个类的反编译的详细信息

2.使用javap -verbose命令分析一个字节码文件时，将会分析该字节文件的魔数、版本号、常量池、类信息、类的构造方法、类中的方法信息、类变量与成员变量等信息。

3.魔数：所有的.class字节码文件的前4个字节都是魔数，魔数值为固定值：0xCAFEBABE.

4.魔数之后4个字节是版本信息，前两个字节表示minor_version(次版本号)，后两个字节表示major_version(主版本号)。这里的版本号为00 00 00 34，换算成十进制，表示次版本号为0，主版本号为52。所以，该文件的版本号为：1.8.0。可以通过java -version命令来验证这一点.

5.常量池(constant_pool):紧接着主版本号之后的就是常量池入口。一个Java类中定义的很多信息都是由常量池来维护和描述的。可以将常量池看作是class文件的资源仓库，比如说Java类中定义的方法与变量信息，都是存储在常量池中。常量池中主要存储两类常量：字面量与符号引用。字面量如文本字符串，Java中声明为final的常量值等，而符号引用如类和接口的全局限定名，字段的名称和描述符，方法的名称和描述符。

6.常量池的总体结构：Java类的所对应的常量池主要是由常量池数量与常量池数组这两部分共同构成。常量池数量紧跟在主版本号后面，占据2个字节；而常量池数组则紧跟在常量池数量之后。常量池数组与一般的数组不同的是，常量池数组中不同的元素的类型、结构都是不同的。长度当然也就不同；但是，每一种元素的第一个数据都是u1类型，该字节是个标志位，占据1个字节。JVM在解析常量池时，会根据这个u1类型来获取元素的具体类型。==值得注意的是，常量池数组中元素的个数 = 常量池数 - 1（其中0暂时不使用）。==目的是满足某些常量池索引值的数据在特定情况下需要表达【不引用任何一个常量池】的含义；根本原因在于，索引为0也是一个常量（保留常量），只不过它不位于常量表中，这个常量就对应null值；所以常量池的索引从1而非0开始。 

![屏幕快照 2020-01-21 下午10.44.45](%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202020-01-21%20%E4%B8%8B%E5%8D%8810.44.45.png)

7.在JVM规范中，每个字段/变量都有描述信息，描述信息主要的作用是描述字段的数据类型、方法的参数列表(包括数量、类型和顺序)与返回值。根据描述符规则，基本数据类型和代表无返回值的void类型都用一个大写字符表示，对象类型规则使用字符L加对象的权限定名称来表示。为了压缩字节码文件的体积。对于基本数据类型，JVM都只使用一个大写字母表示，如下所示：B - byte,C - char,D - double,F - float,I - int,J - long,S - short,Z - boolean,V - void,L - 对象类型，如 Ljava/lang/String;

8.对于数组来说，每一个维度使用一个前置的[来表示，如int[]被记录为[I,String[][][][]被记录为[[Ljava/lang/String;

9.用描述符描述方法时，按照先参数列表，后返回值的顺序来描述。参数列表按照参数的严格顺序放在一组()之内，如方法：String getRealnameByIdAndNickname(int id,String name);的描述符为；（I,Ljava/lang/String;）Ljava/lang/String;

![屏幕快照 2020-01-22 下午11.10.57](%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202020-01-22%20%E4%B8%8B%E5%8D%8811.10.57.png)

