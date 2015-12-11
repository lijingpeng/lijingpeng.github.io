title: Java beta distribution
date: 2015-09-03 14:51:59
tags:
category: Java
---


维基百科介绍：https://zh.wikipedia.org/wiki/%CE%92%E5%88%86%E5%B8%83
https://en.wikipedia.org/wiki/Beta_distribution

```java
import org.apache.commons.math3.distribution.BetaDistribution;

public class test {
    /**
     * @param args
     */
    public static void main(String[] args) {
        double x;
        double b;
        BetaDistribution beta = new BetaDistribution(40.0, 40.0);
        for (int i = 0; i < 100; i++) {
            x = Math.random();
            b = beta.inverseCumulativeProbability(x);
            System.out.println(b);
        }
    }
}
```

实现：
```java
public class BetaDistributionE {
	public double getBetaDistribution(double alpha, double beta){
    double a = alpha + beta;
	double b;
	if(Math.min(alpha, beta) <= 1){
		b = Math.max(1/alpha, 1/beta);
	}
	else{
	b = Math.sqrt((a - 2) / (2*alpha*beta - a));
	}
	double c = alpha + 1/b;
 
	double W = 0;
	boolean reject = true;
	for (reject = true; reject; ) {
 		double U1 = Math.random();
 		double U2 = Math.random();
 		double V = b * Math.log(U1/(1-U1));
 		W = alpha * Math.exp(V);
 		reject = (a*Math.log(a/(beta+W)) + c*V - Math.log(4)) < Math.log(U1*U1*U2);
 	}
 	return (W / (beta + W));
 	}
}
```
相关资源：
1. http://commons.apache.org/proper/commons-math/apidocs/org/apache/commons/math3/distribution/BetaDistribution.html
2. http://stackoverflow.com/questions/13634170/java-using-beta-distribution-to-generate-a-random-number-from-0-to-1
3. go语言实现：https://github.com/purzelrakete/bandit/blob/4fca67f963006845de83860fcd849625251fce57/math/rand.go#L21
4. 非均衡抽样，在一个概率区间内抽样：Paper：Generating Beta Variates with Nonintegral Shape Parameters
