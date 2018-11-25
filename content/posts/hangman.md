---
title: "Hangman"
date: 2018-11-25T10:57:28-05:00
draft: false
---

A fairly effective way to play Hangman offense:

* Consider all words that match the pattern and don't contain guesses that have been denied
* Guess the letter not guessed already that appears in most of these words

Here's some code that plays like that:

    class Player(pattern: List<Char>) {
    
        private val pattern = pattern.toMutableList()
    
        private val possibleWords = wordsOfLengthCachy(pattern.size).toMutableSet()
    
        fun guess(): Char {
            return ('A'..'Z')
                    .filter {
                        it !in pattern
                    }
                    .maxBy { guess ->
                        possibleWords.count { word -> word.contains(guess) }
                    }?: throw IllegalStateException("Cannot guess for pattern ${pattern.joinToString("")}")
        }
    
        fun deny(guess: Char) {
            possibleWords.removeIf { it.contains(guess) }
        }
    
        fun confirm(char: Char, positions: List<Int>) {
            if (positions.any { it < 0 || it >= pattern.size } || positions.any { pattern[it] != '.' }) {
                throw IllegalArgumentException("Invalid positions for pattern $pattern")
            }
            else {
                positions.forEach { pattern[it] = char }
            }
            possibleWords.removeIf { word -> (0 until pattern.size).any { index -> pattern[index] !in listOf('.', word[index]) } }
        }
    
        fun knows(): Boolean {
            return if (pattern.none { it == '.' }) {
                if (possibleWords.size == 1) {
                    true
                }
                else {
                    throw IllegalStateException("Pattern seems complete but still have more than one possible word!")
                }
            }
            else {
                false
            }
        }
    
    } 

To measure how effective it is, we can count how many denials it will encounter solving a word:

    fun solve(word: String): Int {
        val player = Player(word.map { '.' })
        var denials = 0
        while (!player.knows()) {
            val guess = player.guess()
            val positions = (0 until word.length).filter { word[it] == guess }
            if (positions.isEmpty()) {
                player.deny(guess)
                denials++
            }
            else {
                player.confirm(guess, positions)
            }
        }
        return denials
    }
    
A fun thing to do will be to run this against all words in the dictionary.  Doing that gives:

Denials    |      Words    |    %    |    Cumulative % |
-----------|--------------:|--------:|----------------:|
0	       |12429          |19.77    |       19.77     |  
1	       |15110          |24.04    |       43.81     |
2	       |12176          |19.37    |       63.18     |
3	       |8663           |13.78    |       76.96     |
4	       |5650           |8.99     |       85.95     |
5	       |3588           |5.71     |       91.66     |
6	       |2153           |3.43     |       95.08     |
7	       |1322           |2.10     |       97.18     |
8	       |793            |1.26     |       98.45     |
9	       |464            |0.74     |       99.18     |
10	       |248            |0.39     |       99.58     |
11	       |151            |0.24     |       99.82     |
12	       |73             |0.12     |       99.93     |
13	       |25             |0.04     |       99.97     |
14	       |12             |0.02     |       99.99     |
15	       |4              |0.01     |      100.00     |

So we get 90% of words with 5 denials or fewer, so enough to win nearly all the time even if playing only drawing the man.  

We also get to know the best words to play on defense.

These words took 13 denials: FOXING, HAZING, HOMY, HONED, HURRY, JAPES, JOY, KEYS, MUFF, NAKED, NULL, OAK, PIPE, PIPES, POPPING, PUNK, RILL, TILLS, TUCK, WANE, YEN, YOU, YUK, YUKS, ZIT

These words took 14 denials each: JAZZING, KEY, LAX, LOX, PUFF, PULL, SIZE, TILL, WILLS, YOGI, YOKED, YUCK

These words took 15 denials each: TUFT, WILL, YOKING, YUKKING