---
layout: post
title: Assentar els comensals en Groovy
categories: [groovy, programació]
---
He trobat un problema per Internet que es fa amb una recursivitat clàssica i l'he fet servir per aprendre Groovy.

La idea és preparar el dinar de Nadal entre diferents persones tenint en compte el grau de felicitat que tindran els comensals al estar uns al costat dels altres. Per exemple si tenim una entrada com aquesta (el problema estava en anglès...):

    Alice would gain 54 happiness units by sitting next to Bob.
    Alice would lose 79 happiness units by sitting next to Carol.
    Alice would lose 2 happiness units by sitting next to David.
    Bob would gain 83 happiness units by sitting next to Alice.
    Bob would lose 7 happiness units by sitting next to Carol.
    Bob would lose 63 happiness units by sitting next to David.
    Carol would lose 62 happiness units by sitting next to Alice.
    Carol would gain 60 happiness units by sitting next to Bob.
    Carol would gain 55 happiness units by sitting next to David.
    David would gain 46 happiness units by sitting next to Alice.
    David would lose 7 happiness units by sitting next to Bob.
    David would gain 41 happiness units by sitting next to Carol.

S'ha d'aconseguir trobar quina és la màxima felicitat que es pot aconseguir al asseure'ls en una taula circular. Per l'exemple el millor que es pot aconseguir amb la llista anterior és **330**

Resolució
--------------
Primer he de processar el fitxer d'entrada. La solució que he trobat és crear una llista de persones (*persons*), un diccionari amb les afinitats (*relations*).

Per llegir el fitxer faig servir una expressió regular per estalviar-me comparacions

    def relations = [:]
    def persons = []

    def regex = ~/(.*) would (.*) (\d+) happiness.*to (.*)\./

    new File('input.txt').eachLine { line ->
      def match = regex.matcher(line)
      persons << match[0][1]
      int valor = match[0][3].toInteger()
      if (match[0][2] == "lose") {
        valor *= -1
      }
      relations[match[0][1] + "-" + match[0][4]] = valor
    }
    persons.unique()


Al acabar tindré una llista amb les persones, persons, i amb les relacions entre les persones, relations.

Ara només cal provar totes les combinacions mirant quina és la que dóna la relació més alta. En aquest cas provo de començar per cada una de les possibles persones i genero una llista amb el número màxim aconseguit per cada una. Acaba quedant-se amb el número màxim

    println persons.collect {
      calculate(0, persons.minus(it), [it], relations)
    }.max()

Calculate no té secrets, és un recursiu clàssic: va passant els elements de persons a order fins que no en queda cap i es queda amb el valor aconseguit, recula i prova amb una persona diferent per veure si aconsegueix un valor més alt... Va fent fins que ha provat totes les combinacions.

    def calculate(valor, persons, order, relations) {
        String last = order[-1]
        if (persons.size() == 0) {
           return valor + relations[last + "-" + order[0]]
                        + relations[order[0] + "-" + last]
        }
        return persons.collect {
            def adding = relations[last + "-" + it]
                         + relations[it + "-" + last]
            calculate(valor + adding, persons.minus(it), order.plus(it), relations)
        }.max()
    }
