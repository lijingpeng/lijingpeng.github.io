title: Bayesian Bandits原理及在互联网广告行业的应用
date: 2015-09-03 13:48:13
tags:
---

## 1. The Multi-Armed Bandit Problem
Suppose you are faced with N slot machines (colourfully called multi-armed bandits). Each bandit has an unknown probability of distributing a prize (assume for now the prizes are the same for each bandit, only the probabilities differ). Some bandits are very generous, others not so much. Of course, you don’t know what these probabilities are. By only choosing one bandit per round, our task is devise a strategy to maximize our winnings.

Of course, if we knew the bandit with the largest probability, then always picking this bandit would yield the maximum winnings. So our task can be phrased as “Find the best bandit, and as quickly as possible”.

The task is complicated by the stochastic nature of the bandits. A suboptimal bandit can return many winnings, purely by chance, which would make us believe that it is a very profitable bandit. Similarly, the best bandit can return many duds. Should we keep trying losers then, or give up?

A more troublesome problem is, if we have a found a bandit that returns pretty good results, do we keep drawing from it to maintain our pretty good score, or do we try other bandits in hopes of finding an even-better bandit? This is the exploration vs. exploitation dilemma.

## 2. Applications

The Multi-Armed Bandit problem at first seems very artificial, something only a mathematician would love, but that is only before we address some applications:

Internet display advertising: companies have a suite of potential ads they can display to visitors, but the company is not sure which ad strategy to follow to maximize sales. This is similar to A/B testing, but has the added advantage of naturally minimizing strategies that do not work (and generalizes to A/B/C/D… strategies)

1. Ecology: animals have a finite amount of energy to expend, and following certain behaviours has uncertain rewards. How does the animal maximize its fitness?
2. Finance: which stock option gives the highest return, under time-varying return profiles.
3. Clinical trials: a researcher would like to find the best treatment, out of many possible treatments, while minimizing losses.

Many of these questions above are fundamental to the application’s field. It turns out the optimal solution is incredibly difficult, and it took decades for an overall solution to develop. There are also many approximately-optimal solutions which are quite good. The one I wish to discuss is one of the few solutions that can scale incredibly well. The solution is known asBayesian Bandits.

## 3. A Proposed Solution

Any proposed strategy is called an online algorithm (not in the internet sense, but in the continuously-being-updated sense), and more specifically a reinforcement learning algorithm. The algorithm starts in an ignorant state, where it knows nothing, and begins to acquire data by testing the system. As it acquires data and results, it learns what the best and worst behaviours are (in this case, it learns which bandit is the best). With this in mind, perhaps we can add an additional application of the Multi-Armed Bandit problem:

Psychology: how does punishment and reward effect our behaviour? How do humans’ learn?
The Bayesian solution begins by assuming priors on the probability of winning for each bandit. In our vignette we assumed complete ignorance of the these probabilities. So a very natural prior is the flat prior over 0 to 1. The algorithm proceeds as follows:

For each round,
1. Sample a random variable Xb from the prior of bandit b, for all b.
2. Select the bandit with largest sample, i.e. select bandit B=argmaxXb.
3. Observe the result of pulling bandit B, and update your prior on bandit B.
4. Return to 1.

That’s it. Computationally, the algorithm involves sampling from N distributions. Since the initial priors are Beta(α=1,β=1) (a uniform distribution), and the observed result X (a win or loss, encoded 1 and 0 respectfully) is Binomial, the posterior is a Beta(α=1+X,β=1+1−X)(see here for why to is true). 

To answer a question from before, this algorithm suggests that we should not discard losers, but we should pick them at a decreasing rate as we gather confidence that there exist better bandits. This follows because there is always a non-zero chance that a loser will achieve the status of B, but the probability of this event decreases as we play more rounds (see figure below). Below is an implementation of the Bayesian Bandits strategy (which can be skipped for the less Pythonic-ly interested).

```python
from pymc import rbeta
 
rand = np.random.rand
 
class Bandits(object):
    """
    This class represents N bandits machines.
 
    parameters:
        p_array: a (n,) Numpy array of probabilities >0, <1.
 
    methods:
        pull( i ): return the results, 0 or 1, of pulling 
                   the ith bandit.
    """
    def __init__(self, p_array):
        self.p = p_array
        self.optimal = np.argmax(p_array)
 
    def pull( self, i ):
        #i is which arm to pull
        return rand() < self.p[i]
 
    def __len__(self):
        return len(self.p)
 
 
class BayesianStrategy( object ):
    """
    Implements a online, learning strategy to solve
    the Multi-Armed Bandit problem.
 
    parameters:
        bandits: a Bandit class with .pull method
 
    methods:
        sample_bandits(n): sample and train on n pulls.
 
    attributes:
        N: the cumulative number of samples
        choices: the historical choices as a (N,) array
        bb_score: the historical score as a (N,) array
 
    """
 
    def __init__(self, bandits):
 
        self.bandits = bandits
        n_bandits = len( self.bandits )
        self.wins = np.zeros( n_bandits )
        self.trials = np.zeros(n_bandits )
        self.N = 0
        self.choices = []
        self.bb_score = []
 
 
    def sample_bandits( self, n=1 ):
 
        bb_score = np.zeros( n )
        choices = np.zeros( n )
 
        for k in range(n):
            #sample from the bandits's priors, and select the largest sample
            choice = np.argmax( rbeta( 1 + self.wins, 1 + self.trials - self.wins) )
 
            #sample the chosen bandit
            result = self.bandits.pull( choice )
 
            #update priors and score
            self.wins[ choice ] += result
            self.trials[ choice ] += 1
            bb_score[ k ] = result 
            self.N += 1
            choices[ k ] = choice
 
        self.bb_score = np.r_[ self.bb_score, bb_score ]
        self.choices = np.r_[ self.choices, choices ]
        return
```

Below we present a visualization of the algorithm sequentially learning the solution. In the figure below, the dashed lines represent the true hidden probabilities, which are (0.85, 0.60, 0.75)(this can be extended to many more dimensions, but the figure suffers, so I kept it at 3).

![](/images/bb/updating2.png)

Note that we don’t real care how accurate we become about inference of the hidden probabilities — for this problem we are more interested in choosing the best bandit (or more accurately, becoming more confident in choosing the best bandit). For this reason, the distribution of the red bandit is very wide (representing ignorance about what that hidden probability might be) but we are reasonably confident that it is not the best, so the algorithm chooses to ignore it.

###  几篇介绍Bayesian Bandits的原理的文章：

1. https://www.chrisstucchio.com/blog/2013/bayesian_analysis_conversion_rates.html
2. 在线演示博弈过程： https://e76d6ebf22ef8d7e079810f3d1f82ba1e5f145d5.googledrive.com/host/0B2GQktu-wcTiWDB2R2t2a2tMUG8/
3. https://www.chrisstucchio.com/blog/2013/bayesian_bandit.html
4. http://camdp.com/blogs/multi-armed-bandits
5. 贝叶斯书籍：https://github.com/lijingpeng/Probabilistic-Programming-and-Bayesian-Methods-for-Hackers
