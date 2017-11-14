---
title: "Secret Santa"
date: 2017-11-13T19:49:44-05:00
draft: true
---

#### Picking names from a hat

The first time I played Secret Santa we assigned givers/receivers like this:

1. Players put their name in a hat
2. Players take turn to pull names out.  If you pull your own name, put it back and pull again.

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
    
but try it a few times and, on some runs, it will never finish.  It's not hard to see why -- it's possible that when number 4 comes to choose, it finds only itself in the pot and reshuffles uselessly forever.

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
    
This way always seems to stop, so we are doing slightly better:

    (1..100_000).forEach { SecretSanta().draw2(setOf(1, 2, 3, 4)) }
    
With 4 players, there aren't many valid assignments.  We can list them:

               ASSIGNMENT            VALID?

    [(1, 1), (2, 2), (3, 3), (4, 4)]   no
    [(1, 1), (2, 2), (3, 4), (4, 3)]   no
    [(1, 1), (2, 3), (3, 2), (4, 4)]   no
    [(1, 1), (2, 3), (3, 4), (4, 2)]   no
    [(1, 1), (2, 4), (3, 2), (4, 3)]   no
    [(1, 1), (2, 4), (3, 3), (4, 2)]   no
    [(1, 2), (2, 1), (3, 3), (4, 4)]   no
    [(1, 2), (2, 1), (3, 4), (4, 3)]   yes
    [(1, 2), (2, 3), (3, 1), (4, 4)]   no
    [(1, 2), (2, 3), (3, 4), (4, 1)]   yes
    [(1, 2), (2, 4), (3, 1), (4, 3)]   yes
    [(1, 2), (2, 4), (3, 3), (4, 1)]   no
    [(1, 3), (2, 1), (3, 2), (4, 4)]   no
    [(1, 3), (2, 1), (3, 4), (4, 2)]   yes
    [(1, 3), (2, 2), (3, 1), (4, 4)]   no
    [(1, 3), (2, 2), (3, 4), (4, 1)]   no
    [(1, 3), (2, 4), (3, 1), (4, 2)]   yes
    [(1, 3), (2, 4), (3, 2), (4, 1)]   yes
    [(1, 4), (2, 1), (3, 2), (4, 3)]   yes
    [(1, 4), (2, 1), (3, 3), (4, 2)]   no
    [(1, 4), (2, 2), (3, 1), (4, 3)]   no
    [(1, 4), (2, 2), (3, 3), (4, 1)]   no
    [(1, 4), (2, 3), (3, 1), (4, 2)]   yes
    [(1, 4), (2, 3), (3, 2), (4, 1)]   yes