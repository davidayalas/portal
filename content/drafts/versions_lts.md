+++
date        = "2017-05-05"
title       = "Nou principi d'arquitectura: Utilització preferent de versions LTS"
description = "Les versions LTS (de Long Term Support) de productes opensource són les d'elecció principal per Arquitectura CTTI"
section     = "blog"
categories  = ["canigo"]
key         = "LTS1705"
+++

### Què són les versions de Suport a Llarg Termini 

Suport a Llarg Termini (o LTS, Long-Time Support, en anglès) es un tipus especial de versions o edicions de programari que estan designades per a ser suportades per un període més llarg que el normal. És un concepte que ha aparegut majoritàriament en programari opensource.

Es tracta d'una política de gestió del cicle de vida del programari, que aplica criteris d'estabilitat de les versions en un termini més llarg de l'acostumat, alterant la freqüencia de les noves versions per rebaixar el risc, la despesa i la disruptivitat que provoquen.

Al principi d'un period de suport a llarg termini, els desenvolupadors del programari imposen una congelació de les característiques: corregeixen errors i vulnerabilitats, però no introdueixen noves característiques que poden causar problemes de regressió. Al final del periode de suport, aquesta versió arriba al final de la seva vida útil.

### La necessitat d'un suport a llarg termini

El cicle de vida tradicional en molts projectes opensource es desplegar aviat, desplegar sovint, o planificar els desplegaments en períodes regulars. En qualsevolcas, cada nova versió inclou tant correccions funcionals com de seguretat i noves funcionalitats.

Les grans organitzacions, o altre tipus de colectius amb projectes de missió crítica, necessiten les correccions a les vulnerabilitats però sovint prefereixen mantenir la mateixa base de codi per un període estés en el temps sense que hi hagi cap tipus de canvi en les funcionalitats. Es habitual que canvis en les funcionalitats impliquin introducció de nous errors o que es trenquin funcionalitats existents.

Encara que es pogués assegurar al 100% que les noves versions estan lliures de defectes, per aquests tipus d'usuaris, tractar amb noves funcionalitats és sovint molt car. Actualitzar una aplicació requereix la participació de molts equips que poden fer prohibitiu la certificació continua de noves versions.

Les versions LTS d'un programari usualment adrecen aquestes preocupacions desplegant només aquelles actualitzacions que afecten a problemes de seguretat, donat que instalar-les aporta menys risc que no fer-ho.

### Nou Principi d'Arquitectura: fer servir versions LTS sempre que el programari escollit les ofereixi

Les versions LTS ofereixen a la Generalitat una opció ideal per a mantenir el versionat dels components del programari al dia amb una eficiència en cost. També s'adapta millor al cicle de vida habitual de les nostres aplicacions, més llarg que en altres sectors.

En aquest sentit, és un requeriment que totes les arquitectures que es defineixin contemplin aquest tipus de versions de programari. El no fer-ho serà objecte de justificació i de la tramitació de la corresponent excepció d'arquitectura.