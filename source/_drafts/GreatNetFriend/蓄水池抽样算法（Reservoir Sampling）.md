# 蓄水池抽样算法（Reservoir Sampling）

许多年以后，当听说蓄水池抽样算法时，邱simple将会想起，那个小学数学老师带他做“小明对水池边加水边放水，求何时能加满水”应用题的下午。

## 一、问题

我是在一次失败的面试经历中听说蓄水池算法的。之后上网搜了搜，知道是一个数据抽样算法，寥寥几行，却暗藏玄机。主要用来解决如下问题。

**给定一个数据流，数据流长度N很大，且N直到处理完所有数据之前都不可知，请问如何在只遍历一遍数据（O(N)）的情况下，能够随机选取出m个不重复的数据。**

这个场景强调了3件事：

1. 数据流长度N很大且不可知，所以不能一次性存入内存。
2. 时间复杂度为O(N)。
3. 随机选取m个数，每个数被选中的概率为m/N。

第1点限制了不能直接取N内的m个随机数，然后按索引取出数据。第2点限制了不能先遍历一遍，然后分块存储数据，再随机选取。第3点是数据选取绝对随机的保证。讲真，在不知道蓄水池算法前，我想破脑袋也不知道该题做何解。

## 二、核心代码及原理

蓄水池抽样算法的核心如下：



```cpp
int[] reservoir = new int[m];

// init
for (int i = 0; i < reservoir.length; i++)
{
    reservoir[i] = dataStream[i];
}

for (int i = m; i < dataStream.length; i++)
{
    // 随机获得一个[0, i]内的随机整数
    int d = rand.nextInt(i + 1);
    // 如果随机整数落在[0, m-1]范围内，则替换蓄水池中的元素
    if (d < m)
    {
        reservoir[d] = dataStream[i];
    }
}
```

**注：**这里使用已知长度的数组dataStream来表示未知长度的数据流，并假设数据流长度大于蓄水池容量m。

算法思路大致如下：

1. 如果接收的数据量小于m，则依次放入蓄水池。
2. 当接收到第i个数据时，i >= m，在[0, i]范围内取以随机数d，若d的落在[0, m-1]范围内，则用接收到的第i个数据替换蓄水池中的第d个数据。
3. 重复步骤2。

算法的精妙之处在于：**当处理完所有的数据时，蓄水池中的每个数据都是以m/N的概率获得的。**

下面用白话文推导验证该算法。假设数据开始编号为1.

**第i个接收到的数据最后能够留在蓄水池中的概率**=**第i个数据进入过蓄水池的概率*****之后第i个数据不被替换的概率**（第i+1到第N次处理数据都不会被替换）。

1. 当i<=m时，数据直接放进蓄水池，所以**第i个数据进入过蓄水池的概率=1**。
2. 当i>m时，在[1,i]内选取随机数d，如果d<=m，则使用第i个数据替换蓄水池中第d个数据，因此**第i个数据进入过蓄水池的概率=m/i**。
3. 当i<=m时，程序从接收到第m+1个数据时开始执行替换操作，第m+1次处理会替换池中数据的为m/(m+1)，会替换掉第i个数据的概率为1/m，则第m+1次处理替换掉第i个数据的概率为(m/(m+1))*(1/m)=1/(m+1)，不被替换的概率为1-1/(m+1)=m/(m+1)。依次，第m+2次处理不替换掉第i个数据概率为(m+1)/(m+2)...第N次处理不替换掉第i个数据的概率为(N-1)/N。所以，之后**第i个数据不被替换的概率=m/(m+1)\*(m+1)/(m+2)\*...\*(N-1)/N=m/N**。
4. 当i>m时，程序从接收到第i+1个数据时开始有可能替换第i个数据。则参考上述第3点，**之后第i个数据不被替换的概率=i/N**。
5. 结合第1点和第3点可知，当i<=m时，第i个接收到的数据最后留在蓄水池中的概率=1*m/N=m/N。结合第2点和第4点可知，当i>m时，第i个接收到的数据留在蓄水池中的概率=m/i*i/N=m/N。综上可知，**每个数据最后被选中留在蓄水池中的概率为m/N**。

这个算法建立在统计学基础上，很巧妙地获得了“m/N”这个概率。

## 三、深入一些——分布式蓄水池抽样（Distributed/Parallel Reservoir Sampling）

一块CPU的计算能力再强，也总有内存和磁盘IO拖他的后腿。因此为提高数据吞吐量，分布式的硬件搭配软件是现在的主流。

如果遇到超大的数据量，即使是O(N)的时间复杂度，蓄水池抽样程序完成抽样任务也将耗时很久。因此分布式的蓄水池抽样算法应运而生。运作原理如下：

1. 假设有K台机器，将大数据集分成K个数据流，每台机器使用单机版蓄水池抽样处理一个数据流，抽样m个数据，并最后记录处理的数据量为N1, N2, ..., Nk, ..., NK(假设m<Nk)。N1+N2+...+NK=N。
2. 取[1, N]一个随机数d，若d<N1，则在第一台机器的蓄水池中等概率不放回地（1/m）选取一个数据；若N1<=d<(N1+N2)，则在第二台机器的蓄水池中等概率不放回地选取一个数据；一次类推，重复m次，则最终从N大数据集中选出m个数据。

m/N的概率验证如下：

1. 第k台机器中的蓄水池数据被选取的概率为m/Nk。
2. 从第k台机器的蓄水池中选取一个数据放进最终蓄水池的概率为Nk/N。
3. 第k台机器蓄水池的一个数据被选中的概率为1/m。（不放回选取时等概率的）
4. 重复m次选取，则每个数据被选中的概率为**m\*(m/Nk\*Nk/N\*1/m)=m/N**。

## 四、算法验证

写一份完整的代码，用来验证蓄水池抽样的随机性。数据集大小为1000，蓄水池容量为10，做10_0000次抽样。如果程序正确，那么每个数被抽中的次数接近1000次。



```dart
package cn.edu.njupt.qyz;

import java.util.ArrayList;
import java.util.Arrays;
import java.util.LinkedList;
import java.util.List;
import java.util.Random;
import java.util.Set;
import java.util.TreeSet;
import java.util.concurrent.Callable;
import java.util.concurrent.ExecutionException;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.Future;

public class ReservoirSampling {
    
    static ExecutorService exec = Executors.newFixedThreadPool(4);
    
    // 抽样任务，用作模拟并行抽样
    private static class SampleTask implements Callable<int[]>
    {
        // 输入该任务的数据
        private int[] innerData;
        // 蓄水池容量
        private int m;
        
        SampleTask (int m, int[] innerData)
        {
            this.innerData = innerData;
            this.m = m;
        }

        @Override
        public int[] call() throws Exception
        {
            int[] reservoir = sample(this.m, this.innerData);
            return reservoir;
        }
        
    }
    
    // 并行抽样
    public static int[] mutiSample(int m, int[] dataStream) throws InterruptedException, ExecutionException
    {
        Random rand = new Random();
        
        
        int[] reservoir = initReservoir(m, dataStream);
        
        // 生成3个范围内随机数，将数据切成4份
        List<Integer> list = getRandInt(rand, dataStream.length); 
        int s1 = list.get(0);
        int s2 = list.get(1);
        int s3 = list.get(2);
        // 每个任务处理的数据量
        double n1 = s1 - 0;
        double n2 = s2 - s1;
        double n3 = s3 - s2;
        double n4 = dataStream.length - s3;
        
        // 并行抽样
        Future<int[]> f1 = exec.submit(new SampleTask(m, Arrays.copyOfRange(dataStream, 0, s1)));
        Future<int[]> f2 = exec.submit(new SampleTask(m, Arrays.copyOfRange(dataStream, s1, s2)));
        Future<int[]> f3 = exec.submit(new SampleTask(m, Arrays.copyOfRange(dataStream, s2, s3)));
        Future<int[]> f4 = exec.submit(new SampleTask(m, Arrays.copyOfRange(dataStream, s3, dataStream.length)));
        List<Integer> r1 = getList(f1.get());
        List<Integer> r2 = getList(f2.get());
        List<Integer> r3 = getList(f3.get());
        List<Integer> r4 = getList(f4.get());
        
        // 进行m次抽样
        for (int i = 0; i < m; i++)
        {
            int p = rand.nextInt(dataStream.length);
            // 根据随机数落在的范围选择元素
            if (p < s1)
            {
                reservoir[i] = getRandEle(r1, rand.nextInt(r1.size()));
            }
            else if (p < s2)
            {
                reservoir[i] = getRandEle(r2, rand.nextInt(r2.size()));
            }
            else if (p < s3)
            {
                reservoir[i] = getRandEle(r3, rand.nextInt(r3.size()));
            }
            else
            {
                reservoir[i] = getRandEle(r4, rand.nextInt(r4.size()));
            }
        }
        
        return reservoir;
    }
    
    // 根据输入返回随机位置的元素，并且删除该元素，模拟不放回
    private static int getRandEle(List<Integer> list, int idx)
    {
        return list.remove(idx);
    }
    
    // 获取bound范围内的3个随机数，用来分割数据集
    private static List<Integer> getRandInt(Random rand, int bound)
    {
        Set<Integer> set = new TreeSet<>();
        List<Integer> list = new ArrayList<>();
        
        while (set.size() < 3)
        {
            set.add(rand.nextInt(bound));
        }
        for (int e: set)
        {
            list.add(e);
        }
        return list;
    }
    // 数据转换成List
    private static List<Integer> getList(int[] arr)
    {
        List<Integer> list = new LinkedList<>();
        for (int a : arr)
        {
            list.add(a);
        }
        return list;
    }
    
    // 单机版蓄水池抽样
    public static int[] sample(int m, int[] dataStream)
    {
        // 随机数生成器，以系统当前nano时间作为种子
        Random rand = new Random();
        
        int[] reservoir = initReservoir(m, dataStream);
        
        // init
        for (int i = 0; i < reservoir.length; i++)
        {
            reservoir[i] = dataStream[i];
        }

        for (int i = m; i < dataStream.length; i++)
        {
            // 随机获得一个[0, i]内的随机整数
            int d = rand.nextInt(i + 1);
            // 如果随机整数在[0, m-1]范围内，则替换蓄水池中的元素
            if (d < m)
            {
                reservoir[d] = dataStream[i];
            }
        }
        return reservoir;
    }
    
    private static int[] initReservoir (int m, int[] dataStream)
    {
        int[] reservoir;
        
        if (m > dataStream.length)
        {
            reservoir = new int[dataStream.length];
        }
        else
        {
            reservoir = new int[m];
        }
        return reservoir;
    }
    
    // 单机版测试
    public void test()
    {
        // 样本长度
        int len = 1000;
        // 蓄水池容量
        int m = 10;
        // 抽样次数，用作验证抽样的随机性
        int iterTime = 100000;
        // 每个数字被抽到的次数
        int[] freq = new int[len];
        // 样本
        int[] dataStream = new int[len];
        
        // init dataStream
        for (int i = 0; i < dataStream.length; i++)
        {
            dataStream[i] = i;
        }
        
        // count freq
        for (int k = 0; k < iterTime; k++)
        {
            // 进行抽样
            int[] reservoir = sample(m, dataStream);
            // 计算出现次数
            for (int i = 0; i < reservoir.length; i++)
            {
                int ele = reservoir[i];
                freq[ele] += 1; 
            }
        }
        
        printStaticInfo(freq);
    }
    
    // 测试并行抽样
    public void mutiTest() throws InterruptedException, ExecutionException
    {
        // 样本长度
        int len = 1000;
        // 蓄水池容量
        int m = 10;
        // 抽样次数，用作验证抽样的随机性
        int iterTime = 10_0000;
        // 每个数字被抽到的次数
        int[] freq = new int[len];
        // 样本
        int[] dataStream = new int[len];
        
        // init dataStream
        for (int i = 0; i < dataStream.length; i++)
        {
            dataStream[i] = i;
        }
        
        // count freq
        for (int k = 0; k < iterTime; k++)
        {
            // 进行抽样
            int[] reservoir = mutiSample(m, dataStream);
            // 计算出现次数
            for (int i = 0; i < reservoir.length; i++)
            {
                int ele = reservoir[i];
                freq[ele] += 1; 
            }
        }
        printStaticInfo(freq);
    }
    // 打印统计信息
    private void printStaticInfo (int[] freq)
    {
        // 期望、方差和标准差
        double avg = 0;
        double var = 0;
        double sigma = 0;
        // print
        for (int i = 0; i < freq.length; i++)
        {
            if (i % 10 == 9) System.out.println();
            System.out.print(freq[i] + ", ");
            avg += ((double)(freq[i]) / freq.length);
            var += (double)(freq[i] * freq[i]) / freq.length;
        }
        
        // 输出统计信息
        System.out.println("\n===============================");
        var = var - avg * avg;
        sigma = Math.sqrt(var);
        System.out.println("Average: " + avg);
        System.out.println("Variance: " + var);
        System.out.println("Standard deviation: " + sigma);
    }
    
    public static void main (String[] args) throws InterruptedException, ExecutionException
    {
        ReservoirSampling rs = new ReservoirSampling();
        rs.mutiTest();
    }
}
```

单机版输出和并行版的输出类似，截取片段如下：



```undefined
948, 1006, 1014, 1019, 1033, 1040, 948, 1014, 1000, 951, 
1014, 987, 1049, 1043, 1034, 983, 1006, 974, 1060, 1009, 
986, 1021, 1024, 963, 1041, 1028, 988, 1011, 975, 980, 
1055, 1017, 1010, 1018, 1013, 983, 942, 1056, 1003, 1063, 
1004, 1004, 999, 976, 957, 935, 1061, 1018, 1002, 1018, 
1019, 946, 985, 1057, 1012, 965, 978, 1040, 1026, 1064, 
1026, 1018, 980, 996, 1025, 1028, 1006, 944, 986, 981, 
923, 1015, 991, 1019, 1024, 1143, 989, 985, 1022, 1019, 
1004, 1000, 989, 972, 1041, 988, 1050, 932, 975, 1037, 
1016, 983, 1051, 1003, 983, 986, 1017, 1009, 936, 993, 
965, 976, 1001, 1000, 988, 1030, 1050, 1024, 981, 985, 
935, 1023, 996, 1007, 1013, 1046, 1003, 1006, 973, 989, 
943, 
===============================
Average: 1000.0000000000002
Variance: 1011.8799999983748
Standard deviation: 31.81006130139291
```

此外，为了对比单机版与并行版（4线程）的性能差异，使用10_0000大小的数据集，蓄水池容量10，进行100_0000次重复抽样，对比两者的运行时间。结果如下



```undefined
---------单机版----------

===============================
Average: 100.00000000000125
Variance: 100.31497999751264
Standard deviation: 10.015736617818613
---------并行版----------

===============================
Average: 100.00000000000169
Variance: 100.63045999737915
Standard deviation: 10.031473470900432
单机版耗时：2006s
并行版耗时：1265s
```

从输出结果可以看出，算法保证了数据选取的随机性。且并行版算法能够有效提高数据吞吐量。

## 五、应用场景

蓄水池抽样的O(N)时间复杂度，O(m)空间复杂度令其适用于对流数据、大数据集的等概率抽样。比如一个大文本数据，随机输出其中的几行。

## 六、总结

象征性总结：优雅巧妙的算法——蓄水池抽样。

## 七、参考文献

1. [数据工程师必知算法：蓄水池抽样](https://link.jianshu.com?t=http%3A%2F%2Fblog.jobbole.com%2F42550%2F)
2. [【算法34】蓄水池抽样算法 (Reservoir Sampling Algorithm)](https://link.jianshu.com?t=http%3A%2F%2Fwww.cnblogs.com%2Fpython27%2Fp%2FReservoir_Sampling_Algorithm.html)
3. [分布式/并行蓄水池抽样 (Distributed/Parallel Reservoir Sampling)](https://www.jianshu.com/p/eea05fb27e3f)
4. [Distributed/Parallel Reservoir Sampling](https://link.jianshu.com?t=https%3A%2F%2Fballsandbins.wordpress.com%2F2014%2F04%2F13%2Fdistributedparallel-reservoir-sampling%2F)



作者：邱simple
链接：https://www.jianshu.com/p/7a9ea6ece2af
来源：简书
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。