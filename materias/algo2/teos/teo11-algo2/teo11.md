---
layout: page
title: Teórica 11
permalink: /materias/algo2/teo11/
---
# Sorting y búsqueda en memoria secundaria

Bit → Byte → KiloByte → MegaByte → GigaByte → Tera → Peta → ...

### Introducción

- Vamos a ver casos en los que los arreglos que tengamos que ordenar sean muuuuuuy grandes. Por ejemplo, cuando son más grandes que nuestra memoria principal.
- Generalmente, en los dispositivos más grandes caben más cosas (son más lentas) y en los más chicos caben menos (pero son más rápidas).
- En las computadoras, tenemos una CPU y una RAM. Además puede tener un disco  y/o dispositivos de cinta. Hay una gran diferencia en velocidad de acceso.

### Ideas y motivaciones

- Vamos a pensar los mismos problemas que estábamos trabajando pero para el caso en el que los arreglos no quepan en la RAM. Teóricamente, por cada acceso a memoria secundaria, se podrían hacer 1.000.000 de accesos a memoria principal (en cuanto a velocidad).
- Lo que nos va a guiar en el diseño de los algoritmos es **minimizar la cantidad de veces que un elemento hace el viaje 
*memoria secundaria - memoria principal.***
- El tiempo de acceso está dominado por el posicionamiento para obtener el dato, no tanto por la transferencia. Por eso conviene acceder a bloques de datos simultáneamente.

---

# Ordenación - Fusión

Los métodos hacen una pasada sobre el archivo, dividiéndolo en bloques que entren en memoria interna (memoria principal), ordenando esos bloques y luego fusionándolos. Es la misma idea que el *Divide & Conquer.*

El objetivo es minimizar el número de pasadas sobre el archivo.

## Método 1: Fusión Múltiple Equilibrada

Supongamos que tenemos que ordenar los registros con valores EJEMPLODEORDENACIONFUSION (cada letra es un registro en una cinta, entonces sólo pueden ser accedidos secuencialmente).

En memoria sólo hay espacio para 3 registros, pero por fuera tenemos infinitas cintas para usar. (Fusiones a 3 vías).

- Primera pasada:
    
    Leemos 3 registros a la vez, los ordenamos y lo escribimos en la Cinta 1 (vacía).
    
    Luego otros 3 y van a la Cinta 2.
    
    Luego otros 3 y van a la Cinta 3.
    
    Para los siguientes 3 volvemos a la cinta uno y seguimos sucesivamente.
    
- Segunda pasada:
    
    Estamos en esta situación.
    
    ![Teo%2011%20Algo%202%20443f1e5307df4ff3a58c873fb908f3de/Untitled.png](Teo%2011%20Algo%202%20443f1e5307df4ff3a58c873fb908f3de/Untitled.png)
    
    Vamos a combinarlos comparando cada cinta, empezando desde la primera posición y vamos insertándolos en orden. Es igual que el combine del merge sort pero con 3 partes en vez de 2. **Nos queda así:**
    
    ![Teo%2011%20Algo%202%20443f1e5307df4ff3a58c873fb908f3de/Untitled%201.png](Teo%2011%20Algo%202%20443f1e5307df4ff3a58c873fb908f3de/Untitled%201.png)
    
    (las cintas quedan de 9 porque el ordenamiento es por cada bloque de 3x3 que había arriba).
    

### **Complejidad:**

N registros, memoria interna de tamaño M, fusiones a P vías.

- La primera pasada produce aprox. N/M bloques ordenados.
- Se hacen fusiones a P vías en cada paso posterior, lo que requiere aprox. $log_p(\frac{N}{M})$ pasadas.

Por ejemplo, con un archivo de 200 millones de palabras y un millón de palabras en memoria. Fusión de 4 vías. → No más de 5 pasadas.

---

## Método 2: Selección por sustitución

La idea es usar una cola de prioridad de tamaño M para armar bloques, luego pasar los elementos a través de la cola de prioridad escribiendo siempre el más pequeño y sustituyéndolo por el siguiente. Pero, si el nuevo es menor que el que acaba de salir, lo marcamos como si fuera mayor hasta que alguno de esos nuevos llegue al primer lugar de la cola.

- EJEMPLODEORDENACIONFUSION

— explicación —

Propiedad: para secuencias aleatorias, las secuencias creadas son aproximadamente el doble del tamaño del heap utilizado.

Efecto práctico: ganamos una pasada de fusión sobre la secuencia. Si hay algún tipo de orden en las claves, las secuencias que surgen van a ser más grandes (porque van a haber muchos elementos ordenados).

---

## Método 3: Fusión polifásica

Muy parecido al primer método pero sin suponer que tenemos todas las cintas que necesitamos.

En cada paso, se debe distribuir los datos en todas las cintas menos una, aplicando estrategia de "fusión hasta el vaciado". Cuando una cinta se vacía, se puede rebobinar y usar como cinta de salida.

---

# Búsqueda externa (diccionarios)

La idea es encontrar una palabra entre un millón con 2 o 3 accesos a disco

Los métodos usados son más parecidos a los de memoria principal que los de recién. Esta problemática es muy común en Bases de Datos.

Acceso a páginas = bloques contiguos de datos a los que se puede acceder eficazmente.  

**Métodos:**

- Acceso Secuencial Indexado (ISAM)
- Árboles B
- Hashing extensible

## Método 1: Acceso Secuencial Indexado (ISAM)

EJEMPLODEBUSQUEDAEXTERNA

![Teo%2011%20Algo%202%20443f1e5307df4ff3a58c873fb908f3de/Untitled%202.png](Teo%2011%20Algo%202%20443f1e5307df4ff3a58c873fb908f3de/Untitled%202.png)

Se puede acelerar, además de con los índices de cada página, con un índice maestro en memoria principal. Este que diga, para cada disco, cuál es la última clave que tiene.

El único problema de este método es que si hay inserciones y borrados puede llegar a hacer reorganizaciones y demás. Está bueno para estructuras rígidas, que no cambian.

Búsqueda en O(1) e inserción en O(n).

## Método 2: Árboles B

Son árboles balanceados y se usa mucho en la práctica. Está súper balanceado, todas las hojas están a la misma distancia de la raíz (no necesariamente es binario).

La idea es acceder a una página de memoria secundaria por nivel. Entonces los nodos tienen relación con las páginas de disco que mencionamos en el método anterior.

**Antecedente:** Árboles 2-3-4

Cada nodo puede contener 1, 2 ó 3 claves.

1 clave → 2 hijos

2 claves → 3 hijos

3 claves → 4 hijos

![Teo%2011%20Algo%202%20443f1e5307df4ff3a58c873fb908f3de/Untitled%203.png](Teo%2011%20Algo%202%20443f1e5307df4ff3a58c873fb908f3de/Untitled%203.png)

Para insertar vamos hasta abajo de todo y si el nodo está completo, lo partimos y empezamos a subir (explicación completa en el video 3 de la teo 11).

- Búsqueda, inserción y borrado en $O(log(n))$.
- Los algoritmos son complicados por el manejo de los distintos tipos de nodos.

**Volvemos a los Árboles B**

Como los 2-3-4, pero cada nodo puede tener hasta M-1 claves (o sea M hijos/enlaces).

Para desplazarse hay que buscar el intervalo apropiado y bajar por ese enlace. Cuando los nodos están llenos, los partimos en 2 y sube la del medio, como en los 2-3-4.

La idea es que cada nodo corresponda a una página del disco. Incluye claves, significados (de ser necesario) y enlaces.

Un M grande implica mayor costo en cada nodo.

---

## Método 3: Hashing Dinámico o Extensible

Es un mix entre los 2 métodos anteriores con algo de hashing.

Como los árboles B, los registros se almacenan en páginas que cuando se llenan se dividen en 2.

Como en el ISAM, se mantiene un índice al que se accede para encontrar la página que contiene los registros que concuerdan con la clave.

Cada clave va a ser insertada en base a una función de hash. Los elementos que comparten el mismo valor de función de hash van a ir a parar a la misma página.

Va a haber un solo acceso a memoria secundaria por operación.