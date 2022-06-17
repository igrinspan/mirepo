---
layout: page
title: Teórica 9
permalink: /materias/algo2/teo9/
---

# Sorting - Ordenamiento

El ordenamiento consiste en, dado un arreglo de tipo $\alpha$, para el que está definida la relación $<_\alpha$, queremos ordenarlo de menor a mayor o mayor a menor.

Es uno de los problemas más clásicos, útiles y estudiados.

## 1. Selection Sort

Es el que vamos buscando el mínimo y lo ponemos al principio. Luego buscamos el mínimo para el resto del arreglo y lo ubicamos en la segunda posición. Y así sucesivamente hasta el último elemento.

**Algoritmo**

Para todo i desde 0 a n

seleccionar el mínimo entre la posición actual (i) y el final

ubicarlo en la posición i

El algoritmo termina y lo hace devolviendo un arreglo ordenado.

### Invariante del algoritmo

En cualquier paso:

- Desde 0 hasta i ya está ordenado el arreglo.
- Todas las posiciones que quedan después de i tienen que ser mayores a las que quedan antes. Y viceversa.
- El arreglo es una permutación del arreglo original.

Tiene complejidad $O(n^2)\;$$=\frac{n(n-1)}{2}$.

El costo no depende de nada. Siempre es igual. O sea, no importa si el algoritmo ya está ordenado o no. Cuesta lo mismo en el peor y en el mejor caso.

## 2. Insertion Sort

Supongamos que queremos ordenar de menor a mayor. 

Agarramos el segundo elemento y lo movemos hacia atrás hasta que el que esté a derecha sea mayor. Luego hacemos lo mismo para el elemento 3 y así sucesivamente hasta n.

### Invariante

- Los elementos de 0 a i son los elementos de 0 a i originales del arreglo.
- Los elementos de 0 a i están ordenados.
- El arreglo es una permutación del arreglo original

```cpp
**para** i desde 1 hasta n-1 **hacer**
	instertar(i)

insertar(i):
**para** i desde 1 hasta n-1 **hacer**
j = i-1
elem = a[i]

**mientras** j≥0 and a[j]>elem **hacer**
	a[j+1] = a[j]
	j = j-1

a[j+1] = elem
```

Costo $\frac{(n-1)(n-2)}{2}$ $= O(n^2)$.

Si el arreglo está parcialmente ordenado, el costo mejora. Y si está ordenado al revés es el peor caso.

Se dice que un algoritmo es **estable** si mantiene el orden anterior de elementos con igual clave. 

En un algoritmo **estable,** dos claves iguales tienen que mantener el orden que tenían al principio

![Teo%209%20Algo%202%20844575a5671b427db8b7d07ae97e8502/Untitled.png](Teo%209%20Algo%202%20844575a5671b427db8b7d07ae97e8502/Untitled.png)

Si un algoritmo es inestable, puede mantener el orden pero también puede no hacerlo. 

¿Para qué sirve? Cuando trabajamos con claves con múltiples valores, puede ser que en algún caso queramos que las segundas claves de los elementos que coinciden preserven el orden al ordenar los elementos según la primera clave.

---

**Ejemplo:**

Nos dan este arreglo ordenado alfabéticamente y queremos ordenarlo según el turno. Pero queremos un algoritmo estable así no cambiamos el orden alfabético inicial.

![Teo%209%20Algo%202%20844575a5671b427db8b7d07ae97e8502/Untitled%201.png](Teo%209%20Algo%202%20844575a5671b427db8b7d07ae97e8502/Untitled%201.png)

---

¿Selection e Insertion Sort son estables?

Selection Sort → **No**, porque el mínimo lo swapeamos con el inicial así nomás.

Insertion Sort → **Sí**, depende de cuando frenemos en cada paso pero sí.

---

## 3. HeapSort

Hasta ahora vimos el Selection y el Insertion. Ambos no muy eficientes.

Siguiendo la línea de Selection Sort, queremos una estructura que nos ayude a encontrar el mínimo más rápidamente (en vez de recorrer toda la lista).

Ahí aparece el Heap. Podemos ir metiendo los elementos en un heap y luego ir sacándolos.

Pero el **HeapSort** es todavía mejor. Usa la operación Array2Heap, que tiene complejidad $O(n^2)$.

Pasamos de $O(n^2)$ a $O(n.log(n))$. 

```cpp
heapSort(A):
	heap = Array2Heao(A)
	for i desde n-1 hasta 0 hacer
		max = proximo(heap)
		desencolar(heap)
		A[i] = max
	return A

Costo: O(n) + O(n*log(n))
```

En este caso usamos un máx-Heap para ordenar un arreglo de menor a mayor.

¿Es estable? ¿Se puede hacer estable?

---

## 4. Merge Sort

El más "elegante" 👀 👀.

Usa la metodología $Divide\;\&\;Conquer.$ 

Consiste en dividir un problema en problemas similares pero más chicos, leugo resolver esos problemas menores y combinar las soluciones de los problemas menores para obtener la solución del problema original.

Tiene sentido siempre y cuando la división y la combinación no sean caras (y el problema se reduzca en cada paso).

```cpp
mergeSort(A){
	if (n<2){
		el arreglo ya está ordenado
	}

	else{
		dividir el arreglo en 2 partes iguales
		ordenar_recursivamente ambas mitades //mergeSort(A1) y A2.
		combinar ambas mitades en un único arreglo //combinar(A1, A2).
	}

}
```

El costo del algoritmo es de $O(n.log(n))$.

---

## 5. Quick Sort

Idea más o menos parecida al MergeSort (con D&Q).

En mergeSort la división se hacía en la mitad del arreglo y se ordenaban ambas mitades recursivamente.

En vez de cortar el elemento por la mitad, lo cortamos por el elemento mediano. (O sea el que tiene el valor mediano).

```cpp
quickSort(A){
	separamos el arreglo en 2 partes. Una mitar con los
	los elementos menores al mediano y la otra con los mayores

	ordenamos las 2 partes

}
```

¿Qué pasa si no conocemos el elemento mediano? Mmm.

Una opción es usar un algoritmo para encontrar el mediano, pero le va a sumar mucho costo a nuestro algoritmo (aunque sea de costo lineal).

Lamentablemente, no podemos usar como cota superior del peor caso
 $O(n*log(n))$, porque podemos elegir mal el pivote y que quede $O(n^2)$.

Peor caso: $O(n^2)$.

Caso promedio: $O(n*log(n))$.

Una forma de zafar de los casos peores es elegir por ejemplo 5 elementos y elegir el mediano de esos como pivote. Por lo menos no vamos a elegir ninguno de los extremos. Funciona bien en la práctica.

---

# Complejidades

Merge Sort y Heap Sort: $O(n*log(n))$.

Quick Sort, Selection Sort, Insertion Sort: $O(n^2)$.

<aside>
⚠️ Quick Sort en mejor caso: $O(n*log(n))$.
Insertion Sort en mejor caso: $O(n).$

</aside>

**¿Cuál es la eficiencia máxima (complejidad mínima) obtenible en el caso peor?**

$\Omega(n*log(n))$ es lo mínimo que vamos a tardar en el caso peor. No van a existir otros algoritmos que sean más rápidos. Los algoritmos con esa complejidad van a ser óptimos.

Las cotas inferiores son más difíciles de poner que las superiores.

Justificaciones teóricas usando árboles de decisión.

![Teo%209%20Algo%202%20844575a5671b427db8b7d07ae97e8502/Untitled%202.png](Teo%209%20Algo%202%20844575a5671b427db8b7d07ae97e8502/Untitled%202.png)