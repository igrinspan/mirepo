---
layout: page
title: Teórica 6
permalink: /materias/algo2/teo6/
---
# Tries

Los AVL tienen complicación en sus algoritmos, lo que los hacen molestos. Por ejemplo el borrado.

Por eso, para algunas situaciones, vamos a encontrar mejores soluciones.

### Características de los Tries

- Queremos buscar una implementación de un diccionario que no dependa tanto de la cantidad de claves n.
- Nos vamos a enfocar más en el **tamaño de las claves.**
- Son muy rápidos en la práctica y con rendimiento razonable en el peor caso.
- Se suelen usar para **claves de tamaño variable.**
- Se suele usar para "text retrieval".

<aside>
💡 **Idea**: No hacer comparaciones de claves completas, sino de **partes** de ellas.

</aside>

Por ejemplo, si las claves son strings, vamos a trabajar con caracteres. Si las claves son enteros, vamos a trabajar con dígitos (o dígitos binarios/bits).

### Desventajas

- Pueden ser muy ineficientes en términos de memoria. Consumen mucha.
- Las operaciones sobre las componentes de las claves puede ser difícil e ineficiente en algunos lenguajes de alto nivel.

---

## 1era estructura - Árboles de Búsqueda Digital

Son muy parecidos a los ABB, pero la posición no está determinada por comparaciones, sino por los bits o caracteres de la clave. 

Por ejemplo, para guardar una clave, vamos siguiendo nuestra clave moviéndonos a derecha o izquierda en cada paso.

Ejemplo Árbol de Búsqueda Digital

![Teo%206%20Algo%202%20e3ef768b333d4beea2ea245cd5da0244/Untitled.png](Teo%206%20Algo%202%20e3ef768b333d4beea2ea245cd5da0244/Untitled.png)

**Vemos que la complejidad depende de la longitud de las claves.**

- El peor caso es mucho mejor que el de los ABBs
    
    El número de claves es grande y las claves no son tan largas.
    
- La longitud del camino más largo es el mayor número de bits sucesivos iguales. O sea, el mayor **prefijo común.**
- La búsqueda o inserción en un árbol de $n$ claves de $b$ bits, se necesitan en promedio $log_2(n)$ comparaciones de clave completa, y $b$ en el peor caso.
    
    

---

## Árboles Tries

- El nombre proviene de "retrieval". Es un árbol $(k+1) \;ario$ para alfabetos de $k$ elementos.
- La información no va a estar asociada a los nodos del árbol, sino a los **ejes/líneas.** Estos van a representar componentes de las claves.
- **Idea:** Cada subárbol representa al conjunto de claves que comienza con las etiquetas de los ejes que llevan desde la raíz hasta él.
- **Sólo puede haber claves en las hojas**. No están en los nodos internos.

### Propiedades interesantes

- Sólo hacemos comparación de clave completa cuando lleguemos a la hoja. (Entre tanto vamos comparando de a bits o caracteres).
- La estructura del árbol es la misma, independientemente de qué elemento agreguemos primero. Entonces hay un único trie para cada conjunto de claves.
- La **búsqueda/inserción** de $n$ claves de $b$ bits, necesita $log_2(n)$ comparaciones de bits en promedio y $b$ comparaciones en el peor caso.

### Implementación de Tries

1. Memoria estática   → Arreglos
2. Memoria dinámica → Listas

**Usando Arreglos de Punteros**

- **1era posibilidad:** en cada nodo interno tener un arreglo de punteros.

![Teo%206%20Algo%202%20e3ef768b333d4beea2ea245cd5da0244/Untitled%201.png](Teo%206%20Algo%202%20e3ef768b333d4beea2ea245cd5da0244/Untitled%201.png)

Está bueno (es rápido porque sabemos el índice de cada letra), pero **ocupa** **demasiada memoria,** sobre todo si hay letras que se usan poco.

Se va a desperdiciar mucha memoria cuando haya un **alfabeto grande** y un **diccionario chico.**

**Usando Listas Enlazadas**

- **2da posibilidad:** cada vez que necesitamos un caracter, lo agregamos como si fuera una lista enlazada. Cada nodo tiene un puntero a su primer hijo y a su primer hermano.

![Teo%206%20Algo%202%20e3ef768b333d4beea2ea245cd5da0244/Untitled%202.png](Teo%206%20Algo%202%20e3ef768b333d4beea2ea245cd5da0244/Untitled%202.png)

En este caso la raíz no tiene nada, las palabras comienzan en el nivel 1.

- Es eficiente en términos de tiempo si hay pocas claves.
- Es **mucho más eficiente** en términos de memoria.
- Hay que lidiar con las operaciones de listas enlazadas.

### Tries Compactos

Son iguales a los Tries, pero colapsamos (zipeamos) las cadenas que llevan hacia hojas. 

![Teo%206%20Algo%202%20e3ef768b333d4beea2ea245cd5da0244/Untitled%203.png](Teo%206%20Algo%202%20e3ef768b333d4beea2ea245cd5da0244/Untitled%203.png)

Por ejemplo, como ya sabemos que la única palabra que empieza con **B-I-G-G** es **BIGGER,** colapsamos esa cadena y terminamos la rama ahí.

### Tries Compactos 2.0 - PATRICIA

No sólo compactamos las cadenas que van a la hoja, sino todas las cadenas internas también. Un eje puede representar más de un caracter, como en este caso **B-I** y **G-O.**

![Teo%206%20Algo%202%20e3ef768b333d4beea2ea245cd5da0244/Untitled%204.png](Teo%206%20Algo%202%20e3ef768b333d4beea2ea245cd5da0244/Untitled%204.png)

Se aprovecha mucho más el espacio pero los algoritmos se vuelven bastante más complejos, sobre todo para los nodos en los que hay un eje con más de un caracter.

- La altura de un árbol PATRICIA está acotado por el número de claves n, que antes no pasaba.