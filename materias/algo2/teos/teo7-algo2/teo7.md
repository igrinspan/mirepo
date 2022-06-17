---
layout: page
title: Teórica 7
permalink: /materias/algo2/teo7/
---

# Conjuntos y diccionarios con Hashing

Hasta el momento habíamos usado arreglos, listas, árboles, tries.

Habíamos visto que, por ejemplo, un arreglo ordenado estaba bueno para buscar, porque tenía O(log(n)), pero era más costoso para agregar y borrar.

Luego surgieron los árboles binarios, que no resolvían mucho pero dándole una vuelta de tuerca vimos los árboles balanceados AVL, que parecían ser lo mejor.

# Tablas hash

Se utilizan por ejemplo en mapeos entre las variables y la posición de las mismas en memoria, en los lenguajes de programación. También para los mapeos de memoria virtual/física.

Son una generalización del concepto de arreglo. También podemos indexar cosas que no son necesariamente enteros positivos.

## Direccionamiento directo

Por ejemplo, si queremos un diccionario que tenga DNIs como claves e información extra como significado. Supongamos que es para alojar los datos de 10.000 clientes.

![Teo%207%20Algo%202%2089f6dce966d14cc7b378c735d4543de8/Untitled.png](Teo%207%20Algo%202%2089f6dce966d14cc7b378c735d4543de8/Untitled.png)

La búsqueda es en $O(1)$. ✅

Mucho desperdicio de memoria. Hay $10^4$ clientes y $10^8$ DNIs posibles. ❌

Sólo se pueden indexar enteros positivos (justo para DNI nos sirve pero queremos que se puedan otros tipos).

---

**Objetivos:**

1. Indexar otros tipos de datos
2. $n$ = #claves efectivamente usadas. O sea, que no gastemos memoria al pedo.
3. Tiempo de acceso $O(1)$

1. Pre-Hashing: 
    
    Tenemos que tener una **función de correspondencia entre cualquier tipo de datos y un entero.**
    
    - Por ejemplo, como cualquier clave en una computadora se escribe con bits, lo leemos como un entero y ya tenemos esa correspondencia:
        - "hola" → 10011101 → 157
        - "casa" → 1100101 → ...
    - Pero esto no se usa. En la práctica, depende de la implementación de cada lenguaje:
        - hash(key) → int
        - hash($x$) = hash($y$) $\iff$$x$ = $y$ (idealmente)

1. Sólo usar elementos representados:

Tenemos un diccionario con una tupla $\langle$ $T, h$$\rangle$,  donde $T$ es un arreglo con $N$ celdas $(N = tam(T))$ y $h: K\rarr\{0, ..., N-1\}$ es la función hash.

- K es el conjunto de claves posibles.
- $\{0, ..., N-1\}$ es el conjunto de las posiciones de la tabla, llamados también pseudoclaves.
- La posición del elemento en el arreglo se calcula mediante $h$.

![Teo%207%20Algo%202%2089f6dce966d14cc7b378c735d4543de8/Untitled%201.png](Teo%207%20Algo%202%2089f6dce966d14cc7b378c735d4543de8/Untitled%201.png)

 

## Hashing perfecto y colisiones

Función hash perfecta:

- $k_1 \neq k_2 \rarr h(k_1) \neq h(k_1)$
- Esto requiere que N ≥ |K|, lo que es raramente razonable en la práctica.

En general, N << |K|, para no usar tanta memoria, lo que trae **colisiones:**

- Es posible que $k_1 \neq k_2$  e igualmente  $h(k_1) =h(k_1)$, esto es una colisión, y son más frecuentes que lo que creemos.

```
**Ejercicio:**
Proponer una función hash perfecta para el caso en que las 
claves sean strings de largo 3 en el alfabeto {a, b, c}.

a = 00
b = 01
c = 10

h(abc) = 000110 = 6
... etc etc ...
```

### Resolución de colisiones

Hay 2 métodos principales, que se diferencian por la forma de ubicar a los elementos que colisionan.

- **Direccionamiento cerrado o concatenación:** a la i-ésima posición de la tabla se le asocia **la lista** de los elementos tales que $h(k)=i$.
- **Direccionamiento abierto:** todos los elementos se guardan en la tabla, no hay listas. Después veremos cómo.

---

**Ejemplo recital - baños**

Hay que decidir cómo distribuir los baños químicos de un recital. 

Lo ideal sería tener un baño por cada asistente, pero sería **muy poco eficiente** (muy caro). 

Alternativa: tener 100 baños y decir algo como "los últimos 2 dígitos de tu DNI son el baño que te corresponde". 

En este caso va a haber **colisiones** en los baños y se van a formar colas. Esto se relaciona con el **direccionamiento cerrado o concatenación.**

Otro caso puede ser que la gente no quiera hacer colas y por ende, si su baño está ocupado, vayan a otro. Esto se corresponde con el **direccionamiento abierto.**

En ambos casos hay que permitir la búsqueda (saber en dónde buscarlo), y en el direccionamiento abierto está un poco más complicado.

---

**Paradoja del cumpleaños**

Si hay 23 personas en una sala, hay un 50.6% de chances de que dos personas compartan cumpleaños.

Igualmente para la función hash, si hay 23 inserciones en una tabla de 365 posiciones, hay más de $\frac{1}{2}$ de probabilidad de que haya colisión. 

---

### Requisitos de una función hash

- Distribución de la probabilidad de las claves:
    
    $P(k)$ = probabilidad de la clave $k$.
    
- Uniformidad simple:
    
    $\forall j \;\; \sum\limits_{k\in K:h(k)=j}^{}P(k) \approx \frac{1}{|N|}$
    

Se quiere que los elementos se distribuyan en el arreglo de manera uniforme. Pero es difícil construir funciones que satisfagan la uniformidad simple porque la probabilidad $P(k)$ generalmente es desconocida.

![Teo%207%20Algo%202%2089f6dce966d14cc7b378c735d4543de8/Untitled%202.png](Teo%207%20Algo%202%2089f6dce966d14cc7b378c735d4543de8/Untitled%202.png)

En este caso, si usamos **concatenación,** en la posición 1 de la segunda tabla hay una lista de claves.

Ya que **no podemos conocer la distribución de los datos**, queremos una función que sea lo más independiente posible de la distribución.

---

## Direccionamiento cerrado - Concatenación

- **insertar(elem, k)**: insertamos al principio de la lista asociada a la posición $h(k)$, costo $O(1)$.
- **buscar(k):** hay que acceder a esa posición y buscar en la lista. 
Costo: $O(longitud(lista\_k)).$
- **borrar(k)**: búsqueda en al lista asociada a esa posición y luego borrado. Mismo costo que la búsqueda.

Pero **¿cuánto van a medir las listas?** Depende de la distribución de los datos y de la función hash.

En el **peor caso**, los $n$ elementos del diccionario van a parar a la misma dirección. Sería $O(n)$.

**Caso promedio**, en el que tenemos una buena distribución en la tabla:

- n = #elementos en la tabla
- T = tamaño de la tabla
- $\alpha$ = $\frac{n}{|T|}$: factor de carga

Bajo la hipótesis de simplicidad uniforme de la función, si las colisiones se resuelven por concatenación, en promedio una búsqueda fallida requiere $\Theta(1+\alpha)$ y una búsqueda exitosa requiere  $\Theta(1+\frac{\alpha}{2})$. Porque $\alpha$ al parecer es la longitud de la lista.

**Si n ~ |T|, entonces el costo es de $O(1)$.  Por eso es muy importante dimensionar bien T.**

---

## Direccionamiento abierto

- Almacemanos **todos** los elementos en la tabla.
- Las colisiones se resuelven del siguiente modo:
    1. Si la posición está ocupada, buscamos una libre.
- Los distintos métodos de direccionamiento abierto se diferencian en **cómo se hace la búsqueda de la nueva posición.**
- La función hash va a depender del número de intentos realizados. Dirección = $h(k, i)$ para el i-ésimo intento.
- $h(k, i)$ debe generar todas las posiciones de T.

### Inserción

```
**insertar(elem, k, T):**
	
	i <- 0;
	**while** (T[h(k, i)] está ocupada && i<|T|) **do**
		incrementar i;
	**endwhile**

	**if** (i < |T|) **do**
		T[h(k, i)] <- (elem, k)
	**else**
		**return** overflow
	**endif**
```

Puede haber overflow. O sea, que intentemos insertar más elementos de los que caben en la tabla.

![Teo%207%20Algo%202%2089f6dce966d14cc7b378c735d4543de8/Untitled%203.png](Teo%207%20Algo%202%2089f6dce966d14cc7b378c735d4543de8/Untitled%203.png)

### Búsqueda

```
**buscar(k, T):**
	
	i <- 0
	
	**while** ((k != T[h(k, i)].clave) && T[h(k, i)] != null 
	&& i < |T| **do**
		incrementar i;
	**endwhile**
	
	**if** (i < |T|) && T[h(k, i)] != null **do**
		**return** T[h(k, i)]
	**else** 
		**return** noEstá
	**endif**
	 

```

![Teo%207%20Algo%202%2089f6dce966d14cc7b378c735d4543de8/Untitled%204.png](Teo%207%20Algo%202%2089f6dce966d14cc7b378c735d4543de8/Untitled%204.png)

Cuando buscamos 30, como nos topamos con un NULL, devolvimos que no está. Pero...

¿Qué pasa si borramos el 2 y volvemos a buscar el 12? Nos va a tirar que no está. Una posibilidad es, en lugar de poner NULL, marcar al elemento como "borrado". 

### Barrido

Sabemos que la función $h(k, i)$ debe recorrer todas las posiciones de la tabla. Hay varias formas típicas de hacerlo:

1. Barrido lineal
2. Barrido cuadrático 
3. Hashing doble

Se diferencian por su comportamiento  y complejidad de cálculo

1. Barrido Lineal
    
    Es el que ya vimos, el más intuitivo:
    
     $h(k, i) = h'(k) + i  \; mod\; |T|$.
    
    Puede haber **aglomeración primaria:** dos secuencias de barrido siguen colisionando. O sea, los elementos que nos van llegando, medio que no van a ir nunca a su lugar y queda todo corrido.
    
2. Barrido Cuadrático
    
    $h(k,i) = (h'(k) + c_1i+c_2i^2) \;\; mod \; \; |T|$,  con $c_1$ y $c_2$ que cumplan que la función mapee, en algún momento, todas las posiciones de la tabla.
    
    Posibilidad de **algomeración secundaria:** si hay colisión en el primer intento, sigue habiendo colisiones.
    $h'(k_1) = h'(k_2) \rarr h(k_1, i) = h(k_2, i)$
    
3. Hashing Doble
    
    Idea: el barrido también depende de la clave.
    
    $h(k, i) = (h_1(k) + ih_2(k))\;\;mod\;\;|T|$ , donde $h_1$ y $h_2$ son dos funciones de hash.
    
    Reduce los fenómenos de aglomeración secundara y no tiene aglomeración primaria.
    

---

## Construcción de funciones de Hash

- Recordemos que las claves no son únicamente números naturales. Podrían ser strings, por ejemplo.
- Solución: asociar a cada clave un entero. ¿Cómo? Depende.

1er posible método:

 asociar a cada caracter su código ASCII y al string le asociamos el número entero obtenido en una determinada base.

Ejemplo en base 2: 

![Teo%207%20Algo%202%2089f6dce966d14cc7b378c735d4543de8/Untitled%205.png](Teo%207%20Algo%202%2089f6dce966d14cc7b378c735d4543de8/Untitled%205.png)

Otros métodos:

- División
- Partición
- Mid-Square
- Extracción
- ... etc etc ...

El objetivo principal es tener una distribución lo más uniforme posible, y los métodos le diferencian en su complejidad y en el tratamiento de los fenómenos de aglomeración.

### División

- $h(k) = k \;\;mod\;\;|T|$

Baja complejidad. Pero hay que seleccionar bien el tamaño de la tabla.

Aglomeraciones:

- No potencias de 2: si |T| = $2^p$, todas las claves con los $p$ bits menos significativos iguales, colisionan.
- No potencias de 10 si trabajamos con decimales.
- La función debería depender de **todas las cifras** de las claves.
- Una buena elección: un número primo lejos de una potencia de 2 
($h(k) = k\;\;mod\;\;701$ para $|K|$=$2048$ valores posibles). Un número primo porque no queremos que tenga factores comunes.

### Partición

Particionar la clave $k$ en $k_1, k_2, ..., k_n$.

- $h(k)= f(k_1, k_2,...,k_n)$

Por ejemplo, si la clave es una tarjeta de crédito, la función podría ser:

$f(n\_tarjeta) = f(t_0t_1t_2, \;t_3t_4t_5)$ $= (t_0t_1t_2 + t_3t_4t_5)\;\;mod\;\;701$  

### Extracción

Se usa solamente una parte de la clave para calcular la dirección.

Ejemplo: las 6 cifras centrales de una tarjeta de crédito, o los últimos 2 números de un DNI.

El número extraído/obtenido puede ser modificado después.