---
title: "Cheryl's Birthday"
date: 2017-12-19T12:57:55-05:00
draft: false
---

Cheryl's Birthday was [fun a few years ago](https://www.theguardian.com/science/alexs-adventures-in-numberland/2015/apr/13/can-you-solve-the-singapore-primary-maths-question-that-went-viral).  Even more fun is Peter Norvig's [treatment of it](https://github.com/norvig/pytudes/blob/master/ipynb/Cheryl.ipynb) with Python.  What follows here is the same idea reimplemented in Kotlin.

Date extenstions for Strings:

    fun String.month() = this.split(" ")[0]
    fun String.day() = this.split(" ")[1]
    
Our Agent class.  An agent is someone trying to solve the puzzle, whether Albert, Bernard or ourselves.    
    
    class Agent {
        val possibilities = mutableSetOf(
            "May 15",    "May 16",    "May 19",
            "June 17",   "June 18",
            "July 14",   "July 16",
            "August 14", "August 15", "August 17")
        fun tellMonth(date: String)
                = this.possibilities.retainAll { it.month() == date.month() }
        fun tellDay(date: String)
                = this.possibilities.retainAll { it.day() == date.day() }
        fun tellStatement(statement: (String) -> Boolean)
                = this.possibilities.retainAll(statement)
        fun know(): Boolean
                = this.possibilities.size == 1
    }
    
The statements as functions:    
    
    fun statement1(date: String) : Boolean {
        val albert = Agent()
        albert.tellMonth(date)
        return (!albert.know() && albert.possibilities.all {
            val bernard = Agent()
            bernard.tellDay(it)
            !bernard.know()
        })
    }
    
    fun statement2(date: String) : Boolean {
        val bernard = Agent()
        bernard.tellDay(date)
        val knowsAtFirst = bernard.know()
        bernard.tellStatement(::statement1)
        val knowsGivenStatement1 = bernard.know()
        return !knowsAtFirst && knowsGivenStatement1
    }
    
    fun statement3(date: String) : Boolean {
        val albert = Agent()
        albert.tellMonth(date)
        albert.tellStatement(::statement2)
        return albert.know()
    }
    
Us figuring it out step by step:    
    
    fun main(args: Array<String>) {
        val problemSolver = Agent()
        for (statement in listOf(::statement1, ::statement2, ::statement3)) {
            problemSolver.tellStatement(statement)
            if (problemSolver.know()) {
                println("We've heard ${statement.name} " +
                        "and now we know the solution: ${problemSolver.possibilities.first()}")
                break
            }
            else {
                if (problemSolver.possibilities.isEmpty()) {
                    println("We've heard ${statement.name} and now there are no possibilities left!")
                    break
                }
                else {
                    println("We've heard ${statement.name} and we don't know the solution yet, " +
                            "but the possibilities are ${problemSolver.possibilities}")
                }
            }
        }
    }