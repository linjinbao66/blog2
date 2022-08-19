---

titile: 多线程计算圆周率
tags:
  - 多线程
date: 2019-10-03

---

# 多线程计算圆周率

使用公式1-1/3+1/5......

## 代码示例

```text
package learn;

import java.math.BigDecimal;
import java.util.ArrayList;
import java.util.concurrent.*;

/**
 * 多线程计算圆周率
 */
public class Pie {

    public static double sum = 0.0;

    public static void main(String[] args) throws ExecutionException, InterruptedException {
        int tNum = 64;
        ExecutorService pool = Executors.newFixedThreadPool(tNum);

        ArrayList<Future<BigDecimal>> list = new ArrayList<>();

        long tmp = 1000000000;

        long time0 = System.currentTimeMillis();
        System.out.println("开始计算时间："+time0);
        for (int i=0; i< tNum; i++){
            long start = 1+tmp*i;
            long end = start+tmp-1;
            Callable<BigDecimal> caller = new Pie().new MyCaller(start,end);
            Future<BigDecimal> res = pool.submit(caller);

            list.add(res);
        }
        pool.shutdown();

        BigDecimal sum = new BigDecimal(0.0);
        for (Future<BigDecimal> res: list){
            BigDecimal tmp1 = new BigDecimal(res.get().doubleValue());
           sum  = sum.add(tmp1);
        }
        long time1 = System.currentTimeMillis();
        System.out.println(sum);
        System.out.println("结束计算时间："+time1);
        System.out.println("计算总共耗时："+(time1-time0)/1000 + "秒");
    }

    public class MyCaller implements Callable<BigDecimal> {
        private long start;
        private long end;

        public MyCaller(long start, long end) {
            this.start = start;
            this.end = end;
        }

        @Override
        public BigDecimal call() throws Exception {
           double sum = 0.0;
           for (long n = start; n<=end; n++){
               sum+=Math.pow(-1.0,n-1)*4.0/(2.0*n-1.0);
           }
           return new BigDecimal(sum);
        }
    }
}
```

## 计算结果

```text
开始计算时间：1572782003607
3.141592653572425427117925326266235743040967334956115042388091751490719616413116455078125
结束计算时间：1572783503836
计算总共耗时：1500秒

Process finished with exit code 0
```
