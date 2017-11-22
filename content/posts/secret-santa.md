---
title: "Secret Santa"
date: 2017-11-13T19:49:44-05:00
draft: false
---

Things I never knew I never knew about Secret Santa...

#### Names in a hat

The first time I played Secret Santa we assigned givers and receivers like so:

1. Put the names of all the players in a hat
2. Take turns to shake the hat and pick a name out at random.  If you pick your own name, put it back and pick again.

So when I decided to program it this is how I tried to do it.  Here we use integers as the players and they get paired up in giver, receiver pairs:

    fun draw(n: Int) : List<Pair<Int, Int>> {
        if (n < 4) throw IllegalArgumentException()
        val players = (1..n)
        val hat = players.toMutableList()
        Collections.shuffle(hat)
        val result = mutableListOf<Pair<Int, Int>>()
        players.forEach {
            while (hat[0] == it) {
                Collections.shuffle(hat)
            }
            result.add(Pair(it, hat.removeAt(0)))
        }
        return result
    }
        
On a first try it seems to work:

    println(SecretSanta().draw(4))
    // printed [(1, 4), (2, 1), (3, 2), (4, 3)]
    
but try it a few times and, on some runs, it will never finish.  It's not hard to see why -- it's possible that when number 4 comes to choose, it finds only itself in the hat and reshuffles forever.

This could happen in a real names-in-a-hat situation too.  Has it ever happened to you?  When it does, there's nothing to be done but to put all the names back in and try again, so that's what I tried next:

    fun draw2(n: Int) : List<Pair<Int, Int>> {
        if (n < 4) throw IllegalArgumentException()
        val players = (1..n)
        class BadHatException : RuntimeException()
        while (true) {
            val hat = players.toMutableList()
            Collections.shuffle(hat)
            try {
                val result = mutableListOf<Pair<Int, Int>>()
                players.forEach {
                    if (hat.size == 1 && hat[0] == it) {
                        throw BadHatException()
                    }
                    while (hat[0] == it) {
                        Collections.shuffle(hat)
                    }
                    result.add(Pair(it, hat.removeAt(0)))
                }
                return result
            }
            catch (e: BadHatException) {
                // ok, try again
            }
        }
    }
    
This always seems to stop, so we are doing slightly better:

    (1..100_000).forEach { SecretSanta().draw2(4) }
    
With 4 players, there aren't many valid assignments, just these 9:

    (1, 2), (2, 1), (3, 4), (4, 3)
    (1, 2), (2, 3), (3, 4), (4, 1)
    (1, 2), (2, 4), (3, 1), (4, 3)
    (1, 3), (2, 1), (3, 4), (4, 2)
    (1, 3), (2, 4), (3, 1), (4, 2)
    (1, 3), (2, 4), (3, 2), (4, 1)
    (1, 4), (2, 1), (3, 2), (4, 3)
    (1, 4), (2, 3), (3, 1), (4, 2)
    (1, 4), (2, 3), (3, 2), (4, 1)
    
A good test would be to see if our procedure returns each of the nine, and that it returns each one 1/9 of the time:

    val patterns = mutableMapOf<List<Pair<Int, Int>>, Int>()
    (1..100_000).forEach {
        val pattern = SecretSanta().draw2(4)
        patterns[pattern] = patterns.getOrDefault(pattern, 0) + 1
    }
    patterns.entries.sortedByDescending { it.value }.forEach { println(it) }
        
We run it 100,000 times and would expect each pattern to occur around 11,111 times.  But:

Pattern                          | Occurences
---------------------------------|-----------
(1, 4), (2, 1), (3, 2), (4, 3)   | 19241
(1, 2), (2, 4), (3, 1), (4, 3)   | 13004
(1, 2), (2, 1), (3, 4), (4, 3)   | 12943
(1, 3), (2, 4), (3, 2), (4, 1)   | 9775
(1, 4), (2, 3), (3, 1), (4, 2)   | 9720
(1, 4), (2, 3), (3, 2), (4, 1)   | 9685
(1, 3), (2, 4), (3, 1), (4, 2)   | 9591
(1, 3), (2, 1), (3, 4), (4, 2)   | 9532
(1, 2), (2, 3), (3, 4), (4, 1)   | 6509
    
The algorithm is biased.  We are giving special treatment to the last pick and this is skewing the distribution.  There is also an anonymity problem.  You gain information when you witness someone drawing their own name and putting it back.  In an extreme case, if the second-to-last person draws their name and puts it back, then you can be sure that the last persion will end up drawing the second-to-last.  So, what if we reject the draw as soon as we find an element in the wrong place and reshuffle the whole hat:

    fun draw3(n: Int) : List<Pair<Int, Int>> {
        if (n < 4) throw IllegalArgumentException()
        val players = (1..n)
        class BadHatException : RuntimeException()
        while (true) {
            val hat = players.toMutableList()
            Collections.shuffle(hat)
            try {
                val result = mutableListOf<Pair<Int, Int>>()
                players.forEach {
                    if (hat[0] == it) {
                        throw BadHatException()
                    }
                    result.add(Pair(it, hat.removeAt(0)))
                }
                return result
            }
            catch (e: BadHatException) {
                // ok, try again
            }
        }
    }    
    
We will be doing more work here, but the distribution looks better:

Pattern                          | Occurences
---------------------------------|-----------
(1, 2), (2, 4), (3, 1), (4, 3)   | 11221
(1, 4), (2, 3), (3, 1), (4, 2)   | 11203
(1, 3), (2, 1), (3, 4), (4, 2)   | 11176
(1, 4), (2, 1), (3, 2), (4, 3)   | 11151
(1, 3), (2, 4), (3, 1), (4, 2)   | 11092
(1, 3), (2, 4), (3, 2), (4, 1)   | 11087
(1, 4), (2, 3), (3, 2), (4, 1)   | 11042
(1, 2), (2, 3), (3, 4), (4, 1)   | 11022
(1, 2), (2, 1), (3, 4), (4, 3)   | 11006

You would never think to do it this way in a real names-in-a-hat situation but you'd need to if you wanted to be fair.

#### Derangements    
    
At this point let's forget the hat metaphor:

    fun draw4(n: Int) : List<Pair<Int, Int>> {
        if (n < 4) throw IllegalArgumentException()
        val players = (1..n)
        val permutation = players.toMutableList()
        do Collections.shuffle(permutation)
            while (players.any { it == permutation[it - 1] })
        return players.map { Pair(it, permutation[it - 1]) }
    } 
       
We are generating permutations of the players and rejecting them if they are not [derangements](https://en.wikipedia.org/wiki/Derangement).  There were 9 derangements of 4 players.  Let's see how many we get for 5, 6 and 7 players:

    for (i in 5..7) {
        val patterns = mutableSetOf<List<Pair<Int, Int>>>()
        (1..100_000).forEach {
            val pattern = SecretSanta().draw4(i)
            patterns.add(pattern)
        }
        println("Found ${patterns.size} derangements for $i players")
    }
    
Players | Derangements
--------|-------------
5       | 44
6       | 265
7       | 1854
    
Of course, just doing it 100,000 times doesn't mean we hit all of the possible derangements.  To check:

    for (i in (4..11)) {
        val count = generatePermutations((0 until i).toList())
                .filter { perm -> (0 until i).none { perm[it] == it } }.count()
        println("Found $count derangements of $i elements")
    }
 
Players | Derangements
--------|-------------
4       | 9
5       | 44
6       | 265
7       | 1854
8       | 14833
9       | 133496
10      | 1334961
11      | 14684570
 
I'm using a library Steinhaus–Johnson–Trotter algorithm to generate permutations and it gets slow after 11 but we can see the sequence continuing in the [OEIS article](https://oeis.org/A000166).

We have not been tracking how many times we had to shuffle.  We can assume it's not too many times since, so far, we were able to perform draws quite quickly.  But will we have to shuffle more as we have more players?  Counting the shuffles and timing gives:

Players    | Time (ms) | Shuffles
-----------|-----------|---------
10         | 29        | 7 
100        | 0         | 1
1000       | 9         | 5
10000      | 36        | 3
100000     | 172       | 4
1000000    | 1822      | 2
10000000   | 158218    | 6
    
We don't have to shuffle more, but shuffling itself takes longer as we add players.  So the lack of extra shuffling suggests that chances of hitting a derangement remain about the same even as the number of permutations goes through the roof (there are n! permutations of n elements).  Extending the earlier table with the number of permutations

n       | Permutations | Derangements | Ratio
--------|--------------|--------------|--------
0       | 1            | 1            | 1
1       | 1            | 0            | -
2       | 2            | 1            | 2
3       | 6            | 2            | 3
4       | 24           | 9            | 2.66667
5       | 120          | 44           | 2.72727
6       | 720          | 265          | 2.71698
7       | 5040         | 1854         | 2.71845
8       | 40320        | 14833        | 2.71826
9       | 362880       | 133496       | 2.71828
10      | 3628800      | 1334961      | 2.71828
11      | 39916800     | 14684570     | 2.71828

We can see that the ratio levels off, at *e*.  See [the Wikipedia article](https://en.wikipedia.org/wiki/Derangement#Limit_of_ratio_of_derangement_to_permutation_as_n_approaches_.E2.88.9E) for details.

#### Cycles

The first time I organised a Secret Santa game one player succeeded in figuring out the assignments through detective work and confessions.  We had 16 players that year.

    Tom -> Alice -> Ashley -> Nora  
     |                          |
    Eric <- Laine <- Elron <- John
    
    
            Mark -> Ambs -> Andy 
              |               |  
            Laura <- Cory <- Indy
            
    
                    Derek --> Gerry 
                      `-----<----'
                     
Seeing this graph made me think about a few things:

1. It's fun to look at the assignments as graphs
2. It's possible we could have had one big cycle
3. It's possible we could have had 8 pairs

First let's look at this particular graph.  How likely was it that we would end up with 3 components?  To find the real answer we'd have to look at all derangements of 16 elements and there are 7,697,064,251,745 of those.  Instead I sampled 10 million at random:

Components | Occurences | Ratio of total
-----------|------------|---------------
8          | 3          |     0.000000
7          | 554        |     0.000055
6          | 17683      |     0.001768
5          | 211913     |     0.021191
4          | 1156218    |     0.115622
1          | 1698725    |     0.169873
3          | 3090215    |     0.309022
2          | 3824689    |     0.382469
    
So 3 components is common.  The pairs on the other hand will hardly ever happen which is good as that would be a weird game.  What about the particular pattern of a 2 node components, a 6 node and an 8 node?  From a different sample of 10 million:

Pattern                  | Occurences | Ratio of total
-------------------------|------------|---------------
[2, 2, 2, 2, 2, 2, 2, 2] |        3   |   0.000000
[2, 2, 2, 2, 2, 2, 4]    |      145   |   0.000015
[2, 2, 2, 2, 2, 3, 3]    |      420   |   0.000042
[2, 2, 2, 2, 2, 6]       |     1185   |   0.000119
[2, 2, 3, 3, 3, 3]       |     1755   |   0.000176
[2, 2, 2, 2, 4, 4]       |     2197   |   0.000220
[3, 3, 3, 3, 4]          |     3584   |   0.000358
[4, 4, 4, 4]             |     4445   |   0.000445
[2, 2, 2, 2, 3, 5]       |     4647   |   0.000465
[2, 2, 2, 3, 3, 4]       |     8069   |   0.000807
[2, 2, 4, 4, 4]          |     8857   |   0.000886
[2, 2, 2, 2, 8]          |     8879   |   0.000888
[2, 2, 2, 5, 5]          |    11398   |   0.001140
[2, 3, 3, 3, 5]          |    16774   |   0.001677
[2, 2, 2, 4, 6]          |    23567   |   0.002357
[2, 3, 3, 4, 4]          |    23729   |   0.002373
[3, 3, 3, 7]             |    23991   |   0.002399
[2, 2, 2, 3, 7]          |    27059   |   0.002706
[3, 3, 5, 5]             |    30141   |   0.003014
[2, 2, 3, 3, 6]          |    31520   |   0.003152
[2, 2, 6, 6]             |    47000   |   0.004700
[3, 4, 4, 5]             |    56214   |   0.005621
[2, 2, 2, 10]            |    56321   |   0.005632
[2, 2, 3, 4, 5]          |    56515   |   0.005652
[3, 3, 4, 6]             |    62915   |   0.006292
[2, 4, 5, 5]             |    68133   |   0.006813
[2, 4, 4, 6]             |    70954   |   0.007095
[5, 5, 6]                |    90871   |   0.009087
[4, 6, 6]                |    94380   |   0.009438
[2, 3, 3, 8]             |    94532   |   0.009453
[2, 2, 5, 7]             |    96845   |   0.009685
[4, 4, 8]                |   106051   |   0.010605
[2, 2, 4, 8]             |   106113   |   0.010611
[2, 2, 3, 9]             |   126166   |   0.012617
[2, 7, 7]                |   138703   |   0.013870
[2, 3, 5, 6]             |   150604   |   0.015060
[3, 3, 10]               |   150766   |   0.015077
[2, 3, 4, 7]             |   162452   |   0.016245
[4, 5, 7]                |   194590   |   0.019459
[8, 8]                   |   212145   |   0.021215
[3, 6, 7]                |   215861   |   0.021586
[3, 5, 8]                |   227178   |   0.022718
[3, 4, 9]                |   251454   |   0.025145
[2, 6, 8]                |   282413   |   0.028241
[2, 2, 12]               |   282690   |   0.028269
[2, 5, 9]                |   301765   |   0.030177
[2, 4, 10]               |   340009   |   0.034001
[2, 3, 11]               |   411921   |   0.041192
[7, 9]                   |   431182   |   0.043118
[6, 10]                  |   453110   |   0.045311
[5, 11]                  |   494113   |   0.049411
[4, 12]                  |   566147   |   0.056615
[3, 13]                  |   696411   |   0.069641
[2, 14]                  |   971812   |   0.097181
[16]                     |  1699299   |   0.169930
    
It's one of the less rare ones, and happens around 3% of the time.  For fun let's try and work out the probability of hitting the complete single cycle of 16 to see if it matches our experience sampling.  To make the complete cycle, any element can be "first" so we can say there is one way of picking it.  Then the first element can pair with any other but itself, so there are 15 ways of doing that.  Then the next just has to avoid the first and itself, so there are 14 ways of doing that.  The next has 13 then 12 then 11 all the way to the last element which can only match to the first.

    1 * 15 * 14 * 13 * 12 * 11 * 10 * 9 * 8 * 7 * 6 * 5 * 4 * 3 * 2 * 1 
        = 1307674368000
    
So 1,307,674,368,000 ways to make the single cycle.  Divided by the total number of possible derangements:

    1307674368000 / 7697064251745 =~ 0.169893
    
Close enough.  What about the probability of hitting the 8 cycle assignment.  How to count these ways?  I came up with this way to count the number of possible pairings for n players:

    fun countPairedPermutations(n: Int) : Int {
        if (n < 4 || (n % 2) != 0) {
            throw IllegalArgumentException()
        }
        if (n == 4) {
            return 3
        }
        else {
            return (n - 1) * (countPairedPermutations(n - 2))
        }
    }

The idea of the recursion is that the first element can be mapped to any of the remaining n - 1 elements, so there are n - 1 different ways to do that.  Each of those ways then has the n - 2 case number of ways to continue.  This produces:

Players | Paired derangements
--------|--------------------
 4      |       3
 6      |      15
 8      |     105
 10     |     945
 12     |   10395
 14     |  135135
 16     | 2027025
    
This appears to be [this sequence](https://oeis.org/A001147), though I couldn't make sense of the descriptions there.  The figure 2027025 gives the actual probability for the 8 pairs of 16 players as:

    2027025 / 7697064251745 =~ 2.63 / 10 million
    
So finding three earlier was just right!  With more time on the train just now I ran 100 million and got 28.  Shout out to anyone who has experienced paired up Secret Santas with a large number of players -- I hope you deanonymized it and found out how lucky you were!

#### Back to the hat

Back in the real world, we still have a problem in that the hat method has become annoying to implement, what with the high probability of telling people to restart multiple times.  

There's a real world method described [in this video](https://www.youtube.com/watch?v=GhnCj7Fvqt0) which can come up with a valid assignment in one pass and claims to achieve anonymity and fairness.  The idea is you shuffle once and then each participant is assigned to its neighbour in the shuffle with the last in line looping around to be assigned to the first.  The video explains how to make this work with real people and paper.  In code it looks like:

    fun draw5(n: Int) : List<Pair<Int, Int>> {
        if (n < 4) throw IllegalArgumentException()
        val players = (1..n)
        val permutation = players.toMutableList()
        Collections.shuffle(permutation)
        return (0 until n).map {
            Pair(permutation[it], permutation[(it + 1) % n])
        }
    }
    
It's guaranteed to create a derangement and it's uniformly random over its possibilities.  A problem is that the possibilities themselves are restricted to complete cycles, meaning that we would be missing most derangements.  This might not matter if everyone is getting bubble bath and no one cares to guess who got what, but if you enjoy the detective work part then it's less fun this way.  On the other hand, a cycle is quite pleasing.  I did not adopt this method for my program.

#### Smarter shuffling

So far I didn't worry about shuffling, I just used a library shuffle.  What if I write my own shuffle and adapt it to avoid non-derangements?  Then I could avoid ever reshuffling.  The shuffle I was using, java.util.Collections.shuffle, does something like this:

    fun shuffle(n: Int, random: Random) : List<Int> {
        val list = (1..n).toMutableList()
        for (i in list.size downTo 2) {
            list.set(i - 1, list.set(random.nextInt(i), list[i - 1]))
        }
        return list
    }    
    
This is a [Fisher-Yates shuffle](https://en.wikipedia.org/wiki/Fisher–Yates_shuffle).  My attempts to adapt it didn't end well and it seems this is a [tricky problem](https://stackoverflow.com/questions/7279895/shuffle-list-ensuring-that-no-item-remains-in-same-position).  One failed attempt led me to the following:

    fun derange(n: Int, random: Random) : List<Int> {
        val list = (1..n).toMutableList()
        for (i in list.size downTo 2) {
            list.set(i - 1, list.set(random.nextInt(i - 1), list[i - 1]))
        }
        return list
    }
    
which does produce valid derangements, but only complete cycles.  This is called [Sattolo's algorithm](https://danluu.com/sattolo/) and would be an alternative to the draw5() method above.

There is an algorithm described [here](http://epubs.siam.org/doi/pdf/10.1137/1.9781611972986.7) which does produce random derangements without trial and error of permutations.  Here it is, more or less as described in the paper:

    fun randomDerangement(n: Int) : List<Int> {
        fun numDerangements(n: Int): BigDecimal {
            return when (n) {
                0 -> BigDecimal.ONE
                1 -> BigDecimal.ZERO
                else -> BigDecimal.valueOf(n.toLong()).minus(BigDecimal.ONE)
                    .multiply(numDerangements(n - 1).plus(numDerangements(n - 2)))
            }
        }
        val random = Random()
        val A = (1..n).toMutableList()
        val marked = (0 until n).map { false }.toBooleanArray()
        var i = n - 1
        var u = n
        while (u >= 2) {
            if (!marked[i]) {
                var j = random.nextInt(i)
                while (marked[j]) {
                    j = random.nextInt(i)
                }
                A[i] = A.set(j, A[i])
                val p = random.nextDouble()
                val threshold =
                        BigDecimal.valueOf((u - 1).toLong())
                        .multiply(numDerangements(u - 2))
                        .divide(numDerangements(u), MathContext.DECIMAL64).toDouble()
                if (p < threshold) {
                    marked[j] = true
                    u--
                }
                u--
            }
            i--
        }
        return A
    }
    
The key insight is to work out the probability that an exchanged element should subsequently remain fixed (these are the "marked" indices), such that all derangements end up equally likely.  This probability is related to the number of possible derangements which we calculate using the recursion also discussed in the paper.  There's a trial and reject loop in the middle to select the next unfixed element to swap with.  In my version I keep the unmarked indices in a set to avoid the trial and reject method.  I am also memoizing the derangement counts:

    object RandomDerangementCalculator {

        private val random = Random()
        private val derangementCounts = mutableMapOf<Int, BigDecimal>()
    
        private fun derangementCount(n: Int) : BigDecimal {
            return derangementCounts.getOrPut(n, {
                when (n) {
                    0 -> BigDecimal.ONE
                    1 -> BigDecimal.ZERO
                    else -> BigDecimal.valueOf(n.toLong())
                            .minus(BigDecimal.ONE)
                            .multiply(
                                    derangementCount(n - 1)
                                    .plus(derangementCount(n - 2)))
                }
            })
        }
    
        private fun Set<Int>.nth(n: Int) : Int {
            if (this.size < n || n < 0) {
                throw IllegalArgumentException()
            }
            var i = 0
            return this.stream().dropWhile { i++ < n }.findFirst().get()
        }
    
        fun derange(n: Int) : List<Int> {
            val result = (1..n).toMutableList()
            val unmarkedIndices = (0 until n).toMutableSet()
            for (i in n - 1 downTo 0) {
                if (unmarkedIndices.size < 2) {
                    break
                }
                if (unmarkedIndices.contains(i)) {
                    var j = unmarkedIndices.nth(random.nextInt(unmarkedIndices.size - 1))
                    result[i] = result.set(j, result[i])
                    val probability = random.nextDouble()
                    val threshold =
                            BigDecimal.valueOf((unmarkedIndices.size - 1).toLong())
                                    .multiply(derangementCount(unmarkedIndices.size - 2))
                                    .divide(derangementCount(unmarkedIndices.size), MathContext.DECIMAL64).toDouble()
                    if (probability < threshold) {
                        unmarkedIndices.remove(j)
                    }
                    unmarkedIndices.remove(i)
                }
            }
            return result
        }
    }

This appears to be correct under some testing I did, but I did not adopt this method for my Secret Santa program.

My Secret Santa program is [here](https://github.com/snakey1980/secretsanta).