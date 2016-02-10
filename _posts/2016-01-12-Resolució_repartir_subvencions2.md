---
layout: post
title: DAW3 UF5 - Resolució Repartir subvencions 2
categories: [daw, programació, groovy]
---
He tornat a fer el problema de [repartir subvencions](http://blog.utrescu.cat/Repartidor%20de%20subvencions/) en Groovy però ara fent servir expressions regulars...

Com en l'anterior intent faig servir una expressió regular inicial per separar el nom del nen, que no interessa per res, i després faig servir una altra expressió per anar cercant els diferents personatges

    def separarNen = ~/^([^:]+): (.*)/
    def separarPersonatges = ~/([^:]+): ([^-]+)(?: -|$)/

La resta és simplement el mateix d'abans però més senzill d'entendre ja que puc assignar els noms directament...

Trobo especialment interessant la forma de comptar el número de comes

    int numregals = ((regals =~ /,/).count)

El codi ara queda més "bonic":

    def separarNen = ~/^([^:]+): (.*)/
    def separarPersonatges = ~/([^:]+): ([^-]+)(?: -|$)/
    def personatges = [:]

    int totalRegals = 0

    new File('subvencions4.txt').eachLine { line ->
      (line =~ separarNen).each { tot2, nen, regaladors ->
        (regaladors =~ separarPersonatges).each { x, personatge, regals ->

          int numregals = ((regals =~ /,/).count) + 1

          if (!personatges.containsKey(personatge)) {
            personatges[personatge] = numregals
            } else {
              personatges[personatge] += numregals
            }
            totalRegals += numregals
          }
        }
      }
      println personatges
      personatges.each { println "${it.key} : ${it.value/totalRegals*100}" }

Molt interessant.
