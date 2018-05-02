---
title: "Dobble Spot It"
date: 2018-05-02T17:00:26-04:00
draft: false
---

Spot It, also sold as Dobble is a fun card game with a deck of around 50 cards printed with 8 symbols each, with one key rule:  every possible pair of cards has exactly one symbol in common.

This leads to questions like

* What's the maximum number of cards there could be in the deck?
* With the number of cards maxed, how many symbols are there in total?
* How can you construct a deck which conforms to the rule?

For the second question, here's some code that works, at least for symbolsPerCard = 8:

    fun generateDeck(): Set<Set<Int>> {
        return mutableSetOf<Set<Int>>().apply {
            add((0 until symbolsPerCard).toSet())
            (0 until symbolsPerCard - 1).forEach { i ->
                add(setOf(0).plus((0 until symbolsPerCard - 1).map { j -> symbolsPerCard + (symbolsPerCard - 1) * i + j }.toSet()))
            }
            (0 until symbolsPerCard - 1).forEach { i ->
                (0 until symbolsPerCard - 1).forEach { j ->
                    val c = setOf(i + 1).plus((0 until  symbolsPerCard - 1).map { k -> symbolsPerCard + (symbolsPerCard - 1) * k + (i * k + j) % (symbolsPerCard - 1) }.toSet())
                    add(c)
                }
            }
        }.toSet()
    }