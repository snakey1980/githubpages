---
title: "Ghost"
date: 2018-12-05T12:00:54-05:00
draft: false
---
Here's a program that helps (you cheat) at [Ghost](https://en.wikipedia.org/wiki/Ghost_(game)).  I must look into Lexicant some time too.

    fun main(args: Array<String>) {
        repl()
    }
    
    val words = File("/path/to/your/wordlist.txt").readLines().toSet()
    
    fun kills(given: String, word: String, players: Int): Boolean {
        val remaining = word.removePrefix(given).length
        return word.length > 2 && (remaining % players == 1)
    }
    
    fun possibleWords(given: String): Set<String> {
        return words.asSequence().filter { it.startsWith(given) && it.length > given.length }.toSet()
    }
    
    fun wordsThatKill(given: String, possibleWords: Set<String>, players: Int): Set<String> {
        val killers = possibleWords.filter { word -> kills(given, word, players) }
        val indirectKillers = possibleWords.filter { killers.any { killer -> it.startsWith(killer) } }
        return killers.asSequence().plus(indirectKillers).toSet()
    }
    
    interface Response
    data class Challenge(private val given: String): Response {
        override fun toString() = "challenge (I don't believe any words start with $given)"
    }
    data class Surrender(val killers: Set<String>): Response {
        override fun toString() = "surrender (an example of a word that kills you is ${killers.minBy { it.length }}).  You can challenge if you think your opponent is bluffing."
    }
    data class Letter(private val value: Char, private val example: String): Response {
        override fun toString() = "play '$value' (an example of a safe word is $example)"
    }
    
    fun play(given: String, players: Int): Response {
        val possibleWords = possibleWords(given)
        if (possibleWords.isEmpty()) {
            return Challenge(given)
        }
        val wordsThatKillMe = wordsThatKill(given, possibleWords, players)
        val safeWords = possibleWords.minus(wordsThatKillMe)
        return if (safeWords.isEmpty()) {
            Surrender(wordsThatKillMe)
        } else {
            val safeWord = safeWords.sorted().first()
            Letter(safeWord.removePrefix(given).first(), safeWord)
        }
    }
    
    fun repl() {
        println("How many players are there?")
        val players = readLine()?.toIntOrNull()
        if (players == null || players < 2) {
            throw RuntimeException("You must give an integer number of players > 1")
        }
        println()
        generateSequence {
            println("What are the letters so far?")
            val letters = readLine()
            if (letters?.any { it < 'a' || it > 'z' } == true) {
                throw RuntimeException("You must give letters only")
            }
            letters
        }.forEach { given ->
            println("Given '$given' and $players players, I recommend you ${play(given, players)}\n")
        }
    }