---
layout: page
title: Teórica 5
permalink: /materias/algo2/teo5/
---

# Conjuntos y Diccionarios - ABB y AVL

**Conjuntos y diccionarios son los TADs que suelen constituir los problemas más importantes.**

![Teo%205%20Algo%202%201bfcab6a886449fd95be77b84275f457/Untitled.png](Teo%205%20Algo%202%201bfcab6a886449fd95be77b84275f457/Untitled.png)

¿Qué cosa se puede modelar a través del tipo abstracto Diccionario?

Por ejemplo, un sistema que usa DNIs como claves para almacenar datos sobre personas.

- Un diccionario clásico: **clave** son palabras, **significado** sus significados.
- El texto predictivo: **clave** son strings, **significado** un conjunto de las posibles palabras que se pueden formar.
- El padrón electoral: **clave** son DNIs, **significado** nombre y domicilio.
- Sistema de la facultad: **clave** es número de LU y **significado** una lista de materias aprobadas.
- Corrector ortográfico: **clave** son palabras, significado es ... (no nos importa el significado, sólo queremos saber si esa palabra está definida o no.)  → Podemos usar un conjunto para ver esto, fijándonos si la palabra está en el conjunto o no. Los conjuntos son un caso particular de los diccionarios.

 

---

## Representación secuencial de conjuntos y diccionarios.

¿Cómo se podrían representar?

Por ejemplo, con un arreglo o secuencia de tuplas, en el que el primer elemento sea la clave y el segundo el significado. Y además una variable que indique la última posición ocupada.

![Teo%205%20Algo%202%201bfcab6a886449fd95be77b84275f457/Untitled%201.png](Teo%205%20Algo%202%201bfcab6a886449fd95be77b84275f457/Untitled%201.png)

¿Se podrán encontrar mejor representaciones de los diccionarios y conjuntos? O sea, que todas sus operaciones tengan complejidad    **sub-lineal.**

## Árboles Binarios

- Árbol sin ninguna consideración
    - $def$? → $O(n)$
    - $obtener$ → $O(n)$
    - $vacío$ → $O(1)$
    - $definir$ → $O(1) \;u \;O(n)$

Para def? y obtener hay que recorrer todo el árbol, porque no tiene ningún orden particular. Por eso son $O(n)$.

## Árbol Binario de Búsqueda (ordenado)

- Árbol ordenado
    - $def?$ → $O(n)$
    - $obtener$ → $O(n)$
    - $vacío$ → $O(1)$
    - $definir$ → $O(1) \;u \;O(n)$

No cambian las complejidades porque un árbol puede ser ABB pero estar todo a izquierda o todo a derecha y habría que recorrerlo por completo para esas operaciones. (O sea en el peor caso es $O(n)$, sino estuviera mejor **balanceado** se podría hacer más rápido).

![Teo%205%20Algo%202%201bfcab6a886449fd95be77b84275f457/Untitled%202.png](Teo%205%20Algo%202%201bfcab6a886449fd95be77b84275f457/Untitled%202.png)

Vemos ejemplos de Árboles Binarios de Búsqueda.

### Invariante de Representación - ABB

$ABB: ab(nodo) \rarr boolean$

$\forall e: ab(nodo)$

$ABB(e) \equiv Nil?(e) \;\;\lor_{L}$

$[\{(\forall c: clave)(Está(c, Izq(e)) \implies (c <_{clave} LaClave(Raíz(c))))
\;\;\land$ 

$(\forall c: clave)(Está(c, Der(e)) \implies (c >_{clave} LaClave(Raíz(c))))\}
\;\;\land$ 

$ABB(Izq(e)) \;\;\land \;\;ABB(Der(e))

]$

### Algoritmos - ABB

- $vacío()$
- $**definir(clave, significado, arbol)**$
    
    Vamos recorriendo el árbol yendo a derecha o izquierda hasta encontrar el padre. ¿Cuándo encontramos el padre? Cuando nuestra clave es mayor al nodo y no tiene hijo derecho o cuando es menor al nodo y no tiene hijo izquierdo.
    
- $**borrar(u, A)**$
    
    3 casos: u es hoja, u tiene un solo hijo, u tiene 2 hijos.
    
    si es hoja, no hay que cambiar mucho. sólo lo sacamos.
    
    si tiene un hijo: hay que fijarse si tiene hijo derecho o izquierdo y conectarlo con el padre del nodo a eliminar.
    
    si tiene 2 hijos: podemos usar la técnica del **predecesor inmediato** o la del **sucesor inmediato.** Consiste en encontrar el predecesor inmediato, pegar su valor en el nodo que queremos eliminar y eliminar el nodo del predecesor inmediato, que va a ser hoja o tener solamente un hijo (izquierdo), y para eso tenemos los casos anteriores. ¿Cómo los encontramos? Buscando el mínimo del subárbol derecho (sucesor) o el máximo del subárbol izquierdo (predecesor).
    
    tienen complejidad lineal, $O(n)$
    

---

**¿Cómo resolvemos la complejidad lineal?**

## Árboles balanceados

El objetivo es que todas las ramas sean lo más parejas posibles. O sea, que no haya un nivel nuevo si el anterior nivel todavía no está completo.

Para un **árbol completo,** si tenemos $n$ nodos, la altura es $log_2(n)$.

Pero tener árboles completos es **muy caro,** vamos a usar una estructura un poco más flexible. → **balanceo.**

**Balanceo Perfecto:**

Cuando todas las ramas tienen la misma longitud y todos los nodos internos tienen 2 hijos (o 1 si se está completando ese nivel).

Teorema:

Un árbol binario perfectamente balanceado de $n$ nodos tiene altura $log_2(n) +1$

Demo: 

$n_h= n_i+1$, con $n_h$ = #hojas y $n_i$ = #nodos_internos.

Luego las hojas son más del 50% de los nodos.

Supongamos que tenemos un árbol y queremos podarlo. Vamos a ir podando las hojas. O sea en el primer paso, podamos el último nivel, luego el anteúltimo y así sucesivamente hasta llegar a la raíz. 

<aside>
🚨 $**log_2(n)$ da cuántas veces podemos dividir $n$ en $2$  hasta llegar a la unidad.**

</aside>

- [ ]  Ver bien y entender clase 5_4.

---

## Balanceo en Altura  - AVL

Un árbol está balanceado en altura si las alturas de los subárboles izquierdo y derecho **de cada nodo** difieren en, como mucho, una unidad.

¿Cuán altos son los AVL?

Queremos demostrar que la altura de un árbol de $n$ nodos es menor a $log_2(n)$.

<aside>
🚨 Si quiero demostrar que la altura de un árbol con n nodos es menor a $f(n)$, es lo mismo que demostrar que un árbol con altura h tiene más que $f^{-1}(h)$ nodos.

</aside>

Entonces, lo que queremos demostrar es que un árbol con altura $h$ tiene más que $exp(h)$ nodos. O sea, más que $e^h$ nodos.

Mínimo numero de nodos de un AVL según la altura:

- 1 → 1 nodo (la raíz)
- 2 → 2 nodos (la raíz y un hijo)
- 3 → 4 nodos (la raíz, ambos hijos, y un hijo para uno de ellos)
- 4 → 7 nodos
- 5 → 12 nodos

...

![Teo%205%20Algo%202%201bfcab6a886449fd95be77b84275f457/Untitled%203.png](Teo%205%20Algo%202%201bfcab6a886449fd95be77b84275f457/Untitled%203.png)

La cantidad de nodos va siguiendo la **sucesión de Fibonacci - 1.**

Número de nodos
$AVL_{i} = F_{i+2} -1$ .

La sucesión de Fibonacci crece en orden exponencial en i. Luego la altura de un árbol AVL, dada una cantidad de nodos, la máxima altura tiene **orden logarítmico.**

Los árboles de Fibonacci tienen todos los factores de balanceo en $\pm \;1$.

Luego un AVL de $n$ nodos tiene altura $\Theta(lg(n))$.

### Algoritmos de los AVL

Inserción y borrado. 1 hora.

### Inserción:

Si se caga el factor de balanceo, hacemos rotaciones. Puede ser LL, RR, LR o RL.

### Borrado:

Igual que en inserción, si después de borrar se caga algún factor de balanceo, rotamos. Acá usa lo de buscar el máximo del subárbol izquierdo o el mínimo del lado derecho, cuando borra a un nodo con 2 hijos.