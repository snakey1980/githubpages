---
title: "Pirates"
date: 2017-12-22T16:45:09-05:00
draft: false
---

Like [Cheryl's Birthday](https://snakey1980.github.io/posts/cheryl/) this [Pirates Problem](https://en.wikipedia.org/wiki/Pirate_game) is more fun to program than to work out by hand.  The problem goes like this: 

> It happens that pirates often acquire coins and need to share them out amongst themselves.  The way they do this is that the captain -- there is always strict seniority within any group of pirates -- proposes a distribution and the pirates vote on it, the captain having a casting vote.  If the distribution is approved it is implemented, otherwise a mutiny ensues and the new captain proposes a new distribution.  

> Pirates are logical and don't trust each other and their favorite things are, in order: staying alive, getting coins, and killing other pirates.

> In the case of 5 pirates and 100 coins, how will the captain propose to distribute them?

So we are trying to implement

    fun proposeDistribution(coins: Int, pirates: Int) : List<Int>

which will return the captain's proposal given the numbers of coins and pirates. 

No explicit working backwards is required, we just need to think like a Pirate.  Given his coins, the Pirate will ask himself "what is the minimum I have to do to get the votes I need and not get killed?" which leads him to think like a subordinate and ask himself "what happens if the captain dies and what could the captain give me that would be better?".  The captain can then work out the "price" of each vote and "buy" as many as he needs to, starting with the cheapest.

So we start off assuming we get all the coins and everyone else gets zero.  As we go along we can move coins from our position to others.

    val result = listOf(coins).plus((1..(pirates - 1)).map { 0 }).toMutableList()  // e.g. [100, 0, 0, 0, 0]
    
Next, how many votes do we need to buy?

    val votesNeeded = (pirates - 1) / 2
    
This could be 0 if there are only one or two pirates.  Otherwise, we need to buy some votes:

    val caseIfWeDie = proposeDistribution(coins, pirates - 1).toList()
    val cheapestVotes = caseIfWeDie.mapIndexed { index, i -> Pair(index, i) }
                .sortedBy { it.second }.map { it.first }.take(votesNeeded)
    
So we calculate what happens if we die, then order our subordinates by how well they do out of that, from worst to best.  The ones who would do worst are the cheapest votes.  To buy any vote, we need to give them more money than they would get in the us dying case:

    cheapestVotes.forEach {
        val valueToTurnVote = caseIfWeDie[it] + 1
        if (result[0] >= valueToTurnVote) {
            result[it + 1] = valueToTurnVote
            result[0] -= valueToTurnVote
        }
    }    
    
We take the vote buying money out of our cut, so long as we have enough.  And we're done!  Putting it all together:

    fun proposeDistribution(coins: Int, pirates: Int) : List<Int> {
        val result = listOf(coins).plus((1..(pirates - 1)).map { 0 }).toMutableList()
        val votesNeeded = (pirates - 1) / 2
        if (votesNeeded > 0) {
            val caseIfWeDie = distribute(coins, pirates - 1).toList()
            val cheapestVotes = caseIfWeDie.mapIndexed { index, i -> Pair(index, i) }
                    .sortedBy { it.second }.map { it.first }.take(votesNeeded)
            cheapestVotes.forEach {
                val valueToTurnVote = caseIfWeDie[it] + 1
                if (result[0] >= valueToTurnVote) {
                    result[it + 1] = valueToTurnVote
                    result[0] -= valueToTurnVote
                }
            }
        }
        return result
    }
    
And here's the answer to the 5, 100 case:

    println(proposeDistribution(100, 5)) // prints [98, 0, 1, 0, 1]



