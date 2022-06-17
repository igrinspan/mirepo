---
layout: page
title: Teórica 12
permalink: /materias/algo2/teo12/
---
# Skip Lists y Splay Trees

Son estructuras que forman parte de algunos algoritmos probabilísticos que vamos a ver.

# Skip Lists

- Supongamos que tenemos diccionarios con listas enlazadas en orden.
- Son muy simples pero ineficientes, porque la búsqueda es en $O(n)$.
- Pero podemos agregar, a cada nodo, un puntero al siguiente del siguiente (o sea que se salte un elemento). Así avanzaría más rápido nuestro iterador.
- Eso no parece suficiente, podríamos agregar otra cosa (que cada nodo tenga un puntero que avance de a 4, además de los otros 2).
- Así sucesivamente... cada nodo posee una referencia al nodo $2^i$ posiciones más adelante en la lista.

Se dice que los nodos de abajo son de nivel 1, los que tienen la segunda flechita son de nivel 2 y así sucesivamente. En este caso llegan hasta el nivel 4 (el nodo 70).

![Teo%2012%20Algo%202%205f99fa012a7c4535a0ba3392fbd3aee6/Untitled.png](Teo%2012%20Algo%202%205f99fa012a7c4535a0ba3392fbd3aee6/Untitled.png)

El número total de referencias es **sólamente el doble**, a pesar de que parezca que estamos agregando referencias exponencialmente.

Cómo máximo se examinan $O(log_2(n))$ nodos durante una búsqueda. Muy parecido a la búsqueda binaria.

<aside>
⚠️ Se complica para cuando tengamos que agregar cosas. O sea está piola para búsqueda, ¿pero para insertar?

</aside>

**Pero...** hay una manera de resolver ese problema de rigidez.

---

Algoritmos probabilísticos vs. determinísticos

El algoritmo de Montecarlo, para encontrar primalidad de un número.

---

ABB óptimos

### **Splay Trees**

No requieren memoria adicional y tienen muy buena performance en secuencias de acceso no uniformes.

Podemos garantizar $O(mlog(n))$ para secuencias de $m$ operaciones. 

Los ST son asintóticamente tan eficientes como cualquier ABB fijo.

Y los ST son asintóticamente tan eficientes como cualquier ABB que se modifique a través de rotaciones.

— $\bull$ —

Splaying:

Si accedemos a la raíz del árbol, no hacemos nada.

Si accedemos a k y su padre es la raíz, hacemos una rotación simple.

Si accedemos a k y el padre de k no es la raíz, hay 2 casos: rotación zig-zag o rotación zig-zig. Hay que aplicar alguno de los dos.

Como efecto del splaying, se mueven todos los nodos del camino desde la raíz hasta el nodo accedido.

El zig-zag lo hacemos cuando el padre del nodo k no es la raíz y k es hijo derecho y su padre hijo izquiero o viceversa.

El zig-zig lo hacemos cuando k es el hijo izquierdo de un hijo izquierdo o es el hijo derecho de un hijo derecho. 1ro hacemos la rotación sobre el padre p y luego sobre el k.

— $\bull$ —

Inserción y borrado:

Para insertar hacemos como siempre. Buscamos el lugar donde deberíamos insertar y vamos efectuando las rotaciones. El elemento insertado queda en la raíz. 

Para borrar un elemento se realiza una búsqueda al nodo a eliminar, lo que lo deja en la raíz. Se elimina y se obtienen 2 subárboles. El elemento mayor del árbol izquierdo se busca y pasa a ser la raíz. Como era el mayor, no tenía hijo derecho, entonces se cuelga el subárbol derecho original de la nueva raíz.

— $\bull$ —

Lamentablemente no podemos controlar cómo se balancean los árboles. En el peor caso nos podría quedar como una lista enlazada. Pueden traer problemas también porque cada vez que buscamos elementos cambia la forma de la estructura y nos puede llegar a cagar todo. Son autoajustables.

---

Listas auto-ajustantes

Usan política Move-to-Front (MTF). El elemento accedido se mueve a la raíz (al primer nodo de la lista).

Teorema: el costo total para una secuencia de m operaciones es como mucho el doble que el de cualquier implementación usando listas.

---

# Análisis Amortizado

Es un análisis que da garantías pero no sobre el costo de cada operación sino sobre secuencias de operaciones. Entonces no se puede garantizar que cada operación tarde x, pero sí se puede garantizar que m operaciones tarden m*x.

Método del banquero:

Método del potencial: