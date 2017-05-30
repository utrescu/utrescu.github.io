---
layout: post
title: Injecció de dependències amb Guice
categories: [programació, java]
---
Injecció de dependències
--------------------------

La injecció de dependències és un patró de disseny en el que es una classe en comptes de crear els objectes que necessita els rep ja creats. Es basa en un concepte més genèric anomenat Inversió de Control

> La inversió de control diu, més o menys, que una classe no hauria de crear cap classe estàticament sinó que les hauria de rebre des de fora.

Per tant les classes no han de crear els objectes que els hi fan falta (mai faran **new**) sinó que els han de rebre ja creats (via un constructor, amb getters/setters ...)

Per tant l'objectiu final és un dels fonamentals en programació Orientada a Objectes: **reduïr l'acoblament**

> Que no s'hagi de canviar cap classe perquè un dels objectes que fa servir hagi canviat.

El més corrent és implementar aquest patró a través d'algun tipus d'*injector* que serà el que se n'encarregarà d'injectar els objectes creats als objectes que els necessitin.

![Injector](/images/injector.png)

La injecció de dependències millora el manteniment, la flexibilitat i facilita les proves dels objectes.

La injecció de dependències es pot fer servir en qualsevol llenguatge de programació.

### Injecció de dependències en Java

En Java la injecció de dependències es sol fer a través d'interfícies ja que si una classe crea un objecte que en crea un altre a través de l'operador `new` l'objecte dependrà d'aquest segon.

Idealment les classes han de ser tant independents com sigui possible i per tant l'ús de `new` dificulta la independència dels objectes.

En Java hi ha diferents formes de fer Injecció de dependències. El framework més "popular" és Spring però n'hi ha d'altres, CDI, Dagger, Google Guice, ... i fins i tot hi ha una especificació oficial (JSR330)

Google Guice
--------------
Google Guice és un framework de Injecció de Dependències que es pot fer servir per aplicacions on es volen mantenir les relacions de dependència en el codi de l'aplicació i no en fitxers de configuració externs.

Això en general implica que s'ha de crear una  classe on es definirà la configuració

En aquest cas s'injectarà una instància de `CotxeDeCarreres` en cas que alguna classe necessiti un objecte de tipus `Cotxe` (`Cotxe` normalment serà una interfície)

```java
import com.google.inject.AbstractModule;

public class Configuracio extends AbstractModule {
  @Override
  protected void configure() {

    bind(Cotxe.class).to(CotxeDeCarreres.class);

  }
}
```
Per obtenir les classes en un objecte només cal aconseguir-la a partir de l'injector

```java
Injector injector = Guice.createInjector(new Configuracio());
Cotxe cotxet = injector.getInstance(Cotxe.class);
```
Es pot veure que s'obté una variable *cotxet* de tipus `Cotxe` (que segons la configuració serà de tipus `CotxeDeCarreres`) sense crear-lo amb `new` ja que serà injectat pel contenidor Guice.

Això ens permet, per exemple, fer una nova implementació de `Cotxe` i usar-la només canviant-ne la configuració.

En les classes que es reben de l'injector s'hi pot fer servir l'anotació `@Inject` perquè les seves dependències s'injectin automàticament.

L'anotació `@Inject` es pot posar en un constructor, en un getter o en una propietat.

```java
class CotxeDeCarreres {

  Rodes rodes;

  @Inject
  public CotxeDeCarreres(Rodes r) {
    rodes = r;
  }
}
```

### Llibreria

Per poder fer servir Guice en un programa Java cal tenir la llibreria en el CLASSPATH. El més fàcil sol ser fer servir algun gestor de dependències com Maven o Gradle.

En Maven afegim la dependència al pom.xml:

```xml
<dependency>
    <groupId>com.google.inject</groupId>
    <artifactId>guice</artifactId>
    <version>4.1.0</version>
</dependency>
```
En Gradle és més o menys el mateix:

```
compile group: 'com.google.inject', name: 'guice', version: '4.1.0'
```

Exemple
----------------

El model seran "noms de persona" que codificaré en Strings.

### Crear els objectes que s'injectaran

#### Crear una interfície

Es defineix una interfície amb les operacions que es voldran fer. En aquest cas ens proporciona un mecanisme amb dos mètodes:

* Un per afegir noms de persona a una font de dades
* Un per saber quantes persones hi ha emmagatzemades

```java
public interface RepositoriPersones {

  int quantesPersonesHiHa();
  void afegirPersona(String nom);

}
```

#### Implementar la interfíce

Es desenvolupa un objecte que implementa la interfície. En aquest cas les dades estaran a memòria en una llista:

```java
public class RepositoriPersonesMemory implements RepositoriPersones {
   List<String> llistaPersones = Arrays.asList("Pere", "Manel");

   public int quantesPersonesHiHa() {
     return llistaPersones.size();
   }

   public void afegirPersona(String nom) {
     llistaPersones.add(nom);
   }
}
```

Es poden tenir diferents implementacions de la interfície

### Crear l'objecte configuració

Cal definir quins objectes són els que es podran injectar.

La configuració es defineix en una classe que farà les funcions de configuració. La classe de configuració ha de ser un descendent de Module (normalment serà AbstractModule) i implementar-ne el mètode `configure()`

* En el mètode s'hi defineix quins tipus d'objectes són els que s'han d'injectar i quina implementació es fa servir:

```java
public class GuiceConf extends AbstractModule {
  @Override
  protected void configure() {

    bind(RepositoriPersones.class).to(RepositoriPersonesMemory.class);

  }

}
```

Es pot definir d'altres formes, com per exemple crear-lo manualment:

```java
bind(RepositoriPersones.class).to(new RepositoriPersonesMemory.class)
```

#### Injectar dependències en una classe

Es pot indicar a *Guice* que en els objectes que retorna se'ls hi injecti un objecte de la configuració automàticament.

Per exemple al recuperar un `GestorNoms` se li injectarà una instància de `RepositoriPersones`:

```java
public GestorNoms {
  RepositoriPersones repo;

  @Inject
  public GestorNoms(RepositoriPersones repo) {
    this.repo = repo;
  }

  public run() {
    System.out.println(repo.quantesPersonesHiHa());
  }
}
```

### Programa principal

Per fer el programa principal només hem de fer que s'arranqui *Guice* creant un `Injector` al que li donem la classe de configuració.

```java
public static void main(String[] args) {

  Injector injector = Guice.createInjector(new GuiceConf());
  ...
}
```

A partir d'aquest injector es poden obtenir les classes del programa i executar-ne els mètodes

```java
RepositoriPersones repo = injector.getInstance(RepositoriPersones.class);

System.out.println(repo.quantesPersonesHiHa());
repo.afegirPersona("Frederic");
System.out.println(repo.quantesPersonesHiHa());
```

El mateix podem fer amb la classe `GestorNoms` al que al obtenir-la a través de l'injector se li injectarà un `RepositoriPersones` automàticament perquè té l'anotació `@Inject`:

```java
GestorDeNoms noms = injector.getInstance(GestorDeNoms.class);
noms.run();
```

Si s'executen un rere l'altre es veurà que Guice ha creat un objecte diferent cada vegada que injecta un objecte nou. Al executar això:

```java
RepositoriPersones repositori = injector.getInstance(RepositoriPersones.class);
System.out.println(repositori.quantesPersonesHiHa());
repositori.afegirPersona("Frederic");
System.out.println(repositori.quantesPersonesHiHa());
System.out.println("------- GestorDeNoms -------");
GestorDeNoms noms = injector.getInstance(GestorDeNoms.class);
noms.run();
```

La segona execució donarà **2** a pesar de que en l'anterior s'hi havia afegit un element:

```
2
3
------- GestorDeNoms -------
2
```

Es pot aconseguir que siguin la mateixa anotant les implementacions amb `@Singleton`:

```java
@Singleton
public class RepositoriPersonesMemory implements RepositoriPersones{
     ...
}
```

Amb només aquest canvi només hi haurà una sola instància de cada objecte en tot el programa:

```
2
3
------- GestorDeNoms -------
3
```

# Codi de l'exemple

Es pot trobar el codi d'exemple en el meu [repositori de Github](https://github.com/utrescu/GuiceExemple)
