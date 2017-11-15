---
title: "Secret Santa"
date: 2017-11-13T19:49:44-05:00
draft: true
---

#### Names in a hat

The first time I played Secret Santa we assigned givers/receivers like this:

1. Players put their name in a hat
2. Players take turn to shake the hat and pick a name out at random.  If you pull your own name, put it back and pull again.

So this is how I tried to implement it.  Here we use plain integers as the players:

    fun draw(players: Set<Int>) : List<Pair<Int, Int>> {
        if (players.size < 4) throw IllegalArgumentException()
        val pot = players.toMutableList()
        Collections.shuffle(pot)
        val result = mutableListOf<Pair<Int, Int>>()
        players.forEach {
            while (pot[0] == it) {
                Collections.shuffle(pot)
            }
            result.add(Pair(it, pot.removeAt(0)))
        }
        return result
    }
    
On a first try it seems to work:

    println(SecretSanta().draw(setOf(1, 2, 3, 4)))
    // printed [(1, 4), (2, 1), (3, 2), (4, 3)]
    
but try it a few times and, on some runs, it will never finish.  It's not hard to see why -- it's possible that when number 4 comes to choose, it finds only itself in the pot and reshuffles forever.

This could happen in a real names-in-a-hat situation too.  When it does, there's nothing to be done but to put all the names back in and try again, so that's what I tried next:

    fun draw2(players: Set<Int>) : List<Pair<Int, Int>> {
        class BadPotException : RuntimeException()
        if (players.size < 4) throw IllegalArgumentException()
        while (true) {
            val pot = players.toMutableList()
            Collections.shuffle(pot)
            try {
                val result = mutableListOf<Pair<Int, Int>>()
                players.forEach {
                    if (pot.size == 1 && pot[0] == it) {
                        throw BadPotException()
                    }
                    while (pot[0] == it) {
                        Collections.shuffle(pot)
                    }
                    result.add(Pair(it, pot.removeAt(0)))
                }
                return result
            }
            catch (e: BadPotException) {
                // ok, try again
            }
        }
    }
    
This always seems to stop, so we are doing slightly better:

    (1..100_000).forEach { SecretSanta().draw2(setOf(1, 2, 3, 4)) }
    
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
    
A good test would be to see if our procedure returns each of the nine, and that it returns each one one ninth of the time:

    val patterns = mutableMapOf<List<Pair<Int, Int>>, Int>()
    (1..100_000).forEach {
        val pattern = SecretSanta().draw2(setOf(1, 2, 3, 4))
        patterns[pattern] = patterns.getOrDefault(pattern, 0) + 1
    }
    patterns.entries.sortedByDescending { it.value }.forEach { println(it) }
        
We run it 100,000 times and would expect each pattern to occur around 11111 times.  But:

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

    fun draw3(players: Set<Int>) : List<Pair<Int, Int>> {
        class BadPotException : RuntimeException()
        if (players.size < 4) throw IllegalArgumentException()
        while (true) {
            val pot = players.toMutableList()
            Collections.shuffle(pot)
            try {
                val result = mutableListOf<Pair<Int, Int>>()
                players.forEach {
                    if (pot[0] == it) {
                        throw BadPotException()
                    }
                    result.add(Pair(it, pot.removeAt(0)))
                }
                return result
            }
            catch (e: BadPotException) {
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

#### Derangements    
    
At this point let's forget the hat metaphor and more honestly represent what we are doing:

    fun draw4(players: Set<Int>) : List<Pair<Int, Int>> {
        if (players.size < 4) throw IllegalArgumentException()
        val permutation = players.toMutableList()
        do Collections.shuffle(permutation)
            while (players.any { it == permutation[it - 1] })
        return players.map { Pair(it, permutation[it - 1]) }
    }
    
We are generating permutations of the players and rejecting them if they are not "derangements" (https://en.wikipedia.org/wiki/Derangement).  We found there were 9 derangements of 4 players.  Let's see how many we get for 5, 6 and 7 players:

    for (i in 5..7) {
        val patterns = mutableSetOf<List<Pair<Int, Int>>>()
        (1..100_000).forEach {
            val pattern = SecretSanta().draw4((1..i).toSet())
            patterns.add(pattern)
        }
        println("Found ${patterns.size} derangements for $i players")
    }
    
    Found 44 derangements for 5 players
    Found 265 derangements for 6 players
    Found 1854 derangements for 7 players
    
Of course, just doing it 100,000 times doesn't mean we hit all of the possible derangements.  But we can take a look at https://oeis.org/A000166 and see that we did indeed do it.  If we wanted to guarantee it we could generate all permutations and check them.

We have not been tracking how many times we had to shuffle.  We can assume it's not too many times since, so far, we were able to perform draws quite quickly.  But will we have to shuffle more as we have more players?  Counting the shuffles and timing gives:

    10 players took 29ms and performed 7 shuffles
    100 players took 0ms and performed 1 shuffles
    1000 players took 9ms and performed 5 shuffles
    10000 players took 36ms and performed 3 shuffles
    100000 players took 172ms and performed 4 shuffles
    1000000 players took 1822ms and performed 2 shuffles
    10000000 players took 158218ms and performed 6 shuffles
    
We don't have to shuffle more at all, but shuffling itself must take longer, as we would expect.  On the last one my laptop lost its mind.  So the lack of shuffling suggests that derangements remain fairly common even as the number of permutations goes through the roof (there are n! permutations of n elements).  It turns out (see https://en.wikipedia.org/wiki/Derangement again) that the ratio of permutations to derangements tends to 1/e as n goes to infinity.  So we don't need to worry about that.

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

So how likely was the 3 component

7,697,064,251,745