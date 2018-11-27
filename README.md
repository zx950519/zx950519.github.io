## 输入问题
&emsp;&emsp;Codeforces著名世界级选手Petr大爷写的Java输入内部类:  
```
class InputReader {
    public BufferedReader reader;
    public StringTokenizer tokenizer;

    public InputReader(InputStream stream) {
        reader = new BufferedReader(new InputStreamReader(stream), 32768);
        tokenizer = null;
    }

    public String next() {
        while (tokenizer == null || !tokenizer.hasMoreTokens()) {
            try {
                tokenizer = new StringTokenizer(reader.readLine());
            } catch (IOException e) {
                throw new RuntimeException(e);
            }
        }
        return tokenizer.nextToken();
    }

    public int nextInt() {
        return Integer.parseInt(next());
    }

}
```

## 输出问题
&emsp;&emsp;文档：https://www.cs.colostate.edu/~cs160/.Summer16/resources/Java_printf_method_quick_reference.pdf  
&emsp;&emsp;常见输出格式：  
```
    // 定义一些变量，用来格式化输出。
    double d = 345.678;
    String s = "你好！";
    int i = 1234;
    // "%"表示进行格式化输出，"%"之后的内容为格式的定义。
    System.out.printf("%f", d);// "f"表示格式化输出浮点数。
    System.out.println();
    System.out.printf("%9.2f", d);// "9.2"中的9表示输出的长度，2表示小数点后的位数。
    System.out.println();
    System.out.printf("%+9.2f", d);// "+"表示输出的数带正负号。
    System.out.println();
    System.out.printf("%-9.4f", d);// "-"表示输出的数左对齐（默认为右对齐）。
    System.out.println();
    System.out.printf("%+-9.3f", d);// "+-"表示输出的数带正负号且左对齐。
    System.out.println();
    System.out.printf("%d", i);// "d"表示输出十进制整数。
    System.out.println();
    System.out.printf("%o", i);// "o"表示输出八进制整数。
    System.out.println();
    System.out.printf("%x", i);// "d"表示输出十六进制整数。
    System.out.println();
    System.out.printf("%#x", i);// "d"表示输出带有十六进制标志的整数。
    System.out.println();
    System.out.printf("%s", s);// "d"表示输出字符串。
    System.out.println();
    System.out.printf("输出一个浮点数：%f，一个整数：%d，一个字符串：%s", d, i, s);
    // 可以输出多个变量，注意顺序。
    System.out.println();
    System.out.printf("字符串：%2$s，%1$d的十六进制数：%1$#x", i, s);
```
```
345.678000
345.68
+345.68
345.6780 
+345.678 
1234
2322
4d2
0x4d2
你好！
输出一个浮点数：345.678000，一个整数：1234，一个字符串：你好！
字符串：你好！，1234的十六进制数：0x4d2
```

## JAVA-API
```
1.ArrayList去重：List<Integer> unipue = list.stream().distinct().collect(Collectors.toList());
2.判断二维数组是否为空:if(matrix==null||matrix.length==0||(matrix.length==1&&matrix[0].length==0))
3.数组深拷贝:Arrays.copyOf(a, a.length+5);//参数1:旧数组;参数2：新数组的长度
4.Map的降序&升序
    List<Map.Entry<Character, Integer>> list = new ArrayList<Map.Entry<Character, Integer>>(map.entrySet());
    Collections.sort(list,new Comparator<Map.Entry<Character, Integer>>() {
        public int compare(Entry<Character, Integer> o1,
                Entry<Character, Integer> o2) {
            return o2.getValue().compareTo(o1.getValue());
        }
    });
    for(Map.Entry<Character, Integer> mapping:list){}  
5.暴力去除字符串中的空格：String[] words = s.trim().split(" +");  
6.保留两位小数:http://blog.51cto.com/glblong/1312340  

```
## 精度问题
#### 问题分析  
&emsp;&emsp;丫的，前两天做题发现了这么个奇葩的问题，代码如下：  
```
    public class Main {

        public static void main(String[] args) throws Exception {
            System.out.println(0.2 + 0.1);
            System.out.println(0.4 - 0.3);
            System.out.println(0.1 * 0.2);
            System.out.println(0.6 / 0.2);
            System.out.println(4.1 - 1.1);
        }
        //    0.30000000000000004
        //    0.10000000000000003
        //    0.020000000000000004
        //    2.9999999999999996
        //    2.9999999999999996

    }
```  
&emsp;&emsp;尤其是最下面那组4.1-1.1，解题的一个步骤是要求两个数(double类型)差的绝对值，然后向下取整，我直接用Math.floor(Math.abs(4.1-1.1));
发现结果竟然等于2，后来经过网上搜集资料，发现Java在精度上是有问题的！  
理论解释  
&emsp;&emsp;借用《Effactive Java》书中的话，float和double类型的主要设计目标是为了科学计算和工程计算。他们执行二进制浮点运算，这是为了在广域数值范围上提供较为精确的快速近似计算而精心设计的。然而，它们没有提供完全精确的结果，所以不应该被用于要求精确结果的场合。但是，商业计算往往要求结果精确，这时候BigDecimal就派上大用场了。  
&emsp;&emsp;在学习计算机组成原理的时候我们应该都对浮点数有一定的了解，看下这个演示的例子就知道为啥会出现丢失精度的问题：https://www.zhihu.com/question/42024389  
解决办法  
&emsp;&emsp;使用Java中的BigDecimal，主要有如下几种构造方式：  
```
    1.public BigDecimal(double val)    //将double表示形式转换为BigDecimal, 不建议使用

    2.public BigDecimal(int val)　　   //将int表示形式转换成BigDecimal

    3.public BigDecimal(String val)　　//将String表示形式转换成BigDecimal
```  
&emsp;&emsp;为什么不推荐使用double构造BigDecimal对象呢，看下面的实例：  
```
    import java.math.BigDecimal;
    public class Main {

        public static void main(String[] args) throws Exception {
            BigDecimal bigDecimal = new BigDecimal(2);
            BigDecimal bDouble = new BigDecimal(2.3);
            BigDecimal bString = new BigDecimal("2.3");
            System.out.println("bigDecimal=" + bigDecimal);
            System.out.println("bDouble=" + bDouble);
            System.out.println("bString=" + bString);
        }
        //    bigDecimal=2
        //    bDouble=2.29999999999999982236431605997495353221893310546875
        //    bString=2.3

    }
```
&emsp;&emsp;为什么会出现这种情况呢？  
- 1.参数类型为double的构造方法的结果有一定的不可预知性。有人可能认为在Java中写入newBigDecimal(0.1)所创建的BigDecimal正好等于 0.1（非标度值 1，其标度为 1），但是它实际上等于0.1000000000000000055511151231257827021181583404541015625。这是因为0.1无法准确地表示为 double（或者说对于该情况，不能表示为任何有限长度的二进制小数）。这样，传入到构造方法的值不会正好等于 0.1（虽然表面上等于该值）。  
- 2.String 构造方法是完全可预知的：写入 newBigDecimal("0.1") 将创建一个 BigDecimal，它正好等于预期的 0.1。因此，比较而言，通常建议优先使用String构造方法。  
&emsp;&emsp;当必须使用double构造时，可以使用String进行过桥，避免丢失精度，例如：BigDecimal b1 = new BigDecimal(Double.toString(v1));  
加减乘除  
```
    public BigDecimal add(BigDecimal value);                        //加法

    public BigDecimal subtract(BigDecimal value);                   //减法 

    public BigDecimal multiply(BigDecimal value);                   //乘法

    public BigDecimal divide(BigDecimal value);                     //除法
```  
&emsp;&emsp;值得注意的是对于除法，如果不能整除，运行时会出现java.lang.ArithmeticException异常，解决办法是添加额外的参数：public BigDecimal divide(BigDecimal divisor, int scale, int roundingMode)。其中第一参数表示除数，第二个参数表示小数点后保留位数，第三个参数表示舍入模式，只有在作除法运算或四舍五入时才用到舍入模式，有下面这几种：  
- ROUND_CEILING        //向正无穷方向舍入
- ROUND_DOWN           //向零方向舍入
- ROUND_FLOOR          //向负无穷方向舍入
- ROUND_HALF_DOWN      //向（距离）最近的一边舍入，除非两边（的距离）是相等,如果是这样，向下舍入, 例如1.55 保留一位小数结果为1.5
- ROUND_HALF_EVEN      //向（距离）最近的一边舍入，除非两边（的距离）是相等,如果是这样，如果保留位数是奇数，使用ROUND_HALF_UP，如果是偶数，使用ROUND_HALF_DOWN
- ROUND_HALF_UP        //向（距离）最近的一边舍入，除非两边（的距离）是相等,如果是这样，向上舍入, 1.55保留一位小数结果为1.6
- ROUND_UNNECESSARY    //计算结果是精确的，不需要舍入模式
- ROUND_UP             //向远离0的方向舍入  
&emsp;&emsp;例如，常见的四舍五入为：System.out.println("a / b =" + a.divide(b, 10, BigDecimal.ROUND_HALF_UP));  
&emsp;&emsp;此外，四则运算后会产生新的对象，即BigDecimal对象不可变！  
截断操作  
&emsp;&emsp;需要对BigDecimal进行截断和四舍五入可用setScale方法，例如：  

```
    import java.math.*;
    import java.math.BigDecimal;
    public class Main {

        public static void main(String[] args) throws Exception {
            BigDecimal a = new BigDecimal("4.5635");
            a = a.setScale(3, RoundingMode.HALF_UP);    //保留3位小数，且四舍五入
            System.out.println(a);
        }

    }
```
常见变量  
```
    BigDecimal zero = BigDecimal.ZERO;  
    BigDecimal one = BigDecimal.ONE;  
    BigDecimal ten = BigDecimal.TEN; 
```  
比较操作  
```
    BigDecimal one = BigDecimal.valueOf(1);  
    BigDecimal two = BigDecimal.valueOf(2);  
    BigDecimal three = one.add(two);  
    int i1 = one.compareTo(two);//-1  
    int i2 = two.compareTo(two);//0  
    int i3 = three.compareTo(two);//1  
```  
完整API  
&emsp;&emsp;地址：http://www.javaweb.cc/help/JavaAPI1.6/java/math/class-use/BigDecimal.html  
&emsp;&emsp;地址：https://docs.oracle.com/javase/7/docs/api/java/math/BigDecimal.html
源码分析  
&emsp;&emsp;有待补充  
#### 解题模板  

```
class Arith{
    private static final int DEF_DIV_SCALE = 10; //这个类不能实例化
    private Arith(){
    }
    /**
     * 提供精确的加法运算。
     * @param v1 被加数
     * @param v2 加数
     * @return 两个参数的和
     */
    public static double add(double v1,double v2){
        BigDecimal b1 = new BigDecimal(Double.toString(v1));
        BigDecimal b2 = new BigDecimal(Double.toString(v2));
        return b1.add(b2).doubleValue();
    }
    /**
     * 提供精确的减法运算。
     * @param v1 被减数
     * @param v2 减数
     * @return 两个参数的差
     */
    public static double sub(double v1,double v2){
        BigDecimal b1 = new BigDecimal(Double.toString(v1));
        BigDecimal b2 = new BigDecimal(Double.toString(v2));
        return b1.subtract(b2).doubleValue();
    }
    /**
     * 提供精确的乘法运算。
     * @param v1 被乘数
     * @param v2 乘数
     * @return 两个参数的积
     */
    public static double mul(double v1,double v2){
        BigDecimal b1 = new BigDecimal(Double.toString(v1));
        BigDecimal b2 = new BigDecimal(Double.toString(v2));
        return b1.multiply(b2).doubleValue();
    }
    /**
     * 提供（相对）精确的除法运算，当发生除不尽的情况时，精确到
     * 小数点以后10位，以后的数字四舍五入。
     * @param v1 被除数
     * @param v2 除数
     * @return 两个参数的商
     */
    public static double div(double v1,double v2){
        return div(v1,v2,DEF_DIV_SCALE);
    }
    /**
     * 提供（相对）精确的除法运算。当发生除不尽的情况时，由scale参数指
     * 定精度，以后的数字四舍五入。
     * @param v1 被除数
     * @param v2 除数
     * @param scale 表示表示需要精确到小数点以后几位。
     * @return 两个参数的商
     */
    public static double div(double v1,double v2,int scale){
        if(scale<0){
            throw new IllegalArgumentException(
                    "The scale must be a positive integer or zero");
        }
        BigDecimal b1 = new BigDecimal(Double.toString(v1));
        BigDecimal b2 = new BigDecimal(Double.toString(v2));
        return b1.divide(b2,scale,BigDecimal.ROUND_HALF_UP).doubleValue();
    }
    /**
     * 提供精确的小数位四舍五入处理。
     * @param v 需要四舍五入的数字
     * @param scale 小数点后保留几位
     * @return 四舍五入后的结果
     */
    public static double round(double v,int scale){
        if(scale<0){
            throw new IllegalArgumentException("The scale must be a positive integer or zero");
        }
        BigDecimal b = new BigDecimal(Double.toString(v));
        BigDecimal one = new BigDecimal("1");
        return b.divide(one,scale,BigDecimal.ROUND_HALF_UP).doubleValue();
    }
}
```

## 优先队列、大顶堆、小顶堆
#### Top-K问题
```
import java.util.ArrayList;
import java.util.Collections;
import java.util.Comparator;
import java.util.Iterator;
import java.util.List;
import java.util.PriorityQueue;
import java.util.Random;

//固定容量的优先队列，模拟大顶堆，用于解决求topN小的问题
public class FixSizedPriorityQueue<E extends Comparable> {

    private PriorityQueue<E> queue; // 优先队列
    private int maxSize; // 堆的最大容量

    public FixSizedPriorityQueue(int maxSize) {
        if (maxSize <= 0)
            throw new IllegalArgumentException();
        this.maxSize = maxSize;
        this.queue = new PriorityQueue(maxSize, new Comparator<E>() {
            public int compare(E o1, E o2) {
                // 生成最大堆使用o2-o1,生成最小堆使用o1-o2, 并修改 e.compareTo(peek) 比较规则
                return (o2.compareTo(o1));
            }
        });
    }

    public void add_element(E e) {
        if (queue.size() < maxSize) { //未达到最大容量，直接添加
            queue.add(e);
        } else { // 队列已满
            E peek = queue.peek();
            if (e.compareTo(peek) < 0) { //将新元素与当前堆顶元素比较，保留较小的元素
                queue.poll();
                queue.add(e);
            }
        }
    }

    public List<E> sortedList() {
        List<E> list = new ArrayList<E>(queue);
        Collections.sort(list); // PriorityQueue本身的遍历是无序的，最终需要对队列中的元素进行排序
        return list;
    }

    public static void main(String[] args) {
        FixSizedPriorityQueue pq = new FixSizedPriorityQueue(10);
        // 生成数据
        Random random = new Random();
        int rNum = 0;
        for (int i = 1; i <= 100; i++) {
            rNum = random.nextInt(1000);
            pq.add_element(rNum);
        }

        System.out.println("PriorityQueue 本身的遍历是无序的：-----------------------------------");
        Iterable<Integer> iter = new Iterable<Integer>() {
            public Iterator<Integer> iterator() {
                return pq.queue.iterator();
            }
        };
        for (Integer item : iter) {
            System.out.print(item + ", ");
        }
        System.out.println();
        System.out.println("PriorityQueue 排序后的遍历：-----------------------------------");
        while (!pq.queue.isEmpty()) {
            System.out.print(pq.queue.poll() + ", ");
        }
    }
}
```

## 并查集
```
static int n = 20;
static int[] parent = new int[n];
static int[] rank = new int[n];

public static void init(int n) {
    for(int i=0; i<n; i++) {
        parent[i] = i;
        rank[i] = 0;
    }
}

public static int find(int x) {
    if(parent[x] == x) {
        return x;
    }
    else {
        return parent[x] = find(parent[x]);
    }
}

public static void unite(int x, int y) {
    x = find(x);
    y = find(y);
    if(x==y)
        return;
    if(rank[x] < rank[y]) {
        parent[x] = y;
    }
    else {
        parent[y] = x;
        if(rank[x]==rank[y])
            rank[x]++;
    }
}

public static boolean same(int x, int y) {
    return find(x) == find(y);
}
```
#### 朋友圈
&emsp;&emsp;链接：https://leetcode-cn.com/problems/friend-circles/description/  
```
class Solution {
    public int findCircleNum(int[][] M) {
        int count = M.length;
        int[] parent = new int[M.length];
        int[] rank = new int[M.length];
        for(int i=0; i<parent.length; i++) {
            parent[i] = i;
        }
        for(int i=0; i<parent.length; i++) {
            for(int j=0; j<=i; j++) {
                if(M[i][j] == 1) {
                    count = Union(parent, rank, i, j, count);
                }
            }
        }
        return count;
    }
    
    public int find(int[] parent, int x) {
        if(x!=parent[x]) {
            parent[x] = find(parent, parent[x]);
        }
        return parent[x];
    }
    
    public int Union(int[] parent, int[] rank, int x, int y, int count) {
        x = find(parent, x);
        y = find(parent, y);
        if(x==y)
            return count;
        if(rank[x] > rank[y])
            parent[y] = x;
        else {
            if(rank[x]==rank[y])
                rank[y]++;
            parent[x] = y;
        }
        count--;
        return count;
    }
    
}
```

## 矩阵问题
#### 像素翻转
&emsp;&emsp;题干：有一副由NxN矩阵表示的图像，这里每个像素用一个int表示，请编写一个算法，在不占用额外内存空间的情况下(即不使用缓存矩阵)，将图像顺时针旋转90度。给定一个NxN的矩阵，和矩阵的阶数N,请返回旋转后的NxN矩阵,保证N小于等于500，图像元素小于等于256。  

&emsp;&emsp;解法：先转置，再进行列交换(第一列跟最后一列交换，第二列跟倒数第二列交换...)
```
import java.util.*;

public class Transform {
    public int[][] transformImage(int[][] mat, int n) {
        // write code here
        for(int i=0; i<mat.length; i++) {
            for(int j=i; j<mat.length; j++) {
                int tmp = mat[i][j];
                mat[i][j] = mat[j][i];
                mat[j][i] = tmp;
            }
        }
        for(int i=0; i<n; i++) {
            for(int j=0; j<n/2; j++) {
                int tmp = mat[i][n-1-j];
                mat[i][n-1-j] = mat[i][j];
                mat[i][j] = tmp;
            }
        }
        return mat;
    }
}
```
&emsp;&emsp;推广：逆时针旋转90°，依旧先转置，在进行行交换
```
public static int[][] transformImage(int[][] mat, int n) {
        // write code here
        for(int i=0; i<mat.length; i++) {
            for(int j=i; j<mat.length; j++) {
                int tmp = mat[i][j];
                mat[i][j] = mat[j][i];
                mat[j][i] = tmp;
            }
        }
        for(int i=0; i<n/2; i++) {
            for(int j=0; j<n; j++) {
                int tmp = mat[n-1-i][j];
                mat[n-1-i][j] = mat[i][j];
                mat[i][j] = tmp;
            }
        }
        return mat;
    }
```
&emsp;&emsp;推广:逆时针旋转180°，顺时针旋转180°以及更高维度的旋转操作，由旋转90°的基本操作组合即可！  

## 类型转换
#### String转double
```
public static boolean isDouble(String data) {
        try{
            double tmp = Double.parseDouble(data);
            return true;
        }
        catch (Exception e) {

        }
        return false;
    }
```

#### ArrayList转int[]
```
class Transformation {

    // ArrayList转int[]
    public static int[] list2Ints(ArrayList<Integer> arrayList){
        return arrayList.stream().mapToInt(i->i).toArray();
    }

}
```
&emsp;&emsp;持续更新中  

## 位运算问题
&emsp;&emsp;一些使用位运算的奇淫技巧  
#### 判断奇偶
&emsp;&emsp;用if((a & 1) == 0)代替if (a % 2 == 0)  
#### 交换两数
```
    int a = 1, b = 2;
    a ^= b;
    b ^= a;
    a ^= b;
```
#### 取反
&emsp;&emsp;(~a+1)  
#### 取绝对值
```
    int i = a >> 31;
    System.out.println(i == 0 ? a : (~a + 1));
    或
    int j = a >> 31;
    System.out.println((a ^ j) - j);
```
#### 位运算枚举数组子集
```
public static ArrayList<ArrayList<Integer>> getSubset(ArrayList<Integer> L) {
    if (L.size() > 0) {
        ArrayList<ArrayList<Integer>> result = new ArrayList<>();
        for (int i = 0; i < Math.pow(2, L.size()); i++) {   // 集合子集个数=2的该集合长度的乘方
            ArrayList<Integer> subSet = new ArrayList<>();
            int index = i;// 索引从0一直到2的集合长度的乘方-1
            for (int j = 0; j < L.size(); j++) {
                // 通过逐一位移，判断索引那一位是1，如果是，再添加此项
                if ((index & 1) == 1) {// 位与运算，判断最后一位是否为1
                    subSet.add(L.get(j));
                }
                index >>= 1;// 索引右移一位
            }
            result.add(subSet); // 把子集存储起来
        }
        return result;
    } else {
        return null;
    }
}
```
#### 位运算枚举符合特定条件的数组子集
```
public static ArrayList<ArrayList<Integer>> getSubset(ArrayList<Integer> L, int target) {
    if (L.size() > 0) {
        ArrayList<ArrayList<Integer>> result = new ArrayList<>();
        for (int i = 0; i < Math.pow(2, L.size()); i++) {   // 集合子集个数=2的该集合长度的乘方
            ArrayList<Integer> subSet = new ArrayList<>();
            int index = i;// 索引从0一直到2的集合长度的乘方-1
            int sum = 0;
            for (int j = 0; j < L.size(); j++) {
                // 通过逐一位移，判断索引那一位是1，如果是，再添加此项
                if ((index & 1) == 1) {// 位与运算，判断最后一位是否为1
                    subSet.add(L.get(j));
                    sum += L.get(j);
                }
                index >>= 1;// 索引右移一位
            }
            if(sum == target)
                result.add(subSet); // 把子集存储起来
        }
        return result;
    } else {
        return null;
    }
}
```
#### 其他技巧
```
    // 1. 获得int型最大值
    System.out.println((1 << 31) - 1);// 2147483647， 由于优先级关系，括号不可省略
    System.out.println(~(1 << 31));// 2147483647
    // 2. 获得int型最小值
    System.out.println(1 << 31);
    System.out.println(1 << -1);
    // 3. 获得long类型的最大值
    System.out.println(((long)1 << 127) - 1);
    // 4. 乘以2运算
    System.out.println(10<<1);
    // 5. 除以2运算(负奇数的运算不可用)
    System.out.println(10>>1);
    // 6. 乘以2的m次方
    System.out.println(10<<2);
    // 7. 除以2的m次方
    System.out.println(16>>2);
    // 8. 判断一个数的奇偶性
    System.out.println((10 & 1) == 1);
    System.out.println((9 & 1) == 1);
    // 9. 不用临时变量交换两个数（面试常考）
    a ^= b;
    b ^= a;
    a ^= b;
    // 10. 取绝对值（某些机器上，效率比n>0 ? n:-n 高）
    int n = -1;
    System.out.println((n ^ (n >> 31)) - (n >> 31));
    /* n>>31 取得n的符号，若n为正数，n>>31等于0，若n为负数，n>>31等于-1
    若n为正数 n^0-0数不变，若n为负数n^-1 需要计算n和-1的补码，异或后再取补码，
    结果n变号并且绝对值减1，再减去-1就是绝对值 */
    // 11. 取两个数的最大值（某些机器上，效率比a>b ? a:b高）
    System.out.println(b&((a-b)>>31) | a&(~(a-b)>>31));
    // 12. 取两个数的最小值（某些机器上，效率比a>b ? b:a高）
    System.out.println(a&((a-b)>>31) | b&(~(a-b)>>31));
    // 13. 判断符号是否相同(true 表示 x和y有相同的符号， false表示x，y有相反的符号。)
    System.out.println((a ^ b) > 0);
    // 14. 计算2的n次方 n > 0
    System.out.println(2<<(n-1));
    // 15. 判断一个数n是不是2的幂
    System.out.println((n & (n - 1)) == 0);
    /*如果是2的幂，n一定是100... n-1就是1111....
    所以做与运算结果为0*/
    // 16. 求两个整数的平均值
    System.out.println((a+b) >> 1);
    // 17. 从低位到高位,取n的第m位
    int m = 2;
    System.out.println((n >> (m-1)) & 1);
    // 18. 从低位到高位.将n的第m位置为1
    System.out.println(n | (1<<(m-1)));
    /*将1左移m-1位找到第m位，得到000...1...000
    n在和这个数做或运算*/
    // 19. 从低位到高位,将n的第m位置为0
    System.out.println(n & ~(0<<(m-1)));
```

## 线段树
&emsp;&emsp;如果问题带有区间操作，或者可以转化成区间操作，可以尝试往线段树方向考虑。  
&emsp;&emsp;当我们分析出问题是一些列区间操作时：
-1.对区间的一个点的值进行修改
-2.对区间的一段值进行统一的修改
-3.询问区间的和
-4.询问区间的最大值、最小值
&emsp;&emsp;树型实现：  
```
public class Main {
    public static void main(String args[]) throws Exception{
        int[] A = {1, 4, 2, 3, 5, 7, 8, 8};
        SegmentTreeNode root = build(A);
        dfs(root);
        System.out.println(query_max(root, 1, 5));
    }
    public static void dfs(SegmentTreeNode root) {
        if(root==null)
            return;
        System.out.println("当前节点维护区间为:["+root.start+","+root.end+"]  " +
                "最大值为:"+root.max+" 最小值为:"+root.min+" 和为:"+root.sum +" 均值为:" + root.average);
        dfs(root.left);
        dfs(root.right);
    }
    public static SegmentTreeNode build(int[] A) {
        return buildhelper(0, A.length-1, A);
    }
    public static SegmentTreeNode buildhelper(int left, int right, int[] A) {
        if(left>right)
            return null;
        SegmentTreeNode root = new SegmentTreeNode(left, right, A[left]);
        if(left==right)
            return root;
        int mid = (left+right)/2;
        root.left = buildhelper(left, mid, A);
        root.right = buildhelper(mid+1, right, A);
        root.max = Math.max(root.left.max, root.right.max);
        root.min = Math.min(root.left.min, root.right.min);
        root.sum = root.left.sum + root.right.sum;
        root.average = (root.sum)*1.0/(root.end + 1 - root.start);
        return root;
    }
    public static int query_max(SegmentTreeNode root, int start, int end) {
        if(start<=root.start && root.end<=end) {
            return root.max;
        }
        int mid = (root.start + root.end) / 2; // 二分法划分区间
        int res = Integer.MIN_VALUE;
        if(start<=mid) {// 如果查询区间和左边节点区间有交集,则寻找查询区间在左边区间上的最大值
            res = Math.max(res, query_max(root.left, start, end));
        }
        if(mid+1<=end) {// 如果查询区间和右边节点区间有交集,则寻找查询区间在右边区间上的最大值
            res = Math.max(res, query_max(root.right, start, end));
        }
        return res;
    }
}
class SegmentTreeNode {
    public int start, end;
    public int max, min, sum;
    public double average = 0.0;
    public SegmentTreeNode left, right;
    public SegmentTreeNode(int start, int end, int basic) {
        this.start = start;
        this.end = end;
        this.max = basic;
        this.min = basic;
        this.sum = basic;
        this.left = null;
        this.right = null;
    }
}
```
&emsp;&emsp;数组实现  
```
public class Main {
    public static int[] seg;
    public static int n;
    public static void main(String args[]) throws Exception{
        int[] nums = {1, 4, 2, 3};
        n = nums.length;
        if(n>0) {
            seg = new int[4*n];
            build(nums, 0, n-1, 0);
        }
        update(2, 11);
        for(int i=0; i<seg.length; i++)
            System.out.print(seg[i]+" ");
        System.out.println();
        System.out.println(sum(2, 3));
    }
    public static void build(int[] nums, int start, int end, int idx) {
        if(start==end)
            seg[idx] = nums[start];
        else {
            int mid = (start+end) / 2;
            build(nums, start, mid, 2*idx+1);
            build(nums, mid+1, end, 2*idx+2);
            seg[idx] = seg[2*idx+1] + seg[2*idx+2];
        }
    }

    public static void update(int i, int val) {
        update(0, n-1, i, val, 0);
    }
    public static void update(int start, int end, int i, int val, int idx) {
        if(start==end) {
            seg[idx] = val;
            return;
        }
        int mid = (start+end)/2;
        if(i<=mid) {
            update(start, mid, i, val, 2*idx+1);
        }
        else {
            update(mid+1, end, i, val, 2*idx+2);
        }
        seg[idx] = seg[2*idx+1] + seg[2*idx+2];
    }
    public static int sum(int i, int j) {
        return sum(0, n-1, i, j, 0);
    }
    public static int sum(int start, int end, int i, int j, int idx) {
        if(start>j || end<i)
            return 0;
        if(i<=start && j>=end)
            return seg[idx];
        int mid = (start+end)/2;
        return sum(start, mid, i, j, 2*idx+1) + sum(mid+1, end, i, j, 2*idx+2);
    }
}
```
&emsp;&emsp;计算右侧小于当前元素的个数(https://leetcode-cn.com/problems/count-of-smaller-numbers-after-self/description/)  
```
class Solution {
    public List<Integer> countSmaller(int[] nums) {
        Integer[] result = new Integer[nums.length];
        TreeNode root = null;
        // 从右往左以此插入数字
        for(int i = nums.length - 1; i >= 0; i--) {
            root = insert(nums[i], root, result, i, 0);
        }
//        dfs(root);
        return Arrays.asList(result);
    }
    
    public static TreeNode insert(int num, TreeNode node, Integer[] result, int idx, int preSum) {
        // 创建节点
        if(node == null) {
            node = new TreeNode(num);
            node.count = 0;
            node.duplicates = 1;
            result[idx] = preSum;
        }
        // 更新节点
        else if (node.val == num) {
            node.duplicates++;
            result[idx] = preSum + node.count;
        }
        // 在左子树添加
        else if (node.val > num) {
            node.count++;
            node.left = insert(num, node.left, result, idx, preSum);
        }
        // 在右子树添加
        else {
            node.right = insert(num, node.right, result, idx, preSum + node.duplicates + node.count);
        }
        return node;
    }
}
class TreeNode {
    int val;
    int count; // **从左往右**，小于该val的数字的数目
    int duplicates; // 该val对应的数字的重复个数

    TreeNode left;
    TreeNode right;

    TreeNode(int val) {
        this.val = val;
    }
}
```

## 树状数组
&emsp;&emsp;数组的区间求和的复杂度是O(n)，树状数组可以将数组区间求和的复杂度降低到O(lg n)。这对于长数组的高频率区间求和的应用场景来讲，可以提高效率。  

```
public class Main {
    static int length = 4;
    static int[] tree = new int[length+1];
    public static void main(String args[]) throws Exception{
        put(1, 2);
        put(2, 3);
        put(3, 4);
        put(4, 5);
        System.out.println(sum(2, 4));
    }
    public static int sum(int index){
        if (index<1 && index>length) {
            throw new IllegalArgumentException("Out of Range!");
        }
        int sum=0;
        while (index>0) {
            sum+=tree[index];
            index -= lowBit(index);
        }
        return sum;
    }
    public static int sum(int start,int end) {
        return sum(end)-sum(start-1);
    }
    public static void put(int index,int value){
        if (index<1 && index>length) {
            throw new IllegalArgumentException("Out of Range!");
        }
        while (index<=length) {
            tree[index] += value;
            index += lowBit(index);
        }
    }
    public static int get(int index){
        if (index<1&&index>length) {
            throw new IllegalArgumentException("Out of Range!");
        }
        int sum=tree[index];
        int z=index-lowBit(index);
        index--;
        while (index!=z) {
            sum-=tree[index];
            index-=lowBit(index);
        }
        return sum;
    }
    public static int lowBit(int k){
        return k&-k;
    }
}
```
#### 冒泡排序的交换次数
```
public class Main {
    static int n = 0;
    static int MAX_N = 10000;
    static int[] dat = new int[1000000];
    static int[] a = new int[MAX_N];
    public static void main(String args[]) throws Exception{
        Scanner scanner = new Scanner(System.in);
        n = scanner.nextInt();
        for(int i=0; i<n; i++)
            a[i] = scanner.nextInt();
        System.out.println(opt());
    }
    public static int sum(int i) {
        int ans = 0;
        while(i>0) {
            ans += dat[i];
            i -= i& -i;
        }
        return ans;
    }
    public static void add(int i, int x) {
        while (i<=n) {
            dat[i]+=x;
            i+= i& -i;
        }
    }
    public static int opt() {
        int ans = 0;
        for(int i=0; i<n; i++) {
            System.out.println("当前:"+i+" sum():"+(i-sum(a[i])));
            ans += i-sum(a[i]);
            add(a[i], 1);
        }
        return ans;
    }
}
```

## 进制转换
&emsp;&emsp;任意进制间的转换，使用10进制进行桥接即可。利用了StringBuilder、取余等操作即可，实现过程如下：
```
import java.util.*;
import java.io.*;
import java.math.*;
public class Main{

    private static char[] array = "0123456789abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ".toCharArray();
    private static String numStr = "0123456789abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ";

    public static void main(String[] args) throws Exception{
        BufferedReader bufferedReader = new BufferedReader(new InputStreamReader(System.in));
        Scanner in = new Scanner(System.in);
        while(in.hasNext()){
            int from = in.nextInt();
            int to = in.nextInt();
            String input = in.next();
            // 任意进制转任意进制 使用10进制进行桥接
            System.out.println(getData(from, to, input));
        }

    }
    // 获取结果
    public static String getData(int from, int to, String data) {
        Long tmp_10= Hexadecimal_From_Random_To_10(data, from);
        String tmp_random = Hexadecimal_From_10_To_Random(tmp_10, to);
        String ans = tmp_random.replaceFirst("^0*", "");
        return ans;
    }
    // 10进制转任意进制
    public static String Hexadecimal_From_10_To_Random(long data, int N) {
        Long tmp = data;
        Stack<Character> stack = new Stack<Character>();
        StringBuilder result = new StringBuilder(0);
        while (tmp != 0) {
            stack.add(array[new Long((tmp % N)).intValue()]);
            tmp = tmp / N;
        }
        for (; !stack.isEmpty();) {
            result.append(stack.pop());
        }
        return result.length() == 0 ? "0":result.toString();
    }
    // 任意进制转10进制
    public static long Hexadecimal_From_Random_To_10(String number, int N) {
        char ch[] = number.toCharArray();int len = ch.length;
        long result = 0;
        if (N == 10){
            return Long.parseLong(number);
        }
        long base = 1;
        for (int i = len - 1; i >= 0; i--) {
            int index = numStr.indexOf(ch[i]);
            result += index * base;
            base *= N;
        }
        return result;
    }

}
```

## 字典Map的使用
- 1.HashMap
- 2.LinkedHashMap
- 3.TreeMap
&emsp;&emsp;HashMap最基本的Map，遍历时的顺序与插入顺序无关  
&emsp;&emsp;LinkedHashMap是特殊的一种Map，遍历时的顺序与插入顺序相同  
&emsp;&emsp;TreeMap默认是按照key的字典序升序排列，如果想降序的话按照如下操作:  
```
    Map<Integer, Integer> map = new TreeMap<Integer, Integer>(new Comparator<Integer>(){
        /*
         * int compare(Object o1, Object o2) 返回一个基本类型的整型，
         * 返回负数表示：o1 小于o2，
         * 返回0 表示：o1和o2相等，
         * 返回正数表示：o1大于o2。
         */
        public int compare(Integer a,Integer b){
            return b-a;
        }
    });
```
&emsp;&emsp;黑科技：Java8中引入了getOrDefault(key, 找不大时返回的值)方法。  

## 二分搜索
#### Java版lower_bound&upper_bound
```
public static int binary_search(int[] nums, int l, int r, int v) {
    while (l<r) {
        int m = (l+r)/2;
        if(nums[m]==v){
            return m;
        }
        else if(nums[m]>v) {
            r = m-1;
        }
        else {
            l = m+1;
        }
    }
    if(nums[l]==v)
        return l;
    else
        return -1;
}
public static int lower_Bound(int[] nums, int left, int right, int target){
    if(left>=nums.length)
        return -1;
    while(left<right){
        int mid = (right + left) / 2;
        if(mid==nums.length-1)
            break;
        if(nums[mid] >= target) {
            right = mid;
        }
        else if(nums[mid] < target) {
            left = mid+1;
        }
    }
    if(nums[left]==target)
        return left;
    else
        return -1;
}
public static int upper_Bound(int[] nums,int left,int right,int target){
    int tmp = lower_Bound(nums, 0, nums.length-1, target);
    int pian = 0;
    if(tmp==-1)
        return -1;
    else {
        for(int i=tmp+1; i<nums.length; i++) {
            if(nums[i]==target)
                pian++;
            else
                break;
        }
    }
    return tmp+pian;
}
```
#### 割绳子
&emsp;&emsp;有N条绳子，长度分别为Li,如果从其中割出K条相同长度的绳子，这K条绳子每条最长能有多长？  
&emsp;&emsp;挑战程序竞赛P141  
#### 最大化最小值(牛舍问题)
&emsp;&emsp;挑战程序竞赛P142  
#### 最大化平均值
&emsp;&emsp;挑战程序竞赛P143  

#### 折半枚举(双向搜索)
&emsp;&emsp;给定四个数列以及目标数，求和为零的组合数(挑战程序竞赛p160)  
```
static int[] A = {-45, -41, -36, -36, -32, 26};
static int[] B = {22, -27, 53, 30, -38, -54};
static int[] C = {42, 56, -37, -75, -10, -6};
static int[] D = {-16, 30, 77, -46, 62, 45};
static int n = 6;
static int[] CD = new int[n*n];
static Map<Integer, String> map = new LinkedHashMap<>();
static List<String> li = new ArrayList<>();
public static void solve() {
    for(int i=0; i<n; i++) {
        for(int j=0; j<n; j++) {
            CD[i*n+j] = C[i] + D[j];
            if(map.getOrDefault(C[i] + D[j], "").equals("")) {
                map.put(C[i] + D[j], i+","+j+";");
            }
            else {
                map.put(C[i] + D[j], map.get(C[i] + D[j])+i+","+j+";");
            }
        }
    }
    Arrays.sort(CD);
    long res = 0;
    for(int i=0; i<n; i++) {
        for (int j = 0; j < n; j++) {
            int cd = -(A[i] + B[j]);
            if(binary_search(CD, 0, CD.length, cd)==-1)
                continue;
            int up = upper_Bound(CD, 0, CD.length, cd);
            int low = lower_Bound(CD, 0, CD.length, cd);
            if(up-low==0) {
                res++;
                String tmp = map.get(cd);
                tmp+=i+","+j;
                li.add(tmp);
            }
            else {
                res+=up-low;
                String[] tmp = map.get(cd).split(";");
                for(int k=0; k<tmp.length; k++) {
                    String apk = tmp[i]  + i + "," + j;
                    li.add(apk);
                }
            }

        }
    }
    System.out.println(res);
    for(String sp: li){
        String[] half = sp.split(";");
        int c = Integer.parseInt(half[0].split(",")[0]);
        int d = Integer.parseInt(half[0].split(",")[1]);
        int a = Integer.parseInt(half[1].split(",")[0]);
        int b = Integer.parseInt(half[1].split(",")[1]);
        System.out.println(A[a]+","+B[b]+","+C[c]+","+D[d]+" =0");
    }
}
public static int binary_search(int[] nums, int l, int r, int v) {
    while (l<r) {
        int m = (l+r)/2;
        if(nums[m]==v){
            return m;
        }
        else if(nums[m]>v) {
            r = m-1;
        }
        else {
            l = m+1;
        }
    }
    if(nums[l]==v)
        return l;
    else
        return -1;
}
public static int lower_Bound(int[] nums,int l,int r,int v){
    while(l<r){
        int mid = (r + l) / 2;
        if(nums[mid] >= v)
            r = mid;
        else if(nums[mid] < v)
            l = mid+1;
    }
    if(nums[l]==v)
        return l;
    else
        return -1;
}
public static int upper_Bound(int[] nums,int l,int r,int v){
    while(l<r){
        int mid = (r + l) / 2;
        if(nums[mid] <= v)
            l = mid+1;
        else if(nums[mid] > v)
            r = mid;
    }
    if(nums[l-1]==v)
        return l-1;
    else
        return -1;
}
```
#### 三数之和
&emsp;&emsp;https://leetcode-cn.com/problems/3sum/description/  
```
class Solution {
    public List<List<Integer>> threeSum(int[] nums) {
        List<List<Integer>> res = new ArrayList<List<Integer>>();
        if (nums.length < 3) {
            return res;
        }
        Arrays.sort(nums);

        for (int i = 0; i < nums.length; i++) {
            //相同，跳过
            if (i > 0 && nums[i] == nums[i - 1]) {
                continue;
            }
            int l = i + 1;
            int r = nums.length - 1;
            while (l < r) {
                int sum = nums[i] + nums[l] + nums[r];
                if (sum == 0) {
                    res.add(Arrays.asList(nums[i], nums[l], nums[r]));
                   //相同，跳过
                    while (l < r && nums[l] == nums[l + 1]) l++;
                    while (l < r && nums[r] == nums[r - 1]) r--;
                    l++;
                    r--;
                } else if (sum < 0) {
                    l++;
                } else {
                    r--;
                }
            }
        }
        return res;
    }
}
```
#### 四数之和
&emsp;&emsp;https://leetcode-cn.com/problems/4sum/description/  
```
class Solution {
    public List<List<Integer>> fourSum(int[] nums, int target) {
        List<List<Integer>> res = new ArrayList<List<Integer>>();
        if (nums.length < 4) {
            return res;
        }
        Arrays.sort(nums);
        for (int i = 0; i < nums.length; i++) {
            if (i > 0 && nums[i] == nums[i - 1]) {
                continue;
            }
            for (int j = i + 1; j < nums.length; j++) {
                //第二个数的去重处理
                if (j > i + 1 && nums[j] == nums[j - 1]) {
                    continue;
                }
                int l = j + 1;
                int r = nums.length - 1;
                while (l < r) {
                    int sum = nums[i] + nums[j] + nums[l] + nums[r];
                    if (sum == target) {
                        res.add(Arrays.asList(nums[i], nums[j], nums[l], nums[r]));
                        while (l < r && nums[l] == nums[l + 1]) l++;
                        while (l < r && nums[r] == nums[r - 1]) r--;
                        l++;
                        r--;
                    } else if (sum < target) {
                        l++;
                    } else {
                        r--;
                    }
                }
            }
        }
        return res;
    }
}
```

## 数学问题
#### 辗转相除法
```
int gcd(int a, int b) {
    if(b==0)
        return a;
    return gcd(b, a%b);
}
```
#### 埃氏筛
```
public static int sieve(int n) {
    boolean[] is_prime = new boolean[n+1];
    prime = new int[n+1];
    int p = 0;
    for(int i=0; i<=n; i++)
        is_prime[i] = true;
    is_prime[0] = false;
    is_prime[1] = false;
    for(int i=2; i<=n; i++) {
        if(is_prime[i]) {
            prime[p++] = i;
            for(int j=2*i; j<=n; j+=i)
                is_prime[j] = false;
        }
    }
    for(int i=0; i<p; i++) {
        System.out.println(prime[i]);
    }
    return p;
}
```
#### 模运算性质&法则
&emsp;&emsp;参考：https://blog.csdn.net/m0_37154839/article/details/78263267
#### 拓展欧几里得算法
```
public class Main {
    public static void main(String[] args) {
        long[] da = extgcd(5, 7);
        System.out.println("x="+da[1]+",y="+da[2]+",gcd="+da[0]);
    }
    public static long[] extgcd(long a,long b){
        long ans;
        long[] res=new long[3];
        if(b==0) {
            res[0]=a;
            res[1]=1;
            res[2]=0;
            return res;
        }
        long [] temp=extgcd(b,a%b);
        ans = temp[0];
        res[0]=ans;
        res[1]=temp[2];
        res[2]=temp[1]-(a/b)*temp[2];
        return res;
    }
}
```
#### 线段上的格点数
&emsp;&emsp;求p1(x1, y1)与p2(x2, y2)之间的线段上除p1,p2外有多少个格点  
```
res = |x1-x2|与|y1-y2|最大公约数-1
```
#### 数列片段和
&emsp;&emsp;例如，给定数列{0.1, 0.2, 0.3, 0.4}，我们有(0.1) (0.1, 0.2) (0.1, 0.2, 0.3) (0.1, 0.2, 0.3, 0.4) (0.2) (0.2, 0.3) (0.2, 0.3, 0.4) (0.3) (0.3, 0.4) (0.4) 这10个片段。 给定正整数数列，求出全部片段包含的所有的数之和。如本例中10个片段总和是0.1 + 0.3 + 0.6 + 1.0 + 0.2 + 0.5 + 0.9 + 0.3 + 0.7 + 0.4 = 5.0。  
&emsp;&emsp;套路：Sum+=(double)(N-i)*(double)(i+1)*a[i];  
#### 简单约瑟夫问题
&emsp;&emsp;出列问题：令res = 0;for(int i=2;i<=n;i++){res = (res+偏移量)%i};System.out.println((res+1));  
#### 快速幂
&emsp;&emsp;给出三种求出快速幂的方案，原题为：https://leetcode-cn.com/problems/powx-n/description/  
```
class Solution {
    public double myPow(double x, int n) {
        
        // 暴力法，不解释
        //return Math.pow(x, n);
        
        if(n==0)
            return 1.0;
        if(x==0)
            return 0.0;
        
        // 迭代法
        // double res = 1.0;
        // double base = x;
        // int tmp = n;
        // while (tmp!=0) {
        //     if(tmp%2!=0)
        //         res *= base;
        //     base *= base;
        //     tmp /= 2;
        // }
        // return n <0 ? 1.0 / res : res;
        
        // 递归法
        if(n==0)
            return 1;
        double half = myPow(x, n/2);
        if(n%2==0)
            return half * half;
        else if(n>0)
            return half * half * x;
        else
            return half * half / x;
            
        //
        double res = 1.0;
        for (int i = n; i != 0; i /= 2) {
            if (i % 2 != 0) res *= x;
            x *= x;
        }
        return n < 0 ? 1 / res : res;
    }
}
```
#### 超级次方(快速幂取模)
```
class Solution {
    public int superPow(int a, int[] b) {
        int res = 1;
        for(int i=0; i<b.length; i++) {
            res = pow(res, 10)*pow(a, b[i])%1337;
        }
        return res;
    }
    int pow(int x, int n) {
        if(n==0)
            return 1;
        if(n==1)
            return x%1337;
        return (pow(x%1337, n/2)*pow(x%1337, n-n/2))%1337;
    }
}
```
#### 快速幂取模
&emsp;&emsp;计算a的b次方对c取模，算法正确性未探知  
```
    public static void main(String[] args) {
        Scanner in = new Scanner(System.in);
        int a = in.nextInt(), b = in.nextInt(), c = in.nextInt();
        int res = 1;
        a %= c;
        for (; b != 0; b /= 2) {
            if (b % 2 == 1)
                res = (res * a) % c;
            a = (a * a) % c;
        }
        System.out.println(res);
    }
```
#### 矩阵快速幂
```
public static long[][] mutiply(int k, int n, long[][] A){
    long [][] res = new long[n][n];
    for(int i = 0 ; i < res.length ;i++){
        for(int j = 0 ; j< res[i].length ;j++){
            if(i==j){
                res[i][j] = 1;
            }else{
                res[i][j] = 0;
            }
        }
    }
    // 求A^156,而156(10)=10011100(2) 也就有A^156=>(A^4)*(A^8)*(A^16)*(A^128)
    while(k!=0){
        if((k&1)==1)
            res = func(res,A);
        // 移位，参考上面156的例子
        k>>=1;
        A = func(A,A);
    }
    return res;
}
// 矩阵乘法
public static long[][] func(long[][] A,long[][] B){
    long res[][] = new long[A.length][B.length];
    for(int i = 0 ; i < res.length ;i++){
        for(int j = 0 ; j< res[i].length ;j++){
            for(int k = 0 ; k < A[0].length ;k++){
                res[i][j] += A[i][k]*B[k][j];
            }
        }
    }
    return res;
}
```
#### 斐波那契额矩阵快速幂
```
public class Main {
    public static void main(String[] args) {
        int n = 4;
        int[][] m = fb(n);
        System.out.println(m[0][1]);
    }
    // 关联矩阵
    private static final int[][] UNIT = { { 1, 1 }, { 1, 0 } };
    // 全0矩阵
    private static final int[][] ZERO = { { 0, 0 }, { 0, 0 } };
    public static int[][] fb(int n) {
        if (n == 0) {
            return ZERO;
        }
        if (n == 1) {
            return UNIT;
        }		
        // n是奇数
        if ((n & 1) == 0) {
            int[][] matrix = fb(n >> 1);
            return matrixMultiply(matrix, matrix);
        }		
        // n是偶数
        int[][] matrix = fb((n - 1) >> 1);
        return matrixMultiply(matrixMultiply(matrix, matrix), UNIT);
    }


    // 矩阵乘法
    public static int[][] matrixMultiply(int[][] m, int[][] n) {
        int rows = m.length;
        int cols = n[0].length;
        int[][] r = new int[rows][cols];
        for (int i = 0; i < rows; i++) {
            for (int j = 0; j < cols; j++) {
                r[i][j] = 0;
                for (int k = 0; k < m[i].length; k++) {
                    r[i][j] += m[i][k] * n[k][j];
                }
            }
        }
        return r;
    }
}
```
#### 递推公式对某个数是否能整除
&emsp;&emsp;多半是有套路的，比如x%y==z?  
#### 完全平方数
&emsp;&emsp;给定正整数 n，找到若干个完全平方数（比如 1, 4, 9, 16, ...）使得它们的和等于 n。你需要让组成和的完全平方数的个数最少。  
&emsp;&emsp;涉及四平方和定理  
```
class Solution {
    public int numSquares(int n) {
        while(n%4 == 0)
            n/=4;
        if(n%8 == 7)
            return 4;
        for(int i=0; i*i<=n; i++) {
            int j = (int)Math.sqrt(n- i*i);
            if(i*i + j*j==n){
                if(i>0&&j>0)
                    return 2;
                if(i>0||j>0)
                    return 1;
            }
        }
        return 3;
    }
}
```
#### 平方数之和
&emsp;&emsp;给定一个非负整数c，你要判断是否存在两个整数a和b，使得 a*a + b*b = c  
&emsp;&emsp;单循环，左右坐标试探加减即可  

```
class Solution {
    public boolean judgeSquareSum(int c) {
        int max = (int)Math.sqrt(c);
        if(max*max==c)
            return true;
        int min = 0;
        while(min<=max) {
            int tmp = min*min+max*max;
            if(tmp==c)
                return true;
            else if(tmp>c)
                max--;
            else
                min++;
        }
        return false;
    }
}
```
#### 两数相除
&emsp;&emsp;https://leetcode-cn.com/problems/divide-two-integers/description/  
```
public int divide(int dividend, int divisor) {
    boolean isP = true;
    if((dividend<0 && divisor>0) || (dividend>0 && divisor<0))
        isP = false;

    long ldividend = Math.abs((long) dividend);
    long ldivisor = Math.abs((long) divisor);
    if(ldivisor==0)
        return Integer.MAX_VALUE;
    if(ldividend==0 || ldividend<ldivisor)
        return 0;
    long result = divide(ldividend, ldivisor);
    if(result>Integer.MAX_VALUE){
        return isP? Integer.MAX_VALUE : Integer.MIN_VALUE;
    }
    return (int)(isP? result : -result);


}

public long divide(long ldividend, long ldivisor) {
    if(ldividend<ldivisor)
        return 0;
    long sum = ldivisor;
    long res = 1;
    while(ldividend >= (sum+sum)) {
        sum += sum;
        res += res;
    }
    return res + divide(ldividend-sum, ldivisor);
}
```
```
public int divide(int dividend, int divisor) {
        if(dividend > Integer.MAX_VALUE ||dividend < Integer.MIN_VALUE ||divisor > Integer.MAX_VALUE||divisor < Integer.MIN_VALUE){
            return Integer.MAX_VALUE;
        }
        if(divisor == 0){
            return Integer.MAX_VALUE;
        }
        if(dividend == Integer.MIN_VALUE){
            if(divisor == -1 || divisor == 1){
                return divisor == -1 ? ~dividend : dividend;
            }
        }
 
        int flag = 1;  
        long div=dividend, dor=divisor;  
        if (div<0) {  
            flag *= -1;  
            div *= -1;  
        }  
        if (dor<0){  
            flag *= -1;  
            dor *= -1;  
        }  
        int re = 0;  
        while (div >= dor){  
            long aa = 1, temp = dor;  
            while (temp < div>>1){  
                temp <<= 1;  
                aa <<= 1;  
            }  
            div -= temp ;  
            re += aa  ;  
        }  
        return re*flag;  
    }
```

## 日期问题
#### 给定年月日判断是星期几  
```
1.公元一年一月一日为星期一(现在世界各国通用一星期七天的制度。这个制度最早由君士坦丁大帝[Constantine the Great]制定。他在公元321年3月7日正式宣布7天为1周，这个制度一直沿用至今)。
2.算今天到公元一年一月一日有多少天，%7，一周7天，周而复始。
3.每年365天，365=52*7+1，所以，过一年，在算星期的时候，就相当于多了一天。
4.闰年多一天。过一个闰年，在3月及以后就要多加一天。
5.公元一年各月一日的星期：t0[] = {1, 4, 4, 0, 2, 5, 0, 3, 6, 1, 4, 6}。
6.本质：日期+月修正+年修正+闰年修正，模7，得到星期几。
7.我们看上面的常数组t[]，其实就是将t0对应的1、2月-1，其后的-2。这个-2，因为是相对公元一年一月一日，所以y-1、m-1和d-1，其中m-1体现在下标中，y和d合起来就是-2了。前两个月-1，是因为在算闰年的时候，将1、2月的y多减了1，在t中补上。即，t[]不仅仅是月份修正常数，而是一个年月日的综合修正常数。
8.以上这个代码，用常数组隐藏了一些算法细节，使得代码变得相当的帅。
```
```
// 由Tomohiko Sakamoto提供
    public static int dayofweek(int y, int m, int d) {
        int[] t = {0, 3, 2, 5, 0, 3, 5, 1, 4, 6, 2, 4};
        if(m < 3)
            y--;
        return (y + y/4 - y/100 + y/400 + t[m-1] + d) % 7;
    }
```

## 递推问题
#### 全错排公式
```
F(1) = 0;F(2) = 1; F(n) = (n-1)*(F(n-2) + F(n-1))//错排公式为F(n)=n!(1/2!-1/3!+…..+(-1)^n/n!)
```
#### 部分错排公式(新郎问题：n个人中m个错误的情况数量)
```
D(m, n) = F(m) * C(m, n);其中F(m)是全错排数量,C(m, n)是组合数格式为C(up, low)
```

## 数论问题
#### 组合数、排列数求解模板及基本公式变形
![](https://ws1.sinaimg.cn/large/005L0VzSgy1ftwvodb6eej30sg0hzwf6.jpg)  
&emsp;&emsp;其他性质、定理传送门

- 1.https://blog.csdn.net/litble/article/details/75913032
- 2.https://blog.csdn.net/w1y2s312138/article/details/70478078
- 3.https://blog.csdn.net/littlewhite520/article/details/71551123

```
    // 排列数
    public static int A(int up,int bellow) {
        int result=1;
        for(int i=up;i>0;i--) {
            result*=bellow;
            bellow--;
        }
        return result;
    }
    // 阶乘
    public static long jie(long n) {
        long ans = 1;
        for(int i=1; i<=n; i++)
            ans *= i;
        return ans;
    }
    // 组合数 C(up, low)
    public static long C(long m,long n) {
        return jie(n) / (jie(n-m)*jie(m));
    }
```

## 字符串问题
#### 一些基本操作
```
    去除空格:s.replaceAll(" ","");
    去除空格：String[] words = s.trim().split(" +");
    大小写转换:s.toLowerCase();s.toUpperCase();
    提取字母&数字:s.replaceAll("[^a-z^A-Z^0-9]", "");
    字符串逆序:new StringBuffer(s).reverse().toString();
    字符大小写转换:大->小 (char)(c+32);小->大 (char)(c-32);
    字符串单词翻转：String[] words = s.trim().split(" +");Collections.reverse(Arrays.asList(words));return String.join(" ", words);
```

#### 无重复字符的最长子串长度
&emsp;&emsp;算法运行一遍即可了解过程  
```
public class Main {
    public static void main(String[] args) throws Exception{
        Scanner scanner = new Scanner(System.in);
        String d = scanner.nextLine();
        System.out.println(fun(d));
    }
    public static int fun(String s){
        if(s == null || s.length() < 1)
            return 0;
        HashSet<Character> set = new HashSet<Character>();
        int left = 0;
        int max_str = Integer.MIN_VALUE;
        int right = 0;
        int len = s.length();
        while(right < len) {
            if(set.contains(s.charAt(right))) {
                if(right - left > max_str)
                    max_str = right - left;
                while(s.charAt(left) != s.charAt(right)) {
                    set.remove(s.charAt(left));
                    left++;
                }
                for(Character c: set){
                    System.out.print(c+" ");
                }
                System.out.println();
                left++;
                System.out.println(left + " -> " + right);
            }
            else {
                set.add(s.charAt(right));
                for(Character c: set){
                    System.out.print(c+" ");
                }
                System.out.println();
                System.out.println(left + " -> " + right);
            }
            right++;
        }
        max_str = Math.max(max_str, right - left);
        return max_str;
    }
}
```
#### 子串(indexOf)手动实现
&emsp;&emsp;https://leetcode-cn.com/problems/implement-strstr/description/  
```
class Solution {
    public int strStr(String haystack, String needle) {
        if(needle==null||needle.length()==0)
            return 0;    
        for(int i=0; i<haystack.length()-needle.length()+1; i++) {
            boolean flag = true;
            for(int j=0; j<needle.length(); j++) {
                if(haystack.charAt(i+j)!=needle.charAt(j)) {
                    flag = false;
                    break;
                }
            }
            if(flag==true)
                return i;
        }
        return -1;
    }
}
```

## 图论问题
#### 暴力凸包(n*n*n)
```
public class Main {
    public static void main(String args[]) throws Exception{
        Point[] A = new Point[8];
        A[0] = new Point(1,0);
        A[1] = new Point(0,1);
        A[2] = new Point(0,-1);
        A[3] = new Point(-1,0);
        A[4] = new Point(2,0);
        A[5] = new Point(0,2);
        A[6] = new Point(0,-2);
        A[7] = new Point(-2,0);
        Point[] result = getConvexPoint(A);
        System.out.println("集合A中满足凸包的点集为：");
        for(int i = 0;i < result.length;i++)
            System.out.println("("+result[i].getX()+","+result[i].getY()+")");
    }
    public static Point[] getConvexPoint(Point[] A){
        Point[] result = new Point[A.length];
        int len = 0;  //用于计算最终返回结果中是凸包中点的个数
        for(int i = 0;i < A.length;i++){
            for(int j = 0;j < A.length;j++){
                if(j == i)     //除去选中作为确定直线的第一个点
                    continue;
                int[] judge = new int[A.length];   //存放点到直线距离所使用判断公式的结果
                for(int k = 0;k < A.length;k++){
                    int a = A[j].getY() - A[i].getY();
                    int b = A[i].getX() - A[j].getX();
                    int c = (A[i].getX())*(A[j].getY()) - (A[i].getY())*(A[j].getX());
                    judge[k] = a*(A[k].getX()) + b*(A[k].getY()) - c;  //根据公式计算具体判断结果
                }
                if(JudgeArray(judge)){  // 如果点均在直线的一边,则相应的A[i]是凸包中的点
                    result[len++] = A[i];
                    break;
                }
            }
        }
        Point[] result1 = new Point[len];
        for(int m = 0;m < len;m++)
            result1[m] = result[m];
        return result1;
    }
    //判断数组中元素是否全部大于等于0或者小于等于0，如果是则返回true，否则返回false
    public static boolean JudgeArray(int[] Array){
        boolean judge = false;
        int len1 = 0, len2 = 0;
        for(int i = 0;i < Array.length;i++){
            if(Array[i] >= 0)
                len1++;
        }
        for(int j = 0;j < Array.length;j++){
            if(Array[j] <= 0)
                len2++;
        }
        if(len1 == Array.length || len2 == Array.length)
            judge = true;
        return judge;
    }
}
class Point {
    private int x;
    private int y;
    Point(){
        x = 0;
        y = 0;
    }
    Point(int x, int y){
        this.x = x;
        this.y = y;
    }
    public void setX(int x){
        this.x = x;
    }
    public int getX(){
        return x;
    }
    public void setY(int y){
        this.y = y;
    }
    public int getY(){
        return y;
    }
}
```
#### 遍历矩阵的每个点的路径长度
&emsp;&emsp;链接：https://www.nowcoder.com/questionTerminal/2c9e3a1f8a2a487ba399af97781bd0cb  
![](https://ws1.sinaimg.cn/large/005L0VzSgy1fty0an21p0j30ix0kg779.jpg)  
#### 二分图判断
&emsp;&emsp;链接：https://leetcode-cn.com/problems/is-graph-bipartite/description/  
![](http://ww1.sinaimg.cn/large/005L0VzSgy1fw27gh2viaj30tt0dvjs8.jpg)  
```
class Solution {
    int v; // 顶点数
    int[] color; // 顶点颜色
    public boolean isBipartite(int[][] graph) {
        v = graph.length;
        color = new int[v];
        for(int i=0; i<v; i++) {
            if(color[i]==0) {
                if(!dfs(i, 1, graph)) {
                    return false;
                }
            }
        }
        return true;
    }

    public boolean dfs(int v, int c, int[][] graph) {
        color[v] = c; // 把顶点v染成c色
        for(int i=0; i<graph[v].length; i++) {
            if(color[graph[v][i]] == c) // 相邻顶点同色返回false
                return false;
            if(color[graph[v][i]]==0 && !dfs(graph[v][i], -c, graph)) // 相邻顶点未被染色，染成相反颜色
                return false;
        }
        return true;
    }
}
```

## 朋友圈、观众分类、水坑、迷宫搜索类问题

&emsp;&emsp;https://leetcode-cn.com/problems/friend-circles/description/  

![](https://ws1.sinaimg.cn/large/005L0VzSgy1fv4r3gha09j30m00fhdgh.jpg)  

&emsp;&emsp;这种问题本质上可以用DFS/BFS解决，通常给定一个二维矩阵，我们可以利用一个一维矩阵visited[]进行记录。  

```
// 朋友圈问题的两种解法,观众、球迷分类本质上也是朋友圈的变种
class Solution {
    public int findCircleNum(int[][] M) {
        int[] visited = new int[M.length];
        int res = 0;
        for(int i=0; i<M.length; i++) {
            if(visited[i]==0) {
                opt(M, visited, i);
                res++;
            }
        }
        return res;
    }
    
    public static void opt(int[][] M, int[] visited, int i) {
        for(int j=0; j<M.length; j++) {
            if(M[i][j]==1 && visited[j]==0) {
                visited[j] = 1;
                opt(M, visited, j);
            }
        }
        return ;
    }
}

class Solution {
    public static Queue<Integer> q = new LinkedList<>();
    public int findCircleNum(int[][] M) {
        
        int[] visited = new int[M.length];
        int res = 0;
        for(int i=0; i<M.length; i++) {
            if(visited[i]==0) {
                opt(M, visited, i);
                res++;
            }
        }
        return res;
    }
    
    public static void opt(int[][] M, int[] visited, int i) {
        q.offer(i);
        visited[i] = 1;
        while (!q.isEmpty()) {
            int node = q.poll();
            for(int j=0; j<M.length; j++) {
                if(visited[j]==0 && M[node][j]==1) {
                    q.offer(j);
                    visited[j] = 1;
                }
            }
        }
    }
}
```

&emsp;&emsp;水坑问题(DFS)：  
![](https://ws1.sinaimg.cn/large/005L0VzSgy1fv4rg2bzhzj30k60fwwfs.jpg)  
&emsp;&emsp;迷宫最短路径问题(BFS):  
![](https://ws1.sinaimg.cn/large/005L0VzSgy1fv4rkf6nehj30jx09g3zf.jpg)  
![](https://ws1.sinaimg.cn/large/005L0VzSgy1fv4rkz5x11j30jw0hbq5a.jpg)  
&emsp;&emsp;岛屿问题(DFS):https://leetcode-cn.com/problems/number-of-islands/description/  
```
class Solution {
    char[][] data;
    public int numIslands(char[][] grid) {
        int res = 0;
        data = grid;
        for(int i=0; i<grid.length; i++) {
            for(int j=0; j<grid[0].length; j++) {
                if(grid[i][j]=='1') {
                    dfs(i, j);
                    res++;
                }
            }
        }
        return res;
    }
    public void dfs(int x, int y) {
        
        data[x][y] = '0';
        for(int i=0; i<2; i++) {
            for(int j=-1; j<3; j+=2) {
                int nx = x;
                int ny = y;
                if(i==0) {
                    nx += j;
                }
                else{
                    ny += j;
                }
                
                if(nx>=0 && ny>=0 && nx<data.length && ny<data[0].length && data[nx][ny]=='1'){
                    dfs(nx, ny);
                }
            }
        }
        return;
    }
}
```
&emsp;&emsp;岛屿问题改(求最大岛屿面积)(DFS):https://leetcode-cn.com/problems/max-area-of-island/description/  
```
class Solution {
    int opt = 0;
    public int maxAreaOfIsland(int[][] grid) {
        int res = 0;
        
        for(int i=0; i<grid.length; i++) {
            for(int j=0; j<grid[0].length; j++) {
                if(grid[i][j]==1) {
                    dfs(grid, i, j);
                    res = Math.max(opt, res);
                    opt = 0;
                }
            }
        }
        return res;
    }
    
    public void dfs(int[][] grid, int x, int y) {
        grid[x][y] = 0;
        opt++;
        for(int i=0; i<2; i++) {
            for(int j=-1; j<3; j+=2) {
                int nx = x;
                int ny = y;
                if(i==0) {
                    nx += j;
                }
                else{
                    ny += j;
                }
                if(nx>=0 && ny>=0 && nx<grid.length && ny<grid[0].length && grid[nx][ny]==1) {
                    dfs(grid, nx, ny);
                }
            }
        }
    }
}
```

## 贪心问题
#### 超大背包
&emsp;&emsp;挑战程序竞赛p162  

## DP问题

#### 不错的博客讲解
- https://blog.csdn.net/wangdd_199326/article/details/76464333  
#### 多重部分和
&emsp;&emsp;算法竞赛入门，p62(有n钟不同大小的数字ai,每种个，mi个,判断是否可以组合成指定数k)  
```
int n;//多少种数字
int k;//目标和
int a[N]//各个数字
int m[M]//每种数字的个数
boolean dp[n+1][k+1];
boolean opt() {
    dp[0][0] = true;
    for(int i=0; i<n; i++) {
        for(int j=0; j<=k; j++) {
            for(int p=0; p<=j&&p*a[i]<=j; p++) {
                dp[i+1][j] = dp[i+1][j] | dp[i][j-p*a[i]];
            }
        }
    }
    if(dp[n][k])
        return true;
    else
        return false;
}
```
#### 划分数
&emsp;&emsp;算法竞赛入门，p66(有n个无区别的物品，将它们划分成不超过m组，求出方法数模M的余数)  
```
int n, m;
int dp[n+1][m+1];
int opt() {
    dp[0][0] = 1;
    for(int i=1; i<=m; i++) {
        for(int j=0; j<=n; j++) {
            if(j>=i) {
                dp[i][j] = (dp[i-1][j] + dp[i][j-1]) % M;
            }
            else {
                dp[i][j] = dp[i-1][j];
            }
        }
    }
    return dp[m][n];
}
```
#### 最长上升子序列 
&emsp;&emsp;链接：https://leetcode-cn.com/problems/longest-increasing-subsequence/description/  
![](https://ws1.sinaimg.cn/large/005L0VzSgy1fubu8vgkf3j30o408aglw.jpg)  
&emsp;&emsp;经典的DP问题之一，解法很多：  
```
// O(n*n)的解法，逻辑很简单，需要注意的是dp[j]>dp[i]-1这步判断，如果不加，会重复加上一些不符合要求的数
class Solution {
    public int lengthOfLIS(int[] nums) {
        if(nums.length==0)
            return 0;
        if(nums.length==1)
            return 1;
        int[] dp = new int[nums.length];
        dp[0] = 1;
        int res = 0;
        for(int i=1; i<nums.length; i++) {
            dp[i] = 1;
            for(int j=0; j<i; j++) {
                if(nums[j]<nums[i]&&dp[j]>dp[i]-1) {
                    dp[i] = dp[j] + 1;
                }
                //if(nums[j]<nums[i]) {
                //    dp[i] = Math.max(res, dp[j]+1);
                //}
            }
            if(dp[i]>res)
                res = dp[i];
        }
        return res;
    }
}
```

```
// O(n*log(n))的解法
```
#### 最长公共子序列(LCS)
&emsp;&emsp;经典的DP问题之一,可以采用打表的方式或者递归解决，状态转移方程如下：  
![](https://ws1.sinaimg.cn/large/005L0VzSgy1fud189qec7j30j30j4jup.jpg)  
![](https://ws1.sinaimg.cn/large/005L0VzSgy1fud1f2ms99j30io02vjrv.jpg)  
&emsp;&emsp;链接：有待补充  
```
// 解法1采用打表法 printLCS方法利用矩阵print将实际的子序列依次打印出来
    public static int LCS(char[] a, char[] b) {
        int[][] dp = new int[a.length+1][b.length+1];
        int[][] print = new int[a.length+1][b.length+1];
        for(int i=0; i<a.length+1; i++)
            dp[i][0] = 0;
        for(int j=0; j<b.length+1; j++)
            dp[0][j] = 0;
        for(int i=1; i<a.length+1; i++) {
            for(int j=1; j<b.length+1; j++) {
                if(a[i-1]==b[j-1]) {
                    dp[i][j] = dp[i-1][j-1] + 1;
                    print[i][j] = 0;
                }
                else if(dp[i-1][j] >= dp[i][j-1]) {
                    dp[i][j] = dp[i-1][j];
                    print[i][j] = 1;
                }
                else {
                    dp[i][j] = dp[i][j-1];
                    print[i][j] = -1;
                }
            }
        }
        printLCS(print, a, a.length, b.length);
        System.out.println();
        return dp[a.length][b.length];
    }

    public static void printLCS(int[][] data, char[] x, int i, int j) {
        if(i==0 || j==0)
            return;
        if(data[i][j]==0) {
            printLCS(data, x, i-1, j-1);
            System.out.print(x[i-1]);
        }
        else if(data[i][j]==1) {
            printLCS(data, x, i-1, j);
        }
        else {
            printLCS(data, x, i, j-1);
        }
    }
```
```
// 解法2，利用递归求解
    public static int LCS2(char[] a, char[] b, int i, int j) {
        if(i>=a.length || j>=b.length)
            return 0;
        if(a[i]==b[j])
            return LCS2(a, b, i+1, j+1) + 1;
        else
            return Math.max(LCS2(a, b, i+1, j), LCS2(a, b, i, j+1));
    }
```
#### 不同路径
&emsp;&emsp;链接：https://leetcode-cn.com/problems/unique-paths/description/  
![](https://ws1.sinaimg.cn/large/005L0VzSgy1fu0cp794psj30no0idq3o.jpg)  
&emsp;&emsp;典型的数组型DP问题，开二维数组遍历即可，递推公式为:d[i][j] = data[i-1][j] + data[i][j-1];  
```
class Solution {
    public int uniquePaths(int m, int n) {
        if(m==1||n==1) 
            return 1;
        int[][] data = new int[m+1][n+1];
        for(int i=1; i<=m; i++) {
            for(int j=1; j<=n; j++) {
                if(i==1&&j==1) {
                    data[i][j] = 1;
                    continue;
                }
                data[i][j] = data[i-1][j] + data[i][j-1];
            }
        }
        return data[m][n];     
    }  
}
```
&emsp;&emsp;其他解法：实际上机器人总共走了m + n - 2步，其中m - 1步向下走，n - 1步向右走，那么总共不同的方法个数就相当于在步数里面m - 1和n - 1中较小的那个数的取法，实际上是一道组合数的问题。  
```
import java.util.*;

class Solution {
    public int uniquePaths(int m, int n) {
        if(m==1||n==1) 
            return 1;
        double num = 1, denom = 1;
        int small = m > n ? n : m;
        for (int i = 1; i <= small - 1; ++i) {
            num *= m + n - 1 - i;
            denom *= i;
        }
        return (int)(num / denom);
    }  
}
```
#### 三角形最小路径和
&emsp;&emsp;链接：https://leetcode-cn.com/problems/triangle/description/  
![](https://ws1.sinaimg.cn/large/005L0VzSgy1fu8d99k228j30o60aewev.jpg)  
&emsp;&emsp;矩阵类DP问题：  
```
class Solution {
    public int minimumTotal(List<List<Integer>> triangle) {
        for(int i=0; i<triangle.size(); i++) {
            if(i==0)
                continue;
            List<Integer> tmp = triangle.get(i);
            List<Integer> last = triangle.get(i-1);
            for(int j=0; j<tmp.size(); j++) {
                if(j==0) {
                    tmp.set(j, tmp.get(j)+last.get(0));
                    continue;
                }
                if(j==tmp.size()-1) {
                    tmp.set(j, tmp.get(j)+last.get(last.size()-1));
                    continue;
                }
                int min = Math.min(last.get(j), last.get(j-1));
                tmp.set(j, tmp.get(j)+min);
            }
        }
        List<Integer> last = triangle.get(triangle.size()-1);
        int min = Integer.MAX_VALUE;
        for(int i=0; i<last.size(); i++) 
            if(last.get(i) < min)
                min = last.get(i);
        return min;
    }
}
```
#### 最大正方形 
&emsp;&emsp;链接：https://leetcode-cn.com/problems/maximal-square/description/  
![](https://ws1.sinaimg.cn/large/005L0VzSgy1fu8fnlaay6j30of097aa6.jpg)  
```
class Solution {
    public static int maximalSquare(char[][] matrix) {
        if(matrix==null||matrix.length==0||(matrix.length==1&&matrix[0].length==0))
            return 0;
        if(matrix.length==1) {
            for(int i=0; i<matrix[0].length; i++) {
                if(matrix[0][i]=='1')
                    return 1;
            }
            return 0;
        }
        if(matrix[0].length==1) {
            for(int i=0; i<matrix.length; i++) {
                if(matrix[i][0]=='1')
                    return 1;
            }
            return 0;
        }
        int max = 0;
        for(int i=0; i<matrix.length; i++)
            for(int j=0; j<matrix[i].length; j++)
                if(matrix[i][j]=='1')
                    max = 1;
        for(int i=0; i<matrix.length; i++) {
            if(i==0)
                continue;
            for(int j=0; j<matrix[i].length; j++) {
                if(j==0) {
                    continue;
                }
                if(matrix[i][j]=='0') {
                    continue;
                }
                int zs = (int)matrix[i-1][j-1] - (int)('0');
                int ys = (int)matrix[i-1][j] - (int)('0');
                int zx = (int)matrix[i][j-1] - (int)('0');
                int yx = (int)matrix[i][j] - (int)('0');
                System.out.println(zx+" "+ys +" "+zx+" "+yx);
                if(zs>0&&ys>0&&zx>0) {
                    yx += Math.min(Math.min(zs, ys), zx);
                    if(yx > max)
                        max = yx;
                    matrix[i][j] = (char)(yx + '0');
                }
            }
        }
        if(max > 1)
            return max*max;
        else if(max ==1)
            return 1;
        else
            return 0;
    }
}
```
#### 最大子序和
&emsp;&emsp;链接：https://leetcode-cn.com/problems/maximum-subarray/description/  
![](http://ww1.sinaimg.cn/large/005L0VzSly1fu9f8gb932j30od07st90.jpg)  
&emsp;&emsp;一维数组的DP问题，状态转移方程为:f(x) = Max(f(x), f(x-1)+f(x))  
```
class Solution {
    public int maxSubArray(int[] nums) {
        if(nums.length==1)
            return nums[0];
        int max = Integer.MIN_VALUE;
        for(int i=1; i<nums.length; i++) {
            nums[i] = Math.max(nums[i], nums[i-1]+nums[i]);
        }
        for(int i=0; i<nums.length; i++) {
            if(nums[i]>max)
                max = nums[i];
        }
        return max;
    }
}
```
#### 最小路径和
&emsp;&emsp;链接:https://leetcode-cn.com/problems/minimum-path-sum/description/  
![](http://ww1.sinaimg.cn/large/005L0VzSly1fu9fqqw3hoj30o608ct8u.jpg)  
&emsp;&emsp;二维数组的DP问题，注意边界判断，状态转移方程为f[i][j] = f[i][j] + Math.min(f[i][j-1], f[i-1][j])  
```
class Solution {
    public int minPathSum(int[][] grid) {
        if(grid.length==1) {
            int sum = 0;
            int[] tmp = grid[0];
            for(int i=0; i<tmp.length; i++) {
                sum += tmp[i];
            }
            return sum;
        }
        if(grid[0].length==1) {
            int sum = 0;
            for(int i=0; i<grid.length; i++) {
                sum += grid[i][0];
            }
        }
        for(int i=0; i<grid.length; i++) {
            for(int j=0; j<grid[i].length; j++) {
                if(i==0&&j==0)
                    continue;
                if(i==0&&j!=0){
                    grid[i][j] += grid[i][j-1];
                    continue;
                }
                if(j==0&&i!=0) {
                    grid[i][j] += grid[i-1][j];
                    continue;
                }
                int tmp = Math.min(grid[i][j-1], grid[i-1][j]);
                grid[i][j] += tmp;
            }
        }
        return grid[grid.length-1][grid[0].length-1];
    }
}
```
#### 使用最小花费爬楼梯
&emsp;&emsp;链接：https://leetcode-cn.com/problems/min-cost-climbing-stairs/description/  
![](http://ww1.sinaimg.cn/large/005L0VzSly1fu9gjwa83zj30o40cd74z.jpg)  
&emsp;&emsp;一维数组DP问题，有点小坑。值得注意的是，需要新开一个长度比cost大1的新数组dp，令dp[0]=cost[0],dp[1]=cost[1],然后运行状态转移方程dp[i]=cost[i]+min(dp[i-1], dp[i-2]),此外需要额外注意的是cost[cost.length]=0。  
```
class Solution {
    public int minCostClimbingStairs(int[] cost) {
        if(cost.length<2)
            return cost[0];
        if(cost.length==2)
            return Math.min(cost[0], cost[1]);
        int[] dp = new int[cost.length+1];
        dp[0] = cost[0];
        dp[1] = cost[1];
        for(int i=2; i<=cost.length; i++) {
            int tmp = 0;
            if(i<cost.length)
                tmp = cost[i];
            dp[i] = tmp + Math.min(dp[i-1], dp[i-2]);
        }
        return dp[cost.length];
    }
}
```
#### 打家劫舍
&emsp;&emsp;链接：https://leetcode-cn.com/problems/house-robber/description/  
![](http://ww1.sinaimg.cn/large/005L0VzSly1fu9hutgtnvj30o60as0t9.jpg)  
&emsp;&emsp;一维数组DP问题，坑比较多。需要额外对数组长度为3及其以下的情况剪枝，当长度为4以上才可以进行正常的迭代。新开一个数组dp，状态转移方程为：dp[i] = nums[i]+max(dp[i-2], dp[i-3])  
```
class Solution {
    public int rob(int[] nums) {
        if(nums.length==0)
            return 0;
        if(nums.length==1)
            return nums[0];
        if(nums.length==2)
            return Math.max(nums[0], nums[1]);
        if(nums.length==3)
            return Math.max(nums[0]+nums[2], nums[1]);
        int[] dp = new int[nums.length];
        dp[0] = nums[0];
        dp[1] = nums[1];
        dp[2] = Math.max(nums[0]+nums[2], nums[1]);
        int res = dp[2];
        for(int i=3; i<nums.length; i++) {
            dp[i] = nums[i] + Math.max(dp[i-2], dp[i-3]);
            if(dp[i]>res)
                res = dp[i];
        }
        return res;
    }
}
```
#### 整数拆分
&emsp;&emsp;链接：https://leetcode-cn.com/problems/integer-break/description/  
![](http://ww1.sinaimg.cn/large/005L0VzSly1fu9inrre5bj30oh08q3yu.jpg)  
&emsp;&emsp;一维数组DP问题，注意2、3的特殊情况，从4开始可以开始迭代。状态转移方程形式不同，参考代码即可。  
```
class Solution {
    public int integerBreak(int n) {
        if(n==2)
            return 1;
        if(n==3)
            return 2;
        if(n==4)
            return 4;
        int[] dp = new int[n+1];
        dp[0] = 0;
        dp[1] = 1;
        dp[2] = 2;
        dp[3] = 3;
        for(int i=4; i<=n; i++) {
            int tmp = Integer.MIN_VALUE;
            for(int j=1; j<=(int)(i/2); j++) {
                if(dp[j]*dp[i-j]>tmp)
                    tmp = dp[j]*dp[i-j];
            }
            dp[i] = tmp;
        }
        return dp[n];
    }
}
```
#### 买卖股票的最佳时机
&emsp;&emsp;链接：https://leetcode-cn.com/problems/best-time-to-buy-and-sell-stock/description/  
![](https://ws1.sinaimg.cn/large/005L0VzSgy1fuannswa68j30o10abaaf.jpg)  
&emsp;&emsp;一维数组DP问题，可用暴力可解决。设一个变量min保存最低值，另一个变量res保存当前最优差值。  
```
class Solution {
    public int maxProfit(int[] prices) {
        if(prices.length==0)
            return 0;
        if(prices.length==1)
            return 0;
        if(prices.length==2&&prices[0]>=prices[1])
            return 0;
        // int[] dp = new int[prices.length];
        // int res = Integer.MIN_VALUE;
        // for(int i=1; i<prices.length; i++) {
        //     int tmp = Integer.MAX_VALUE;
        //     for(int j=0; j<i; j++) {
        //         if(prices[j]<tmp)
        //             tmp = prices[j];
        //     }
        //     if(tmp>=prices[i])
        //         dp[i] = 0;
        //     else
        //         dp[i] = prices[i] - tmp;
        //     if(dp[i]>res)
        //         res = dp[i];
        // }
        // return res;
        
        int min = Integer.MAX_VALUE;
        int res = 0;
        for(int i=0; i<prices.length; i++) {
            min = Math.min(min, prices[i]);
            res = Math.max(res, prices[i] - min);
        }
        return res;
    }
}
```
#### 完全平方数
&emsp;&emsp;链接:https://leetcode-cn.com/problems/perfect-squares/description/  
![](https://ws1.sinaimg.cn/large/005L0VzSgy1fvamcptsb9j30q909c0sx.jpg)  
```
class Solution {
    public int numSquares(int n) {
        int[] dp = new int[n+1];
        for(int i=1; i<=n; i++)
            dp[i] = Integer.MAX_VALUE;
        dp[0] = 0;
        for(int i=0; i<=n; i++) {
            for(int j=1; i+j*j<=n; j++) {
                dp[i+j*j] = Math.min(dp[i+j*j], dp[i]+1);
            }
        }
        return dp[n];
    }
}
```
## 二叉树问题
#### 如何判断二叉树b是否是二叉树a的子树
```
public boolean isSubTree(TreeNode root1,TreeNode root2) {
    if(root2 == null)
        return true;
    if(root1==null && root2!=null)
        return false;
    if(root1.val != root2.val)
        return false;
    else
        return isSubTree(root1.left, root2.left) && isSubTree(root1.right, root2.right);
}
```
#### 求二叉树深度
```
import java.util.*;
import java.math.*;
public class Solution {
    public int TreeDepth(TreeNode root) {
        if (root == null)
            return 0;
        return 1 + Math.max(TreeDepth(root.left), TreeDepth(root.right));
    }
}
public class Solution {
    public int TreeDepth(TreeNode root) {
            if(root==null){
                return 0;
        }          
        int nLelt=TreeDepth(root.left);
        int nRight=TreeDepth(root.right);     
        return nLelt>nRight?(nLelt+1):(nRight+1);
    }
}
```
#### 判断二叉树是否对称
题目描述
请实现一个函数，用来判断一颗二叉树是不是对称的。注意，如果一个二叉树同此二叉树的镜像是同样的，定义其为对称的。
```
public class Solution {
    boolean isSymmetrical(TreeNode pRoot) {
        if (pRoot == null)
            return true;
        return func(pRoot.left, pRoot.right);
    }
    boolean func (TreeNode r1, TreeNode r2) {
        
        if(r1==null && r2==null)
            return true;
        if(r1==null || r2==null)
            return false;
        return r1.val==r2.val && func(r1.left, r2.right) && func(r1.right, r2.left);
    }
}
```
#### 二叉树中序遍历的下一个结点
题目描述
给定一个二叉树和其中的一个结点，请找出中序遍历顺序的下一个结点并且返回。注意，树中的结点不仅包含左右子结点，同时包含指向父结点的指针。
```
解释：
节点可以分成两大类：
1、有右子树的，那么下个结点就是右子树最左边的点
2、没有右子树的，也可以分成两类：
    a)是父节点左孩子 ，那么父节点就是下一个节点
    b)是父节点的右孩子找他的父节点的父节点的父节点...直到当前结点是其父节点的左孩子位置。如果没有，那么他就是尾节点。
```
```
public class Solution {
    public TreeLinkNode GetNext(TreeLinkNode pNode) {
        if (pNode == null) 
            return null;
        if (pNode.right!=null) {
            return visit(pNode.right);
        }
        while (pNode.next != null) {
            if (pNode.next.left == pNode) 
                return pNode.next;
            pNode = pNode.next;
        }
        return null;
    }
    
    public TreeLinkNode visit(TreeLinkNode pNode) {
        if (pNode.left != null)
            pNode = pNode.left;
        return pNode;
    }
}
```
#### 二叉搜索树的后序遍历序列
题目描述
输入一个整数数组，判断该数组是不是某二叉搜索树的后序遍历的结果。如果是则输出Yes,否则输出No。假设输入的数组的任意两个数字都互不相同。
```
public class Solution {
    public boolean VerifySquenceOfBST(int [] sequence) {
        if(sequence.length==0) {
            return false;
        }
        return func(sequence, 0, sequence.length-1);
    }
    
    public boolean func(int [] data, int start, int end) {
        if(end <= start) 
            return true;
        int i=start;
        for(; i<end; i++) {
            if(data[i] > data[end]) 
                break;
        }
        for(int j=i; j<end; j++) {
            if(data[j] < data[end]) 
                return false;
        }
        return func(data, start, i-1) && func(data, i, end-1);
    }
}
```
#### 二叉树中和为某一值的路径
题目描述
输入一颗二叉树和一个整数，打印出二叉树中结点值的和为输入整数的所有路径。路径定义为从树的根结点开始往下一直到叶结点所经过的结点形成一条路径。
```
public class Solution {
    ArrayList<ArrayList<Integer>> data = new ArrayList<ArrayList<Integer>>();
    ArrayList<Integer> tmp = new ArrayList<Integer>();
    public ArrayList<ArrayList<Integer>> FindPath(TreeNode root,int target) {
        if(root==null) {
            return new ArrayList<ArrayList<Integer>>();
        }
        tmp.add(root.val);
        target -= root.val;
        if(target==0 && root.left==null && root.right==null) 
            data.add(new ArrayList<Integer>(tmp));
        FindPath(root.left, target);
        FindPath(root.right, target);
        tmp.remove(tmp.size()-1);
        return data;
    }
}
```
#### 判断二叉树是否是镜像
```
class Solution {
    
    public boolean isSymmetric(TreeNode root) {
        if(root==null)
            return true;
        return judge(root.left, root.right);
    }
    
    public boolean judge(TreeNode left, TreeNode right){
        if (left == null && right == null) {
            return true;
        }
        if (left != null && right == null) {
            return false;
        }
        if (left == null && right != null) {
            return false;
        }
        if (left.val != right.val) {
            return false;
        }
        return judge(left.left, right.right) && judge(left.right, right.left);
    }
    
}
```
#### 获取多叉树的深度
&emsp;&emsp;https://leetcode-cn.com/problems/maximum-depth-of-n-ary-tree/description/  
```
class Solution {
    int res = 0;
    public int maxDepth(Node root) {
        if(root==null)
            return 0;
        if(root!=null&&root.children.size()==0)
            return 1;
        opt(root, 1);
        return res;
    }
    
    public void opt(Node root, int depth) {
        if(root==null)
            return;
        for(int i=0; i<root.children.size(); i++) {
            Node tmp = root.children.get(i);
            if(tmp!=null)
                res = Math.max(depth+1, res);
            opt(tmp, depth+1);

        }
    }
}
```
#### 叶子相似的树
&emsp;&emsp;https://leetcode-cn.com/problems/leaf-similar-trees/description/  
```
class Solution {
    Queue<Integer> q1 = new LinkedList<>();
    Queue<Integer> q2 = new LinkedList<>();
    public boolean leafSimilar(TreeNode root1, TreeNode root2) {
        judge(root1, 1);
        judge(root2, 0);
        while(!q1.isEmpty()){
            int t1 = q1.poll();
            int t2 = q2.poll();
            if(t1!=t2)
                return false;
        }
        return true;
    }
    public void judge(TreeNode root, int flag) {
        if(root.left==null&&root.right==null)
            if(flag==1)
                q1.offer(root.val);
            else
                q2.offer(root.val);
        else {
            if(root.left!=null)
                judge(root.left, flag);
            if(root.right!=null)
                judge(root.right, flag);
        }
    }
}
```
####  二叉树的右视图
&emsp;&emsp;https://leetcode-cn.com/problems/binary-tree-right-side-view/description/  
```
class Solution {
    Queue<TreeNode> q1 = new LinkedList<>();
    Queue<TreeNode> q2 = new LinkedList<>();
    public List<Integer> rightSideView(TreeNode root) {
        List<Integer> res = new ArrayList<>();
        if(root==null)
            return new ArrayList<>();
        if(root.left==null&&root.right==null) {
            res.add(root.val);
            return res;
        }
        q1.offer(root);
        while(!q1.isEmpty()||!q2.isEmpty()) {
            int last = 0;
            if(q1.isEmpty()==true) {
                while(!q2.isEmpty()) {
                    TreeNode tmp = q2.poll();
                    if(tmp.left!=null)
                        q1.offer(tmp.left);
                    if(tmp.right!=null)
                        q1.offer(tmp.right);
                    last = tmp.val;
                }
            }
            else{
                while(!q1.isEmpty()) {
                    TreeNode tmp = q1.poll();
                    if(tmp.left!=null)
                        q2.offer(tmp.left);
                    if(tmp.right!=null)
                        q2.offer(tmp.right);
                    last = tmp.val;
                }
            }
            res.add(last);
        }
        return res;   
    }
}
```
#### 递增顺序查找树
&emsp;&emsp;https://leetcode-cn.com/problems/increasing-order-search-tree/description/  
```
class Solution {
    List<Integer> data = new ArrayList<>();
    public TreeNode increasingBST(TreeNode root) {
        if(root==null)
            return null;
        if(root.left==null&root.right==null)
            return root;
        opt(root);
        TreeNode tn = new TreeNode(data.get(0));
        TreeNode tmp = tn;
        tn.left = null;
        tn.right = null;
        for(int i=1; i<data.size(); i++) {
            TreeNode n = new TreeNode(data.get(i));
            n.left = null;
            n.right = null;
            tmp.right = n;
            tmp = n;
        }
        return tn;
    }
    
    public void opt(TreeNode root) {
        if(root==null)
            return;
        opt(root.left);
        data.add(root.val);
        opt(root.right);
    }
}
```
#### 判断是否是平衡二叉树
&emsp;&emsp;https://leetcode-cn.com/problems/balanced-binary-tree/description/  

```
class Solution {
    public boolean isBalanced(TreeNode root) {
        if(root==null)
            return true;
        if(Math.abs(depth(root.left)-depth(root.right))>1)
            return false;
        else
            return isBalanced(root.left)&&isBalanced(root.right);
    }
    public int depth(TreeNode root) {
        if(root==null)
            return 0;
        return 1 + Math.max(depth(root.left), depth(root.right));
    }
}
```
#### 二叉树路径问题
&emsp;&emsp;https://leetcode-cn.com/problems/path-sum/description/  
```
class Solution {
    public boolean hasPathSum(TreeNode root, int sum) {
        if(root==null)
            return false;
        else if(root.left==null&&root.right==null&&sum==root.val){
                return true;
        }
        else{
            return hasPathSum(root.left,sum-root.val) || hasPathSum(root.right,sum-root.val);
        }
    }
}
```
&emsp;&emsp;https://leetcode-cn.com/problems/path-sum-ii/description/  
```
class Solution {
    List<List<Integer>> data = new ArrayList<>();
    List<Integer> tmp = new ArrayList<>();
    public List<List<Integer>> pathSum(TreeNode root, int sum) {
        dfs(root, sum);
        return data;
    }
    
    public void dfs(TreeNode root, int res) {
        if(root==null)
            return;
        if(root.left==null&&root.right==null) {
            if(res-root.val==0){
                tmp.add(root.val);
                data.add(new ArrayList(tmp));
                tmp.remove(tmp.size()-1);
            }
            return;
        }
        else{
            tmp.add(root.val);
            dfs(root.left, res-root.val);
            dfs(root.right, res-root.val);
            tmp.remove(tmp.size()-1);
        }
    }
}
```
#### 根据二叉树创建字符串
&emsp;&emsp;https://leetcode-cn.com/problems/construct-string-from-binary-tree/description/  
```
class Solution {
    public String tree2str(TreeNode t) {
        return dfs(t);
    }
    public String dfs(TreeNode t) {
        if(t==null) {
            return "";
        }
        String sb = "";
        sb += t.val;
        if(t.left!=null && t.right!=null)
            sb +="("+dfs(t.left)+")"+"("+dfs(t.right)+")";
        if(t.left==null && t.right!=null)
            sb += "()"+"("+dfs(t.right)+")";
        if(t.left!=null && t.right==null)
            sb += "("+dfs(t.left)+")";
        return sb;
    }
}
```
## 栈问题
#### 火车扳倒叉问题
&emsp;&emsp;大量参考：https://blog.csdn.net/mate_ge/article/details/51006724  
&emsp;&emsp;可以判断给定序列是否合理、求可行序列数量有待用卡特兰数优化  
```
public class Main {
    static Map<String, Integer> map = new HashMap<>();
    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);
        /*输入入栈元素的个数*/
        int n = scanner.nextInt();
        int[] sequence = new int[n];
        /*按顺序输入入栈元素的编号*/
        for (int i=0; i<n; i++){
            sequence[i] = scanner.nextInt();
        }
        /*按顺序输入待测序列*/
        int[] inp = new int[n];
        for (int i=0; i<n; i++){
            inp[i] = scanner.nextInt();
        }
        /*没有出栈的压栈到stack栈中，输出的时候逆序输出*/
        Stack<Integer> stack = new Stack<Integer>();
        /*出栈的元素压栈到queue栈中，输出的时候正序输出*/
        Stack<Integer> queue = new Stack<Integer>();
        /**1、初始状态有stack、queue两个栈，可以进行两个操作，一个是sequence中的元素入栈stack，对应in函数，
         *    一个是出栈操作，stack中的元素出栈（stack中元素不为空），压到queue栈中，第一个操作执行完了之后执行第二个。
         * *2、每次执行读入元素或者元素出栈之后都可以重复刚才所说操作，继续读入数据，然后出栈。
         * *3、当读取到sequence最后一个元素的时候，先正序输出queue中的元素，再逆序输出stack中的元素，
         *     然后回退。
         * *4、回退的时候要把原来的操作撤销，比如读入数据的stack.push(sequence[nextPosition]);
         *    in函数执行完了之后要撤销就是stack.pop();
         *    对应的out函数也是：queue.push(stack.pop())对应：
         * if (!queue.isEmpty()) stack.push(queue.pop());
         * */
        next(stack,sequence,0,queue);
        String judeg = "";
        for(int i=0; i<n; i++) {
            if(i!=0)
                judeg += " ";
            judeg += inp[i];
        }
        if(map.getOrDefault(judeg, 0)==0)
            System.out.println("No");
        else
            System.out.println("yes");
    }
    public static void in(Stack<Integer> stack, int[] sequence, int nextPosition, Stack<Integer> queue){
        // 判断是否到最后队列满
        if (nextPosition == sequence.length) {
            String result = "";
            for(int i=0; i< queue.size(); i++){
                result+=queue.elementAt(i)+" ";
            }
            for (int i=stack.size()-1; i>=0; i--){
                result+=stack.elementAt(i)+" ";
            }
//            System.out.println(result.substring(0,result.length()-1));
            map.put(result.substring(0,result.length()-1), 1);
            return;
        }
        // 队列未满
        next(stack, sequence, nextPosition, queue);
    }
    public static void out(Stack<Integer> stack, int[] sequence, int nextPosition, Stack<Integer> queue){
        next(stack,sequence,nextPosition,queue);
    }
    public static void next(Stack<Integer> stack, int[] sequence, int nextPosition, Stack<Integer> queue){
        // 压入下一个元素
        stack.push(sequence[nextPosition]);
        in(stack, sequence, nextPosition+1, queue);
        stack.pop();
        if (!stack.isEmpty()) {
            queue.push(stack.pop());
            out(stack, sequence, nextPosition, queue);
            if (!queue.isEmpty()) stack.push(queue.pop());
        }
    }
}
```
## 奇淫技巧
#### 尺取法
```
// 数列
static int n = 10;
static int S = 15;
static int[] a = {5, 1, 3, 5, 10, 7, 4, 9, 2, 8};
static int[] sum = new int[n+1];
public static void solve() {
    int res = n + 1;
    int s = 0;
    int t = 0;
    int sum = 0;
    for(;;) {
        while (t<n && sum <S) {
            sum += a[t++];
        }
        if(sum < S)
            break;
        // 取本次窗口长度与最优值最小
        res = Math.min(res, t-s);
        System.out.println("本次窗口下标："+s+"->"+t+" 累加和："+sum);
        System.out.println("减掉下标："+s+"的内容："+a[s]);
        System.out.println("当前最优值："+res);
        sum -= a[s++];

    }
    if(res > n) {
        res = 0;
    }
    System.out.println(res);
}
```
```
// n个数，求最小区间覆盖着n个数中所有的不相同的数字
static int N = 1000008 ;
static int[] a = new int[N] ;
static Set<Integer> set = new HashSet<Integer>() ;
static Map<Integer , Integer> map = new HashMap<Integer, Integer>() ;

static void solve() {
    Scanner in = new Scanner(System.in);
    int n = in.nextInt() , size = 0 ;
    set.clear() ;
    map.clear() ;
    for(int i = 0 ; i < n ; i++){
        set.add(a[i] = in.nextInt() ) ;
    }
    size = set.size() ;

    int start = 0 , end = 0 , sum = 0 ;
    int res = n ;
    for(;;){
        while(end < n && sum < size){
            Integer cnt = map.get(a[end]) ;
            if(cnt == null){
                sum++ ;
                map.put(a[end] , 1) ;
            }
            else map.put(a[end] , cnt+1) ;
            end++ ;
        }
        if(sum < size) break ;
        res = Math.min(end - start , res) ;
        int cnt = map.get(a[start]) ;
        if(cnt == 1){
            map.remove(a[start]) ;
            sum-- ;
        }
        else map.put(a[start] , cnt-1) ;
        start++ ;
    }
    System.out.println(res);
}
```
#### 开关问题
&emsp;&emsp;挑战程序竞赛P150  
```
static int N = 7;
    static int[] dir = {1, 1, 0, 1, 0, 1, 1};
    static int[] f = new int[N];

    static int calc(int K) {
        for(int i=0; i<f.length; i++) {
            f[i] = 0;
        }
        int res = 0;
        int sum = 0;
        for(int i=0; i+K<=N; i++) {
            if((dir[i]+sum)%2 != 0) {
                res++;
                f[i] = 1;
            }
            sum += f[i];
            if(i-K+1>=0) {
                sum -= f[i-K+1];
            }
        }

        for(int i=N-K+1; i<N; i++) {
            if((dir[i]+sum)%2!=0) {
                return -1;
            }
            if(i-K+1>=0) {
                sum -= f[i-K+1];
            }
        }
        return res;
    }
    static void solve() {
        int K=1;
        int M = N;
        for(int k=1; k<=N; k++) {
            int m = calc(k);
            if(m>=0 && M>m) {
                M = m;
                K = k;
            }
        }
        System.out.println("K="+K+" M="+M);
    }
```
#### 弹性碰撞
&emsp;&emsp;挑战程序竞赛P158  
```
public class Main {
    public static void main(String[] args) {
        solve();
    }
    static int N = 2;
    static int H = 10;
    static int R = 10;
    static int T = 100;
    static double g = 10.0;
    static double[] y = new double[N];
    static double calc(int T) {
        if(T<0)
            return H;
        double t = Math.sqrt(2*H/g);
        int k = (int)(T/t);
        if(k%2==0) {
            double d = T - k * t;
            return H - g*d*d/2;
        }
        else {
            double d = k * t + t - T;
            return H - g*d*d/2;
        }
    }
    static void solve() {
        for(int i=0; i<N; i++) {
            y[i] = calc(T-i);
        }
        Arrays.sort(y);
        for(int i=0; i<N; i++) {
            DecimalFormat df = new DecimalFormat("#.00");

            System.out.print(df.format(y[i]+2*R*i/100.0)+" ");
            if(i+1==N)
                System.out.print("\n");
            else
                System.out.print(" ");
        }
    }
}
```
#### 黑白棋问题
&emsp;&emsp;挑战程序竞赛P152
```
static int M = 4;
static int N = 4;
static int[][] tile = {{1,0,0,1}, {0,1,1,0}, {0,1,1,0}, {1,0,0,1}};
static int[][] opt = new int[M][N];
static int[][] flip = new int[M][N];
static int[] dx = {-1, 0, 0, 0, 1};
static int[] dy = {0, -1, 0, 1, 0};

static int get(int x, int y) {
    int c = tile[x][y];
    for(int d=0; d<5; d++) {
        int x2 = x + dx[d];
        int y2 = y + dy[d];
        if(0<=x2 && x2<M && 0<=y2 && y2<N) {
            c += flip[x2][y2];
        }
    }
    return c % 2;
}
static int calc() {
    for(int i=1; i<M; i++) {
        for(int j=0; j<N; j++) {
            if(get(i-1, j)!=0) {
                flip[i][j] = 1;
            }
        }
    }
    for(int j=0; j<N; j++) {
        if(get(M-1, j)!=0) {
            return -1;
        }
    }
    int res = 0;
    for(int i=0; i<M; i++) {
        for(int j=0; j<N; j++) {
            res += flip[i][j];
        }
    }
    return res;
}
static void solve() {
    int res = -1;
    for(int i=0; i< 1<<N; i++) {
        for(int j=0; j<flip.length; j++)
            for (int k=0; k<flip[0].length; k++)
                flip[j][k] = 0;
        for(int j=0; j< N; j++)
            flip[0][N-j-1] = i >> j & 1;
        int num = calc();
        if(num >=0 && (res<0 || res>num)) {
            res = num;
            for(int j=0; j<flip.length; j++) {
                for (int k=0; k<flip[0].length; k++) {
                    opt[j][k] = flip[j][k];
                }
            }
        }
    }
    if(res < 0) {
        System.out.println("无解");
    }
    else {
        for(int i=0; i<M; i++) {
            for(int j=0; j<N; j++) {
                System.out.print(opt[i][j]);
                if(j+1==N)
                    System.out.print("\n");
                else
                    System.out.print(" ");
            }
        }
    }
}
```
#### 坐标离散化
&emsp;&emsp;直线将区域划分为多少个区块(挑战程序竞赛p164)  
&emsp;&emsp;暂时没有Java代码  
