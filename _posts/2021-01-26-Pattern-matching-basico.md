---
layout: post
title: "Todo lo que quieres saber y nunca te atreviste a preguntar del pattern matching. Parte I."
author: Alfonso Roa
intro: "Pattern matching de la A a la J"
image: /img/pattern.jpg
intro_image_ratio: is-16by9
toc: true
---
## ¿Que es el pattern matching?

Una estructura de control, pensada para comprobar si un elemento cumple ciertas condiciones. Si no la conocías antes es similar en sintaxis a un `switch`, pero nos permite mayor precisión.
Para usarlo solo necesitamos poner a continuación del elemento sobre el que queremos aplicarlo la palabra reservada `match` e indicar cada uno de los casos que nos interesan.

Comencemos con uno ejemplo sencillo:


```scala
val stringExample = "hola"
stringExample match {
    case "hola" => "saludos"
    case "adios" => "hasta pronto"
    case _ => "???"
}
```




<span style="color:cyan">stringExample</span>: <span style="color:green">String</span> = <span style="color:green">"hola"</span>  
<span style="color:cyan">res3_1</span>: <span style="color:green">String</span> = <span style="color:green">"saludos"</span>  



Vamos a describir que está ocurriendo aquí: tenemos nuestro valor asignado y empezamos el matcheado. En este caso, queremos comprobar si el valor a procesar es igual al string "hola". En caso de ser correcto, se ejecutará el codigo de su parte derecha, si no, pasará al siguiente caso y repetirá lo misma comprobación hasta que uno coincida. Podéis ver que el último caso, se representa solo con `_`, esta es la forma en scala de indicar "cualquier otro caso". De esta forma, si ninguno de los anteriores ha sido satisfactorio, sabemos que siempre ejecutará la parte derecha de este código.

Esta es la principal diferencia respecto a `switch`. El patter matching únicamente ejecutará el primer trozo de código que satisfaga la condición, por lo que el orden que indiquemos importa.


```scala
val stringExample = "hola"
stringExample match {
    case _ => "???"
    case "hola" => "saludos"
    case "adios" => "hasta pronto"
}
```
<span style="color:red">Compilador</span>
```
cmd4.sc:3: patterns after a variable pattern cannot match (SLS 8.1.1)  
    case _ => "???"  
            ^cmd4.sc:4: unreachable code due to variable pattern on line 3  
    case "hola" => "saludos"  
                    ^cmd4.sc:5: unreachable code due to variable pattern on line 3  
    case "adios" => "hasta pronto"  
                    ^cmd4.sc:4: unreachable code  
    case "hola" => "saludos"  
                    ^
```


<span style="color:cyan">stringExample</span>: <span style="color:green">String</span> = <span style="color:green">"hola"</span>  
<span style="color:cyan">res4_1</span>: <span style="color:green">String</span> = <span style="color:green">"???"</span>  



En este ejemplo pusimos en primer caso el comodín `_`, por lo que cualquier valor al ser comparado ejecutará el código de la derecha. El resto de casos son inaccesibles, lo que hará que nos muestre un error de compilación `patterns after a variable pattern cannot match` que nos indica que tenemos código inalcanzable. Todos los casos que hay tras el `case _` se podrían borrar. Esto nos da una pista de que nuestro pattern matching podría estar mal o que podemos prescindir de casos inalcanzables.

## Pero esto es solo la punta del iceberg.

Como hemos visto, pattern matching no solo podemos hacer condiciones de igualdad, si no que podemos hacer una variada cantidad de acciones.
### Comparar con múltiples casos.

En caso de tener múltiples valores que pueden satisfacer un mismo caso, podemos representarlo separandolos con `|`.


```scala
val stringExample1 = "hola"
stringExample1 match {
    case "hola" | "holi" => "saludos"
    case "adios" => "hasta pronto"
    case _ => "???"
}

val stringExample2 = "holi"
stringExample2 match {
    case "hola" | "holi" => "saludos"
    case "adios" => "hasta pronto"
    case _ => "???"
}
```




<span style="color:cyan">stringExample1</span>: <span style="color:green">String</span> = <span style="color:green">"hola"</span>  
<span style="color:cyan">res5_1</span>: <span style="color:green">String</span> = <span style="color:green">"saludos"</span>  
<span style="color:cyan">stringExample2</span>: <span style="color:green">String</span> = <span style="color:green">"holi"</span>  
<span style="color:cyan">res5_3</span>: <span style="color:green">String</span> = <span style="color:green">"saludos"</span>  



## Asignación a un valor

En caso de cumplir la condición, muchas veces necesitamos recoger el valor extraído para poder procesarlo.


```scala
val stringExample = "ya estoy"
stringExample match {
    case "hola" => "saludos"
    case "adios" => "hasta pronto"
    case x => s"el valor $x no está contemplado"
}
```




<span style="color:cyan">stringExample</span>: <span style="color:green">String</span> = <span style="color:green">"ya estoy"</span>  
<span style="color:cyan">res6_1</span>: <span style="color:green">String</span> = <span style="color:green">"el valor ya estoy no est\u00e1 contemplado"</span>  



Podemos ver, que hemos cambiado nuestro comodín `_` por `x`, lo que permite que se pueda usar en la parte derecha y podemos garantizar que si llegamos a este caso nuestro valor nunca contendrá los valores `"hola"` y `"adios`". Realmente el comodín no indica "en cualquier otro caso", es una asignación igual que con `x`. La diferencia es que `_` no puede ser llamada desde la parte derecha.

Aquí quiero hacer otro inciso, y es la forma en la que puedes nombrar al valor donde vamos a asignar el elemento sobre el que hacemos el matching, ya que scala tiene unas reglas.

Pongamos el siguiente ejemplo, en el que asignamos el valor a un nombre ya existente.


```scala
val x = "soy x"

val stringExample = "ya estoy"
stringExample match {
    case "hola" => "saludos"
    case "adios" => "hasta pronto"
    case x => s"el valor $x no está contemplado"
}
```




<span style="color:cyan">x</span>: <span style="color:green">String</span> = <span style="color:green">"soy x"</span>  
<span style="color:cyan">stringExample</span>: <span style="color:green">String</span> = <span style="color:green">"ya estoy"</span>  
<span style="color:cyan">res7_2</span>: <span style="color:green">String</span> = <span style="color:green">"el valor ya estoy no est\u00e1 contemplado"</span>  



En este caso, el valor `x` de dentro del `match` impedirá en el contexto de la derecha que se pueda acceder al valor `x` externo. Esto se llama ocultamiento de valor o "variable shadowing", y puede llevar a confusión en algunos casos. Desgraciadamente, el compilador de scala, NO nos dará una advertencia en caso de que ocurra esto.

¿Y si yo quisiera comparar un caso con el contenido de un valor que tengo definido fuera del `match`? En el pattern matching se puede, pero siguiendo unas reglas, ya que, como hemos visto, un valor con el que nos gustaría comparar puede convertirse en una nueva asignación. Entonces ¿cómo podría crear un caso si es igual a mi valor externo `x`? Indicando que este nombre de valor no es para asignarlo, si no compararlo, rodeando el valor con comillas `` `x` ``:


```scala
val x = "soy x"

val stringExample = "soy x"
stringExample match {
    case "hola" => "saludos"
    case "adios" => "hasta pronto"
    case `x` => s"es el valor que tenía en x"
    case _ => "ninguno de los anteriores"
}
```

<span style="color:cyan">x</span>: <span style="color:green">String</span> = <span style="color:green">"soy x"</span>  
<span style="color:cyan">stringExample</span>: <span style="color:green">String</span> = <span style="color:green">"soy x"</span>  
<span style="color:cyan">res8_2</span>: <span style="color:green">String</span> = <span style="color:green">"es el valor que ten\u00eda en x"</span>  



Y si no te fías de que esto sea así, pongamos un ejemplo que no coincida.


```scala
val x = "soy x"

val stringExample2 = "no soy x"
stringExample2 match {
    case "hola" => "saludos"
    case "adios" => "hasta pronto"
    case `x` => s"es el valor que tenía en x"
    case _ => "ninguno de los anteriores"
}
```

<span style="color:cyan">x</span>: <span style="color:green">String</span> = <span style="color:green">"soy x"</span>  
<span style="color:cyan">stringExample2</span>: <span style="color:green">String</span> = <span style="color:green">"no soy x"</span>  
<span style="color:cyan">res9_2</span>: <span style="color:green">String</span> = <span style="color:green">"ninguno de los anteriores"</span>  

Otra forma más sencilla es seguir [la guia de estilo de scala](https://docs.scala-lang.org/style/naming-conventions.html#constants-values-variable-and-methods). En ella se indica que los valores constantes definidos a nivel de clase han de empezar con mayúscula, lo que reconoce que no es un valor a asignar.

```scala
val X = "soy x"

val stringExample = "soy x"
stringExample match {
    case "hola" => "saludos"
    case "adios" => "hasta pronto"
    case X => s"es el valor que tenía en x"
    case _ => "ninguno de los anteriores"
}
```

<span style="color:cyan">X</span>: <span style="color:green">String</span> = <span style="color:green">"soy x"</span>  
<span style="color:cyan">stringExample</span>: <span style="color:green">String</span> = <span style="color:green">"soy x"</span>  
<span style="color:cyan">res10_2</span>: <span style="color:green">String</span> = <span style="color:green">"es el valor que ten\u00eda en x"</span>  




```scala
val X = "soy x"

val stringExample2 = "no soy x"
stringExample2 match {
    case "hola" => "saludos"
    case "adios" => "hasta pronto"
    case X => s"es el valor que tenía en x"
    case _ => "ninguno de los anteriores"
}
```




<span style="color:cyan">X</span>: <span style="color:green">String</span> = <span style="color:green">"soy x"</span>  
<span style="color:cyan">stringExample2</span>: <span style="color:green">String</span> = <span style="color:green">"no soy x"</span>  
<span style="color:cyan">res11_2</span>: <span style="color:green">String</span> = <span style="color:green">"ninguno de los anteriores"</span>  



### Refinar la condición.

Ahora que sabemos asignar el valor dentro de un caso podemos llegar a otra de las ventajas del pattern matching como es el poder refinar la condición sin tener que ser siempre por igualdad, como hemos hecho hasta el momento. Pongamos el ejemplo donde quisieramos tratar los strings que comiencen por `h` de manera distinta, excepto el caso que tenemos ya, comparando con la palabra `"hola"`. Con los conocimientos que tenemos actualmente, nuestro código quedaría algo tal que así:


```scala
val stringExample2 = "habana"
stringExample2 match {
    case "hola" => "saludos"
    case "adios" => "hasta pronto"
    case x => if (x.head == 'h') "comienza por h" else "ninguno de los anteriores"
}
```




<span style="color:cyan">stringExample2</span>: <span style="color:green">String</span> = <span style="color:green">"habana"</span>  
<span style="color:cyan">res12_1</span>: <span style="color:green">String</span> = <span style="color:green">"comienza por h"</span>  



Pero la utilidad del pattern matching es la de aplanar todos los posibles casos y no empezar a anidar casos más complejos, por lo que podemos hacer uso de la asignación del valor y hacer filtrado de casos de la siguiente manera:


```scala
val stringExample2 = "habana"
stringExample2 match {
    case "hola" => "saludos"
    case "adios" => "hasta pronto"
    case x if x.head == 'h' => "comienza por h"
    case _ => "ninguno de los anteriores"
}
```




<span style="color:cyan">stringExample2</span>: <span style="color:green">String</span> = <span style="color:green">"habana"</span>  
<span style="color:cyan">res13_1</span>: <span style="color:green">String</span> = <span style="color:green">"comienza por h"</span>  



De esta manera, vemos claramente los 4 posibles casos estructurados con su condición primero, y no tenemos que rebuscar lógica escondida a lo largo de todo el código. Lo único a tener en cuenta es que este `if` no necesita paréntesis en la condición, a diferencia de la estructura de control en scala.

### Comprobación del tipo del elemento matcheado.

Hasta ahora, ¿todo bien? Comencemos a trabajar con más tipos, además de nuestro querido `String`. ¿Cómo podríamos sacar casos distintos si es string, si es Integer, y un último para el resto? En principio sería algo tal que así:



```scala
def matchfun(x: Any): String =
 x match {
     case x if x.isInstanceOf[String] =>
       val xString = x.asInstanceOf[String]
       s"tengo el string $xString"
     case x if x.isInstanceOf[Int] =>
       val xInt = x.asInstanceOf[Int]
       s"tengo el integer $xInt"
     case _ => "es otro tipo"
 }
```




defined <span style="color:green">function</span> <span style="color:cyan">matchfun</span>  




```scala
matchfun("hola")
matchfun(42)
matchfun(12.4)
```




<span style="color:cyan">res15_0</span>: <span style="color:green">String</span> = <span style="color:green">"tengo el string hola"</span>  
<span style="color:cyan">res15_1</span>: <span style="color:green">String</span> = <span style="color:green">"tengo el integer 42"</span>  
<span style="color:cyan">res15_2</span>: <span style="color:green">String</span> = <span style="color:green">"es otro tipo"</span>  



Esta forma genera mucho código repetitivo, por lo que la sintaxis del pattern matching nos da una forma más concisa de describirlo. Esta nos sirve no solo para saber si es de un tipo, también hará automáticamente el cambio de tipo para que trabajemos en nuestro contexto de la derecha.


```scala
def matchfun2(x: Any): String =
 x match {
     case x: String => s"tengo el string $x"
     case x: Int => s"tengo el integer $x"
     case _ => "es otro tipo"
 }
```




defined <span style="color:green">function</span> <span style="color:cyan">matchfun2</span>  




```scala
matchfun2("hola")
matchfun2(42)
matchfun2(12.4)
```




<span style="color:cyan">res17_0</span>: <span style="color:green">String</span> = <span style="color:green">"tengo el string hola"</span>  
<span style="color:cyan">res17_1</span>: <span style="color:green">String</span> = <span style="color:green">"tengo el integer 42"</span>  
<span style="color:cyan">res17_2</span>: <span style="color:green">String</span> = <span style="color:green">"es otro tipo"</span>  



Esta forma es mucho más concisa y nos evita mancharnos las manos con funciones como `asInstanceOf`.

#### Cuidado con las comprobaciones de algunas clases

Una cosa que tenemos que tener en cuenta es que estas comprobaciones se hacen en tiempo de ejecución. La JVM tiene una limitación y es los tipos paramétricos no existen durante la ejecución. En otras palabras, que si quisieramos comparar y saber si es de tipo `List[String]` o `List[Int]` no podríamos fácilmente, ya que la información sobre qué tipo de elemento contiene se pierde durante la ejecución:


```scala
def matchfun2(x: List[Any]): String =
 x match {
     case x: List[String] => s"tengo una lista de string de longitud ${x.size}"
     case x: List[Int] => s"tengo una lista de integer de longitud ${x.size}"
     case _ => "es otro tipo de lista"
 }
```
<span style="color:red">Compilador</span>
```
cmd18.sc:3: non-variable type argument String in type pattern List[String] (the underlying of List[String]) is unchecked since it is eliminated by erasure  
        case x: List[String] => s"tengo una lista de string de longitud ${x.size}"  
                ^cmd18.sc:4: non-variable type argument Int in type pattern List[Int] (the underlying of List[Int]) is unchecked since it is eliminated by erasure  
        case x: List[Int] => s"tengo una lista de integer de longitud ${x.size}"  
                ^cmd18.sc:4: unreachable code  
        case x: List[Int] => s"tengo una lista de integer de longitud ${x.size}"  
                              ^
```




defined <span style="color:green">function</span> <span style="color:cyan">matchfun2</span>  




```scala
matchfun2(List("hola", "adios"))
matchfun2(List(42))
matchfun2(List(4.2f))
```




<span style="color:cyan">res19_0</span>: <span style="color:green">String</span> = <span style="color:green">"tengo una lista de string de longitud 2"</span>  
<span style="color:cyan">res19_1</span>: <span style="color:green">String</span> = <span style="color:green">"tengo una lista de string de longitud 1"</span>  
<span style="color:cyan">res19_2</span>: <span style="color:green">String</span> = <span style="color:green">"tengo una lista de string de longitud 1"</span>  



Como puedes comprobar, el pattern matching no puede ver más allá de que se trata de una lista y el tipo contenido nunca lo tiene en cuenta, por lo que siempre entrará en el primer caso. Se podría comprobar qué tipo de elemento contiene en el primer elemento, pero siempre tendremos el problema en listas vacías, ya que nos resultará imposible poder comprobarlo. Eso si, como todo posible punto de error, el compilador nos informará para que lo tengamos en cuenta con el siguiente warning `List[String] is unchecked since it is eliminated by erasure`, o traducido, "List[String] no está chequeado porque ha sido borrado". Esto ocurre porque el compilador lo traducirá a `case x: List => ...` borrando el tipo de la lista. Como os podréis imaginar esto no ocurre solo con listas, si no con todos los tipos paramétricos.

### ADT's en pattern matching

Ya hemos visto que scala permite comprobar el tipo del elemento para poder realizar una acción para cada tipo. Es por esto que quiero pararme a comentar una particularidad de la programación funcional, que por supuesto se aplica en scala, y es el uso de los Tipos Algebráicos de Datos, Algebraic Data Types en inglés o ADT para que sea más corto. Esto es una representación de los datos que se basa en el producto y suma de tipos, por ejemplo un producto de String e integer sería la siguiente `case class`


```scala
case class Usuario(nombre: String, edad: Int)
```




defined <span style="color:green">class</span> <span style="color:cyan">Usuario</span>  



Y una  suma de tipos se puede representar de múltiples maneras, pero la más común es el uso de `sealed trait` por ejemplo.


```scala
sealed trait Trabajador
case class Currito(nombre: String) extends Trabajador
case class Jefe(nombre: String, subordinados: List[Trabajador]) extends Trabajador
```




defined <span style="color:green">trait</span> <span style="color:cyan">Trabajador</span>  
defined <span style="color:green">class</span> <span style="color:cyan">Currito</span>  
defined <span style="color:green">class</span> <span style="color:cyan">Jefe</span>  



En este vemos una representación de lo que sería un trabajador: o es alguien con subordinados a su cargo, o es un currante sin nadie a su cargo. Al ser un `sealed trait` solo permite estas dos posibilidiades de tipo de trabajador y no se puede extender en ningún otro lado. Esta forma de representación de datos es muy usada en scala, incluso en elementos que nos da el lenguaje, como sería Option, que tiene dos posibles elementos: Some, que indica que contiene un elemento, o None, que no contiene ninguno.

Dada la particularidad de esta suma de tipos, el pattern matching suele ser una herramienta muy común y útil para poder actuar según el tipo de dato que podamos encontrarnos.


```scala
def quienEs(t: Trabajador): String =
t match {
    case j: Jefe => s"${j.nombre} es jefe de ${j.subordinados.size} empleados"
    case c: Currito => s"${c.nombre} es un gran trabajador"
}

def tengoDato(o: Option[Int]): String =
o match {
    case s: Some[Int] => s"tenemos el valor ${s.get}"
    case None => "no tenemos valor" // en este caso, no comparamos por tipo, si no contra el objeto único que representa un Option vacío
}
```




defined <span style="color:green">function</span> <span style="color:cyan">quienEs</span>  
defined <span style="color:green">function</span> <span style="color:cyan">tengoDato</span>  




```scala
quienEs(Jefe("JM", List(Currito("Ar"), Currito("J"))))
quienEs(Currito("Ar"))

tengoDato(Some(23))
tengoDato(None)
```




<span style="color:cyan">res23_0</span>: <span style="color:green">String</span> = <span style="color:green">"JM es jefe de 2 empleados"</span>  
<span style="color:cyan">res23_1</span>: <span style="color:green">String</span> = <span style="color:green">"Ar es un gran trabajador"</span>  
<span style="color:cyan">res23_2</span>: <span style="color:green">String</span> = <span style="color:green">"tenemos el valor 23"</span>  
<span style="color:cyan">res23_3</span>: <span style="color:green">String</span> = <span style="color:green">"no tenemos valor"</span>  



### Extractores

Pero no nos quedemos solo con las limitaciones, porque al fin llega uno de los elementos más potentes del pattern matching (y mi favorito), los extractores. Pongamos que ya somos mayorcitos y no trabajamos solo con tipos simples como String, Int, etc, si no que ya tenemos estructuras más complejas, por ejemplo una tupla. En esta tupla nos gustaría hacer varios casos, segun el contenido de esta. Con nuestro conocimiento actual, podríamos hacer algo tal que así:


```scala
def tuplaMatch(x: (String, Int)): String =
 x match {
     case x if x._1 == "hola" & x._2  == 10 => "hola con valor igual que 10"
     case x if x._1 == "hola" & x._2 > 10 => "hola con valor mayor que 10"
     case x if x._1 == "hola" => "hola con valor menor a 10"
     case x => s"la palabra es ${x._1} con valor ${x._2}"
 }
```




defined <span style="color:green">function</span> <span style="color:cyan">tuplaMatch</span>  




```scala
tuplaMatch(("hola", 10))
tuplaMatch(("hola", 42))
tuplaMatch(("hola", 2))
tuplaMatch(("adios", 42))
```




<span style="color:cyan">res25_0</span>: <span style="color:green">String</span> = <span style="color:green">"hola con valor igual que 10"</span>  
<span style="color:cyan">res25_1</span>: <span style="color:green">String</span> = <span style="color:green">"hola con valor mayor que 10"</span>  
<span style="color:cyan">res25_2</span>: <span style="color:green">String</span> = <span style="color:green">"hola con valor menor a 10"</span>  
<span style="color:cyan">res25_3</span>: <span style="color:green">String</span> = <span style="color:green">"la palabra es adios con valor 42"</span>  



El código es correcto, pero hasta el momento la principal ventaja del pattern matching es una descripción muy gráfica de la lógica en la comprobación. Aquí es donde entra el uso de los extractores, con los que podemos comprobar o asignar los elementos internos de una manera mucho más parecida a la representación de la construcción de la clase.

Por ejemplo, para crear una tupla la forma en la que lo hacemos es poniendo los elementos necesarios entre paréntesis y separados por una coma. En este caso a ser una tupla de dos elementos que se podría representar tal que así:


```scala
val tupla: (String, Int) = ("texto", 1)
```




<span style="color:cyan">tupla</span>: (<span style="color:green">String</span>, <span style="color:green">Int</span>) = (<span style="color:green">"texto"</span>, <span style="color:green">1</span>)




```scala
def tuplaMatch2(x: (String, Int)): String =
 x match {
     case ("hola", 10) => "hola con valor igual que 10" // hacemos uso de comparación
     case ("hola", x2) if x2 > 10 => "hola con valor mayor que 10" // hacemos uso de comparación y de asignación de un elemento interno
     case ("hola", _) => "hola con valor menor a 10"
     case (x1, x2) => s"la palabra es ${x1} con valor ${x2}" // asignamos ambos elementos de la tupla
 }
```




defined <span style="color:green">function</span> <span style="color:cyan">tuplaMatch2</span>  




```scala
tuplaMatch(("hola", 10))
tuplaMatch(("hola", 42))
tuplaMatch(("hola", 2))
tuplaMatch(("adios", 42))
```




<span style="color:cyan">res28_0</span>: <span style="color:green">String</span> = <span style="color:green">"hola con valor igual que 10"</span>  
<span style="color:cyan">res28_1</span>: <span style="color:green">String</span> = <span style="color:green">"hola con valor mayor que 10"</span>  
<span style="color:cyan">res28_2</span>: <span style="color:green">String</span> = <span style="color:green">"hola con valor menor a 10"</span>  
<span style="color:cyan">res28_3</span>: <span style="color:green">String</span> = <span style="color:green">"la palabra es adios con valor 42"</span>  



Hay que tener en cuenta que el uso de extractores no es solo para pattern matching, también se pueden  usar en las asignaciones


```scala
val tupla: (String, Int) = ("texto", 1)
val (primero, segundo) = tupla
```




<span style="color:cyan">tupla</span>: (<span style="color:green">String</span>, <span style="color:green">Int</span>) = (<span style="color:green">"texto"</span>, <span style="color:green">1</span>)
<span style="color:cyan">primero</span>: <span style="color:green">String</span> = <span style="color:green">"texto"</span>  
<span style="color:cyan">segundo</span>: <span style="color:green">Int</span> = <span style="color:green">1</span>  



Los extractores no solo permiten comparar por igualdad, o asignar los elementos internos, también permiten comprobar el tipo de los elementos internos.


```scala
def tuplaAnyMatch(x: (Any, Any)): String =
  x match {
      case (x: String, y: String) => s"dos strings primero: $x segundo: $y"
      case (x: String, _) => s"solo el primero es string: $x"
      case (_, x: String) => s"solo el segundo es string: $x"
      case _ => "ninguno es string"
  }
```




defined <span style="color:green">function</span> <span style="color:cyan">tuplaAnyMatch</span>  




```scala
tuplaAnyMatch(("hola", "adios"))
tuplaAnyMatch(("hola", 42))
tuplaAnyMatch((1, "adios"))
tuplaAnyMatch((1, 42))
```




<span style="color:cyan">res31_0</span>: <span style="color:green">String</span> = <span style="color:green">"dos strings primero: hola segundo: adios"</span>  
<span style="color:cyan">res31_1</span>: <span style="color:green">String</span> = <span style="color:green">"solo el primero es string: hola"</span>  
<span style="color:cyan">res31_2</span>: <span style="color:green">String</span> = <span style="color:green">"solo el segundo es string: adios"</span>  
<span style="color:cyan">res31_3</span>: <span style="color:green">String</span> = <span style="color:green">"ninguno es string"</span>  



Como habrás visto, los extractores son una herramienta muy potente que permite simplificar nuestro código. Esto es tan común, que cuando trabajamos con `case classes` (elemento fundamental para crear ADT's) scala nos crea extractores que siguen la misma estructura de los constructores.


```scala
sealed trait Trabajador
case class Currito(nombre: String) extends Trabajador
case class Jefe(nombre: String, subordinados: List[Trabajador]) extends Trabajador
```




defined <span style="color:green">trait</span> <span style="color:cyan">Trabajador</span>  
defined <span style="color:green">class</span> <span style="color:cyan">Currito</span>  
defined <span style="color:green">class</span> <span style="color:cyan">Jefe</span>  




```scala
def dibujaJerarquia(t:Trabajador, nivel: Int = 0):Unit = {
    val blancos = "  " * nivel
    t match{
        case Currito(n) => println(s"${blancos}- $n 🧑‍🏭")
        case Jefe(n, l) =>
          println(s"${blancos}- $n 😎")
          l.foreach(dibujaJerarquia(_, nivel + 1))
    }
}
```




defined <span style="color:green">function</span> <span style="color:cyan">dibujaJerarquia</span>  




```scala
val empresa = Jefe("A", List(
         Jefe("B", List(Currito("C"), Currito("D"))),
         Jefe("E", List(Currito("F"), Currito("G"))),
         Currito("H")
      )
    )

dibujaJerarquia(empresa)
```

```
    - A 😎
      - B 😎
        - C 🧑‍🏭
        - D 🧑‍🏭
      - E 😎
        - F 🧑‍🏭
        - G 🧑‍🏭
      - H 🧑‍🏭
```

<span style="color:cyan">empresa</span>: <span style="color:green">Jefe</span> = <span style="color:yellow">Jefe</span>(
<span style="color:green">"A"</span>,
<span style="color:yellow">List</span>(
<span style="color:yellow">Jefe</span>(<span style="color:green">"B"</span>, <span style="color:yellow">List</span>(<span style="color:yellow">Currito</span>(<span style="color:green">"C"</span>), <span style="color:yellow">Currito</span>(<span style="color:green">"D"</span>))),
<span style="color:yellow">Jefe</span>(<span style="color:green">"E"</span>, <span style="color:yellow">List</span>(<span style="color:yellow">Currito</span>(<span style="color:green">"F"</span>), <span style="color:yellow">Currito</span>(<span style="color:green">"G"</span>))),
<span style="color:yellow">Currito</span>(<span style="color:green">"H"</span>)
    )
)

## Resumen final
Como habrás visto, el pattern matching es una herramienta que permite simplificar códigos muy complejos de una manera estructurada. No es algo que sea exclusivo de scala, ni siquiera es el primero en tenerlo (otros lenguajes ya han introducido herramientas similares) pero podrás ver que siempre va de la mano con el paradigma funcional. Una tendencia que cada vez vemos en más lenguajes.
En el siguiente post, podrás ver elementos avanzados para poder crear tus propios extractores,  como mejorar los posibles errores que puedan surgirte y más.

## Repositorio
Si prefieres leer este post en formato notebook, lo tienes subido en [github](https://github.com/alfonsorr/pattern-matching-post)