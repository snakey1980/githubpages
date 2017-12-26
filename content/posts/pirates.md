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

No explicit working backwards is required, we just need to think like a pirate.  Given his coins, the captain will ask himself "what is the minimum I have to do to get the votes I need and not get killed?" which leads him to think like a subordinate and ask himself "what happens if the captain dies and what could the captain give me that would be better?".  The captain can then work out the "price" of each vote and "buy" as many as he needs to, starting with the cheapest.

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
    
In this case the proposal is the best one possible and also will not result in a mutiny.  But it's possible that the captain can do his best but still come up with a distribution that produces a mutiny.  Let's say we have just one coin:



<table>
<thead>
<tr>
<th>Pirates</th>
<th style="text-align:right">Proposal</th>
<th>Votes for - against</th>
<th>Mutiny?</th>
</tr>
</thead>

<tbody>

<tr>
<td>7</td>
<td style="text-align:right">[0, 1, 0, 0, 0, 0, 0]</td>
<td>2 - 5</td>
<td>yes</td>
</tr>

<tr>
<td>6</td>
<td style="text-align:right">[0, 1, 0, 0, 0, 0]</td>
<td>2 - 4</td>
<td>yes</td>
</tr>

<tr>
<td>5</td>
<td style="text-align:right">[0, 1, 0, 0, 0]</td>
<td>2 - 3</td>
<td>yes</td>
</tr>

<tr>
<td>4</td>
<td style="text-align:right">[0, 1, 0, 0]</td>
<td>2 - 2</td>
<td>no</td>
</tr>

<tr>
<td>3</td>
<td style="text-align:right">[0, 0, 1]</td>
<td>2 - 1</td>
<td>no</td>
</tr>

<tr>
<td>2</td>
<td style="text-align:right">[1, 0]</td>
<td>1 - 1</td>
<td>no</td>
</tr>

<tr>
<td>1</td>
<td style="text-align:right">[1]</td>
<td>1 - 0</td>
<td>no</td>
</tr>
</tbody>
</table>

If there are 5 or more pirates then we can't avoid a mutiny, and giving the coin to the first subordinate is pointless and just happens to come out of our algorithm because of the order of our pirates.

In these cases the captain isn't getting any of his three favorite things.  We need to know his fourth favorite thing to determine what the algorithm should do.  For example we could say

> After staying alive, getting money and killing pirates, a pirate's favorite thing is expressing greed through defiant, provocative gestures

We can add this to our implementation by calculating whether or not we have enough coins to buy all the votes we need and only distributing coins if we do:

    fun proposeDistribution(coins: Int, pirates: Int) : List<Int> {
        val result = listOf(coins).plus((1..(pirates - 1)).map { 0 }).toMutableList()
        val votesNeeded = (pirates - 1) / 2
        if (votesNeeded > 0) {
            val caseIfWeDie = proposeDistribution(coins, pirates - 1).toList()
            val cheapestVotesAndValuesToTurnThem = caseIfWeDie.mapIndexed { index, i -> Pair(index, i + 1) }
                    .sortedBy { it.second }.take(votesNeeded)
            if (cheapestVotesAndValuesToTurnThem.map { it.second }.sum() <= result[0]) {
                cheapestVotesAndValuesToTurnThem.forEach {
                    if (result[0] >= it.second) {
                        result[it.first + 1] = it.second
                        result[0] -= it.second
                    }
                }
            }
        }
        return result
    }
    
Then:

    println(proposeDistribution(1, 7)) // prints [1, 0, 0, 0, 0, 0, 0]