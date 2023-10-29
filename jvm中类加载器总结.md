### 1.类加载器 是 **java 虚拟机** 提供给**应用程序**去获取类和接口字节码数据的的技术

类加载的流程：



![image-20231011142649372](./assets/image-20231011142649372.png)

获取字节码文件 ：由 类加载器完成，类加载器只负责加载二进制 字节码文件

jvm虚拟机由c++编写 ，负责 生成 方法区对象 ，和生成堆上Class对象

### 2.类加载器的应用场景

![image-20231011143115146](./assets/image-20231011143115146.png)

#### 3.类加载器的分类

1. java虚拟机底层源码实现 ：
   - 启动类加载器 bootstrap
2. Java代码中实现
   - 扩展类加载器Extension ，应用程序类加载器

![image-20231011143353696](./assets/image-20231011143353696.png)

### 4.使用arthas观察当前jvm信息

- arthas 相关命令：`https://arthas.aliyun.com/doc/commands.html`
- `classloader`
  - 打印以下信息
  - <img src="./assets/image-20231011153454695.png" alt="image-20231011153454695" style="zoom: 80%;" />
  - numberOfInstance ：类加载器的个数  loaderCountTotal：加载的核心类
  - **启动类加载器个数为1** 
  - $:代表静态内部类
  - arthas类加载器个数为1
  - ExClassLoader：扩展类加载器
  - DelegatingClassLoader :提高反射效率类加载器
  - AppClassLoader：应用程序类加载器
  - PlatformClassLoade：java9之后才有：载Java SE平台的系统模块
    - 扩展：类加载器之间存在一个层次结构，其中`PlatformClassLoader`是`Bootstrap ClassLoader`的子加载器，而`System ClassLoader`是`PlatformClassLoader`的子加载器。

### 5：启动类加载器：由hotspot虚拟机提供，使用c++编写

1. 功能：默认加载/jre/lib

2. 注意点：java9之后jdk包含jre

3. <img src="./assets/image-20231011155719416.png" alt="image-20231011155719416" style="zoom:50%;" />

4. <img src="./assets/image-20231011160713269.png" alt="image-20231011160713269" style="zoom:50%;" />

5. 校验启动类加载器：

   1. 创建一个maven项目，编写一个java脚本，并且打jar包，可以把jar包放在自定位置

      ![image-20231011162734299](./assets/image-20231011162734299.png)

   2. 在idea测试启动类中勾选jvm配置： `-add jvm options`

   3. 输入需要加载的启动类jar包路径：`-Xbootclasspath/a:`+jar包绝对路径

      ![image-20231011163137999](./assets/image-20231011163137999.png)

   4. 执行代码，启动类的类加载器为null，则**启动类加载器（bootstrap）**，

   5. **总结：由此得出，启动类加载器bootstrap加载类到jvm虚拟机，为了安全考虑，程序员是不能去访问启动类加载器和jvm的**

      - <img src="./assets/image-20231011164054965.png" alt="image-20231011164054965" style="zoom:50%;" />
      - <img src="./assets/image-20231011164129677.png" alt="image-20231011164129677" style="zoom: 50%;" />

      

### 6.扩展类加载器

1. ![image-20231011173048346](./assets/image-20231011173048346.png)

2. 在idea测试启动类中勾选jvm配置： `-add jvm options` 

   <img src="./assets/image-20231011173245264.png" alt="image-20231011173245264" style="zoom:50%;" />

3. 配置`-Djava.ext.dirs=`+ext目录;自定义jar包目录（不是某个jar包的目录，而是要导入jar包的父目录）

4. 测试编写代码

   ​	![image-20231011174026395](./assets/image-20231011174026395.png)

5. 成功加载之前的自定义类，所以**扩展类加载 /jre/lib/ext/ 和我们自定义jar包的 字节码文件**



### 7.应用程序类加载器

1. 导入maven commos坐标

   ![image-20231011174351701](./assets/image-20231011174351701.png)

2. 编写自定义student类

3. 执行代码

   ​	![image-20231011174548609](./assets/image-20231011174548609.png)

4. 由此判断，应用程序类加载器：**加载自定义类，和第三方库的类**



### 8.通过启动Arthas来校验结果是否正确

![image-20231011211005903](./assets/image-20231011211005903.png)

启动Arthas jar包后 ，可以看到

1. 扩展类加载器确实是加载的 /jre/lib/ext/ 下面的所有文件
2. 应用程序类加载器 
   - /lib/ext 扩展jar
   - /lib/**    运行环境所依赖的jar
   - /target/classes  当前编译路径下的 **字节码文件**
   - /.m2/  本地环境所依赖的jar包 

遗留问题？ 应用程序不应该只加载 第三方jia包和 当前编译路径下的 **字节码文件** 吗？

这就涉及 类的 双亲委派机制

