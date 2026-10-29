## 1  最简单的处理器

### 1.1  设置注解

注意：`@Retention(RetentionPolicy.SOURCE)`，这里的注解的生命周期是 SOURCE。因为处理的是源代码，而并非是编译后的 class 文件。当然，因此也不能用反射。

```java
package org.Annotation.Simplest;

import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

@Target({
        ElementType.TYPE,
        ElementType.METHOD,
        ElementType.CONSTRUCTOR,
        ElementType.ANNOTATION_TYPE,
        ElementType.PACKAGE,
        ElementType.FIELD,
        ElementType.LOCAL_VARIABLE
})
@Retention(RetentionPolicy.SOURCE)
public @interface Simple {
    String value() default "-default-";
}
```

### 1.2  编写注解处理器

步骤

```java
@SupportedAnnotationTypes("org.Annotation.Simplest.Simple")
@SupportedSourceVersion(SourceVersion.RELEASE_21)
public class SimpleProcessor extends AbstractProcessor {
```

1. 注册注解
2. 设置处理器支持最高版本
3. 继承抽象处理器
4. 重写 process 方法

```java
package org.Annotation.Simplest;

import javax.annotation.processing.AbstractProcessor;
import javax.annotation.processing.RoundEnvironment;
import javax.annotation.processing.SupportedAnnotationTypes;
import javax.annotation.processing.SupportedSourceVersion;
import javax.lang.model.SourceVersion;
import javax.lang.model.element.Element;
import javax.lang.model.element.ElementKind;
import javax.lang.model.element.ExecutableElement;
import javax.lang.model.element.TypeElement;
import java.util.Set;

/**
 * @BelongsProject: untitled
 * @BelongsPackage: org.Annotation.Simplest
 * @Author: Lee
 * @CreateTime: 2026-04-22 13:47
 * @Description: Simple 注解处理器
 * @Version: 1.0
 */
// 参数是注解类
// 告诉 javac 该处理器只对某个注解感兴趣，只有遇到该注解时，process 方法才会触发
@SupportedAnnotationTypes("org.Annotation.Simplest.Simple") // 必选
@SupportedSourceVersion(SourceVersion.RELEASE_21)           // 该处理器支持的JAVA最高版本
public class SimpleProcessor extends AbstractProcessor {

    @Override
    public boolean process(Set<? extends TypeElement> annotations, RoundEnvironment env) {
        for(TypeElement anno : annotations){ // 遍历该处理器负责的注解类型并打印
            System.out.println(anno);
        }
        for (Element el : env.getElementsAnnotatedWith(Simple.class)){ // 处理被 @Simple 标记的所有元素
            display(el);
        }
        return  false; // 表示该处理器不独占这些注解
        // 如果 return true; 则表示本处理器已经处理完毕，后续处理器不会看到这些注解
    }

    /**
     *  解析 Java 元素
     * @param el 元素，包括类、方法、字段等
     */
    private void display(Element el){
        // 打印 el 信息
        System.out.println("==== " + el + " ====");
        System.out.println(
                el.getKind() +                              // 元素类型
                        " : " + el.getModifiers() +         // 权限
                        " : " + el.getSimpleName() +        // 名称
                        " : " + el.asType()                 // 元素类型镜像：类->声明类型；方法->返回值类型
        );

        // 针对类的深入打印
        if(el.getKind().equals(ElementKind.CLASS)){
            TypeElement te = (TypeElement) el;
            System.out.println(te.getQualifiedName());      // 全限定名
            System.out.println(te.getSuperclass());         // 父类
            System.out.println(te.getEnclosedElements());   // 该类的成员
        }

        // 针对函数的深入打印
        if(el.getKind().equals(ElementKind.METHOD)){
            ExecutableElement exe = (ExecutableElement) el;
            System.out.println(exe.getReturnType());        // 返回值类型
            System.out.println(exe.getSimpleName());        // 方法名
            System.out.println(exe.getEnclosedElements());  // 方法内部元素
        }
    }
}
```

### 1.3  对要处理的类进行标注

```java
package org.Annotation.Simplest;

/**
 * @BelongsProject: untitled
 * @BelongsPackage: org.Annotation.Simplest
 * @Author: Lee
 * @CreateTime: 2026-04-22 12:02
 * @Description: TODO
 * @Version: 1.0
 */
@Simple
public class SimpleTest {
    @Simple
    int i;

    @Simple
    public SimpleTest() {}

    @Simple
    public void foo() {
        System.out.println("SimpleTest.foo()");
    }

    @Simple
    public void bar(String s, int i, float f) {
        System.out.println("SimpleTest.bar()");
    }

    @Simple
    public static void main(String[] args) {
        @Simple
        SimpleTest st = new SimpleTest();
        st.foo();
    }
}
```

### 1.4  Javac 命令

在 src 文件夹下运行该命令

```bash
javac -processor org.Annotation.Simplest.SimpleProcessor org/Annotation/Simplest/SimpleTest.java
```

## 2  更复杂的处理器

当你创建用于 javac 注解处理器时，你不能使用 Java 的反射特性，因为你处理的是源代码，而并非是编译后的 class 文件。各种 mirror 解决这个问题的方法是，通过允许你在未编译的源代码中查看方法、字段和类型。

### 2.1  设置注解

```java
package org.Annotation.ifx;

import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

@Target(ElementType.TYPE)
@Retention(RetentionPolicy.SOURCE)
public @interface ExtracInterface {
    String interfaceName() default "-!!-"; // 接口名称
}
```

### 2.2  编写注解处理器

```java
package org.Annotation.ifx;

import javax.annotation.processing.*;
import javax.lang.model.SourceVersion;
import javax.lang.model.element.*;
import javax.lang.model.util.Elements;
import java.io.IOException;
import java.io.Writer;
import java.util.ArrayList;
import java.util.List;
import java.util.Set;
import java.util.stream.Collectors;

/**
 * @BelongsProject: untitled
 * @BelongsPackage: org.Annotation.ifx
 * @Author: Lee
 * @CreateTime: 2026-04-22 18:21
 * @Description: TODO
 * @Version: 1.0
 */
@SupportedAnnotationTypes("org.Annotation.ifx.ExtracInterface")
@SupportedSourceVersion(SourceVersion.RELEASE_21)
public class IfaceExtractorProcessor extends AbstractProcessor {

    private final ArrayList<Element> interfaceMethods = new ArrayList<>(); // 存放 接口内公共的、非静态的方法
    Elements elementUtils;
    private ProcessingEnvironment processingEnv;

    @Override
    public void init(ProcessingEnvironment processingEnv) {
        this.processingEnv = processingEnv;
        this.elementUtils = processingEnv.getElementUtils();
    }

    @Override
    public boolean process(Set<? extends TypeElement> annos, RoundEnvironment env) {
        for(Element el : env.getElementsAnnotatedWith(ExtracInterface.class)){
            // 获取接口名
            String interfaceName = el.getAnnotation(ExtracInterface.class).interfaceName();

            // 获取接口方法列表
            for (Element enclosed : el.getEnclosedElements()){
                // enclosed 是 公共的、非静态的方法
                if(enclosed.getKind().equals(ElementKind.METHOD) &&
                        enclosed.getModifiers().contains(Modifier.PUBLIC) &&
                        !enclosed.getModifiers().contains(Modifier.STATIC)
                ){
                    interfaceMethods.add(enclosed);
                }
            }

            // 写入接口
            if(!interfaceMethods.isEmpty()){
                try {
                    writeInterfaceFile(interfaceName);
                } catch (IOException e) {
                    throw new RuntimeException(e);
                }
            }
        }
        return false;
    }

    /**
     * 编写接口代码
     * @param interfaceName
     * @throws IOException
     */
    private void writeInterfaceFile(String interfaceName) throws IOException {

        // 获取包名
        String pacageName = elementUtils.getPackageOf(interfaceMethods.getFirst()).toString();
        String pacageFullName  = "package " + pacageName  + ";\n";
        System.out.println(pacageFullName);

        String interfaceFullName = pacageName + "." + interfaceName;

        Writer writer = processingEnv.getFiler().createSourceFile(interfaceFullName).openWriter();

        // 写入 package 包全限定名
        writer.write(pacageFullName);
        System.out.println(pacageName);

        // 写入 public interface {
        writer.write("public interface " + interfaceName + "{\n");

        // 写入方法声明
        for(Element el : interfaceMethods){
            ExecutableElement method = (ExecutableElement) el;

            // public returntype name(args);
            String signature = " public " +
                    method.getReturnType() + " " +
                    method.getSimpleName() +
                    createArgList(method.getParameters()) + ";\n";
            System.out.println(signature);
            writer.write(signature);
        }
        writer.write("}");
        writer.close();
    }

    /**
     *  根据 parameters 获取参数列表的字符串
     * @param parameters 参数
     * @return 返回参数列表 (int a1,String a2,double a3, …)
     */
    private String createArgList(List<? extends VariableElement> parameters){
        String args = parameters.stream().map(
                p->p.asType() + " " + p.getSimpleName()) // type name
                .collect(Collectors.joining(", "));
        return "(" + args + ")";
    }
}
```

### 2.3  对要处理的类进行标注

```java
package org.Annotation.ifx;

/**
 * @BelongsProject: untitled
 * @BelongsPackage: org.Annotation.ifx
 * @Author: Lee
 * @CreateTime: 2026-04-22 18:16
 * @Description: TODO
 * @Version: 1.0
 */
// 要生成的接口名称
@ExtracInterface(interfaceName = "IMultiplier")
public class Multiplier {
    public boolean flag = false;
    private int n = 0;
    public int multiply(int x, int y){
        int total = 0;
        for(int i = 0; i < x; i++){
            total = add(total, y);
        }
        return total;
    }
    public int fortySeven(){
        return 47;
    }
    private  int add(int x, int y){
        return x+y;
    }
    public double timesTen(double arg){
        return arg*10;
    }

    public static void main(String[] args) {
        Multiplier m = new Multiplier();
        System.out.println("11 * 16 = " + m.multiply(11, 16));
    }
}
```
