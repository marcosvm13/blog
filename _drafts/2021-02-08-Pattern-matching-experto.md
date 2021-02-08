---
layout: post
title: "Todo lo que quieres saber y nunca te atreviste a preguntar del pattern matching. Parte II."
author: Alfonso Roa
intro: "Pattern matching de la K a la Z"
image: /img/pattern.jpg
intro_image_ratio: is-16by9
toc: true
---

## Conocimientos avanzados de pattern matching
En el anterior post ya vimos todas las posibilidades que nos ofrece el pattern matching para realizar nuestras condiciones. Ahora verás como puedes llevar esta herramienta a un nivel superior

## Bricomanía: crea tus propios extractores
Ya vimos que los extractores nos permiten descomponer el elemento que pasamos al `match` sacando información interna.

¿Y cómo puede ser esto posible? ¿Cuándo se que algo se puede descomponer o no? Muy sencillo, tenemos que ver si existe en una clase u objeto un método llamado `unapply`. Este es el truco que usa scala para poder descomponer algo. Scala tiene ya preparado este método en el objeto de compañía para todos sus tipos básicos (tuplas, listas o toda case class que creas).

##### Extractores básicos

Lo primero a tener en cuenta son los elementos que entran en el método, en este caso siempre tiene que ser uno y del tipo que queremos descomponer, y lo que devolverá siempre ha de ser un Option, que será del tipo extraido.

Veamos un ejemplo primero, en el que queremos comprobar si un string se puede transformar a un integer. El problema es que el método `toInteger` que scala nos provee lanza excepciones, por lo que no es seguro usarlo desde un pattern matching. Sin embargo, nosotros podemos crear una clase que lo permita. 


```scala
object ValidIntString { // creamos el objeto que permitirá extraer el string si es valido
    def unapply(string: String): Option[Int] = // esperamos descomponer un string, y poder sacar un integer si es posible
    try {
        Some(string.toInt) // Si logra ejecutar sin excepciones, devolverá un some con el valor
    } catch {
        case _ : Throwable => None // en caso que no fuera integer, lanzaría excepción, por lo que devolvemos None
    }
}
```




defined <span style="color:green">object</span> <span style="color:cyan">ValidIntString</span>  



Ahora podemos usar nuestro flamante nuevo extractor


```scala
"123" match {
    case ValidIntString(n) => s"es un integer con valor $n"
    case _ => "no es integer"
}

"hola" match {
    case ValidIntString(n) => s"es un integer con valor $n"
    case _ => "no es integer"
}
```




<span style="color:cyan">res36_0</span>: <span style="color:green">String</span> = <span style="color:green">"es un integer con valor 123"</span>  
<span style="color:cyan">res36_1</span>: <span style="color:green">String</span> = <span style="color:green">"no es integer"</span>  



Funciona perfectamente, pero ¿y si quisiera devolver más de un elemento, como por ejemplo hacen en la tupla que vimos antes? Simplemente tenemos que devolver una tupla. Por ejemplo si el valor se puede transformar en integer queremos devolver una tupla que contenga ese valor y su doble.


```scala
object ValidIntStringWithDouble { // creamos el objeto que permitirá extraer el string si es valido
    def unapply(string: String): Option[(Int, Int)] = // esperamos descomponer un string, y poder sacar una tupla de integes si es posible
    try {
        val x = string.toInt
        Some(x, x * 2) // Si logra ejecutar sin excepciones, devolverá un Some con el valor
    } catch {
        case _: Throwable => None // en caso que no fuera integer, lanzaría excepción, por lo que devolvemos None
    }
}
```




defined <span style="color:green">object</span> <span style="color:cyan">ValidIntStringWithDouble</span>  




```scala
"123" match {
    case ValidIntStringWithDouble(n, n2) => s"es un integer con valor $n y su doble $n2"
    case _ => "no es integer"
}

"hola" match {
    case ValidIntStringWithDouble(n, n2) => s"es un integer con valor $n y su doble $n2"
    case _ => "no es integer"
}
```




<span style="color:cyan">res38_0</span>: <span style="color:green">String</span> = <span style="color:green">"es un integer con valor 123 y su doble 246"</span>  
<span style="color:cyan">res38_1</span>: <span style="color:green">String</span> = <span style="color:green">"no es integer"</span>  



##### Extractores variádico

Si necesitamos un número indeterminado de elementos a devolver podemos hacer uso de la variante variádico unapplySeq, en la que devolveremos una secuencia de elementos. Cuando se hace la extracción en el match, se tiene en cuenta el número de elementos que se le pasan como argumento.


```scala
object SplitDecimals { // creamos el objeto que permitirá extraer el string si es valido
    def unapplySeq(string: String): Option[List[Int]] = // esperamos descomponer un string, y devolver un número indefinido de parámetros.
    try {
        val x = string.toFloat
        val hasDecimals = x % 1 != 0 // si es par tendrá 2 elementos la lista, si es impar solo uno
        if (hasDecimals)
          Some(List((x / 1).toInt, (x % 1 * 1000000).toInt)) // Al ser par devolvemos 2 elementos
        else
          Some(List((x / 1).toInt)) // Al ser impar devolvemos solo uno
    } catch {
        case _: Throwable => None // en caso que no fuera integer, lanzaría excepción, por lo que devolvemos None
    }
}
```




defined <span style="color:green">object</span> <span style="color:cyan">SplitDecimals</span>  




```scala
"123.0000" match {
    case SplitDecimals(n1, n2) => s"tiene decimales: $n1 . $n2"
    case SplitDecimals(n1) => s"es entero y tenemos $n1 solo"
    case _ => "no es numerico"
}

"123.56454" match {
    case SplitDecimals(n1, n2) => s"tiene decimales: $n1 . $n2"
    case SplitDecimals(n1) => s"es entero y tenemos $n1 solo"
    case _ => "no es numerico"
}

"hola" match {
    case SplitDecimals(n1, n2) => s"tiene decimales: $n1 y $n2"
    case SplitDecimals(n1) => s"es entero y tenemos $n1 solo"
    case _ => "no es numerico"
}
```




<span style="color:cyan">res40_0</span>: <span style="color:green">String</span> = <span style="color:green">"es entero y tenemos 123 solo"</span>  
<span style="color:cyan">res40_1</span>: <span style="color:green">String</span> = <span style="color:green">"tiene decimales: 123 . 564537"</span>  
<span style="color:cyan">res40_2</span>: <span style="color:green">String</span> = <span style="color:green">"no es numerico"</span>  



#### Extractores Booleanos

Por último, scala permite otro tipo de extractor en el que no interesa extraer un elemento si no ver si cumple una propiedad. Al igual que hacemos en la parte de refinado, en la que podemos comprobar si cumple una condición declarándolo explicitamente. Tambien podríamos encapsular esa lógica para darle un nombre legible. Para hacer esto también tenemos que crear un extractor con el método unapply, pero en vez de devolver un `Option`, solo tenemos que devolver un `Boolean`


```scala
object IsEaven {
    def unapply(int: Int): Boolean = int % 2 == 0
}
```




defined <span style="color:green">object</span> <span style="color:cyan">IsEaven</span>  




```scala
54 match {
    case IsEaven() => "el valor es par"
    case _ => "el valor es impar"
}

45 match {
    case IsEaven() => "el valor es par"
    case _ => "el valor es impar"
}
```




<span style="color:cyan">res42_0</span>: <span style="color:green">String</span> = <span style="color:green">"el valor es par"</span>  
<span style="color:cyan">res42_1</span>: <span style="color:green">String</span> = <span style="color:green">"el valor es impar"</span>  



Y aunque en ejemplos anteriores siempre usamos `object`para crear nuestro extractor, también podemos hacer un extractor que requiera parámetros usando `class`. Por requermientos de la sintaxis tenemos que instanciarlo antes, para no confundir los parámetros del extractor con las comparaciones que queremos hacer.


```scala
case class GreaterThan(c: Int) {
    def unapply(int: Int): Boolean = int > c
}
```




defined <span style="color:green">class</span> <span style="color:cyan">GreaterThan</span>  




```scala
val mayor45 = GreaterThan(45)

54 match {
    case mayor45() => "el valor es mayor que 45"
    case _ => "el valor es menor o igual"
}

23 match {
    case mayor45() => "el valor es mayor que 45"
    case _ => "el valor es menor o igual"
}
```




<span style="color:cyan">mayor45</span>: <span style="color:green">GreaterThan</span> = <span style="color:yellow">GreaterThan</span>(<span style="color:green">45</span>)
<span style="color:cyan">res44_1</span>: <span style="color:green">String</span> = <span style="color:green">"el valor es mayor que 45"</span>  
<span style="color:cyan">res44_2</span>: <span style="color:green">String</span> = <span style="color:green">"el valor es menor o igual"</span>  



#### Usos de extractores ya implementados en scala

Con estos ejemplos podemos ver que los extractores no solo sirven para facilitarnos acceder a los elementos, si no que también nos permiten hacer validaciones. Por ejemplo, en scala se usa para permitir el uso de regex en pattern matching y poder extraer los elementos (o grupos) que capturamos.


```scala
val fecha = raw"(\d{4})-(\d{2})-(\d{2})".r

"2004-01-20" match {
    case fecha(year, month, day) => s"año: $year, mes: $month, dia: $day"
    case _ => "no es una fecha"
}

"hola" match {
    case fecha(year, month, day) => s"año: $year, mes: $month, dia: $day"
    case _ => "no es una fecha"
}
```




<span style="color:cyan">fecha</span>: <span style="color:green">scala</span>.<span style="color:green">util</span>.<span style="color:green">matching</span>.<span style="color:green">Regex</span> = (\d{4})-(\d{2})-(\d{2})
<span style="color:cyan">res45_1</span>: <span style="color:green">String</span> = <span style="color:green">"a\u00f1o: 2004, mes: 01, dia: 20"</span>  
<span style="color:cyan">res45_2</span>: <span style="color:green">String</span> = <span style="color:green">"no es una fecha"</span>  



Otro caso es poder matchear las listas, esperando un número específico de elementos


```scala
List(1, 2, 3) match {
    case List(n1) => s"tiene solo un elemento $n1"
    case List(n1, n2) => s"tiene dos elementos $n1, $n2"
    case List(n1, n2, n3) => s"tiene tres elementos $n1, $n2, $n3"
    case l => s"tiene demasiados elementos, exactamente  ${l.size}"
}

List(1) match {
    case List(n1) => s"tiene solo un elemento $n1"
    case List(n1, n2) => s"tiene dos elementos $n1, $n2"
    case List(n1, n2, n3) => s"tiene tres elementos $n1, $n2, $n3"
    case l => s"tiene demasiados elementos, exactamente  ${l.size}"
}

List(1, 2, 3, 4) match {
    case List(n1) => s"tiene solo un elemento $n1"
    case List(n1, n2) => s"tiene dos elementos $n1, $n2"
    case List(n1, n2, n3) => s"tiene tres elementos $n1, $n2, $n3"
    case l => s"tiene demasiados elementos, exactamente  ${l.size}"
}
```




<span style="color:cyan">res46_0</span>: <span style="color:green">String</span> = <span style="color:green">"tiene tres elementos 1, 2, 3"</span>  
<span style="color:cyan">res46_1</span>: <span style="color:green">String</span> = <span style="color:green">"tiene solo un elemento 1"</span>  
<span style="color:cyan">res46_2</span>: <span style="color:green">String</span> = <span style="color:green">"tiene demasiados elementos, exactamente  4"</span>  



### Obtener el elemento original y poder aplicar un patrón.

El uso de extractores es muy común, pero podemos llegar al caso en el que nos interesaría poder tener el valor original en la condición además de una extracción. Para esto podemos hacer uso del símbolo `@`, con el que podemos asignar el valor original a un valor y a continuación del símbolo descomponerlo con un patrón.


```scala
val fecha = raw"(\d{4})-(\d{2})-(\d{2})".r

"2004-01-20" match {
  case d @ fecha(year, month, day) => s"año: $year, mes: $month, dia: $day original $d"
  case _ => "no es una fecha"
}
```




<span style="color:cyan">fecha</span>: <span style="color:green">scala</span>.<span style="color:green">util</span>.<span style="color:green">matching</span>.<span style="color:green">Regex</span> = (\d{4})-(\d{2})-(\d{2})  
<span style="color:cyan">res47_1</span>: <span style="color:green">String</span> = <span style="color:green">"a\u00f1o: 2004, mes: 01, dia: 20 original 2004-01-20"</span>  



## ¡Ahora todo a la vez!

Haciendo uso de todo lo visto hasta ahora y a modo de recordatorio, se pueden realizar comparativas complejas en muy poco código:


```scala
val fecha = raw"(\d{4})-(\d{2})-(\d{2})".r // regex para fechas con guion 2020-01-01

val anio19xx = raw"19(\d{2})".r // regex para números de 4 cifras que comienzan por 19xx y extrae el xx

val anioEspecial = "2001"

def queDiaEs(dateStr: String): String =
dateStr match {
    // comparación con un valor tras la extracción
    case fecha(`anioEspecial`, _, _) => "mi año especial :D"
    // comparación con literal tras extracción
    case fecha(_, "01", "01") => s"feliz año nuevo!"
    // asignación del valor original y comparación en la extracción
    case d @ fecha(_, "02", "29") => s"es año bisiesto $d"
    // varios posibles casos de un elemento extraido
    case fecha("1800" | "1700", _, _) => "eso es muy viejo"
    // refinamiento tras extracción
    case fecha(year, month, day) if year.reverse == (month + day) => s"la fecha es capicua $year$month$day"
    // extracción de un elemento obtenido por una extracción
    case fecha(anio19xx(year19), month, day) => s"$day del $month del año $year19"
    case fecha(year, month, day) => s"año: $year, mes: $month, dia: $day"
    case _ => "no es una fecha"
}
```




<span style="color:cyan">fecha</span>: <span style="color:green">scala</span>.<span style="color:green">util</span>.<span style="color:green">matching</span>.<span style="color:green">Regex</span> = (\d{4})-(\d{2})-(\d{2})
<span style="color:cyan">anio19xx</span>: <span style="color:green">scala</span>.<span style="color:green">util</span>.<span style="color:green">matching</span>.<span style="color:green">Regex</span> = 19(\d{2})
<span style="color:cyan">anioEspecial</span>: <span style="color:green">String</span> = <span style="color:green">"2001"</span>  
defined <span style="color:green">function</span> <span style="color:cyan">queDiaEs</span>  




```scala
queDiaEs("2001-03-01")
queDiaEs("2021-01-01")
queDiaEs("2020-02-29")
queDiaEs("1800-03-29")
queDiaEs("1700-03-29")
queDiaEs("2020-02-02")
queDiaEs("1995-02-03")
queDiaEs("2021-31-10")
queDiaEs("2021")
```




<span style="color:cyan">res49_0</span>: <span style="color:green">String</span> = <span style="color:green">"mi a\u00f1o especial :D"</span>  
<span style="color:cyan">res49_1</span>: <span style="color:green">String</span> = <span style="color:green">"feliz a\u00f1o nuevo!"</span>  
<span style="color:cyan">res49_2</span>: <span style="color:green">String</span> = <span style="color:green">"es a\u00f1o bisiesto 2020-02-29"</span>  
<span style="color:cyan">res49_3</span>: <span style="color:green">String</span> = <span style="color:green">"eso es muy viejo"</span>  
<span style="color:cyan">res49_4</span>: <span style="color:green">String</span> = <span style="color:green">"eso es muy viejo"</span>  
<span style="color:cyan">res49_5</span>: <span style="color:green">String</span> = <span style="color:green">"la fecha es capicua 20200202"</span>  
<span style="color:cyan">res49_6</span>: <span style="color:green">String</span> = <span style="color:green">"03 del 02 del a\u00f1o 95"</span>  
<span style="color:cyan">res49_7</span>: <span style="color:green">String</span> = <span style="color:green">"a\u00f1o: 2021, mes: 31, dia: 10"</span>  
<span style="color:cyan">res49_8</span>: <span style="color:green">String</span> = <span style="color:green">"no es una fecha"</span>  



### Un error que todos cometemos

Vimos al comienzo que el compilador nos daba un mensaje de advertencia cuando teníamos casos que no eran alcanzables, pero planteo otra duda. ¿Qué ocurre si tenemos un caso que no está contemplado?


```scala
def tengoDato(o: Option[Int]): String =
o match {
    case Some(0) => s"tenemos el valor 0" // solo contemplamos Some con el valor 0
    case None => "no tenemos valor"
}
```

<span style="color:red">Compilador</span>
```
cmd50.sc:2: match may not be exhaustive.  
It would fail on the following input: Some((x: Int forSome x not in 0))  
o match {  
^
```



defined <span style="color:green">function</span> <span style="color:cyan">tengoDato</span>  



Ya vemos en este caso que solo con la definición ya nos advierte el compilador de que algo falta. Pero si aun así hacemos caso omiso, al ejecutar:


```scala
tengoDato(Some(1))
```

```
scala.MatchError: Some(1) (of class scala.Some)  
    ammonite.$sess.cmd50$Helper.tengoDato(cmd50.sc:2)  
    ammonite.$sess.cmd51$Helper.<init>(cmd51.sc:1)  
    ammonite.$sess.cmd51$.<init>(cmd51.sc:7)  
    ammonite.$sess.cmd51$.<clinit>(cmd51.sc:-1)  
```

Obtenemos un error en la ejecución de tipo `scala.MatchError`. Y ya hemos dicho que esto en scala hay que evitarlo siempre que sea posible.

Como bien sabrás, scala está muy orientado a que se programe según el paradigma funcional, lo que nos lleva a las funciones puras. Si no conocías el concepto de función pura, podemos resumirlo como que a todo elemento que entra en una función, tiene que devolver un valor. Pero una excepción no es un valor.

Hay que tener en cuenta que esta comprobación de exhausividad solo funciona si trabajamos con ADT's. Si realizamos este match en un tipo primitivo como son `String` o `Int`, el compilador no va a poder darnos estas advertencias.


```scala
def noExhaustivo(o: Int): String =
o match {
    case 0 => s"tenemos el valor 0"
    case 1 => s"tenemos el valor 1"
}
```




defined <span style="color:green">function</span> <span style="color:cyan">noExhaustivo</span>  




```scala
noExhaustivo(0)
noExhaustivo(1)
noExhaustivo(2)
```

```
scala.MatchError: 2 (of class java.lang.Integer)  
    ammonite.$sess.cmd52$Helper.noExhaustivo(cmd52.sc:2)  
    ammonite.$sess.cmd53$Helper.<init>(cmd53.sc:3)  
    ammonite.$sess.cmd53$.<init>(cmd53.sc:7)  
    ammonite.$sess.cmd53$.<clinit>(cmd53.sc:-1)  
```

Por este motivo siempre es recomendable poner un caso por defecto (`case _ => `) que lo evite si estamos haciendo match sobre un elemento primitivo o clases que no sean ADTs.

### Un truco si eres nuevo (o no te fias ni de ti mismo)

El compilador de scala sabe que un pattern matching es un posible foco de excepciones, por lo que en muchos casos, si ve que no se contemplan todos los casos de entrada, o lo que es lo mismo, no es exhaustivo, nos dará un advertencia a nivel de warning. 

Yo te doy un consejo, es buena practica hacer que una build falle si hay algún mensaje. Solo tienes que añadir la siguiente opción en tu proyecto sbt:

```scala
scalacOptions += "-Xfatal-warnings"
```

o si eres de los que trabaja con maven, en el plugin de scala añade junto a tus opciones:

```xml
<executions>
  <execution>
    <configuration>
      <args>  
        <arg>-Xfatal-warnings</arg>
      </args>
    </configuration>
  </execution>
</executions>
```


Con esto ya tenemos todas las herramientas necesarias para convertirnos en un ninja del patter matching. Podemos hacer una gran y compleja lógica de una manera muy legible y mantenible y teniendo al compilador supervisor.

## Funciones parciales

Siempre que se ha hablado de pattern matching hemos hecho mucho hincapié en que ha de ser una función pura, es decir, contemplar todos los casos de entrada y que den respuesta a cada uno de ellos. Sin embargo algunas veces fuera del `match` solo queremos contemplar una parte de los casos. Esto en scala se llaman funciones parciales y tienen un [interfaz](https://www.scala-lang.org/api/current/scala/PartialFunction.html) ya creado para este propósito y que sigue la siguiente estructura:

```scala
trait PartialFunction[-A, +B]{
    def isDefinedAt(x: A): Boolean
    def apply(v1: A): B
}
```

El método `apply` es un método que no es necesario llamar de forma explícita, si no que se llama directamente solo pasándole los parámetros. En este caso sería el equivalente a la parte derecha tras la flecha (`=>`) del pattern matching y, como podrás imaginar, `isDefinedAt` es el método que representa la condición. Si está definido para el valor a comprobar devuelve verdadero y ejecutaríamos el método apply.


```scala
val isEven: PartialFunction[Int, String] = new PartialFunction[Int, String]{
    def isDefinedAt(x: Int): Boolean = x % 2 == 0
    def apply(v1: Int): String = v1+" is even"
}

```




<span style="color:cyan">isEven</span>: <span style="color:green">PartialFunction</span>[<span style="color:green">Int</span>, <span style="color:green">String</span>] = <function1>



Esto no quiere decir que nosotros tengamos que crear una instancia de `PartialFunction` de forma explícita, porque para eso tenemos la sintaxis que usabamos anteriormente.


```scala
val isEven: PartialFunction[Int, String] = {
  case x if x % 2 == 0 => x + " is even"
}
```




<span style="color:cyan">isEven</span>: <span style="color:green">PartialFunction</span>[<span style="color:green">Int</span>, <span style="color:green">String</span>] = <function1>



Con esto, vemos uno de los grandes secretos del pattern matching. En el fondo solo es azúcar sintáctico que el compilador traduce al interfaz `PartialFunction`.

¿Y qué usos podemos hacer de las funciones parciales? Pues podemos aplicarlas cuando necesitamos saber dos cosas: sobre qué podemos aplicar esta función, y si es el caso, qué queremos hacer con ellas. Por ejemplo el método `collect` de las colecciones, con el que seleccionamos solo los que pasan la criba y los transformados como se indica:


```scala
List(1, 2, 3, 4).collect(isEven)
```




<span style="color:cyan">res56</span>: <span style="color:green">List</span>[<span style="color:green">String</span>] = <span style="color:yellow">List</span>(<span style="color:green">"2 is even"</span>, <span style="color:green">"4 is even"</span>)



Otra de las particularidades de las funciones parciales es que se pueden combinar para contemplar más casos.


```scala
val isEven: PartialFunction[Int, String] = {
  case x if x % 2 == 0 => x+" is even"
}

val isOdd: PartialFunction[Int, String] = {
  case x if x % 2 == 1 => x+" is odd"
}

val pf: PartialFunction[Int, String] = isEven.orElse(isOdd)

List(1, 2, 3, 4).map(pf)
```




<span style="color:cyan">isEven</span>: <span style="color:green">PartialFunction</span>[<span style="color:green">Int</span>, <span style="color:green">String</span>] = <function1>
<span style="color:cyan">isOdd</span>: <span style="color:green">PartialFunction</span>[<span style="color:green">Int</span>, <span style="color:green">String</span>] = <function1>
<span style="color:cyan">pf</span>: <span style="color:green">PartialFunction</span>[<span style="color:green">Int</span>, <span style="color:green">String</span>] = <function1>
<span style="color:cyan">res57_3</span>: <span style="color:green">List</span>[<span style="color:green">String</span>] = <span style="color:yellow">List</span>(<span style="color:green">"1 is odd"</span>, <span style="color:green">"2 is even"</span>, <span style="color:green">"3 is odd"</span>, <span style="color:green">"4 is even"</span>)



En este caso, somos nosotros los que tenemos que asegurar que es exhaustivo.

## Aplicación de pattern matching en lambdas

Muchas veces cuando queremos hacer un pattern matching es creando una lambda, por ejemplo en un método map.


```scala
val optval = Some(4)

optval.map(x => x match {
    case 1 => "es 1"
    case 2 => "es el número 2"
    case _ => "es otro número"
})


```




<span style="color:cyan">optval</span>: <span style="color:green">Some</span>[<span style="color:green">Int</span>] = <span style="color:yellow">Some</span>(<span style="color:green">4</span>)
<span style="color:cyan">res58_1</span>: <span style="color:green">Option</span>[<span style="color:green">String</span>] = <span style="color:yellow">Some</span>(<span style="color:green">"es otro n\u00famero"</span>)



Como hemos visto en el ejemplo anterior, si la lógica que queremos en esa lambda solo se compone de un pattern matching, podemos simplificar el código. Scala permite realizar un pattern matching con los parámetros pasados a la lambda cambiando el inicio `x => x match {` por las llaves que tienen los casos directamente:


```scala
val optval = Some(4)

optval.map{
    case 1 => "es 1"
    case 2 => "es el número 2"
    case _ => "es otro número"
}
```




<span style="color:cyan">optval</span>: <span style="color:green">Some</span>[<span style="color:green">Int</span>] = <span style="color:yellow">Some</span>(<span style="color:green">4</span>)
<span style="color:cyan">res59_1</span>: <span style="color:green">Option</span>[<span style="color:green">String</span>] = <span style="color:yellow">Some</span>(<span style="color:green">"es otro n\u00famero"</span>)



Y tranquilo, si lo que esperas ha de ser una función completa o pura, ya te avisará el compilador si es exhaustivo o no (en los casos que vimos previamente).


```scala
val foo: List[Option[Int]] = List(Some(4), None, Some(1))

foo.map{
    case Some(_) => 5
}

// tenemos advertencia de compilación y además error en la ejecución
```

<span style="color:red">Compilador</span>
```
cmd60.sc:3: match may not be exhaustive.
It would fail on the following input: None
val res60_1 = foo.map{
                     ^
```

```
    scala.MatchError: None (of class scala.None$)  
      ammonite.$sess.cmd60$Helper.$anonfun$res60_1$1(cmd60.sc:3)  
      ammonite.$sess.cmd60$Helper.$anonfun$res60_1$1$adapted(cmd60.sc:3)  
      scala.collection.immutable.List.map(List.scala:297)  
      ammonite.$sess.cmd60$Helper.<init>(cmd60.sc:3)  
      ammonite.$sess.cmd60$.<init>(cmd60.sc:7)  
      ammonite.$sess.cmd60$.<clinit>(cmd60.sc:-1)  
```

Y si ve que espera una función parcial, ahí eres tú el responsable de que esté bien creado. El compilador no puede hacer todo por tí.


```scala
val foo: List[Option[Int]] = List(Some(4), None, Some(1))

foo.collect{
    case Some(_) => 5
}

// al ser parcial, no tiene responsabilidad el compilador
```




<span style="color:cyan">foo</span>: <span style="color:green">List</span>[<span style="color:green">Option</span>[<span style="color:green">Int</span>]] = <span style="color:yellow">List</span>(<span style="color:yellow">Some</span>(<span style="color:green">4</span>), <span style="color:green">None</span>, <span style="color:yellow">Some</span>(<span style="color:green">1</span>))
<span style="color:cyan">res61_1</span>: <span style="color:green">List</span>[<span style="color:green">Int</span>] = <span style="color:yellow">List</span>(<span style="color:green">5</span>, <span style="color:green">5</span>)



## Scala 3

Ahora toca mirar al futuro próximo. En pocos meses tras la fecha de este post saldrá una nueva versión de scala denominada dotty o scala 3. En esta se ha reescrito el compilador y van a tener muchas novedades. Respecto al pattern matching, no va a haber grandes cambios. Todo lo que se ha visto para indicar la condición se mantiene tal cual. Pero sí merece la pena destacar unos puntos y ya de paso los escribimos con la sintaxis nueva que nos permite scala 3, omitiendo llaves y cambiandolas por `:`.

### Match como 'función'
Un pequeño cambio respecto a la palabra `match`: sigue siendo una palabra reservada, pero ahora se puede usar como llamada a un método, es decir, usando punto respecto al valor. Eso si, tras `match` se pueden eliminar las llaves, sin necesidad de poner `:`
```scala
45.match
    case 1 => "es 1"
    case 2 => "es el número 2"
    case _ => "es otro número"

```
[link para ejemplo interactivo](https://scastie.scala-lang.org/alfonsorr/qeozSdkzTvOTseiCc0OmnA)

Esta forma trata de permitirnos cambiar la prioridad para procesar el pattern matching, permitiéndonos integrarlo fácilmente con otros elementos, por ejemplo:

```scala
if 4.match
     case 5 => true
     case _ => false
then
  "valor 5"
else
  "otro valor"
```
[link para ejemplo interactivo](https://scastie.scala-lang.org/alfonsorr/A4nNc65pSTuvrtWvSkFfaA/8)

### Extractores 'irrefutables'

Otra de las mejoras que se tiene en scala 3 es la creación de extractores. Una limitación que tienen actualmente los extractores de scala 2 es la obligación de devolver los elementos extraidos en un Option, excepto para el caso del extractor Booleano, lo que representa que esa extracción puede no ir bien. Esto impide crear extractores que sabemos que siempre irán correctamente, o como lo llaman en la documentación, 'irrefutables'. Como por ejemplo el siguiente.

```scala
object PreviousAndNextNumber:
    def unapply(int: Int): Option[(Int, Int)] = Some((int-1, int + 1))

5 match
    case PreviousAndNextNumber(_, 6) => "valor valido"
    case _ => "valor invalido"
````
[link para ejemplo interactivo](https://scastie.scala-lang.org/alfonsorr/Z0kg5fykTh6hE6c2g8GGvQ/1)

Como se ve, en este extractor se devuelve un elemento `Option[Int]` pero nunca va a haber posibilidad de que devolvamos `None`. Esto en scala 3 se ha mejorado, permitiendo el uso de extractores que no solo devuelvan `Option`, sino también `Product`. Recordad que a este último pertenecen tuplas y todas las `case class`.

```scala
object PreviousAndNextNumber:
    def unapply(int: Int): (Int, Int) = (int-1, int + 1)1

5 match
    case PreviousAndNextNumber(_, 6) => "valor valido"
    case _ => "valor invalido"

val PreviousAndNextNumber(prev, next) = 5

```
[link para ejemplo interactivo](https://scastie.scala-lang.org/alfonsorr/IYjlD9BWT9S1z6xRipQpIA/9)

Esto permite el uso de extractores en las asignaciones, cosa que no se suele usar mucho en scala 2 si no sabemos con exactitud si puede lanzar excepciones en caso de no ser válida la condición.

Y como último apunte ya que hablamos de extractores, se permitirá su uso en los `for comprehesion`, pero eso es tema para otro post ;)

## Resumen final
Conociendo que hacen los métodos unapply, o las funciones parciales podemos aplicar la herramienta adecuada en cada momento. Siempre con la intención de hacer un código legible y mantenible.
En scala el `pattern matching` siempre ha sido una de sus caracteristicas estrella, y ha madurado mucho. Tanto que en nuevas versiones solo tiene algunas mejoras para poder reutilizar algunos de sus elementos en más lugares.

## Repositorio
Si prefieres leer este post en formato notebook, lo tienes subido en [github](https://github.com/alfonsorr/pattern-matching-post)