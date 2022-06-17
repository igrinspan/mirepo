---
layout: page
title: Teórica 10
permalink: /materias/algo2/teo10/
---
# Divide & Conquer

El D&C es una técnica algorítmica para encarar problemas.

La metodología consiste en:

- Dividir un problema en problemas similares pero más chicos.
- Resolver esos problemas menores.
- Combinar las soluciones para obtener la solución del problema original.

Tiene sentido siempre que el *divide* y el *merge* no sean caros.

Ya vimos que es usado por el **Merge Sort** y **Quick Sort.**

```cpp
//Idea general del algoritmo

AlgoritmoD&C(X){
	if (X es suficientemente chico){
		adhoc(X) //hacemos lo que tengamos que hacer
	}
	else{
		Dividir X en X1, X2, ..., Xk
		for (i=0 to k){
			res_i = AlgoritmoD&C(Xi)
		}
		Combinar soluciones res_i para construir RES
	}
	return RES
}
```

---

### Ejemplo: Algoritmo de Karatsuba

Queremos multiplicar $2$ enteros de $n$ cifras en base $b$. ($X$ e $Y$).

Con el algoritmo general, se tarda $O(n^2)$. 

Pero aplicando D&C, la complejidad es de $O(n^{1.59})$, que es considerablemente mejor.

Hay una variable del algoritmo, llamado **Algoritmo de Strassen**, que logró para el producto de matrices, pasar de $O(n^3)$ a $O(n^{2.8})$. Consiste en dividir cada matriz en 4.

---

## ¿Cómo se calcula la complejidad de un D&C?

Ecuaciones de recurrencia. 

Dividimos en $a$ subproblemas de tamaño máximo  $\frac{n}{c}$. El costo de determinar subproblemas (*divide*) y unirlos (*conquer*) es de $b*n^d$. (Suponemos que el caso base cuesta $b$).

No necesariamente $a$ coincide con $c$, hay algoritmos que pueden llegar a dividir en $4$ problemas de tamaño $\frac{n}{2}$, por ejemplo.

### Costo: recurrencia típica

La recurrencia va hasta que n sea 1, dividiendo en cada paso por c.

Vamos a ir dividiendo por $c$ y multiplicando por $a$.

$T(n)=aT(\frac{n}{c}) + bn^d = aT(c^{k-1}) + bn^d$

$= ... = a^2(aT(c^{k-3}) + b(c^{k-2})^d) + abc^{d(k-1)} + bc^{kd}$

$= ... =$

$= ... = a^jT(c^{k-j}) + \sum\limits_{i=0}^{j-1}a^ibc^{d(k-i)}$

$=...= bn^d\sum\limits_{i=0}^{\log_cn}(\frac{a}{c^d})^i$

Un quilombo.

 

## Teorema Maestro