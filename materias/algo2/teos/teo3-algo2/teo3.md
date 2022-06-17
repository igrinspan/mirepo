---
layout: page
title: Teórica 3
permalink: /materias/algo2/teo3/
---
# Análisis de complejidad

Para resolver problemas, queremos que nuestros algoritmos:

- **Funcionen correctamente**
    - modelen bien el problema
    - los algoritmos terminen
    - devuelvan el resultado correcto
- **Sean eficientes** en términos del consumo de recursos.

**¿Cuáles son los recursos que se consumen?**

- Tiempo de ejecución
- Espacio en memoria
- Cantidad de procesadores
- Utilización de la red de comunicaciones

Otros criterios (que no vamos a tener en cuenta):

- Claridad de la solución
- Facilidad de codificación

En general no se pueden optimizar todos al mismo tiempo, hay que **balancear** y darle más importancia a algunos recursos que a otros.

Nosotros vamos a priorizar **el tiempo y el espacio**.  Principalmente el tiempo. ¿Por qué? Porque el tiempo no se recupera.

**¿Cómo se puede medir la comlpejidad de los algoritmos?**

- Forma empírica/experimental (medir segundos, ticks de clock, etc.)
- Forma teórica

Como la forma empírica puede cambiar mucho según el contexto en el que ejecutemos, la capacidad de la computadora, etc., **vamos a usar la teórica** porque permite tener resultados mucho más generales.

### Vemos el ejemplo de búsqueda secuencial en un array.

¿Depende de cuál buscamos? ¿Depende de la cantidad de elementos del array? 

### Análisis Teórico

Ventajas del enfoque teórico:

- El análisis se puede a ser **a priori**, antes de codear.
- Vale para **todas las instancias** del problema.
- Es **independiente del lenguaje de programación.**
- Es **independiente de la máquina**.
- Es **independiente de la pericia del programador**.

1. Se basa en un **modelo de máquina** consensuado.
2. En función del **tamaño del input.**
3. Para distintos **tipos de input.**
4. **Análisis asintótico.**

---

### **Modelo de cómputo**

- Queremos una medida universal válida para distintas implementaciones del algoritmo.
- La idea es tener un banco de pruebas teórico/virtual.
- Definimos una máquina ideal
- Medida del tiempo: número de pasos u operaciones que se ejecutan en esa máquia para determinado input.
- Medida del espacio: número de posiciones de memoria que se utilizan en esa máquina para determinado input.

### **Operaciones Elementales.**

- t(l) va a ser la función que mida el número de operaciones elementales para la instancia l.
- Las OE serán aquellas que el procesador realiza en tiempo acotado por una constante (no depende del tamaño de entrada).
- OEs: operaciones aritméticas básicas, comparaciones lógicas, transferencias de control, asignaciones a variables de tipos básicos, etc.

**Cálculo de OE**

- Vamos a considerar que el tiempo de una OE es 1.
- El tiempo de ejecución de una secuencia consecutiva de instrucciones se calcula sumando los tiempos de ejecución de cada instrucción.

ejemplo:

```c
buscar: int x, array A -> int 
l <- 1                                                   1
encontré <- falso                                        1
mientras ¬encontré:                                      2*4  
	si A[l] = x entonces encontré <- verdadero             2*3 + 3   
	l <- l+1                                               3*4                              
print(l-1)                                               1       

Buscar(5, [2, 6, 3, 5, 8])
aprox 30 Operaciones Elementales.

Buscar(6, [2, 6, 3, 5, 8]) un poco menos
Buscar(8, [2, 6, 3, 5, 8]) un poco más

¿Y si el arreglo en vez de tener 5 elementos tiene 10? 
→ Va a tener más operaciones elementales.
```

**Vamos a empezar a pensar la complejidad en función del tamaño del input.**

Reglas generales del cálculo de Operaciones Elementales:

- Si tenemos un `if C then S1 else S2` va a tardar T = T(comparación) + máx{T(S1), T(S2)}
- Si tenemos un `while C do S` va a ser T = T(C) + nro_iteraciones*(T(S)+T(C)).

<aside>
⚠️ T(S) y T(C) pueden ir variando en cada iteración, y si queremos analizar **bien** la complejidad va a haber que tenerlo en cuenta.

</aside>

- Llamada a una función `F(p1, p2, .., pn)`, vamos a medir 1 por la llamada, luego por la evaluación de los parámetros y luego por el T de la función. → T= 1 + T(p1) + T(p2) + ... + T(pn) + T(F).

---

### Tamaño del input

¿Por qué se mide la complejidad en función del tamaño de la entrada?

- Queremos complejidad **relativa, no absoluta.**
- Es una medida general. Queremos predecir, no medir para una instancia particular.
- Más abstracto que pensarlo para cada input. Podría haber infinitos inputs, imposible de calcular eso.

**T(n): complejidad temporal para una entrada de tamaño n.**

**S(n): complejidad espacial para una entrada de tamaño n.**

Como puede haber 2 inputs del mismo tamaño que sean distintos, vamos a tener que ser un poco más específicos. Por ejemplo: no es lo mismo trabajar el arreglo [1, 1, 2, 1, 0] que el [99999, 151895, 29115025, 942015, 1029401]. Es más probable que si tenemos que operar con los elementos de el segundo arreglo tardemos más.

Entonces, se suelen estudiar **tres casos** para un mismo algoritmo:

1. Análisis del **peor caso**
2. Análisis del **caso medio**
3. Análisis del **mejor caso**

<aside>
🚨 El que más vamos a usar va a ser el **análisis de caso peor,** porque nos da **garantías** sobre nuestro algoritmo. O sea podemos asegurar que va a tener esa complejidad o una menor.

</aside>

1. **Análisis del peor caso**
    
    $T_{peor}{(n)}$  es el tiempo de ejecución del algoritmo sobre la instancia de tamaño n que implica mayor tiempo de ejecución.
    
    Por ejemplo, para búsqueda lineal en un arreglo, el peor caso es n, que es cuando el elemento está al final del arreglo.
    
2. **Análisis del mejor caso**
    
    Indica el tiempo de ejecución sobre la instancia más rápida entre los inputs de tamaño n. No es muy utilizada.
    
3. **Análisis del caso medio/promedio**
    
    Corresponde al tiempo "**promedio".** O sea, al tiempo "**esperado"** sobre instancias "**típicas".**
    
    Se define como la esperanza matemática de la variable aleatoria definida por todas las posibles ejecuciones para un input del tamaño dado, con las probabilidades de que estas ocurran para esa entrada.
    
    $T_{prom}{(n)} = \sum_{instancias\;l\;tq\;|l|=n}{P(l)*t(l)}$. 
    
    Es una medida de gran utilidad, **pero...** debemos conocer la distribución estadística del input, para poder hacer hipótesis realistas sobre lo que va a ingresar.
    
    Además, si tenemos un algoritmo que tarda 1 o 100 pasos no sirve dar el tiempo promedio (50) porque claramente nunca va a tardar eso.
    

Ejemplo búsqueda secuencial:

- $T_{peor}{(n)\;\; = \;8 + 5(n-1)}.$   Cuando el elemento está al final del arreglo.
- $T_{mejor}{(n) = 9}$.                          Cuando el elemento está al final del arreglo.
- $T_{prom}{(n) \;= 8 + 5(\frac{n}{2})}$.            Cuando está en la mitad.

Para ver la complejidad espacial, tenemos en cuenta el tamaño del arreglo y el uso de las variables i, x y encontré:   $S(n) = n + 3$.

Si el arreglo estuviera ordenado ¿cambiarían los tiempos?. NO.

Pero... podemos cambiar el algoritmo por el de búsqueda binaria, tarda menos.

---

### Principio de invarianza

Dado un algoritmo y dos máquinas que tardan $T_{1}(n)$ y $T_{2}(n)$, existe una constante real $c > 0$ y un $n_{0} \in \N$  tales que $\forall \; n ≥ n_{0}$ se verifica que: 

$$
T_{1}(n) ≤ c*T_{2}(n)
$$

O sea, dos ejecuciones distintas del mismo algoritmo sólo difieren en un factor constante para entradas suficientemente grandes.

### Análisis asintótico

Cuando el tamaño de los datos es pequeño no habrá diferencias significativas en el uso de distintos algoritmos, pero para entradas más grandes los costos pueden **variar de manera significativa.**

- **El orden** (logarítmico, lineal, cuadrático, exponencial) de la función $T(n)$ que mide la complejidad de un algoritmo, es el que **expresa el comportamiento dominante** cuando el tamaño de entrada es grande.
- **El comportamiento asintótico** es el comportamiento para valores de entrada suficientemente grandes.

El objetivo del estudio de la complejidad es determinar el comportamiento asintótico del algoritmo.

Medidas del comportamiento asintótico:

- $O$ (O grande) cota superior.
- $\Omega$  (Omega) cota inferior.
- $\Theta$ (theta) orden exacto de la función.

**Cota superior - Notación $O$**

- Esta notación sirve para representar la cota superior del tiempo de ejecución de un algoritmo.
- La notación $f \in O(g)$ significa que $f$ **no crece más rápido que alguna función proporcional a $g$.**
- Si para un algoritmo sabemos que $T_{peor} \in O(g)$, entonces se puede asegurar que **en todos los casos** el tiempo será como mucho proporcional a la cota.
    
    ![Teo%203%20Algo%202%20d2eccf49e81d4599ad0ae45384cc1335/Untitled.png](Teo%203%20Algo%202%20d2eccf49e81d4599ad0ae45384cc1335/Untitled.png)
    
    ¿Cómo encontramos esos valores de $n_0$ y $k$?
    
    $$
    100n^2 + 300n ≤ kn^2 \\
    
    $$
    
    $$
    \frac{100n^2}{n^2} + \frac{300n}{n^2} ≤ \frac{kn^2}{n^2} \\ 
    
    $$
    
    $$
    100 + \frac{300}{n} ≤ k
    $$
    
    Entonces podemos elegir, por ejemplo, $k = 400$ y $n_0 = 1$ y se cumple
    

Funciones de complejidad temporal:

- Complejidad constante $O(1)$
- Complejidad logarítmica $O(log(n))$
- Complejidad lineal $O(n)$
- Complejidad nlogn $O(nlog(n))$
- Complejidad cuadrática $O(n^2)$
- Complejidad cúbica $O(n^3)$
- Complejidad polinómica $O(n^k)$
- Complejidad exponencial $O(2^n)$

**Cota Inferior - Notación $\Omega$**

- Sirve para representar la cota inferior del tiempo de ejecución de un algoritmo.

<aside>
🚨 No confundir con el análisis del mejor caso.

</aside>

![Teo%203%20Algo%202%20d2eccf49e81d4599ad0ae45384cc1335/Untitled%201.png](Teo%203%20Algo%202%20d2eccf49e81d4599ad0ae45384cc1335/Untitled%201.png)

Por ejemplo:  $100n^2 + 200n + 40 \in \Omega(n^2)$ y  $100n^2 + 200n + 40 \in \Omega(n)$, porque esa función esta acotada **inferiormente** por $n$ y $n^2$.

**Orden exacto - Notación $\Theta$**

- Cota **por arriba** y **por abajo**

![Teo%203%20Algo%202%20d2eccf49e81d4599ad0ae45384cc1335/Untitled%202.png](Teo%203%20Algo%202%20d2eccf49e81d4599ad0ae45384cc1335/Untitled%202.png)

Ejemplo: $100n^2 + 300n + 1000 \in \Theta(n^2)$. Podemos probarlo:

Las cotas asintóticas son suficientes para decidir el mejor algoritmo, no hay que preocuparnos tanto por las constantes.