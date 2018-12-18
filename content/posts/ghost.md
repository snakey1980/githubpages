---
title: "Ghost"
date: 2018-12-05T12:00:54-05:00
draft: false
---
Here's code that helps (you cheat) at [Ghost](https://en.wikipedia.org/wiki/Ghost_(game)).  It passes some basic tests but there must be bugs -- I haven't tested it all.  It's more complicated than it might be as it works for N players and does not assume the other players are playing perfectly.  Run repl() to get responses, or playOut() to see it play itself.

    val WORDS = File("/path/to/your/dictionary").readLines().toSet().filter { it.length > 2 }
    
    val PARTIAL_WORDS: Set<String> = (0..WORDS.map { it.length }.max()!!).flatMap { length -> WORDS.filter { it.length >= length }.map { it.substring(0, length) }.toSet() }.toSet()
    
    interface Response { val thisPlayer: Int }
    data class Challenge(override val thisPlayer: Int, val given: String): Response {
        override fun toString() = "Player ${thisPlayer + 1}: challenge (no words start with '$given')."
    }
    data class Surrender(override val thisPlayer: Int, val example: String): Response {
        override fun toString() = "Player ${thisPlayer + 1}: surrender (an example that kills is '$example') or challenge for the hell of it."
    }
    interface Play: Response {
        val value: Char
    }
    data class PlayPerfect(override val thisPlayer: Int, override val value: Char, private val example: String, private val playerWhoGetsKilled: Int): Play {
        override fun toString() = "Player ${thisPlayer + 1}: play '$value'.  An example of an ending word is '$example' which kills player ${playerWhoGetsKilled + 1}."
    }
    data class PlaySpeculative(override val thisPlayer: Int, override val value: Char, private val example: String, private val playerWhoGetsKilled: Int): Play {
        override fun toString() = "Player ${thisPlayer + 1}: play '$value'.  We should lose if the other players are playing perfectly, but it's impossible we might end on '$example' which kills ${playerWhoGetsKilled + 1}"
    }
    data class PlayVerySpeculative(override val thisPlayer: Int, override val value: Char): Play {
        override fun toString() = "Player ${thisPlayer + 1}: play '$value'.  We should lose if the other players are playing perfectly because no word starts like this but it's possible they may not challenge."
    }
    
    fun play(given: String, players: Int): Response {
        if (players < 2) {
            throw IllegalArgumentException("Need at least 2 players")
        }
        val thisPlayer = given.length % players
        return if (given !in PARTIAL_WORDS) {
            Challenge(thisPlayer, given)
        } else {
            val killsWithPerfectPlay = killsWithPerfectPlay(given, players)
            if (killsWithPerfectPlay == thisPlayer) {
                val speculativeWord = WORDS.filter { it.startsWith(given) && whichPlayerGetsKilled(it, players) != thisPlayer }.minWith(compareBy<String> { it.length }.thenBy { it })
                if (speculativeWord != null) {
                    PlaySpeculative(thisPlayer, speculativeWord[given.length], speculativeWord, whichPlayerGetsKilled(speculativeWord, players))
                }
                else {
                    val play = ('a'..'z').firstOrNull { given.plus(it) !in WORDS }
                    if (play != null) {
                        PlayVerySpeculative(thisPlayer, play)
                    }
                    else {
                        val destinedWord = WORDS.filter { it.startsWith(given) && killsWithPerfectPlay(it, players) == thisPlayer }.minWith(compareBy<String> { it.length }.thenBy { it })!!
                        Surrender(thisPlayer, destinedWord)
                    }
                }
            } else {
                val play = ('a'..'z').filter { given.plus(it) in PARTIAL_WORDS }.first { killsWithPerfectPlay(given.plus(it), players) == killsWithPerfectPlay }
                val destinedWord = WORDS.filter { it.startsWith(given.plus(play)) && killsWithPerfectPlay(it, players) == killsWithPerfectPlay }.minWith(compareBy<String> { it.length }.thenBy { it })!!
                PlayPerfect(thisPlayer, play, destinedWord, killsWithPerfectPlay)
            }
        }
    }
    
    fun whichPlayerGetsKilled(word: String, players: Int): Int {
        if (word !in WORDS) {
            throw IllegalArgumentException("$word is not a valid Ghost word.")
        }
        var result = (3..word.length).first { word.substring(0, it) in WORDS }
        while (result > players) {
            result -= players
        }
        return result - 1
    }
    
    fun killsWithPerfectPlay(string: String, players: Int): Int {
        if (string !in PARTIAL_WORDS) {
            throw IllegalArgumentException("$string is not a valid Ghost string.  Someone should have challenged already")
        }
        return if (string in WORDS) {
            whichPlayerGetsKilled(string, players)
        } else {
            val thisPlayer = string.length % players
            val nextGeneration = ('a'..'z').map { string.plus(it) }.filter { it in PARTIAL_WORDS }.toSet().map { killsWithPerfectPlay(it, players) }
            val furthest = compareBy<Int> { (thisPlayer - it).sign }.thenBy { it }
            nextGeneration.filter { it != thisPlayer }.maxWith(furthest)?: thisPlayer
        }
    }
    
    fun playOut(start: String, players: Int) {
        generateSequence(start to play(start, players)) { previous ->
            val (given, response) = previous
            println("'$given' -> $response")
            if (response is Play) {
                given.plus(response.value).let { it to play(it, players) }
            }
            else {
                null
            }
        }.toList()
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
            println("'$given', $players players -> ${play(given, players)}\n")
        }
    }
    