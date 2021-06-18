# 关于lab3的一些实现细节

> 说明：
>
> * 本篇blog仅针对[HIT 2021春 Software Construction Lab3](https://github.com/Edmund-Lai/JavaBlog/blob/main/Blog02/instruction.pdf)
> * 由于本次的Lab几乎没有任何往届的材料参考，所有的分析仅代表个人观点，欢迎任何的质疑、讨论与批评指正
> * 本篇blog针对Lab中的一些具体细节的实现，它们之间的关联较小，您可以单独查看各个细节
> * 内容可能并不完善，仍处在更新中，也会添加其它的主题
> * 由于各种原因本人暂时恕不上传源代码，如有需要请私下联系

## 索引

1. [如何实现GUI](# 如何实现GUI)
3. [正则表达式的使用](# 正则表达式的使用)
4. [日期的表示](# 日期的表示)

## 如何实现GUI

简单来说，这就是一个选API的过程，一般来说有以下四种方式：

* AWT：最早的GUI界面库
* Swing：对AWT的扩展，相对AWT轻量，最常用的方式
* JavaFX：jdk8引入，相对晦涩，但也是一种比较常用的方式
* SWT：开发Eclipse所用的界面库

本人选择的是Swing来实现GUI，对于GUI的开发，个人习惯使用前端展示，而本次的lab要求用jdk内置的包并不是本人擅长的，这里也只是做一个简单的介绍，从本人做这个lab目前碰到的问题来说已经够用了

下面给出一个例子，这个例子仅仅作Swing开发思路说明：

```java
public class demoGUI {
    public static void createGUI() {
        //窗口标题
        JFrame frame = new JFrame("Swing Demo");
        
        //点击窗口右上角的"×"的默认行为，这里默认行为是退出，可以设置为隐藏，参数是DISPOSE_ON_CLOSE
        frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

        //上面的操作成功创建了一个处在图层最底层的面板，下面的操作是向面板上添加一个容器
        Container contentPane = frame.getContentPane();
        contentPane.setLayout(new FlowLayout());

        //在“图层”上添加控件
        contentPane.add(new JLabel("Hello,World"));//一个JLabel对象
        contentPane.add(new JButton("测试"));//一个JButton对象

        //设置窗口参数
        frame.setSize(400, 300);

        //显示窗口
        frame.setVisible(true);
    }
    
    public static void main(String[] args) {
        javax.swing.SwingUtilities.invokeLater(demoGUI::createGUI);//GUI窗口创建
    }
}
```

从本人的角度理解，Swing的开发有些类似于使用Photoshop进行绘图，需要有最底下一层的图层，然后逐层覆盖，每一图层上的内容就是这里添加的控件，这里我们称“图层”为容器。

### 针对本次Lab的GUI绘制

以画出下面这个窗口为例进行说明：

<img src="D:\Typora\figure\image-20210617155003713.png" alt="image-20210617155003713" style="zoom: 50%;" />

实现的代码如下，您可以暂时跳过这段代码，这段代码中注释有`point XX`的内容本人会重点进行解释，您可以直接参考这些point的解释

```java
public static void dateDurationFrame() {
    //背景面板
    JFrame durationFrame = new JFrame("排班表系统日期设置");
    durationFrame.setDefaultCloseOperation(JFrame.DISPOSE_ON_CLOSE);
    durationFrame.setLayout(new GridLayout(4, 1));//point 1
    
    //参数设置，这两行代码像前面一个例子那样放在最后或者放在这里没有影响
    durationFrame.setVisible(true);
    durationFrame.setSize(600, 300);

    //三句提示信息
    String panelsName[] = {"请输入排班系统的起始日期及结束日期：", "起始日期 (yyyy-MM-dd)",
            "结束日期 (yyyy-MM-dd)"};
    List<JPanel> panelsList = new ArrayList<>();
    List<JTextField> textList = new ArrayList<>();
    
    
    for (int i = 0; i < panelsName.length; i++) {
        //创建panel对象
        JPanel newPanel = new JPanel();//point 2
        panelsList.add(newPanel);
        newPanel.setLayout(new FlowLayout());

        //panel内部结构、panel内部布局
        newPanel.add(new JLabel(panelsName[i]));
        if (i > 0) {
            JTextField newText = new JTextField(LINE_WIDTH);
            textList.add(newText);
            newPanel.add(newText);
        }

        //添加panel到背景板中
        durationFrame.add(newPanel);
    }

    //确认按钮
    JButton enterButton = new JButton("确认");
    durationFrame.add(enterButton);

    //事件触发，point3
    enterButton.addActionListener((e -> {
        List<String> gotString = new ArrayList<>();
        for (int i = 0; i < panelsName.length - 1; i++) {
            gotString.add(textList.get(i).getText());
        }
        String[] dateStart = gotString.get(0).split("-");
        String[] dateEnd = gotString.get(1).split("-");
        if (dateStart.length > 3 || dateEnd.length > 3) {
            JOptionPane.showMessageDialog(durationFrame, "日期输入格式有误");
            textList.get(0).setText("");
            textList.get(1).setText("");
            return;
        }


        DutyRosterApp dutyRosterApp = null;
        try {
            //可以暂时只考虑try中的内容
            //------------------------------------------------------------------
            dutyRosterApp = new DutyRosterApp(Integer.valueOf(dateStart[0]), Integer.valueOf(dateStart[1]), Integer.valueOf(dateStart[2]),
                    Integer.valueOf(dateEnd[0]), Integer.valueOf(dateEnd[1]), Integer.valueOf(dateEnd[2]));
            mainFrame(dutyRosterApp);
            durationFrame.dispose();
            //-------------------------------------------------------------------
        } catch (DutyRosterDurationException exception) {
            JOptionPane.showMessageDialog(durationFrame, exception.getMessage());
            for (JTextField textField : textList) {
                textField.setText("");
            }
        } catch (ArrayIndexOutOfBoundsException exception) {
            JOptionPane.showMessageDialog(durationFrame, "日期输入有误！");
            for (JTextField textField : textList) {
                textField.setText("");
            }
        }

    }));
}
```

#### point 1 - 布局

布局，简单来说就是当一个容器中有多个控件时，控件会面临一个如何排列的问题，排列的方式就称为布局

本次实验中，本人使用了三种布局方式：FlowLayout(流式布局)、BorderLayout(边界布局)、GridLayout(网格布局)

**流式布局**：

简单来说，就是所有的控件从左到右排列，排满了就下一行

<img src="D:\Typora\figure\image-20210617161102802.png" alt="image-20210617161102802" style="zoom:50%;" />

**边界布局**：

这种布局将整个容器分成5个部分，在插入控件时可以选择控件插入的位置：

<img src="D:\Typora\figure\image-20210617161139882.png" alt="image-20210617161139882" style="zoom:50%;" />

**网格布局**：

这个非常好理解，相当于将整个容器划分成若干个网格，所有的控件按照顺序依次插入网格：

<img src="D:\Typora\figure\image-20210617161442715.png" alt="image-20210617161442715" style="zoom:50%;" />

以上面的代码案例来说，对JFrame类的对象设置了网格布局：

```java
durationFrame.setLayout(new GridLayout(4, 1));//四行一列
```

这里将容器划分成四行一列，如图所示

<img src="D:\Typora\figure\figure5.png" style="zoom:50%;" />

然后在每一个网格中添加控件

#### point 2 - JPanel

JPanel，简单来说和JButton、JLabel类似，也是一种控件，但是它可以使用布局，在它自己的内部添加其它控件，从而实现比较复杂的布局效果。还是以下图为例：

<img src="D:\Typora\figure\image-20210617155003713.png" alt="image-20210617155003713" style="zoom: 50%;" />

在第二个网格中，可以观察到有两个内容：

1. 添加提示信息“起始日期(yyyy-MM-dd)”
2. 一个空白输入框

这两个内容从左到右排列，因此使用流式布局即可：

```java
JPanel newPanel = new JPanel();
newPanel.setLayout(new FlowLayout());//流式布局
```

前者使用JLabel类对象，后者使用JTextField类对象，依次加入JPanel对象中：

```java
newPanel.add(new JLabel(panelsName[i]));//添加提示信息
if (i > 0) {//第一行不需要输入框
    JTextField newText = new JTextField(LINE_WIDTH);
    newPanel.add(newText);//添加空白输入框
}
```

#### point 3 - 事件触发

事件触发对应用户点击确认按钮后需要执行的代码，代码框架如下：

```java
enterButton.addActionListener(new ActionListener() {
    @Override
    public void actionPerformed(ActionEvent e) {
        //执行逻辑
    }
});
```

这里的执行逻辑应该包括如下的内容：

1. 使用`textList.get(i).getText()`依次取出两个输入框中的内容，**此时的信息以String存在的**
2. 利用`Integer.valueOf(String对象)`将String对象转换成int数值
3. 利用int数值初始化排班表

**注意：建议您以显式的提示信息，类似于“(yyyy-MM-dd)”这样，提示用户按照固定的输入格式输入，否则将难以解析实际的数值**

### 常用控件和方法

1. 单行文本框 (JTextField) : `getText`用于获取文本框当前的内容，`setText`用于修改当前文本框的内容(一般用于在用户输入成功后清空所有文本框)
2. 多行文本框 (JTextArea) : 同单行文本框

3. 标签 (JLable) : 使用`new JLabel(提示信息)`创建JLable对象时直接初始化提示信息
4. 按钮 (JButton)  : 绑定监视器，处理触发的事件(即前文中的[事件触发](# point 3 事件触发))

5. 复选框 (JCheckBox)

6. 单选框 (JRadioButton)

7. 滚动窗格组件 (JScrollPane)

上述的这几个类的使用方法都非常类似，具体使用哪一个方法此处不赘述，大部分情况下您可以直接观察这些类中定义的方法名得知

**注意**：Swing和AWT中类名相当类似，一般来说就是差一个首字母‘J’，比如AWT中的按钮是`Button`类，而Swing中的按钮是`JButton`类，使用Swing时一定不要误添加AWT中的组件，事实上这么做编译器不会报错，这样的问题会难以排查

## 正则表达式的使用

无论您用任何的语言进行开发，正则表达式都将是您**在文本中提取关键信息的重要而强大的工具**

如果您的时间已经非常紧迫，可以直接参考如下的代码做相关的修改即可：

```java
public DutyRosterApp(String filename) throws IOException {
    BufferedReader bf = new BufferedReader(new FileReader(filename));
    StringBuilder sb = new StringBuilder();
    String line = new String();
    while ((line = bf.readLine()) != null) {
        sb.append(line);
    }
    String stringInfo = sb.toString();//提取文件中所有信息

    //扫描员工列表，这一步必须先进行，员工列表未初始化的情况下不能对员工的工作时间进行安排
    employeesList = new HashSet<>();
    Pattern patternEmployeesSet = Pattern.compile("([A-Z][a-z]+[A-Z][a-z]+)\\{([A-Z][A-Za-z\\s]+),(1(?:3\\d|4[4-9]|5[0-35-9]|6[67]|7[013-8]|8\\d|9\\d)-\\d{4}-\\d{4})\\}");
    Matcher matcherEmployeesSet = patternEmployeesSet.matcher(stringInfo);
    while (matcherEmployeesSet.find()) {
        String name = matcherEmployeesSet.group(1);//员工姓名
        String position = matcherEmployeesSet.group(2);//员工职位
        String[] phoneNumberInfo = matcherEmployeesSet.group(3).split("-");
        StringBuilder phoneNumber = new StringBuilder();
        for (String s : phoneNumberInfo) {
            phoneNumber.append(s);
        }
        //员工手机号码已获取，存储在phoneNumber中
        Employee employee = new Employee(name, position, phoneNumber.toString());
        //后续逻辑...
    }

    //时间段信息扫描，此时可以创建DutyRoster类对象
    Pattern patternDuration = Pattern.compile("Period\\{(\\d{4})-(\\d{2})-(\\d{2}),(\\d{4})-(\\d{2})-(\\d{2})\\}");
    Matcher matcherDuration = patternDuration.matcher(stringInfo);
    if (!matcherDuration.find()) {
        throw new RuntimeException("输入正则表达式有误");
    }
    //整个值班时间段的起始日
    String yearStart = matcherDuration.group(1);
    String monthStart = matcherDuration.group(2);
    String dayStart = matcherDuration.group(3);
    //整个值班时间段的结束日
    String yearEnd = matcherDuration.group(4);
    String monthEnd = matcherDuration.group(5);
    String dayEnd = matcherDuration.group(6);
    //后续逻辑...

    //值班安排信息扫描
    Pattern patternArrange = Pattern.compile("([A-Z][a-z]+[A-Z][a-z]+)\\{(\\d{4})-(\\d{2})-(\\d{2}),(\\d{4})-(\\d{2})-(\\d{2})\\}");
    Matcher matcherArrange = patternArrange.matcher(stringInfo);
    while (matcherArrange.find()) {
        String name = matcherArrange.group(1);//员工姓名获取，需要在员工列表中检索对应的员工类对象
        Employee employeeChosen = null;
        for (Employee employee : employeesList) {
            if (employee.getName().equals(name)) {
                employeeChosen = employee;
                break;
            }
        }
        if (employeeChosen == null) {
            throw new NoSuchEmployeeException("被安排员工不属于员工列表！");
        }
        //当前员工值班起始日
        yearStart = matcherArrange.group(2);
        monthStart = matcherArrange.group(3);
        dayStart = matcherArrange.group(4);
        //当前员工值班结束日
        yearEnd = matcherArrange.group(5);
        monthEnd = matcherArrange.group(6);
        dayEnd = matcherArrange.group(7);
		//后续逻辑...
    }

}
```

好的，如果您还在看的话，本人接下来会对java中正则表达式的使用做一些介绍

为了使用正则表达式进行匹配，您需要使用两个类：`Pattern`和`Matcher`，使用框架如下：

```java
Pattern pattern = Pattern.compile(正则表达式模式串);
Matcher matcher = pattern.matcher(被匹配的文本);
while(matcher.find()){
    String info = matcher.group(检索需要子串所在的索引);
    //对info进行操作，实现后续的逻辑
}
```

本篇blog**不会**对正则表达式模式串做相关介绍，该部分内容在课程的ppt上有对应的内容，您可以自行查看

对Matcher类对象进行操作时最反常的一点就是`matcher.find()`并不会返回我们需要的结果，我们需要用`String info = matcher.group(索引值)`来获取我们需要的字符串，这里的原因和`Matcher`类内部的设计非常有趣和巧妙，下面进行解释

### Matcher类中的groups数组及分组原理

为了解释这个问题，我们需要用一个更简单的例子入手，给出如下的文本：

```
I am happy to join with you today in what will go down in history as the greatest demonstration for freedom in the history of our nation.
```

假设我们需要在上述的文本中寻找“happy”这个单词，那么我们的代码如下所示：

```java
public static void main(String[] args) {
    String stringInfo = "上述文本";
    Pattern pattern = Pattern.compile("happy");
    Matcher matcher = pattern.matcher(stringInfo);
    while (matcher.find()){
        String result = matcher.group(0);//得到的result就是"happy"
		//后续逻辑
    }
}
```

我们为了获取这个单词，会使用`matcher.group(0)`

为了解释这个索引为什么是0，我们需要对这个过程进行debug，将断点设置在`String result = matcher.group(0)`这一行，得到如下的结果，重点注意红框中的内容：

<img src="D:\Typora\figure\figure6.png" style="zoom:50%;" />

#### `first`变量和`last`变量

它们分别对应5和10，您此时可能已经猜到了，这个代表的含义就是“happy”在原来文本中的位置：

```
I am happy to join ...
```

第一个字母的索引为0，从左往右数，发现第一个字符‘h’恰好对应的索引是5；而最后一个字符‘y’对应的字符是9，这里的原因稍后解释

#### `oldLast`变量

这个您可能也能猜到，这个变量的作用仅仅是给下一次循环中调用`matcher.find()`一个起始的匹配位置，因为前面第一次匹配到了，所以下一次要从第一次匹配到的位置再次开始

#### `groups`数组

这个数组是关键所在，这个数组被声明为`int[] groups`，它就是用于存储我们之前的`first`变量和`last`变量，但是我们不能直接访问，需要调用类似`matcher.group(0)`这样的方法才能访问，所以我们将目光转移到这个方法，其源码如下：

```java
public String group(int group) {
    //忽略异常处理
    if (first < 0)
        throw new IllegalStateException("No match found");
    if (group < 0 || group > groupCount())
        throw new IndexOutOfBoundsException("No group " + group);
    if ((groups[group*2] == -1) || (groups[group*2+1] == -1))
        return null;
    
    //重点只有下面这一行
    return getSubSequence(groups[group * 2], groups[group * 2 + 1]).toString();
}
```

这个数组调用一个`getSubSequence`方法，从方法名可知这个方法是从原文本中截取一段子串返回，参数是`groups[group * 2]`和`groups[group * 2 + 1]`，我们带入group=0，那么参数分别就是`groups[0]`和`groups[1]`，此时我们再回看groups数组的内容：

<img src="D:\Typora\figure\image-20210618081940410.png" alt="image-20210618081940410" style="zoom: 67%;" />

`groups[0]`和`groups[1]`恰好是5和10，而这恰好就是我们需要的“happy”的首尾索引，置于为什么尾部字符索引是10，只需要查看`getSubSequence`方法的specification即可，它指明了返回时遵循“左闭右开”的原则(Java中大部分的API都是这样)，实际上不包括索引为10的那个字符

#### 分组

现在您可能好奇，既然有`matcher.group(0)`，那么有没有`matcher.group(1)`、`matcher.group(2)`呢？——这取决于您是否进行了分组

关于什么是分组，这里以一个例子给出，例如您此时得到了“happy”字符串，您想进一步从“happy”字符串中得到“ha”和“ppy”这两个子串，那么您现在需要的就是分组，代码如下：

```java
public static void main(String[] args) {
    String stringInfo = "上述文本";
    Pattern pattern = Pattern.compile("(ha)(ppy)");//仅有这里不同，表示需要进行分组
    Matcher matcher = pattern.matcher(stringInfo);
    while (matcher.find()){
        String result1 = matcher.group(0);//还是得到happy
        String result2 = matcher.group(1);//得到ha
        String result3 = matcher.group(2);//得到ppy
		//后续逻辑
    }
}
```

相比于前面不分组的场景，我们代码上的区别仅有匹配模式串从“happy”变成了"(ha)(ppy)"，简单来说这个就是分组：第一对括号里的是第一组，第二对括号里的是第二组，以此类推。此时我们同样进行debug，可以得到如下结果：

<img src="D:\Typora\figure\figure8.png" style="zoom: 50%;" />

我们可以发现仅仅只有groups数组的内容发生了变化，groups[2]、groups[3]、groups[4]、groups[5]分别有了内容5、7、7、10，稍加观察便可知道，5和7对应的是子串“ha”，7和10对应的是子串“ppy”，而我们获取这两个子串分别使用了`matcher.group(1)`和`matcher.group(2)`，此时我们再回看`group`方法的关键代码：

```java
return getSubSequence(groups[group * 2], groups[group * 2 + 1]).toString();
```

分别代入group=1和group=2，可以得到groups[2]、groups[3]即groups[4]、groups[5]这里**恰好对应了两组子串的索引**！自此，我们终于发现了这个`group`方法背后的秘密，我们使用`matcher.group(1)`和`matcher.group(2)`本质上就是获取到了第一组和第二组对应子串。

## 日期的表示

此次lab中比较麻烦的一点就是需要使用日期、计算日期，特别是计算天数差距

这里直接说结论：使用`LocalDate`类，使用方法如下：

```java
//使用 LocalDate.of()得到LocalDate类对象，参数分别是年、月、日
LocalDate dateStart = LocalDate.of(yearStart, monthStart, dayStart);
LocalDate dateEnd = LocalDate.of(yearEnd, monthEnd, dayEnd);
//计算两个LocalDate类对象表示的日期相差的天数
long duration = dateEnd.toEpochDay() - dateStart.toEpochDay();
```

此外，这个类还提供了一些其它方法：

* getYear：获取年
* getMonth：获取月
* getDayOfMonth：当天是当前月的第几天
* getDayOfYear：当天是当前年的第几天
* getDayOfWeek：当天是当前周的第几天

使用这几个方法处理本次的lab已绰绰有余
