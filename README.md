# JMH(Java Microbenchmark Harness)
- A toolkit that helps you implement Java micro-benchmarks correctly 
- Writing a correct Java microbenchmark is hard 
  - JVM optimization, etc 

# JMH Config 
```java 
Options opt = new OptionsBuilder()
    .include(JMHSetOperation.class.getSimpleName())
    .warmupIterations(3)
    .measurementIterations(5)
    .forks(1)
    .build();
```
- `include`: specifies the class to be benchmarked 
- `warmupIterations`: number of warm-up iterations to be executed before the actual benchmarking starts 
- `measurementIterations`: the number of iterations to be executed for measuring the performance of the benchmarked code
- `forks`: number of times the benchmark should be executed in separate JVM instances

# JMH Result Interpretation 
```text
# Run progress: 66.67% complete, ETA 00:01:26
# Fork: 1 of 1
# Warmup Iteration   1: 8196.310 us/op
# Warmup Iteration   2: 8226.681 us/op
# Warmup Iteration   3: 8239.040 us/op
Iteration   1: 8135.843 us/op
Iteration   2: 8177.210 us/op
Iteration   3: 8654.256 us/op
Iteration   4: 8448.569 us/op
Iteration   5: 8843.438 us/op
```
- During the warmup and measurement iterations, the program's performance was gradually increasing
```text
Benchmark                Mode  Cnt     Score      Error  Units
SetAdd.addHashSet        avgt    5    69.738 ±   32.364  us/op
SetAdd.addLinkedHashSet  avgt    5    66.778 ±    2.004  us/op
SetAdd.addTreeSet        avgt    5  8451.863 ± 1170.506  us/op
```
- `Mode`: Type of benchmark mode used, which ia `avgt`(average time) in this case 
- `Cnt`: Number of iterations performed for each benchmark
- `Score`: Indicates the average time it takes to execute a single operation in microseconds(`us/op`)
- `Error`: Indicates the standard deviation of the measurements
- `Units`: Unit of measurement used, which is microseconds per operation

# Writing Good Benchmarking Code 
## Loop Optimization 
- Don't put your benchmark code inside a loop 
- JVM is good at optimizing loops, so you may end up with a different result that what you expected  
- Avoid loops in your benchmark methods, unless they are part of the code you want to measure 

## Dead Code Elimination 
- If JVM detects that the result of some computation is never used, the JVM may consider this computation dead code and eliminate it.
```java 
package com.jenkov;

import org.openjdk.jmh.annotations.Benchmark;

public class MyBenchmark {

    @Benchmark
    public void testMethod() {
        int a = 1;
        int b = 2;
        int sum = a + b; // <-- dead code 
    }

}
```
- `sum` is never used -> Code related to `a`, `b` and `sum` can all be eliminated
- How to prevent dead code elimination ? 
  - Return the result(`sum` in this case) 
    - ```java 
      package com.jenkov;

        import org.openjdk.jmh.annotations.Benchmark;

        public class MyBenchmark {

            @Benchmark
            public int testMethod() {
                int a = 1;
                int b = 2;
                int sum = a + b;

                return sum;
            }

        }
      ```
  - Pass the calculated value into a `block hole` provided by JMH  
    - ```java
      package com.jenkov;

      import org.openjdk.jmh.annotations.Benchmark;
      import org.openjdk.jmh.infra.Blackhole;

      public class MyBenchmark {

          @Benchmark
          public void testMethod(Blackhole blackhole) {
              int a = 1;
              int b = 2;
              int sum = a + b;
              blackhole.consume(sum);
          }
      }  
      ```
      
## Constant Folding 
- A calculation which is based on constants will often result in exact same result, regardless of how many times the calculation is performed 
```java
package com.jenkov;

import org.openjdk.jmh.annotations.Benchmark;

public class MyBenchmark {

    @Benchmark
    public int testMethod() {
        int a = 1;
        int b = 2;
        int sum = a + b;

        return sum;
    }

}
```
- Above code might be optimized into 
```java
package com.jenkov;

import org.openjdk.jmh.annotations.Benchmark;

public class MyBenchmark {

    @Benchmark
    public int testMethod() {
        int sum = 3;

        return sum;
    }

}
```
- Avoid constant folding by using state objects
```java
package com.jenkov;

import org.openjdk.jmh.annotations.*;

public class MyBenchmark {

    @State(Scope.Thread)
    public static class MyState {
        public int a = 1;
        public int b = 2;
    }


    @Benchmark 
    public int testMethod(MyState state) {
        int sum = state.a + state.b;
        return sum;
    }
}
```
# Reference 
- https://jenkov.com/tutorials/java-performance/jmh.html
