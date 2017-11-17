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

    [(1, 2), (2, 1), (3, 4), (4, 3)]
    [(1, 2), (2, 3), (3, 4), (4, 1)]
    [(1, 2), (2, 4), (3, 1), (4, 3)]
    [(1, 3), (2, 1), (3, 4), (4, 2)]
    [(1, 3), (2, 4), (3, 1), (4, 2)]
    [(1, 3), (2, 4), (3, 2), (4, 1)]
    [(1, 4), (2, 1), (3, 2), (4, 3)]
    [(1, 4), (2, 3), (3, 1), (4, 2)]
    [(1, 4), (2, 3), (3, 2), (4, 1)]
    
A good test would be to see if our procedure returns each of the nine, and that it returns each one 1/9 of the time:

    val patterns = mutableMapOf<List<Pair<Int, Int>>, Int>()
    (1..100_000).forEach {
        val pattern = SecretSanta().draw2(4)
        patterns[pattern] = patterns.getOrDefault(pattern, 0) + 1
    }
    patterns.entries.sortedByDescending { it.value }.forEach { println(it) }
        
We run it 100,000 times and would expect each pattern to occur around 11,111 times.  But:

    [(1, 4), (2, 1), (3, 2), (4, 3)]=19241
    [(1, 2), (2, 4), (3, 1), (4, 3)]=13004
    [(1, 2), (2, 1), (3, 4), (4, 3)]=12943
    [(1, 3), (2, 4), (3, 2), (4, 1)]=9775
    [(1, 4), (2, 3), (3, 1), (4, 2)]=9720
    [(1, 4), (2, 3), (3, 2), (4, 1)]=9685
    [(1, 3), (2, 4), (3, 1), (4, 2)]=9591
    [(1, 3), (2, 1), (3, 4), (4, 2)]=9532
    [(1, 2), (2, 3), (3, 4), (4, 1)]=6509
    
The algorithm is biased.  We are giving special treatment to the last pick and this is skewing the distribution.  So, what if we reject the draw as soon as we find an element in the wrong place:

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

    [(1, 2), (2, 4), (3, 1), (4, 3)]=11221
    [(1, 4), (2, 3), (3, 1), (4, 2)]=11203
    [(1, 3), (2, 1), (3, 4), (4, 2)]=11176
    [(1, 4), (2, 1), (3, 2), (4, 3)]=11151
    [(1, 3), (2, 4), (3, 1), (4, 2)]=11092
    [(1, 3), (2, 4), (3, 2), (4, 1)]=11087
    [(1, 4), (2, 3), (3, 2), (4, 1)]=11042
    [(1, 2), (2, 3), (3, 4), (4, 1)]=11022
    [(1, 2), (2, 1), (3, 4), (4, 3)]=11006

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
    
    Found 44 derangements for 5 players
    Found 265 derangements for 6 players
    Found 1854 derangements for 7 players
    
Of course, just doing it 100,000 times doesn't mean we hit all of the possible derangements.  To check:

    for (i in (4..11)) {
        val count = generatePermutations((0 until i).toList())
                .filter { perm -> (0 until i).none { perm[it] == it } }.count()
        println("Found $count derangements of $i elements")
    }
 
    Found 9 derangements of 4 elements
    Found 44 derangements of 5 elements
    Found 265 derangements of 6 elements
    Found 1854 derangements of 7 elements
    Found 14833 derangements of 8 elements
    Found 133496 derangements of 9 elements
    Found 1334961 derangements of 10 elements
    Found 14684570 derangements of 11 elements

I'm using a library Steinhaus–Johnson–Trotter algorithm to generate permutations and it gets slow after 11 but we can see the sequence continuing in the [OEIS article](https://oeis.org/A000166).

We have not been tracking how many times we had to shuffle.  We can assume it's not too many times since, so far, we were able to perform draws quite quickly.  But will we have to shuffle more as we have more players?  Counting the shuffles and timing gives:

          10 players took     29ms and performed 7 shuffles
         100 players took      0ms and performed 1 shuffles
        1000 players took      9ms and performed 5 shuffles
       10000 players took     36ms and performed 3 shuffles
      100000 players took    172ms and performed 4 shuffles
     1000000 players took   1822ms and performed 2 shuffles
    10000000 players took 158218ms and performed 6 shuffles
    
We don't have to shuffle more, but shuffling itself takes longer as we add players.  So the lack of shuffling suggests that derangements remain fairly common even as the number of permutations goes through the roof (there are n! permutations of n elements).  [It turns out](https://en.wikipedia.org/wiki/Derangement#Limit_of_ratio_of_derangement_to_permutation_as_n_approaches_.E2.88.9E) that the ratio of permutations to derangements ends up approximately 1/e.

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

1. It's fun to look at the assignments as cyclic graphs
2. It's possible we could have had one big cycle
3. It's possible we could have had 8 pairs

First let's look at this particular graph.  How likely was it that we would end up with 3 components?  To find the real answer we'd have to look at all derangements of 16 elements and there are 7,697,064,251,745 of those.  Instead I sampled 10 million at random:

          3 instances of 8 components (ratio 0.000000)
        554 instances of 7 components (ratio 0.000055)
      17683 instances of 6 components (ratio 0.001768)
     211913 instances of 5 components (ratio 0.021191)
    1156218 instances of 4 components (ratio 0.115622)
    1698725 instances of 1 components (ratio 0.169873)
    3090215 instances of 3 components (ratio 0.309022)
    3824689 instances of 2 components (ratio 0.382469)
    
So 3 components is common.  The pairs on the other hand will hardly ever happen which is good as that would be a weird game.  What about the particular pattern of a 2 node components, a 6 node and an 8 node?  From a different sample of 10 million:

          3 instances of [2, 2, 2, 2, 2, 2, 2, 2] (ratio 0.000000)
        145 instances of [2, 2, 2, 2, 2, 2, 4]    (ratio 0.000015)
        420 instances of [2, 2, 2, 2, 2, 3, 3]    (ratio 0.000042)
       1185 instances of [2, 2, 2, 2, 2, 6]       (ratio 0.000119)
       1755 instances of [2, 2, 3, 3, 3, 3]       (ratio 0.000176)
       2197 instances of [2, 2, 2, 2, 4, 4]       (ratio 0.000220)
       3584 instances of [3, 3, 3, 3, 4]          (ratio 0.000358)
       4445 instances of [4, 4, 4, 4]             (ratio 0.000445)
       4647 instances of [2, 2, 2, 2, 3, 5]       (ratio 0.000465)
       8069 instances of [2, 2, 2, 3, 3, 4]       (ratio 0.000807)
       8857 instances of [2, 2, 4, 4, 4]          (ratio 0.000886)
       8879 instances of [2, 2, 2, 2, 8]          (ratio 0.000888)
      11398 instances of [2, 2, 2, 5, 5]          (ratio 0.001140)
      16774 instances of [2, 3, 3, 3, 5]          (ratio 0.001677)
      23567 instances of [2, 2, 2, 4, 6]          (ratio 0.002357)
      23729 instances of [2, 3, 3, 4, 4]          (ratio 0.002373)
      23991 instances of [3, 3, 3, 7]             (ratio 0.002399)
      27059 instances of [2, 2, 2, 3, 7]          (ratio 0.002706)
      30141 instances of [3, 3, 5, 5]             (ratio 0.003014)
      31520 instances of [2, 2, 3, 3, 6]          (ratio 0.003152)
      47000 instances of [2, 2, 6, 6]             (ratio 0.004700)
      56214 instances of [3, 4, 4, 5]             (ratio 0.005621)
      56321 instances of [2, 2, 2, 10]            (ratio 0.005632)
      56515 instances of [2, 2, 3, 4, 5]          (ratio 0.005652)
      62915 instances of [3, 3, 4, 6]             (ratio 0.006292)
      68133 instances of [2, 4, 5, 5]             (ratio 0.006813)
      70954 instances of [2, 4, 4, 6]             (ratio 0.007095)
      90871 instances of [5, 5, 6]                (ratio 0.009087)
      94380 instances of [4, 6, 6]                (ratio 0.009438)
      94532 instances of [2, 3, 3, 8]             (ratio 0.009453)
      96845 instances of [2, 2, 5, 7]             (ratio 0.009685)
     106051 instances of [4, 4, 8]                (ratio 0.010605)
     106113 instances of [2, 2, 4, 8]             (ratio 0.010611)
     126166 instances of [2, 2, 3, 9]             (ratio 0.012617)
     138703 instances of [2, 7, 7]                (ratio 0.013870)
     150604 instances of [2, 3, 5, 6]             (ratio 0.015060)
     150766 instances of [3, 3, 10]               (ratio 0.015077)
     162452 instances of [2, 3, 4, 7]             (ratio 0.016245)
     194590 instances of [4, 5, 7]                (ratio 0.019459)
     212145 instances of [8, 8]                   (ratio 0.021215)
     215861 instances of [3, 6, 7]                (ratio 0.021586)
     227178 instances of [3, 5, 8]                (ratio 0.022718)
     251454 instances of [3, 4, 9]                (ratio 0.025145)
     282413 instances of [2, 6, 8]                (ratio 0.028241)
     282690 instances of [2, 2, 12]               (ratio 0.028269)
     301765 instances of [2, 5, 9]                (ratio 0.030177)
     340009 instances of [2, 4, 10]               (ratio 0.034001)
     411921 instances of [2, 3, 11]               (ratio 0.041192)
     431182 instances of [7, 9]                   (ratio 0.043118)
     453110 instances of [6, 10]                  (ratio 0.045311)
     494113 instances of [5, 11]                  (ratio 0.049411)
     566147 instances of [4, 12]                  (ratio 0.056615)
     696411 instances of [3, 13]                  (ratio 0.069641)
     971812 instances of [2, 14]                  (ratio 0.097181)
    1699299 instances of [16]                     (ratio 0.169930)
    
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

    4  ->       3
    6  ->      15
    8  ->     105
    10 ->     945
    12 ->   10395
    14 ->  135135
    16 -> 2027025
    
This appears to be [this sequence](https://oeis.org/A001147), though I couldn't make sense of the descriptions there.  The figure 2027025 gives the actual probability for the 8 pairs of 16 players as:

    2027025 / 7697064251745 =~ 2.63 / 10 million
    
So finding three earlier was just right!  With more time on the train just now I ran 100 million and got 28.  Shout out to anyone who has experienced paired up Secret Santas with a large number of players -- I hope you de-anonymised it and found out how lucky you were!

#### Back to the hat

Back in the real world, we still have a problem in that the hat method has become annoying to implement, what with the high probability of telling people to restart multiple times.  There's a real world method described [in this video](https://www.youtube.com/watch?v=GhnCj7Fvqt0) which can come up with a valid assignment in one pass.  The idea is you shuffle once and then each participant is assigned to its neighbour in the shuffle with the last in line looping around to be assigned to the first.  The video explains how to make this work with real people and paper.  In code it looks like:

    fun draw5(n: Int) : List<Pair<Int, Int>> {
        if (n < 4) throw IllegalArgumentException()
        val players = (1..n)
        val permutation = players.toMutableList()
        return (0 until n).map { 
            Pair(permutation[it], permutation[(it + 1) % n])
        }
    }

It's guaranteed to create a derangement and it's uniformly random over its possibilities.  A problem is that the possibilities themselves are restricted to complete cycles, meaning that we would be missing most derangements.  This might not matter if everyone is getting bubble bath and no one cares to guess who got what, but it somewhat spoils the detective work part.  On the other hand, a cycle is quite pleasing.  I did not adopt this method for my program.

#### Smarter shuffling

So far I didn't worry about shuffling, I just used a library shuffle.  What if I write my own shuffle and adapt it to avoid non-derangements?  Then I could avoid ever reshuffling.  The shuffle I was using, java.util.Collections.shuffle, does something like this:

    fun shuffle(n: Int, random: Random) : List<Int> {
        val list = (1..n).toMutableList()
        for (i in list.size downTo 2) {
            list.set(i - 1, list.set(random.nextInt(i), list[i - 1]))
        }
        return list
    }    
    
This is a (Fisher-Yates shuffle)[https://en.wikipedia.org/wiki/Fisher–Yates_shuffle].  My attempts to adapt it didn't end well and it seems this is a [tricky problem](https://stackoverflow.com/questions/7279895/shuffle-list-ensuring-that-no-item-remains-in-same-position).  One failed attempt led me to the following:

    fun derange(n: Int, random: Random) : List<Int> {
        val list = (1..n).toMutableList()
        for (i in list.size downTo 2) {
            list.set(i - 1, list.set(random.nextInt(i - 1), list[i - 1]))
        }
        return list
    }
    
which does produce valid derangements, but only complete cycles.  This is called [Sattolo's algorithm](https://danluu.com/sattolo/) and would be an alternative to the draw5() method above.

My Secret Santa program is [here](https://github.com/snakey1980/secretsanta).