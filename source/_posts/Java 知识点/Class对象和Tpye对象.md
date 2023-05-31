---
title: Class 对象和 Tpye 对象知识笔记
---

## 类对象
1. 常见的类对象如下：
   - class：外部类、成员类（成员内部类、静态内部类）、局部内部类、匿名类
   - interface：接口
   - []：数组
   - enum：枚举类
   - annotation：注解类@interface
   - primitive：基础（原始）数据类型
   - void 空类
  
2. 测试例子说明
```java
@Test
public void test() {
    
    // 外部类类对象
    System.out.println(Object.class);
    
    // 接口类对象
    System.out.println(List.class);

    // 数组类对象
    System.out.println(String[].class);
    System.out.println(int[][].class);
    
    // 注解类对象
    System.out.println(Override.class);
    // void 类对象
    System.out.println(void.class);
    
    // 类对象，比较
    int[] a = new int[10];
    int[] b = new int[20];
    System.out.println(a.getClass() == b.getClass()); // true
}
```

## Type 对象
Type 是 Java 中所有类型共同的超级接口，其中包含 raw types（原始类型）, parameterized types（参数化类型）, array types（数组类型）, type variables（类型变量类型） and primitive types（原生基础类型）。

Type 接口的类 UML 图如下，它有 4 个子接口 GenericArrayType，ParameterizedType，TypeVariable，WildcardType 和一个实现类 Class。
![Type 类 UML 图](/images/Type-UML.png)

- Class 类，表示 raw types（原始类）和基础类型，指的是普通类、枚举、接口、注解、数组、基本类型（int，double），等等不是泛型的类型；
- GenericArrayType 表示的泛型数组类型，如 T[] 等等；
- ParameterizedType 表示的是泛型类型，指的是 List<T>，Map<K,V> 等泛型类型;
- TypeVariable 表示的是泛型类型变量，如 List<T> 中的泛型变量 T；
- WildcardType 通配符泛型也叫做泛型表达式类型，如 List<? extends Number>。

测试说明例子，如下：
```java

/**
     * 测试例子输出结果如下：
     * ParameterizedType = java.util.Map<java.lang.String, java.lang.Object> actualTypeArguments = java.lang.String,java.lang.Object
     * ParameterizedType = java.util.List<java.lang.String> actualTypeArguments = java.lang.String
     * ParameterizedType = java.util.List<? extends java.lang.Number> actualTypeArguments = ? extends java.lang.Number
     * ParameterizedType = java.lang.Class<?> actualTypeArguments = ?
     * GenericArrayType = T[] arrayType = T
     * K
     * V
     */
    @Test
    public void test02() {
        TypeTest<String> typeTest = new TypeTest<String>();
        // 通过反射的方式，获取所有生命定义的字段
        Field[] declaredFields = typeTest.getClass().getDeclaredFields();
        for (Field declaredField : declaredFields) {
            // 反射获取字段的 Type 类型对象
            Type type = declaredField.getGenericType();
            // 判断类型
            if (type instanceof ParameterizedType) {
                ParameterizedType pt = (ParameterizedType) type;
                // 泛型变量的实际类型
                String actualTypeArguments = Arrays.stream(pt.getActualTypeArguments()).map(item -> item.getTypeName()).collect(Collectors.joining(","));
                System.out.println("ParameterizedType = " + pt.getTypeName() + " actualTypeArguments = " + actualTypeArguments);
            }
            if (type instanceof WildcardType) {
                WildcardType wt = (WildcardType) type;
                System.out.println("WildcardType = " + wt.getTypeName());
            }
            if (type instanceof GenericArrayType) {
                GenericArrayType gat = (GenericArrayType) type;
                System.out.println("GenericArrayType = " + gat.getTypeName() + " arrayType = " + gat.getGenericComponentType().getTypeName());
            }
        }

        // 通过 map 实例的方式只能获取泛型定义的变量，取不到实际的泛型变量类型，只有通过类成员变量然后通过反射 Field 获取 type 的方式的才能取到，如上。
        Map<String, Object> map = new HashMap<String, Object>();
        TypeVariable<? extends Class<? extends Map>>[] typeParameters = map.getClass().getTypeParameters();
        for (TypeVariable<? extends Class<? extends Map>> typeParameter : typeParameters) {
            // 输出 K, V
            System.out.println(typeParameter.getTypeName());
        }
    }

```

## 获取泛型类型的实际类型
泛型的本质是参数化类型，想要获取泛型参数化类型的实际类型，只有通过反射成员变量、继承、构造函数等方式获取 Type 对象强转 ParameterizedType 类型对象，或者直接通过放射获取 ParameterizedType 的方式，才能获取到泛型类型变量的实际类型。因为只有 ParameterizedType 对象的 getActualTypeArguments 方法能能够方法泛型的实际变量类型。

1. 通过反射类成员变量的方式获取：
```java
import java.lang.reflect.Field;
import java.lang.reflect.ParameterizedType;
import java.lang.reflect.Type;
import java.lang.reflect.TypeVariable;

public class GenericTest {

    private Map<String, Object> memberMap = new HashMap<String, Object>();

    public static void main(String[] args) {

		Map<String, Object> map = new HashMap<String, Object>();
		// 直接通过 Map 对象的方式，只能获取 map 对象的声明的泛型变量，获取不是实际的泛型变量类型 
		TypeVariable<? extends Class<? extends Map>>[] typeParameters = map.getClass().getTypeParameters();
		for (TypeVariable<? extends Class<? extends Map>> typeParameter : typeParameters) {
			// 会打印 Map<K, V> 中的 K 和 V
			System.out.println(typeParameter.getTypeName());
		}
		// 间接通过类成员变量的方式，可以获取 Map对象实际的泛型变量类型
		Field field = ClassAndTypeTest.class.getDeclaredField("memberMap");
		Type type = field.getGenericType();
		ParameterizedType pt = (ParameterizedType)type;
		for(Type t : pt.getActualTypeArguments()) {
			// 会打印出泛型变量的实际类型，java.lang.String 和 java.lang.Object
			System.out.println(t.getTypeName());
		}
	}
}

2. 通过成员方法的参数和返回类型的方式获取泛型的实际类型
```java

public class HelloWorld {

    @Test
    public void test(){
        try {

            Method helloMethod = TypeTest.class.getDeclaredMethod("hello", List.class);
            System.out.println(helloMethod.getName());
            // 获取 Type 对象，然后强转 ParameterizedType 对象，因为只有 ParameterizedType 类型对象，才能获取泛型变量的实际类型。 
            Type genericReturnType = helloMethod.getGenericReturnType();
            if (genericReturnType instanceof ParameterizedType) {
                ParameterizedType pt =  (ParameterizedType)genericReturnType;
                System.out.println("返回的泛型类型 = " + Arrays.stream(pt.getActualTypeArguments()).map(Type::getTypeName).collect(Collectors.joining()));
            }

            Type[] genericParameterTypes = helloMethod.getGenericParameterTypes();
            for (Type genericParameterType : genericParameterTypes) {
                if (genericParameterType instanceof ParameterizedType) {
                    ParameterizedType pt =  (ParameterizedType)genericParameterType;
                    System.out.println("参数的泛型类型 = " + Arrays.stream(pt.getActualTypeArguments()).map(Type::getTypeName).collect(Collectors.joining()));
                }
            }
        } catch (NoSuchMethodException e) {
            e.printStackTrace();
        }
    }

    public List<Integer> hello(List<String> paramList){
        return new ArrayList<Integer>();
    }
}

```

3. 通过继承方式获取泛型变量的实际类型
```java
public class Hello<T>{
    public static void main(String[] args) {
        Hello<String> hello = new Hello<String>();
        // 直接通过 class 对象的只能超类或者接口的 Type，所以想要获取 Hello 类的泛型变量实际类型，必须通过其子类方能获取，相当包了一层，如下 World 类
        Type genericSuperclass1 = hello.getClass().getGenericSuperclass();
        Type[] genericInterfaces = hello.getClass().getGenericInterfaces();

        World world = new World();
        // 获取 type 类型对象，然后强转 ParameterizedType
        Type genericSuperclass = world.getClass().getGenericSuperclass();
        if (genericSuperclass instanceof ParameterizedType) {
            ParameterizedType pt = (ParameterizedType)genericSuperclass;
            System.out.println("获取父类的泛型类型 = " + Arrays.stream(pt.getActualTypeArguments()).map(Type::getTypeName).collect(Collectors.joining()));
        }
    }
}

publi class World extend Hello<String> {

}

```

4. 通过构造方法的方式获取泛型变量的实际类型
```java
public class Hello {

    public Hello(List<String> params){

    }


    public static void main(String[] args) {
        // 方式1
        Constructor<?>[] constructors = Param.class.getConstructors();
        for (Constructor<?> constructor : constructors) {
            // 构造函数能获取参数的 Type 类型，所以通过构造函数的方式获取泛型变量的实际类型
            for (Type genericParameterType : constructor.getGenericParameterTypes()) {
                ParameterizedType pt = (ParameterizedType)genericParameterType;
                System.out.println(Arrays.stream(pt.getActualTypeArguments()).map(Type::getTypeName).collect(Collectors.joining()));
            }
        }
        // 方式2
        Constructor<Param> constructor = Param.class.getConstructor(List.class);
        Type[] genericParameterTypes = constructor.getGenericParameterTypes();
        ParameterizedType pt = (ParameterizedType)genericParameterTypes[0];
        System.out.println(Arrays.stream(pt.getActualTypeArguments()).map(Type::getTypeName).collect(Collectors.joining()));
    }

}

```



## 常见的 TypeRefece 原理


