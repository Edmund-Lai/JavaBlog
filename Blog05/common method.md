# 常用方法与操作

> 写在前面：
>
> * 考虑到HIT软件构造的考试为手写Java代码，有许多不便之处，特别是本人比较依赖于IDE，经常忘记方法名，所以给出一篇blog，用于记录常用的方法以及一些不太熟悉的操作，基本上作为一篇自用的note，格式不太严谨还请原谅
> * ~~作为软件构造blog的最后一篇收尾~~

## 集合

### Collection

Collection接口的实现、继承关系如下：

<img src="D:\Typora\figure\figure1.png" alt="figure1" style="zoom:50%;" />

Collection接口中有声明如下常用的方法：

* add、addAll
* size
* clear
* isEmpty
* contains、containsAll
* remove、removeAll（不建议使用，最好使用iterator来进行remove）
* retainAll(Colloction c)：取交集放在当前集合中，也就是说会修改当前集合，但是不影响c
* equals
* toArray：返回数组形式
* iterator
  * next
  
  * hasNext
  
  * remove：特别是在List中，最好使用这个remove代替List接口实现类直接调用remove方法，这种问题主要发生在下面的情况中
  
    ```java
    List<Integer> list=new ArrayList<Integer>();
    list.add(1);
    list.add(2);
    list.add(3);
    list.add(3);
    list.add(4);
    
    for(int i=0;i<list.size();i++){
       if(list.get(i)==3) list.remove(i);
    }
    System.out.println(list);//结果为1、2、3、4
    ```
  
    原因：在第一次remove的时候，第二个3和后面的元素的索引被自动调整，随后循环变量i自增，就跳过了第二个3。为了避免这种问题，**遍历最好直接使用迭代器遍历，移除元素也用迭代器配套的remove方法**
  
  * **特别注意**：增强型for循环的底层实际使用的就是迭代器，所以一般的数组无法使用增强for型循环

### List 及 Set

**List接口中相对于Collection加的方法**：

* get(int index)
* indexOf：通过调用equals方法找到对应元素第一次出现的索引
* lastIndexOf：同indexOf，找的是最后一次出现的索引
* remove(int index)
  * 这个remove会带来很奇怪的问题，比如List的泛型是Integer，调用remove(2)删除的是索引为2的元素，而不是值为2的元素，后者需要使用remove(new Integer(2))来解决
* set(int index, Object element)：将索引为index的元素修改成element
* subList(int from, int to)：左开右闭规则

**Set接口中没有定义额外的方法**，所以只能使用Collection中定义的方法

### Map

Map接口的实现、继承关系如下：

<img src="D:\Typora\figure\figure2.png" alt="figure2" style="zoom:50%;" />

Map接口中有声明如下的常用方法：

* put、putAll
* remove(key)：通过key移除整个Entry
* clear
* get(key)：通过key找到value
* containsKey、containsValue
* size
* keySet、values、entrySet
  * **特别注意**：Map不能直接进行遍历，需要调用上面这三个方法得到对应的Set进行遍历，最后一个方法entrySet得到的Set对象的泛型标签为Map.Entry<K, V>

## 大小比较、排序

简单来说就是两个接口：Comparable、Comparator

### Comparable 接口

这个接口要求**需要进行排序的类直接实现**，并且要求在Comparable后加上对应类的泛型标签（一般就是这个实现类本身），下面给出一段示例代码：

```java
public class Person implements Comparable<Person>{
    private String name;

    public Person(String name) {
        this.name = name;
    }
    
    public String getName() {
        return this.name;
    }

    //Comparable的实现类需要实现下面这个方法
    @Override
    public int compareTo(Person o) {
        /*
        在这里写两个类进行比较的方式
        返回1表示当前对象“大于”比较目标o；
        返回0表示当前对象“等于”比较目标o；
        返回-1表示当前对象“小于”比较目标o；
        */
    }
}
```

在实现`compareTo`方法后需要进行排序时，直接调用`Arrays.sort(被排序数组)`即可

**注意**：这里给`sort`方法传入的**是一个数组，不是List的实现类**，因此这种排序方式相对局限，适用情况较少

### Comparator 接口

和Comparable接口的使用方法一样，这个接口也可以用于数组的排序，示例代码如下：

```java
Person[] people = new Person[n];
people[0] = new Person(...);
people[1] = new Person(...);
...
//比较
Arrays.sort(array, new Comparator<Person>() {
    @Override
    public int compare(Person o1, Person o2) {
        /*
        在这里写两个类进行比较的方式
        返回1表示当前对象“大于”比较目标o；
        返回0表示当前对象“等于”比较目标o；
        返回-1表示当前对象“小于”比较目标o；
        */
    }
});
```

同样，一般需要排序的都是集合，因此，如下的方式更加常用：

```java
List<Person> people = new ArrayList<>();

Comparator<Person> comparator = new Comparator<>() {
    @Override
    public int compare(Person o1, Person o2) {
        //比较方式
    }
};

people.sort(comparator);//直接使用List对象调用sort方法，方法中传一个Comparator实现类
```

## String

### String和数值类型、char[]互转

String -\> 数值：Integer.valueOf()，Long.valueOf()，根据数值的包装类调用

数值 -\> String：String.valueOf()，该方法有很多重载的方法，直接传入数值即可

String -\> char\[\]：str.toCharArray()，返回值就是一个char\[\]

char\[\] -\> String：new String (chars)，利用构造器，向其中传入一个char\[\]来实例化String对象

### 比大小

str1.compareTo(str2)

* 返回值 == 1 表示str1大于str2
* 返回值 == 0 表示str1等于str2
* 返回值 == -1 表示str1小于str2
* 这个compareTo就是之前Comparable接口的那个方法，言下之意就是String类实现了Comparable\<String\>接口

### String类的其它常用方法

* length
* toUpperCase / toLowerCase
* split
* trim
* charAt(int index)
* subString(int beginIndex) / subString(int beginIndex, int endIndex)
* contains(String subString)
* replace(char oldChar, char newChar)

## 枚举类

### 定义枚举类

* 自定义

  ```java
  class Season{
      //声明Season类的属性
      private final String name;
      private final String description;
      
      //私有化构造器
      private Season(String name, String description){
          this.name = name;
          this.description = description;
      }
      
      //提供自定义枚举类的多个对象
      //通过类直接获取对象
      public static final Season SPRING = new Season("Spring","...");
      public static final Season SUMMER = new Season("Summer","...");
      public static final Season FALL = new Season("Fall","...");
      public static final Season WINTER = new Season("Winter","...");
      
      //其它方法的提供：toString、getXX
      //...
  }
  ```

* 使用enum关键字定义枚举类（推荐）

  ```java
  enum Season{
      //上来必须直接提供当前枚举类对象
      SPRING("Spring","..."),
      SUMMER("Summer","..."),
      FALL("Fall","..."),
      WINTER("Winter","..."); //对象之间用逗号隔开，最后一个用分号
      
      //声明Season类的属性
      private final String name;
      private final String description;
      
      //私有化构造器
      private Season(String name, String description){
          this.name = name;
          this.description = description;
      }
      
      //其它方法的提供：toString、getXX
      //...
  }
  ```

  **注意**：

  * 定义的枚举类默认继承的不是Object，而是Enum类，这个类重写了toString方法，其默认的效果是打印枚举类对象的名字，比如上述例子中调用`Season.SPRING.toString()`的结果就是“SPRING”

### Enum类

使用enum关键字定义的类都是继承这个类的，所以可以直接调用这个类的方法，主要关注这个类中的三个方法即可：

* values()：一个静态方法，通过枚举类直接调用，获取枚举类内部定义的所有对象，返回值为当前枚举类的数组
* valueOf(Stirng name)：根据name在枚举类中找到名字和name一样的枚举类对象，找不到报错，注意这个方法区分大小写
* equals()：直接使用即可，不必重写，因为都是单例的，所以使用`==`判断时必然相等

## 日期与时间

`System.currentTimeMillis()`可以返回当前时间与1970年1月1日0时0分0秒之间的时间差，单位为毫秒，**该方法常用于计算时间差**

### Date类

```java
Date date1 = new Date();//使用当前的系统时间初始化Date类
Date date2 = new Date(value);//根据时间戳value初始化Date类

date1.getTime();//获取Date类对象的时间戳
```

### LocalDate、LocalTime、LocalDateTime类

```java
//实例化方式一：使用now方法，根据当前系统时间实例化
LocalDate localDate = LocalDate.now();
LocalTime localTime = LocalTime.now();
LocalDateTime localDateTime = LocalDateTime.now();

//实例化方式二：使用of方法，指定日期时间进行初始化
LocalDateTime localDateTime = LocalDateTime.of(2021,7,3,10,0,0);
//剩下两个也是一样的

//常用方法：
/*
plusXX()：在原有的时间基础上加上XX，可以是日、月、秒等，这三个类都是immutable的，所以这里事实上是返回了加上对应时间后的一个新的对象
toEpochDay()：时间戳，以日为单位，用于计算天数的差值
*/
```
