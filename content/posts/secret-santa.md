---
title: "Secret Santa"
date: 2017-11-13T19:49:44-05:00
draft: true
---

# Secret Santa

I wrote a program to assign givers and receivers in a game of Secret Santa and learnt a few things along the way.

## Generating assignments

The first time I played Secret Santa we assigned givers like this:

1. Players put their name in a hat
2. Players take turn to pull names out.  If you pull your own name, put it back and pull again.

So this is how I tried to implement it.  Here we use plain integers as the players, 1 thru n:

    internal fun draw(n: Int) : List<Pair<Int, Int>> {
        val pot = (1..n).toMutableList()
        Collections.shuffle(pot)
        val result = mutableListOf<Pair<Int, Int>>()
        (1..n).forEach {
            while (pot[0] == it) {
                Collections.shuffle(pot)
            }
            result.add(Pair(it, pot.removeAt(0)))
        }
        return result
    }
    
On a first try it seems to work:

    println(SecretSanta().draw(4)) 
    // prints [(1, 4), (2, 3), (3, 1), (4, 2)]
    
but try it a few times and, on some runs it will never finish.