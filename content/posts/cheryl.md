---
title: "Cheryl's Birthday"
date: 2017-12-19T12:57:55-05:00
draft: false
---

Cheryl's Birthday was [fun a few years ago](https://www.theguardian.com/science/alexs-adventures-in-numberland/2015/apr/13/can-you-solve-the-singapore-primary-maths-question-that-went-viral).  Even more fun is Peter Norvig's [treatment of it](https://github.com/norvig/pytudes/blob/master/ipynb/Cheryl.ipynb) with Python.  What follows here is just straight up copied from Norvig reimplemented in Kotlin.  Kotlin is quite nice here as it's more natural to use types and some OO and you get the quite satisfying idea of Albert, Bernard and us (the one solving the problem) being the same type of thing.

    data class Date(val month: String, val day: Int)
    
    class Agent {
        val possibilities = mutableSetOf<Date>()
        fun tellPossibilities(possibilities: Set<Date>) = this.possibilities.addAll(possibilities)
        fun tellMonth(date: Date) = this.possibilities.retainAll { it.month == date.month }
        fun tellDay(date: Date) = this.possibilities.retainAll { it.day == date.day }
        fun tellStatement(statement: (Date) -> Boolean) = this.possibilities.retainAll(statement)
        fun know(): Boolean = this.possibilities.size == 1
    }
    
    fun main(args: Array<String>) {
    
        val possibilities = listOf(
                "May 15",    "May 16",    "May 19",
                "June 17",   "June 18",
                "July 14",   "July 16",
                "August 14", "August 15", "August 17")
                .map { Date(it.split(" ")[0], it.split(" ")[1].toInt()) }
                .toSet()
    
        fun statement1(date: Date) : Boolean {
            val albert = Agent()
            albert.tellPossibilities(possibilities)
            albert.tellMonth(date)
            return (!albert.know() && albert.possibilities.all {
                val bernard = Agent()
                bernard.tellPossibilities(possibilities)
                bernard.tellDay(it)
                !bernard.know()
            })
        }
    
        fun statement2(date: Date) : Boolean {
            val bernard = Agent()
            bernard.tellPossibilities(possibilities)
            bernard.tellDay(date)
            val knowsAtFirst = bernard.know()
            bernard.tellStatement(::statement1)
            val knowsGivenStatement1 = bernard.know()
            return !knowsAtFirst && knowsGivenStatement1
        }
    
        fun statement3(date: Date) : Boolean {
            val albert = Agent()
            albert.tellPossibilities(possibilities)
            albert.tellMonth(date)
            albert.tellStatement(::statement2)
            return albert.know()
        }
    
        val problemSolver = Agent()
        problemSolver.tellPossibilities(possibilities)
        for (statement in listOf(::statement1, ::statement2, ::statement3)) {
            problemSolver.tellStatement(statement)
            if (problemSolver.know()) {
                println("We've heard ${statement.name} and now we know the solution: ${problemSolver.possibilities.first()}")
                break
            }
            else {
                println("We've heard ${statement.name} and we don't know the solution yet, but the possibilities are ${problemSolver.possibilities}")
            }
        }
    }