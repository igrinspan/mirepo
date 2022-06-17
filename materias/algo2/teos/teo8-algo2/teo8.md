---
layout: page
title: Teórica 8
permalink: /materias/algo2/teo8/
---

# Colas de prioridad y heaps

Colas → FIFO (First In First Out)

Pilas  → LIFO (Last In First Out)

# Colas de prioridad

**El orden de salida no tiene que ver con el orden de llegada**, sino con una característica particular de los elementos que le da más o menos prioridad que al resto.

La prioridad suele ser un entero, pero puede ser cualquier tipo $\alpha$ con un orden $<_\alpha$ asociado. Hay correspondencia entre máxima prioridad y un valor máximo o mínimo del tipo $\alpha$.

Por ejemplo, en una guardia, la prioridad está dada por la gravedad de la enfermedad.

<aside>
⚠️ **Prioridad alta**: sale antes, es más importante.
**Prioridad baja**: sale después, menos importante.

</aside>

El **TAD Cola de prioridad** es prácticamente igual al de la cola, sólo que además del tipo toma un orden asociado a ese tipo. Tanto la operación **próximo** como **desencolar** toman al elemento de mayor prioridad.

### ¿Cómo se podría implementar una cola de prioridad?

Listas y arreglos: sí, muy parecido a las listas y arreglos normales.

Árboles AVL: podemos hacer que el mínimo esté a la izquierda y crezca hacia derecha, es lo mismo que hacíamos para enteros.

Nueva estructura, muy asociada a la cola de prioridad.

# Heap

Es mucho más sencillo que hacer un AVL y es la implementación más eficiente para las colas de prioridad. Significa "montón".

La estructura tiene un ordenamiento no tan completo. O sea no está toda ordenada.

## Representación de Colas de Prioridad - Heap

- Cola de prioridad($\alpha, <_\alpha$) se representa con heap.
- Invariante de representación:
    1. Árbol binario perfectamente balanceado. (O sea la longitud de todas las ramas difiere en, como mucho, 1).
    2. La clave (prioridad) de cada nodo es mayor o igual que la de sus hijos, si tiene.
    3. Todo subárbol es un heap.
    4. Es izquierdista, o sea, está lleno desde la izquierda. (no siempre).

Como todo subárbol es un heap y la prioridad de cada nodo es mayor o igual a la de sus hijos, en la raíz está el nodo de máxima prioridad. 

Estos se denominan **máx-heap**, existe también el **mín-heap**, cambiando mayor por menor.

- Podemos ver quién está en la raíz en $O(1)$.

## Implementación de heaps

Todas las representaciones usadas para árboles binarios valen:

- Representación con punteros (hijo-padre).
- Representación con arrays (muy eficiente). La posición de los elementos nos va a dar información sobre su prioridad en relación con otros elementos. Por eso es una representación implícita.

### **Representación con arrays**

- Cada nodo $v$ es almacenado en la posición $p(v)$.
    - Si $v$ es la raíz, entonces $p(v) = 0$.
    - Si $v$ es el hijo izquierdo de $u$, entonces $p(v) = 2p(u)+1$
    - Si $v$ es el hijo derecho de $u$, entonces $p(v) = 2p(u)+2$

![Teo%208%20Algo%202%20426d6a6b2b5841abae2b91e272481ee1/Untitled.png](Teo%208%20Algo%202%20426d6a6b2b5841abae2b91e272481ee1/Untitled.png)

![Teo%208%20Algo%202%20426d6a6b2b5841abae2b91e272481ee1/Untitled%201.png](Teo%208%20Algo%202%20426d6a6b2b5841abae2b91e272481ee1/Untitled%201.png)

Se puede calcular con una suma y una multiplicación la posición de los hijos y está bueno porque también, dependiendo de la posición de un nodo, podemos encontrar al padre. 

Fácil de navegar, muy eficientes en términos de espacio.

Implementación estática, puede haber que duplicar el arreglo cuando queremos agregar un elemento. No va a ser útil para árboles que no estén completos.

---

### Algoritmos (independientemente de la representación)

- Crear_Vacío:
    - Fácil, $O(1)$.
- Próximo:
    - El elemento de máxima prioridad está en la posición 0 del array, $O(1)$.
- Inserción/Encolar:
    - Metemos el nodo en la posición más izquierdista posible, independientemente de su valor.
    - Nos fijamos que sea menor que el padre. Si es mayor, los intercambiamos. Comparar con su nuevo padre y hacemos lo mismo, hasta que sea menor o igual a su padre.
    
    ![Teo%208%20Algo%202%20426d6a6b2b5841abae2b91e272481ee1/Untitled%202.png](Teo%208%20Algo%202%20426d6a6b2b5841abae2b91e272481ee1/Untitled%202.png)
    
    ```
    Encolar:
    	insertar elemento al final del heap
    	subir(elemento)
    
    Subir:
      while (elem != raíz) && prioridad(elem)>prioridad(padre)
    		intercambiar(elem, padre)
    ```
    
- Desencolar: (eliminar la raíz)
    - Sacamos a la raíz.
    - Mandamos el último elemento a la raíz.
    - Nos fijamos que sea mayor que su hijo mayor (al principio no va a pasar), entonces lo vamos haciendo bajar por la rama de su hijo mayor, intercambiándolo en cada paso hasta que sus hijos sean menores o sea una hoja.
    
    ```
    Desencolar:
    	reemplazar el primer elemento con la última hoja
    	eliminar la última hoja
    	bajar(raíz) //comparando con su hijo mayor
    
    Bajar:
    	while (p != hoja) && prioridad(p) < prioridad(hijo_p)
    		intercambiar(p, hijo_p)
    
    donde hijo_p es el hijo de mayor prioridad
    ```
    
    <aside>
    ⚠️ Está bueno ir comparando con su hijo mayor porque sabemos que si es mayor a ese, entonces es mayor a ambos.
    Además, si los tenemos que intercambiar, sabemos que el nuevo padre va a ser mayor a sus dos hijos.
    
    </aside>
    

### Complejidades

Crear un heap vacío y encontrar el próximo cuestan $O(1)$.

Tanto para encolar como para desencolar, los costos son proporcionales a la altura del heap: $O(log (n))$.

---

## Array2Heap

Dado un array cualquiera $arr$, lo transforma en un array que representa un heap a través de una permutación de sus elementos.

**Algoritmo básico:**

```
array2heap(heap, arr):
	for i=0 to len(arr):
		encolar(heap, arr[i])
```

A medida que se nos va armando el heap, se hace más costoso encolar. El costo de encolar **todos los elementos** es de  $\approx$ $\;\Theta(nlog(n))$.

**Algoritmo de Floyd:**

Es una estrategia bottom-up. Vamos a suponer la representación del arreglo como un árbol binario común, sin ordenamiento. 

A partir de eso, vamos a ir buscando, para cada subárbol, desde abajo hacia arriba, si es un máx(o mín)-heap. Si nos fijamos un subárbol y no es un máx-heap, vamos bajando su raíz por el lado de su hijo más grande hasta que no tenga un hijo más grande que él (intercambiándolos en cada paso).

El **peor caso** se da cuando cada llamada a bajar hace el máximo número de intercambios.

$O(n)$ para encontrar la clave, $O(1)$ para eliminarla y $O(n)$ para reconstruir la representación de heap. 

También nos va a servir para ordenar arreglos.