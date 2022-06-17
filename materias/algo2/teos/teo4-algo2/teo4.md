---
layout: page
title: Teórica 4
permalink: /materias/algo2/teo4/
---
# Diseño jerárquico de TADs

Especificación → Diseño → Implementación

- En las primeras clases hablamos mucho de la **especificación,** cuando armamos TADs y eso para los distintos problemas.
- Después vimos la complejidad algorítmica.
- Ahora vamos a concentrarnos más en el **cómo.**

### ¿Qué significa diseñar?

Pasar de la descripción del **qué** al **cómo** del problema. Gracias a las ideas de complejidad vamos a tener más herramientas para poder **elegir el cómo.**

Vamos a empezar a tener en cuenta temas como **tiempo, espacio, encapsulamiento** y **ocultamiento de información.**

Cambiar de paradigma desde el utilizado para especificar al utilizado para programar de forma ordenada. O sea: lenguaje de especificación → lenguaje de diseño.

---

### Ejemplo - TAD Conjunto

- Vemos 2 implementaciones posibles:
    - Un **arreglo redimensionable** ordenado
        - Inserción: $O(n)$
        - Búsqueda: $O(log(n))$
    - Una **secuencia**
        - Inserción: $O(1)$
        - Búsqueda: $O(n)$

¿Cuál nos conviene? → Depende de qué queramos hacer / de qué necesitemos. 

Si vamos a insertar mucho más de lo que vamos a buscar, la **secuencia.** Si vamos a hacer más búsquedas que inserciones, conviene el **arreglo ordenado.**

---

No se puede pasar de la especificación directo a la implementación, por eso está la etapa de diseño.

## Diseñar

**A nivel conceptual:**

- Pasar del **qué** al **cómo**
- **Cambiar de paradigma** (funcional → imperativo)
- Resolver los problemas que surgen como consecuencia de eso

**Más concretamente:**

- Proveer una **representación** para los valores
- Definir las **funciones del tipo**
- Demostrar que eso es **correcto**

¿Por qué diseño jerárquico? Porque vamos a pensar la resolución separando responsabilidades en su construcción. O sea, vamos a ir de a poco. Ejemplo:

$conjuntoNat →_{se\,representa\,con} secuenciaNat →_{se\,representa\,con} punteros$.

### Contexto de uso

¿Cómo **diferenciamos 2 soluciones**? (Como por ejemplo las del TAD Conjunto.)

Depende del **contexto de uso y los requerimientos de eficiencia**.

![Teo%204%20Algo%202%208ec4771175ef4ff693b1e9a274cd622b/Untitled.png](Teo%204%20Algo%202%208ec4771175ef4ff693b1e9a274cd622b/Untitled.png)

Para llegar a la mejor solución, vamos a guiarnos por una metodología. 

### Metodología de diseño

Vamos a usar una filosofía ***top-down***. Arrancamos de lo más abarcativo y nos vamos interiorizando.

- Elección del tipo a diseñar
- Introducción de los elementos no funcionales (eficiencia, contexto de uso y buenas prácticas.)
- Vinculación entre la representación y su abstracción
- Iteración sobre los tipos restantes. O sea tal vez tenemos tipos dentro del tipo a diseñar de los que también nos vamos a tener que encargar.

Podemos dividir esta metodología en 2 sectores:

- **Los aspectos de la interfaz:** 
describen todo elemento relacionado con los aspectos de uso de dicho tipo. O sea, todo lo que resulta visible desde afuera.  **AFUERA**.
- **Las pautas de implementación:** 
son todo aspecto que hable de cuestiones vinculadas a "los medios a través de los cuales el tipo garantiza esos aspectos de uso", es decir, cómo se implementa.  **ADENTRO**.

La **interfaz** de un módulo de diseño debe explicarle al usuario todos los aspectos relativos a los servicios que exporta:

**Servicios exportados**: describe para cada operación su complejidad, aspectos de aliasing, efectos sobre los argumentos, etc.

**Interfaz**: define las operaciones exportadas junto con su precondición y postcondición. Establece la **relación entre la implementación de las operaciones y su especificación**. 

<aside>
🚨 Algo que nos puede pasar en la interfaz es darnos cuenta de que debemos **cambiar la aridad** de una función. 
También nos puede pasar que nos convenga ***mergear* 2 funciones o separar 2 funciones**, dependiendo de qué queremos devolver y cuándo y etc.

</aside>

### Transparencia referencial, aliasing, etc.

En los sevicios exportados, tenemos que explicitar si estamos haciendo algo de esto en nuestras funciones.

1. **Transparencia referencial**
    
    Es una característica de una función. Una función es **referencialmente transparente** si el resultado sólo depende de sus parámetros.
    
    ```c
    f(x){
     return (x+1);
    }
    
    **f(4) + f(3) es referencialmente transparente**
    
    g(x){
    	y = var_global*(x+1);
    	var_global++;
    	return y;
    } 
    
    **g(4) + g(3) no es referencialmente transparente**
    ```
    
    Vemos que en el paradigma funcional, todas las operaciones eran referencialmente transparentes. O sea no dependía del contexto. 
    
    Ahora, en el paradigma imperativo, nos podemos encontrar con funciones **no referencialmente transparentes.**
    
2. **Aliasing**
    
    Significa la posibilidad de tener más de un nombre para la misma cosa. En concreto, **dos punteros hacia el mismo objeto**.
    
    Vemos una función que dado un arbol, devuelve la raíz y los subárboles izquierdo y derecho. 
    
    ```c
    Podar(in A: ab(elem), out I: ab(elem), out R: elem, out D: ab(elem))
    ```
    
    - Implementación 1: armar **copias** de los subárboles y devolverlas como I y D.
    - Implementación 2: devolver en I y D **referencias** a los subárboles de A.
    
    En el segundo caso estaríamos provocando ***aliasing,*** entonces cualquier modificación que le hagamos a I ó a D luego de llamar a esta función va a repercutir en el árbol original A.
    
3. **Manejo de errores**
    
    ```c
    encolar(inout C: cola, in e: elem)
    vs.
    encolar(inout C: cola, in e: elem, out s: status)
    ```
    
    No es lo mismo usar una función o la otra y hay que elegirla según el uso que le queramos dar y **siempre** explicitarlo en nuestra interfaz.
    

---

# Ejemplo - Diseño Conjunto(nat)

## Contexto de uso:

- Obtención del mínimo en $O(1)$
- Quitar un elemento en $O(n)$
- Agregar un elemento en $O(1)$

Una opción sería tener una variable que indique el mínimo. Luego agregar lo agrega en cualquier lugar y quitar "recorre" todo el conjunto hasta encontrar el elemento y lo quita.

## Interfaz

### **Servicios exportados:**

(observadores, generadores, otras)

**pertenece:** no produce aliasing ni efectos colaterales. Complejidad O(n).

**vacío:** no produce aliasing ni etc. Orden O(1).

**agregar:** no produce aliasing, sí modifica el conjunto que le entra. Orden O(1).

**quitar:** no produce aliasing, sí modifica el conjunto. Orden de complejidad O(n).

**vacío?:** no produce efectos. Orden O(1).

**mínimo:** no produce efectos. Orden O(1).

---

### Relación entre los valores imperativos y los valores lógicos

Quisiéramos poder decir lo siguiente:

$pertenece(\textbf{in} \;\;conjuntoDeNat \;\;C, \textbf{in} \;\; nat \;\;n) → res\;\; bool$

**pre** $\{true\}$

**post** $\{res =_{obs} n \in C\}$

<aside>
🚨 Pero **estamos mezclando 2 mundos.** Estamos mezclando lenguaje del tipo imperativo con valores lógicos o funcionales, como la igualdad observacional. ****

</aside>

¿Qué diferencia existe entre los valores que pueden tomar las variables imperativas respecto de los valores lógicos? ¿Cómo lo resolvemos?

Le agregamos ***sombreritos*** a las variables, galerazo.

$pertenece(\textbf{in} \;\;conjuntoDeNat \;\;C, \textbf{in} \;\; nat \;\;n) → res\;\; bool$

$pre\{\textbf{true}\}$

$post\{\hat{res} =_{obs} \hat n \in \hat C\}$

---

¿Por qué tanto énfasis en la interfaz? ¿No es más facil dar el código y listo?

- Es muy importante el proceso de **abstracción.** En cada paso reconocer los aspectos importantes.
- **Ocultar información.** Cada módulo está diseñado para revelar lo mínimo posible sobre cómo funciona.
- **Encapsulamiento**, para que el consumidor tenga visibilidad sobre el funcionamiento, pero no sobre los datos.

**Ventajas:**

- La implementación puede cambiar y mejorar sin afectar su uso.
- Ayuda a modularizar y facilita la comprensión.
- Favorece el reuso. Si tenemos un módulo que hace tal cosa lo podemos reutilizar en otro.
- El sistema es más resistente a cambios y mantenible.

---

## Representación

La representación de un módulo de diseño implica tomar en cuenta cómo se satisfacen los requerimientos declarados en la interfaz.

- **Estructura:** describe la estructura interna.
- **Relación entre la representación y la abstracción:** expone toda restricción sobre la estructura de representación para que pueda ser considerada una implementación correcta del tipo. Vincula los valores con su contraparte abstracta.
- **Algoritmos:** el pseudo-código de las operaciones, tanto las exportadas como las auxiliares, incluyendo el cálculo de su complejidad.
- **Servicios usados:** declara las demandas de complejidad, el aliasing o efectos colaterales que los servicios usados deban satisfacer.

### Estructura de Representación

Describe los valores sobre los cuales se representará el género que se está implementando. Tenemos que ser capaces de representar **todos** los términos del género de la especificación y que las operaciones cumplan las exigencias del **contexto de uso.**

---

—Ejemplo: Conjunto rápido—

Nos piden implementar un Conjunto(nat) de la siguiente manera:

- Los números del 1 al 100 deben manejarse en $O(1)$.
- El resto, en $O(n)$.
- Hay que obtener la cardinalidad en $O(1)$.

Idea:

- Podemos usar un arreglo de 100 elementos booleanos en el que cada índice coincida con el número que contiene, [0, 0, 1, ..., 0].
- Para el resto de los números otro arreglo dimensionable en el que se vayan agregando y sacando normalmente los números que quedan.
- Además, tener una variable que indique cuántos elementos hay.

Cómo lo escribimos:

*conjunto_rapido_nat* **se representa con** 
tupla < 

- *rapido*: **arreglo**[1...100] de **bool** x
- *resto*: **secu(nat)** x
- *cant*: **nat**

> 

<aside>
✅ Todos los conjuntos son representables por esta estructura.

</aside>

Pero.. ¿cualquier instancia de esta estructura representa un conjunto válido?

- <[0, ..., 0], <>, 8254> **no es válido** porque debería tener cant = 0.
- <[0, ..., 0], <36, 104, 22>, 3> **no es válido** porque los menores a 100 van en el arreglo inicial.

---

# Invariante de representación

Tenemos que poder separar con facilidad las instancias válidas e inválidas.

Para:

- Documentar la estructura.
- Agregar las precondiciones como una garantía con la que cuentan nuestros algoritmos.
- Agregar las postcondiciones de modo que nuestros algoritmos no rompan la estructura.
- Como condición necesaria para establecer una relación con la abstracción.
- Usar como guía a la hora de escribir los algoritmos.
- Podemos programar el invariante para chequar instancias inválidas/corruptas.

El **invariante de representación** es una función booleana que tiene como dominio el género de representación y da **true** si es válido y **false** si es inválido. (En realidad el dominio es la versión abstracta del género, con ^).

Por ejemplo, si representamos $T_1$  sobre $T_2$:

$Rep: \hat T_2 \rarr bool$ 

$(\forall t: \hat T_2) \; Rep(t) \equiv$  ... condiciones para que t represente instancia válida de $T_1$

---

Volvemos a nuestro ejemplo:

¿Qué debería decir el invariante?

- *Resto* sólo tiene números mayores a 100, si tiene alguno.
- *Resto* no tiene números repetidos.
- *Cant* tiene la longitud de *resto* más la cantidad de posiciones de *rápido* que están en **true.**

De manera formal:

$Rep: \hat{estr} \rarr bool$

$(\forall e: \hat{estr}) Rep(e) \equiv$

$(1) (\forall n:nat) (esta?(n, \;e.resto) \implies n>100) \; \land$

$(2)(\forall n:nat) (cantApariciones(n, \;e.resto) ≤ 1) \; \land$

$(3) e.cant=long(e.resto) + contarTrues(e.rápido)$

---

# Función de abstracción

Es una herramienta que nos permite vincular una estructura con algún valor abstracto al que representa.

$Abs: \hat T_2 \;e \rarr \hat T_1\,(Rep(e))$

Lo que hace es tomar una instancia abstracta de la estructura de representación y devolver una instancia abstracta del tipo representado. Obviamente la instancia debe cumplir el invariante de representación.

Un término se suele caracterizar en base a los observadores, pero se pueden usar los generadores.

---

Vemos el ejemplo del conjunto de naturales normal.

![Teo%204%20Algo%202%208ec4771175ef4ff693b1e9a274cd622b/Untitled%201.png](Teo%204%20Algo%202%208ec4771175ef4ff693b1e9a274cd622b/Untitled%201.png)

$Abs: <s, n>\;:\;<secuenciaDeNat, nat> \rarr conj[nat]$ 

$(\forall <s,n>\;:\;<secu[nat], nat>) (Abs(<s,n>)) =_{obs} c \;|$

$(\forall n':nat)(esta?(s, n') \iff n' \in c)$ 

Donde

$esta?:secu[nat] \times nat \rarr bool$

$esta?([], n) \equiv false$

$esta?(a \bullet s, n) \equiv (a=n) \lor esta?(s, n)$

---

Ahora vemos el conjuntoRapido

**estr** es
tupla < 

- *rapido*: **arreglo**[1...100] de **bool** x
- *resto*: **secu(nat)** x
- *cant*: **nat**

>

$Abs: \hat{estr} \; e \rarr conjRapido$

$Abs(e) =_{obs} c \;/$

$(\forall n:nat)(n \in C \iff ((n ≤100 \; \land \; e.rápido[n]) \lor$ 

$(n>100 \; \land \; está?(n, e.resto)))$

---

### Propiedades de la Función de Abstracción

- Es una función total, cuando está restringida a Rep(e).
- No tiene por qué ser inyectiva. 2 estructuras diferenter pueden representar el mismo término de un TAD.
- Todo término de un TAD debería ser representable por la estructura, según lo especificado por el contexto de uso.

# Los algoritmos

En la interfaz teníamos esto

$Ag(\textbf{inout}\;C: conjRapido, \textbf{in}\;e: nat)$

$Pre\{\hat C =_{obs} C_0 \land \hat e \notin C\}$

$Post\{\hat C =_{obs} Agregar(C_0, \hat e)\}$

Implementación:

$imp\_Ag(\textbf{inout}\;C: conjRapido, \textbf{in}\;e: nat)$

$Pre\{\hat C =_{obs} C_0 \land \hat e \notin C \land Rep(c)\}$

$C.cant++$

$if \;(e<100) \;then \;c.rápido[e]=true \;else \;ag\_en\_secu(C,e) \;fi$

$Post\{\hat C =_{obs} Agregar(C_0, \hat e) \land Rep(C)\}$

Tenemos una función auxiliar `ag_en_secu(inout E: estr, in e: nat)`.

$ag\_en\_secu(inout\;E:\;estr,\;in\;e:\;nat)$

$Pre\{\hat E =_{obs} E_0 \land e>100\}$

Notar que acá no vale Rep(E). Esto es porque en el principio del algoritmo aumentamos la cardinalidad del conjunto antes de agregarle un elemento, y entonces ahora, en el medio del algoritmo, no es una instancia válida.

$InsertarAlFinal(E.resto, e)$

$\{\hat E.resto =_{obs} E_0.resto \circ \hat e\}$

<aside>
🚨 No es necesario que **la estructura de representación** se cumpla **en el medio del algoritmo.** Es suficiente **que se cumpla cuando inicia y cuando termina.**

</aside>