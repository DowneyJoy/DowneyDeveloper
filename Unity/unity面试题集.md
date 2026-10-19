1. 重载和重写的区别
      1)   封装、继承、多态所处位置不同，重载在同类中，重写在父子类中。

      2)   定义方式不同，重载方法名相同参数列表不同，重写方法名和参数列表都相同。

      3)   调用方式不同，重载使用相同对象以不同参数调用，重写用不同对象以相同参数调用。

      4)   多态时机不同，重载时编译时多态，重写是运行时多态。

2. 面向对象的三大特点
      1.继承： 提高代码重用度，增强软件可维护性的重要手段，符合开闭原则。继承最主要的作用就是把子类的公共属性集合起来，便与共同管理，使用起来也更加方便。你既然使用了继承，那代表着你认同子类都有一些共同的特性，所以你把这些共同的特性提取出来设置为父类。继承的传递性：传递机制 a▶b; b▶c; c具有a的特性 。继承的单根性：在C#中一个类只能继承一个类，不能有多个父类。

      2.封装： 封装是将数据和行为相结合，通过行为约束代码修改数据的程度，增强数据的安全性，属性是C#封装实现的最好体现。就是将一些复杂的逻辑经过包装之后给别人使用就很方便，别人不需要了解里面是如何实现的，只要传入所需要的参数就可以得到想要的结果。封装的意义在于保护或者防止代码（数据）被我们无意中破坏。

      3.多态性： 多态性是指同名的方法在不同环境下，自适应的反应出不同得表现，是方法动态展示的重要手段。多态就是一个对象多种状态，子类对象可以赋值给父类型的变量。

3.简述值类型和引用类型有什么区别
     1.值类型存储在内存栈中，引用类型数据存储在内存堆中，而内存单元中存放的是堆中存放的地址。

     2.值类型存取快，引用类型存取慢。
    
     3.值类型表示实际数据，引用类型表示指向存储在内存堆中的数据的指针和引用。
    
     4.栈的内存是自动释放的，堆内存是.NET 中会由 GC 来自动释放。
    
     5.值类型继承自 System.ValueType,引用类型继承自 System.Object。

4.请简述private，public，protected，internal的区别
     public：对任何类和成员都公开，无限制访问

     private：仅对该类公开
    
     protected：对该类和其派生类公开
    
     internal：只能在包含该类的程序集中访问该类
    
     protected internal：protected + internal

5.C#中所有引用类型的基类是什么
     引用类型的基类是System.Object值类型的基类是 System.ValueType   同时，值类型也隐式继承自System.Object

6.请简述ArrayList和 List的主要区别
     ArrayList不带泛型 数据类型丢失

     List带泛型 数据类型不丢失
    
     ArrayList需要装箱拆箱 List不需要

7.请简述GC（垃圾回收）产生的原因，并描述如何避免？
     GC为了避免内存溢出而产生的回收机制

     避免：1）减少 new 产生对象的次数
    
                2）使用公用的对象（静态成员）
    
                3）将 String 换为 StringBuilder

8.请描述Interface与抽象类之间的不同
     1.接口不是类 不能实例化 抽象类可以间接实例化

     2.接口是完全抽象 抽象类为部分抽象
    
     3.接口可以多继承 抽象类是单继承

9.请简述关键字Sealed用在类声明和函数声明时的作用
     类声明时可防止其他类继承此类，在方法中声明则可防止派生类重写此方法。

10.反射的实现原理？
     可以在加载程序运行时，动态获取和加载程序集，并且可以获取到程序集的信息反射即在运行期动态获取类、对象、方法、对象数据等的一种重要手段

主要使用的类库：System.Reflection

核心类：

1.Assembly描述了程序集

2.Type描述了类这种类型

3.ConstructorInfo描述了构造函数

4.MethodInfo描述了所有的方法

5.FieldInfo描述了类的字段

6.PropertyInfo描述类的属性

通过以上核心类可在运行时动态获取程序集中的类，并执行类构造产生类对象，动态获取对象的字段或属性值，更可以动态执行类方法和实例方法等。

11..Net与 Mono 的关系？
     .Net是一个语言平台，Mono为.Net提供集成开发环境，集成并实现了.NET的编译器、CLR 和基础类库，使得.Net既可以运行在windows也可以运行于 linux，Unix，Mac OS 等。

12.在类的构造函数前加上static会报什么错?为什么?
构造函数格式为public+类名如果加上 static 会报错（静态构造函数不能有访问、型的对象，静态构造函数只执行一次；运行库创建类实例或者首次访问静态成员之前，运行库调用静态构造函数；静态构造函数执行先于任何实例级别的构造函数；显然也就无法使用this和 base 来调用构造函数。

13.C# String类型比 stringBuilder 类型的优势是什么?
如果是处理字符串的话，用string中的方法每次都需要创建一个新的字符串对象并且分配新的内存地址，而 stringBuilder 是在原来的内存里对字符串进行修改，所以在字符串处理

方面还是建议用stringBuilder这样比较节约内存。但是 string 类的方法和功能仍然还是比 stringBuilder 类要强。

string类由于具有不可变性（即对一个 string 对象进行任何更改时，其实都是创建另外一个 string 类的对象），所以当需要频繁的对一个 string 类对象进行更改的时候，建议使用StringBuilder 类，StringBuilder 类的原理是首先在内存中开辟一定大小的内存空间，当对此 StringBuilder 类对象进行更改时， 如果内存空间大小不够， 会对此内存空间进行扩充，而不是重新创建一个对象，这样如果对一个字符串对象进行频繁操作的时候，不会造成过多的内存浪费，其实本质上并没有很大区别，都是用来存储和操作字符串的，唯一的区别就在于性能上。

String主要用于公共 API，通用性好、用途广泛、读取性能高、占用内存小。

StringBuilder主要用于拼接 String，修改性能好。

不过现在的编译器已经把String的 + 操作优化成 StringBuilder 了， 所以一般用String 就可以了

String是不可变的，所以天然线程同步。

StringBuilder可变，非线程同步。

14.C#函数 Func(string a, string b)用 Lambda 表达式怎么写?
(a,b) => {};

15.数列1,1,2,3,5,8,13...第 n 位数是多少?用 C#递归算法实现
public int CountNumber(int num) {

       if (num == 1 || num == 2) {
    
           return 1;
    
       } else {
    
           return CountNumber(num -1) + CountNumber(num-2);
    
       }

  }

16.冒泡排序（手写代码）
public static void BubblingSort(int[]array) {

      for (int i = 0; i < array.Length; i++){
    
          for (int j = array.Length - 1; j > 0; j--){
    
              if (array[j] < array[i]) {
    
                  int temp = array[j];
    
                  array[j] = array[j-1];
    
                  array[j - 1] = temp;
    
              }
    
          }
    
      }

  }

17.C#中有哪些常用的容器类，各有什么特点。
List,HashTable,Dictionary,Stack,Queue

List：索引泛型容器 访问速度快 修改速度慢

HashTable/Dictionary：散列表格式 查询效率高 空间占用较大

Stack：后进先出

Queue:先进先出

18.C#中常规容器和泛型容器有什么区别，哪种效率高？
不带泛型的容器需要装箱和拆箱操作速度慢所以泛型容器效率更高数据类型更安全

19.有哪些常见的数值类？
简单值类型--包括 整数类型、实数类型、字符类型、布尔类型

复合值类型--包括 结构类型、枚举类型

20.C#中委托和接口有什么区别？各用在什么场合？
接口（interface）是约束类应该具备的功能集合，约束了类应该具备的功能，使类从千变万化的具体逻辑中解脱出来，便于类的管理和扩展，同时又合理解决了类的单继承问题。

C#中的委托是约束方法集合的一个类，可以便捷的使用委托对这个方法集合进行操作。

在以下情况中使用接口：

1.无法使用继承的场合

2.完全抽象的场合

3.多人协作的场合

以上等等

在以下情况中使用委托：多用于事件处理中

21.C#中unsafe关键字是用来做什么的？什么场合下使用？
非托管代码才需要这个关键字一般用在带指针操作的场合

22.C#中ref和out关键字有什么区别？
ref修饰参数，表示进行引用传递，out修饰参数也表示进行引用传递，但传递的引用只为带回返回值 ref又进又出 out不进只出

23.For，foreach，Enumerator.MoveNext的使用，与内存消耗情况
for循环可以通过索引依次进行遍历，foreach和Enumerator.MoveNext通过迭代的方式进行遍历。内存消耗上本质上并没有太大的区别。但是在Unity中的Update中，一般不推荐使用foreach 因为会遗留内存垃圾。

24.函数中多次使用string的+=处理，会产生大量内存垃圾（垃圾碎片），有什么好的方法可以解决。
通过StringBuilder那进行append，这样可以减少内存垃圾

25.当需要频繁创建使用某个对象时，有什么好的程序设计方案来节省内存？
设计单例模式进行创建对象或者使用对象池

26.JIT和AOT区别
Just-In-Time -实时编译

执行慢安装快占空间小一点

Ahead-Of-Time -预先编译

执行快安装慢占内存占外存大

27.给定一个存放参数的数组，重新排列数组
void SortArray(Array arr){Array.Sort(arr);}

28.Foreach循环迭代时，若把其中的某个元素删除，程序报错，怎么找到那个元素？以及具体怎么处理这种情况？(注：Try.....Catch捕捉异常，发送信息不可行)
foreach不能进行元素的删除，因为迭代器会锁定迭代的集合，解决方法：记录找到索引或者key值，迭代结束后再进行删除。

29.GameObject a=new GameObject()  GameObject b=a  实例化出来了A，将A赋给B，现在将B删除，问A还存在吗？
存在，b删除只是将它在栈中的内存删除，而A对象本身是在堆中，所以A还存在

30.你拥有A块钱，一瓶水B块钱，每瓶水可以得到一个瓶盖，每C个瓶盖可以换一瓶水请写出函数求解上面题目，上面题目ABC为参数

    public static int Buy(int a,int b,int c) {
     
        return a/b + ForCap(c,a/b);
     
    }
     
    public static int ForCap(int c,int d) {
     
        if (d
     
        return 0;
     
        } else {
     
        return d/c + ForCap(c,d/c + d%c);
     
        }
     
    }

31.有一排开关，第一个人把所有的开关打开，第二个人按2的倍数的开关，第三个人按3的倍数的开关，以此类推，现在又n个开关，k个人，写函数求最后等两者的开关，输入参数n和k
static void Main(string[] args) {

       int n = int.Parse(Console.ReadLine());
     
       int k = int.Parse(Console.ReadLine());
     
       Function(100,100);

  }

  static void Function(int n, int k) {

       int i, j = 0;

bool[] a = new bool[1000]; //初始false：关灯，true:开灯 

for (i = 1; i <= k; i++)      //k个人

for (j = 1; j <= n; j++)  //n个灯

               if (j % i == 0)

a[j] = !a[j]; //取反,false变true,原来开变关，关变开

for (i = 1; i <= n; i++) //最后输出a[i]的值就可以了

if (a[i]) //灯亮着

                   Console.WriteLine(i);

}
32.数制转换，将任意整数转换成8进制形式
static void Main(string[] args) {

      int n;
     
      n =int.Parse(Console.ReadLine());

Console.WriteLine("输入的10进制为：{0}",n);

Console.Write("转换为8进制数为: ");

      d2o(n);

}

static void d2o(int n) {

     if (n > 7) {
     
          d2o(n / 8);
     
     }
     
     Console.Write(n%8);

}
33.找出200以内的素数。
static void Main(string[] args) {

    int count = 0;
     
    for (int i = 1; i < 200; i++) {//外层循环：要判断的数
     
        for (int j = 2; j <=i; j++){
     
            if (i % j == 0&& i!=j) {
     
                 break;
     
             }
     
             if (j == i ) {//结束的条件:最后一个数还没有被整除 
     
                  count++;
     
                  Console.WriteLine(i);
     
             }
     
        }
     
    }
     
    Console.WriteLine(count);

}
34.打印杨辉三角形
public static void YHSJ(){

    int [][]a= new int[7][] ;
     
    a[0] = new int[1];  //a[0][0]=1;
     
    a[1] = new int[2] ;
     
    for (int i = 0; i < 7; i++) {
     
        a[i] = new int[i+1] ;  
     
        a[i][0] =1;
     
        a[i][i]=1;
     
        if(i>1) {//求出中间的数据
     
    for(int j=1;j
     
                a[i][j]= a[i-1][j-1]+a[i-1][j];
     
            }
     
        }
     
     }
     
     for (int i=0; i
     
         for (int k = 0; k < a.Length-1-i; k++) {
     
             Console.Write("");
     
         }
     
         for(int j=0;j
     
             Console.Write(a[i][j] + "");
     
         }
     
         Console.WriteLine();
     
     }

}
35.中国有句俗话“三天打鱼两天晒网”，某人从2000年1月1日起开始“三天打鱼两天晒网”，问这个人在今后的某天中“打鱼”还是”晒网”
public static void Compute(){

Console.WriteLine ((DateTime.Now - DateTime.Parse("2000-01-01")).Days%5<3?"打鱼":"晒网");

}
36.假设当前市场价一只鸡10元，一只鸭12元5角。请写一个函数ShowPrice，输入参数分别为鸡和鸭的个数（非负整型），功能为显示出总价钱，精确到分。例如调用ShowPrice（5,10）后输出175.00。请注意程序的可读性和易于维护性。
static void ShowPrice(int num_chicken, int num_duck)  {

      float totalPrice = 0.00f;
     
      float price_chicken = 10f;
     
      float price_duck = 12.5f;
     
      totalPrice = num_chicken * price_chicken + num_duck * price_duck;

Console.WriteLine("总价钱为：{0:0.00}", totalPrice);

  }
37.请写一个函数，用于返回n!（阶乘）结果末尾连续0的个数，如GetZeroCount（5）返回1,因为5！ = 120，末尾连续1个0
  static void Main(string[] args) {

        int fac = Factorial(5);
     
        Console.WriteLine(CountZero(fac));
     
    }
     
    public static int Factorial(int n) {
     
        if (n == 1) {
     
            return 1;
     
        } else {
     
            return n * jiecheng(n - 1);
     
        }
     
    }

//求连续的0的个数

    public static int CountZero(int num) {

int result = 0; //最后的结果

        String numStr = num.ToString();
     
        for (int i = numStr.Length - 1; i >= 0; i--) {
     
            if (numStr[i] == '0') {
     
               result ++;
     
            } else {
     
                break;
     
            }
     
        }
     
        return result;
     
    }
38、C#中 委托和事件的区别
大致来说，委托是一个类，该类内部维护着一个字段，指向一个方法。事件可以被看作一个委托类型的变量，通过事件注册、取消多个委托或方法。本篇分别通过委托和事件执行多个方法，从中体会两者的区别。

□ 通过委托执行方法


class Program 

    {
     
        static void Main(string[] args)
     
        {
     
            Example example = new Example();
     
            example.Go();
     
            Console.ReadKey();
     
        }
     
    }

 



    public class Example
     
    {
     
        public delegate void DoSth(string str);

 



        internal void Go()
     
        {
     
            //声明一个委托变量，并把已知方法作为其构造函数的参数
     
            DoSth d = new DoSth(Print);

 



            string str = "Hello,World";

 



            //通过委托的静态方法Invoke触发委托
     
            d.Invoke(str);
     
        }

 



        void Print(string str)
     
        {
     
            Console.WriteLine(str);
     
        }
     
    }
以上，
○ 在CLR运行时，委托DoSth实际上就一个类，该类有一个参数类型为方法的构造函数，并且提供了一个Invoke实例方法，用来触发委托的执行。
○ 委托DoSth定义了方法的参数和返回类型
○ 通过委托DoSth的构造函数，可以把符合定义的方法赋值给委托
○ 调用委托的实例方法Invoke执行了方法

但，实际上让委托执行方法还有另外一种方式，那就是：委托变量(参数列表)

public class Example 

    {
     
        public delegate void DoSth(object sender, EventArgs e);

 



        internal void Go()
     
        {
     
            //声明一个委托变量，并把已知方法作为其构造函数的参数
     
            DoSth d = new DoSth(Print);

 



            object sender = 10;
     
            EventArgs e = new EventArgs();

 



            d(sender, e);
     
        }

 



        void Print(object sender, EventArgs e)
     
        {
     
            Console.WriteLine(sender);
     
        }
     
    }
以上，

○ 委托DoSth的参数列表和方法Print的参数列表还是保持一致
○ 委托DoSth中的参数object sender通常用来表示动作的发起者，EventArgs e用来表示动作所带的参数。

而实际上，委托变量(参数列表)，事件就是采用这种形式执行方法的。

□ 通过事件执行方法

public class Example 

    {
     
        public delegate void DoSth(object sender, EventArgs e);
     
        public event DoSth myDoSth;

 



        internal void Go()
     
        {
     
            //声明一个委托变量，并把已知方法作为其构造函数的参数
     
            DoSth d = new DoSth(Print);

 



            object sender = 10;
     
            EventArgs e = new EventArgs();

 



            myDoSth += new DoSth(d);
     
            myDoSth(sender, e);
     
        }

 



        void Print(object sender, EventArgs e)
     
        {
     
            Console.WriteLine(sender);
     
        }
     
    }
以上，

○ 声明了事件myDoSth,事件的类型是DoSth这个委托
○ 通过+=为事件注册委托
○ 通过DoSth委托的构造函数为事件注册委托实例
○ 采用委托变量(参数列表)这种形式，让事件执行方法

而且，通过+=还可以为事件注册多个委托。

public class Example 

    {
     
        public delegate void DoSth(object sender, EventArgs e);
     
        public event DoSth myDoSth;

 



        internal void Go()
     
        {
     
            //声明一个委托变量，并把已知方法作为其构造函数的参数
     
            DoSth d = new DoSth(Print);
     
            DoSth d1 = new DoSth(Say);

 



            object sender = 10;
     
            EventArgs e = new EventArgs();

 



            //为事件注册多个委托
     
            myDoSth += new DoSth(d);
     
            myDoSth += new DoSth(d1);

 



            myDoSth(sender, e);
     
        }

 



        void Print(object sender, EventArgs e)
     
        {
     
            Console.WriteLine(sender);
     
        }

 



        void Say(object sender, EventArgs e)
     
        {
     
            Console.WriteLine(sender);
     
        }
     
    }
以上，通过+=为事件注册1个或多个委托实例，实际上，还可以为事件直接注册方法。

 

public class Example 

    {
     
        public delegate void DoSth(object sender, EventArgs e);
     
        public event DoSth myDoSth;

 



        internal void Go()
     
        {
     
            object sender = 10;
     
            EventArgs e = new EventArgs();

 



            //为事件注册多个委托
     
            myDoSth += Print;
     
            myDoSth += Say;

 



            myDoSth(sender, e);
     
        }

 



        void Print(object sender, EventArgs e)
     
        {
     
            Console.WriteLine(sender);
     
        }

 



        void Say(object sender, EventArgs e)
     
        {
     
            Console.WriteLine(sender);
     
        }
     
    }   
□ 通过EventHandler执行方法

先来看EventHandler的源代码。



可见，EventHandler就是委托。现在就使用EventHandler来执行多个方法。

 

public class Example 

    {
     
        public event EventHandler myEvent;

 



        internal void Go()
     
        {
     
            object sender = 10;
     
            EventArgs e = new EventArgs();

 



            //为事件注册多个委托
     
            myEvent += Print;
     
            myEvent += Say;

 



            myEvent(sender, e);
     
        }

 



        void Print(object sender, EventArgs e)
     
        {
     
            Console.WriteLine(sender);
     
        }

 



        void Say(object sender, EventArgs e)
     
        {
     
            Console.WriteLine(sender);
     
        }
     
    }
总结：
○ 委托就是一个类，也可以实例化，通过委托的构造函数来把方法赋值给委托实例
○ 触发委托有2种方式: 委托实例.Invoke(参数列表)，委托实例(参数列表)
○ 事件可以看作是一个委托类型的变量
○ 通过+=为事件注册多个委托实例或多个方法
○ 通过-=为事件注销多个委托实例或多个方法
○ EventHandler就是一个委托

  1.事件的声明只是在委托前面加一个event关键词，虽然你可以定义一个public，但是有了event关键词后编译器始终会把这个委托声明为private，然后添加1组add，remove方法。add对应+=，remove对应-=。这样就导致事件只能用+=，-=来绑定方法或者取消绑定方法。而委托可以用=来赋值，当然委托也是可以用+=，-=来绑定方法的（面试我的那个哥们好像说不行）。

  2.委托可以在外部被其他对象调用，而且可以有返回值（返回最后一个注册方法的返回值）。而事件不可以在外部调用，只能在声明事件的类内部被调用。我们可以使用这个特性来实现观察者模式。大概就是这么多。下面是一段测试代码。


namespace delegateEvent
{
    public delegate string deleFun(string word);

    public class test
    {
        public event deleFun eventSay;
        public deleFun deleSay;
        public void doEventSay(string str)
        {
            if (eventSay!=null)
                eventSay(str);
        }
    }

 


    class Program
    {
        static void Main(string[] args)
        {
            test t = new test();
            t.eventSay += t_say;
            t.deleSay += t_say;
            t.deleSay += t_say2;
            //t.eventSay("eventSay"); 错误 事件不能在外部直接调用
            t.doEventSay("eventSay");//正确 事件只能在声明的内部调用
            string str = t.deleSay("deleSay");//正确 委托可以在外部被调用 当然在内部调用也毫无压力 而且还能有返回值（返回最后一个注册的方法的返回值）
            Console.WriteLine(str);
            Console.Read();
        }
     
        static string t_say(string word)
        {
            Console.WriteLine(word);
     
            return "return "+word;
        }
     
        static string t_say2(string word)
        {
            Console.WriteLine(word);
     
            return "return " + word + " 2";
        }
    }
}
39、有限状态机和行为树的区别
附录：
C#常用的数据结构详解 ：Array，ArrayList，List，LinkedList，Queue，Stack，Dictionary<K,T>
1、数组是最简单的数据结构。其具有如下特点：

数组存储在连续的内存上。
数组的内容都是相同类型。
数组可以直接通过下标访问。
数组创建

1 int size = 10;

2 int[] test = new int[size];

创建一个新的数组时将在 CLR 托管堆中分配一块连续的内存空间，来盛放数量为size，类型为所声明类型的数组元素。如果类型为值类型，则将会有size个未装箱的该类型的值被创建。如果类型为引用类型，则将会有size个相应类型的引用被创建。

由于是在连续内存上存储的，所以它的索引速度非常快，访问一个元素的时间是恒定的也就是说与数组的元素数量无关，而且赋值与修改元素也很简单。

string[] test2 = new string[3];

//赋值

test2[0] = "chen";

test2[1] = "j";

test2[2] = "d";

//修改

test2[0] = "chenjd";

但是有优点，那么就一定会伴随着缺点。由于是连续存储，所以在两个元素之间插入新的元素就变得不方便。而且就像上面的代码所显示的那样，声明一个新的数组时，必须指定其长度，这就会存在一个潜在的问题，那就是当我们声明的长度过长时，显然会浪费内存，当我们声明长度过短的时候，则面临这溢出的风险。这就使得写代码像是投机，小匹夫很厌恶这样的行为！针对这种缺点，下面隆重推出ArrayList。

2.ArrayList：
　　为了解决数组创建时必须指定长度以及只能存放相同类型的缺点而推出的数据结构。ArrayList是System.Collections命名空间下的一部分，所以若要使用则必须引入System.Collections。正如上文所说，ArrayList解决了数组的一些缺点。

不必在声明ArrayList时指定它的长度，这是由于ArrayList对象的长度是按照其中存储的数据来动态增长与缩减的。
ArrayList可以存储不同类型的元素。这是由于ArrayList会把它的元素都当做Object来处理。因而，加入不同类型的元素是允许的。
　　ArrayList的操作：

ArrayList test3 = new ArrayList();

//新增数据

test3.Add("i");

test3.Add("j");

test3.Add("d");

test3.Add("is");

test3.Add(25);

test3[4] = 26;

test3.RemoveAt(4);

说了那么一堆”优点“，也该说说缺点了吧。为什么要给”优点”打上引号呢？那是因为ArrayList可以存储不同类型数据的原因是由于把所有的类型都当做Object来做处理，也就是说ArrayList的元素其实都是Object类型的，辣么问题就来了。

ArrayList不是类型安全的。因为把不同的类型都当做Object来做处理，很有可能会在使用ArrayList时发生类型不匹配的情况。
如上文所诉，数组存储值类型时并未发生装箱，但是ArrayList由于把所有类型都当做了Object，所以不可避免的当插入值类型时会发生装箱操作，在索引取值时会发生拆箱操作。这能忍吗？

注：为何说频繁的没有必要的装箱和拆箱不能忍呢？所谓装箱 (boxing)：
    就是值类型实例到对象的转换(百度百科)。那么拆箱：就是将引用类型转换为值类型咯（还是来自百度百科）。如下

//装箱，将值类型转成引用类型
int  info = 1989;  
object obj=(object)info;  

//拆箱，引用类型转换成值类型
object obj = 1;
int info = (int)obj;

显然，从原理上可以看出，装箱时，生成的是全新的引用对象，这会有时间损耗，也就是造成效率降低。那么为了解决ArrayList不安全类型与装箱拆箱的缺点，所以出现了泛型的概念，作为一种新的数组类型引入。也是工作中经常用到的数组类型。和ArrayList很相似，长度都可以灵活的改变，最大的不同在于在声明List集合时，我们同时需要为其声明List集合内数据的对象类型，这点又和Array很相似，其实List内部使用了Array来实现。

3、List

List<string> test4 = new List<string>(); 

//新增数据

test4.Add(“Fanyoy”); 

test4.Add(“Chenjd”);  

//修改数据 

test4[1] = “murongxiaopifu”; 

//移除数据 

test4.RemoveAt(0);

这么做最大的好处就是：(1)即确保了类型安全;(2)也取消了装箱和拆箱的操作;(3)它融合了Array可以快速访问的优点以及ArrayList长度可以灵活变化的优点。

也就是链表了。和上述的数组最大的不同之处就是在于链表在内存存储的排序上可能是不连续的。这是由于链表是通过上一个元素指向下一个元素来排列的，所以可能不能通过下标来访问。如图



既然链表最大的特点就是存储在内存的空间不一定连续，那么链表相对于数组最大优势和劣势就显而易见了。

向链表中插入或删除节点无需调整结构的容量。因为本身不是连续存储而是靠各对象的指针所决定，所以添加元素和删除元素都要比数组要有优势。

链表适合在需要有序的排序的情境下增加新的元素，这里还拿数组做对比，例如要在数组中间某个位置增加新的元素，则可能需要移动移动很多元素，而对于链表而言可能只是若干元素的指向发生变化而已。

有优点就有缺点，由于其在内存空间中不一定是连续排列，所以访问时候无法利用下标，而是必须从头结点开始，逐次遍历下一个节点直到寻找到目标。所以当需要快速访问对象时，数组无疑更有优势。

综上，链表适合元素数量不固定，需要经常增减节点的情况。

4、Queue

在Queue这种数据结构中，最先插入在元素将是最先被删除；反之最后插入的元素将最后被删除，因此队列又称为“先进先出”（FIFO—first in first out）的线性表。通过使用Enqueue和Dequeue这两个方法来实现对 Queue 的存取。



一些需要注意的地方：

先进先出的情景。

默认情况下，Queue的初始容量为32， 增长因子为2.0。

当使用Enqueue时，会判断队列的长度是否足够，若不足，则依据增长因子来增加容量，例如当为初始的2.0时，则队列容量增长2倍。

5、Stack

与Queue相对，当需要使用后进先出顺序（LIFO）的数据结构时，我们就需要用到Stack了。



一些需要注意的地方：

后进先出的情景。

默认容量为10。

使用pop和push来操作。

6、Dictionary<K,T>

提到字典就不得不说Hashtable哈希表以及Hashing（哈希，也有叫散列的），因为字典的实现方式就是哈希表的实现方式，只不过字典是类型安全的，也就是说当创建字典时，必须声明key和item的类型，这是第一条字典与哈希表的区别。关于哈希表的内容推荐看下这篇博客哈希表。关于哈希，简单的说就是一种将任意长度的消息压缩到某一固定长度，比如某学校的学生学号范围从0000099999，总共5位数字，若每个数字都对应一个索引的话，那么就是100000个索引，但是如果我们使用后3位作为索引，那么索引的范围就变成了000999了，当然会冲突的情况，这种情况就是哈希冲突(Hash Collisions)了。扯远了，关于具体的实现原理还是去看小匹夫推荐的那篇博客吧，当然那篇博客上面那个大大的转字也是蛮刺眼的。。。

回到Dictionary<K,T>，我们在对字典的操作中各种时间上的优势都享受到了，那么它的劣势到底在哪呢？对嘞，就是空间。以空间换时间，通过更多的内存开销来满足我们对速度的追求。在创建字典时，我们可以传入一个容量值，但实际使用的容量并非该值。而是使用“不小于该值的最小质数来作为它使用的实际容量，最小是3。”，当有了实际容量之后，并非直接实现索引，而是通过创建额外的2个数组来实现间接的索引，即int[] buckets和Entry[] entries两个数组（即buckets中保存的其实是entries数组的下标），这里就是第二条字典与哈希表的区别，还记得哈希冲突吗？对，第二个区别就是处理哈希冲突的策略是不同的！字典会采用额外的数据结构来处理哈希冲突，这就是刚才提到的数组之一buckets桶了，buckets的长度就是字典的真实长度，因为buckets就是字典每个位置的映射，然后buckets中的每个元素都是一个链表，用来存储相同哈希的元素，然后再分配存储空间。



因此，我们面临的情况就是，即便我们新建了一个空的字典，那么伴随而来的是2个长度为3的数组。所以当处理的数据不多时，还是慎重使用字典为好，很多情况下使用数组也是可以接受的。

综上所述：