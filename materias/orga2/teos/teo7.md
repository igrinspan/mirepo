---
layout: page
title: Teórica 7
permalink: /materias/orga2/teo7/
---
# Teo 7 - Orga 2

# Interrupciones

### Fuentes de Interrupción

- ***Hardware***
    - Señales eléctricas enviadas desde los dispositivos de *hardware*, concentradas en un controlador externo que las prioriza y envía a la CPU secuencialmente.
    - Son **asincrónicas** al *clock* del procesador. Y nadie puede saber cuándo va a llegar una interrupción, son **no determinísticas.**

- ***Software***
    - Se producen cuando el *software* ejecuta INT n.
    - Son **determinísticas**, porque si ponés INT en el código, sabés que ahí se va a producir una interrucpión.
    
- **Internas (excepciones)**
    - Las genera la propia CPU cuando hay algo que le impide completar la ejecución de la instrucción en curso.
    - Ejemplos: división por cero, *page fault*, violación de protección.

**¿Cómo se identifican las fuentes de interrupción?**

Una interrupción se identifica unívocamente mediante un valor de 8 bits llamado **tipo.** Intel maneja **256 tipos**, todas las combinaciones de 8 bits.

1. ¿Cómo se obtiene el tipo?
    
    Es provisto por el HW en interrupciones que ingresan por el pin **INTR** del procesador.
    
    Acompaña al código de operación en la instrucción **INT type,** para el caso de es SW. Por ejemplo **INT 80h**.
    
    En algunos casos está asociado a una instrucción. Por ejemplo: **INTO** es una instrucción que genera una interrupción tipo 4.
    
    Cada excepción tienen asignado un tipo específico.
    

Una vez que identificamos la fuente de interrupción, hay que vectorizarla. Esto es suspender lo que estamos ejecutando, guardar el estado del procesador y que se pueda atender la interrupción, "saltando" hacia el código de su rutina de interrupción.

### Vectorización de las interrupciones

### Interrupt Descriptor Table (IDT)

Cuando se trabaja en Modo Protefido se define la tabla IDT que almacena descriptores de interrupción. La tabla tiene 256 entradas, una por cada tipo. Los descriptores miden 64 bits → la IDT mide 2k.

La **IDT** puede contener 3 tipos de descriptores:

1. **Interrupt Gate** (de 16, 32 o 64 bits)
2. **Trap Gate** (de 16, 32 o 64 bits)
3. **Task Gate** (sólo en 32 bits)

En los 3 casos se trata de **descriptores de sistema (S=0).** 

<aside>
⚠️ Si metemos descriptores de segmentos de datos o de código u otro de sistema diferente, el procesador generará una **excepción de protección general** al intentar vectorizar la interrupción.

</aside>

**Formato de los descriptores en arquitectura 32**

![teo7-1][1]

El Task Gate lo vemos después.

Abajo están el Interrupt Gate y el Trap Gate. La única diferencia son los 3 bits (8, 9 y 10). 

**¿Cómo se llega a la IDT?**

Tenemos el IDTR, igual al GDTR.

![teo7-2][2]

**¿Y cómo vectorizamos la interrupción?**

1. Se genera una INT n.
2. Vamos a la IDT a la puerta para INTs n.
3. Ahora tenemos el descriptor.
4. En el descriptor leemos el selector de segmento al que tenemos que ir. Puede estar en la GDT o LDT. 
5. Ahora lees el nuevo descriptor de segmento. Hay que verificar que este descriptor sea de código, porque debe ser una rutina de atención de interrupciones. Sino, general protection fault.
6. De este descriptor sacamos la dirección base del segmento que contiene el código de la rutina. Y del descriptor de la IDT sacamos el offset a la primera instrucción.

**Manejo de pila**

Nuestro ESP apunta a un lugar determinado. Cuando nos llega la interrupción: guardamos los flags, el CS y el EIP.

 

### Excepciones

Las excepciones se dividen en 3 tipos:

- **Fault:** es una excepción que tiene solución. Puede corregirse y retomar la ejecución de esa instrucción sin perder continuidad. El procesador guarda en la pila la dirección de la instrucción que produjo la falla.
- **Trap:** excepción producida a continuación de una instrucción de trap. Algunas permiten seguir con la ejecución, otras no. El procesador guarda en la pila la dirección de la instrucción siguiente a la instrucción *trapeada.*
- **Abort:** excepción que no permite recuperar la ejecución ni la tarea que la causó. Reporta errores de hardware o inconsistencias en tablas del sistema

Cuando la interrupción es una excepción, el procesador guarda en la pila, además de los flags y eso, el **error code**. Algunas excepciones tienen y otras no.

¿Para qué sirve el **error code**?

Da pistas acerca del origen de la excepción. Tiene 32 bits, pero sólo se leen los 16 menos significativos.

![teo7-3][3]

1. **1**  *EXT:* External Event (bit 0): 
Se setea para indicar que la excepción ha sido causada por un evento externo al procesador.

2. **2**  *IDT:* Descriptor Location (bit 1): 
Cuando está seteado indica que el campo Segment Selector Index se refiere a un descriptor de puerta en la IDT: Cuando está en cero indica que dicho campo se refiere a un descriptor en la GDT o en la LDT de la tarea actual.

3. **3**  *TI:* GDT-LDT (bit 2): 
Tiene significado cuando el bit anterior está en cero. Indica a que tabla de descriptores corresponde el selector del campo Indice: 0 GDT, 1 LDT.

---

Las Interrupciones y Excepciones, en un sistema multitarea, las maneja el **kernel.**

### Tipos de Interrupciones

Más importantes:

- Tipo 6: **invalid opcode exception #UD**.
Esta excepción deja al CS:EIP apuntando a la instrucción que generó la excepción.
- Tipo 7: **device not available exception #NM**. Responde a 2 situaciones: Ejecutar SIMD o FPU con el flag CR0.TS = 1. Ejecutar SIMD con CR0.EM = 1. Deja al CS:EIP apuntando a la instrucción que generó la excepción.
- Tipo 8: **double fault exception #DF.**
Ocurre cuando ocurre una excepción mientras se está vectorizando otra excepción. En general, el procesador intenta manejar las excepciones en serie. Pero en ocasiones no se pueden procesar en serie, lo que produce una doble falta.
Podemos saber si se pueden o no procesar serialmente, porque estas excepciones se dividen en 3 categorías: Benign, Contributory y Page Fault.
    
    Si la primera expeción es Benigna, la segunda se va a poder procesar en serie. 
    Si la primera es Contributiva, la segunda generará #DF sólo si se trata de una Contributiva. 
    Si la primera es un Page Fault, sólo se tratará en serie si la segunda es Benigna. 
    
- Tipo 11: **segmento no presente #NP.** 
Se produce cuando se intenta cargar el descriptor seleccionado por CS, DS, ES, FS o GS y su atributo P es 0. También se genera si LLDT catga un selector de LDT con P = 0.
- Tipo 12: **stack fault #SS.** 
Se genera cada vez que una instrucción que utiliza SS para direccionar memoria viola el límite efectivo de la pila (del segmento pila). También se genera si se escribe en SS un selector cuyo P = 0. Pone código de error en la pila.
- Tipo 13: **general protection fault #GP.** 
Engloba todas las violaciones generales al sistema de protección que no estén comprendidas en Stack Fault, TSS inválida o page fault. Tiene código de error.
- Tipo 14: **page fault #PF**
Engloba todas las violaciones de páginas (violaciones al sistema de protección relacionadas con el Sistema de Paginación).
Tiene 5 causas generales.

## Hardware para las interrupciones

Intel usa el PIC (Programable Interrupt Controller).

![teo7-4][4]

Puede controlar 8 dispositivos de hardware. Las interrupciones le entran al Interrupt Request Register. Este, junto con el Priority Resolver y el In-Service Register, se comunica con el Control Logic. El Control Logic le envía la señal INT al procesador, para avisarle que tiene una interrupción. El procesador le responde con la señal INTA cuando está listo para atenderla. 

Hay un buffer que nos conecta al bus de datos. Y un buffer y comparador de cascada. Este último es porque el PIC está pensado para conectarse con otros PICs, y dice si este PIC es *máster* o *slave* según el caso.

Actualmente se usa el APIC, que es una versión avanzada del PIC.

### **¿Cómo trabaja el PIC?**

El IRR es un registro que simplemente guarda el estado de esas líneas, y avisa al Priority Resolver cuando alguna de estas líneas se activa. El Priority Resolver resuelve, cuando hay más de una línea prendida, cuál priorizar. ¿Cómo prioriza? En general IR0>IR1>..>IR7. También puede pasar que haya una interrupción en curso y se produzca otra interrupción (**añidamiento**). Eso también lo resuelve.

El priority resolver se fija en el Interrupt Mask Register (IMR). Este registro lo maneja el procesador, y dice qué Interrupción se debe tener en cuenta. (Pueden estar habilitadas IR4 e IR5 y el resto no, por ejemplo). Además de eso están los flags **dentro del procesador**, que si están en 0, el PIC puede hacer toda la bola pero el procesador ni atiende las interrupciones. 

El In-Service Register (ISR) pone en 1 el bit de la interrupción que está en servicio. O sea, si el procesador está atendiendo una IR3, el bit 3 del ISR debería estar seteado.

Una vez que la Control Logic mandó la interrupción, espera pulsos por la entrada INTA. (En particular 2 pulsos).

El procesador está en la suya, con el flag de interrupciones en 1 y recibe una interrupción IR1. En ese caso termina la instrucción en curso (y si es una instrucción atómica, también la siguiente), guarda su estado y envía un pulso por el INTA. 

Hay dos modos de trabajo: **fin de interrupción automático o no.**
En el primero hay un solo pulso de INTA y usa añidamiento full o algo así. 

**Diagrama Cascada**

![teo7-5][5]

En cascada, el PIC máster es el único que envía interrupciones al procesador. Se conecta los PICs slave en las líneas más altas (IR7) del PIC máster.

### ¿Cómo se inicializa el PIC?

Vamos a tener que inicializarlo. La PC está programada con **fin de interrupción NO automático.** (Es una de las cosas que vamos a tener en cuenta). **Se activa por flanco, no por nivel.** (otra cosa).

El dispositivo se inicializa mediante una secuencia continua de entre 2 y 4 palabras de inicialización. La secuencia se compone de ICWs (Initialization Control Words). Cada ICW es un byte que setea determinados bits relacionados con el funcionamiento del dispositivo.

- Dependiendo de cómo lo programemos, vamos a tener 2, 3 o 4 ICWs.
- La ICW1 y la ICW2 se envían siempre.
- La ICW3 se envía si tenemos más de un PIC en cascada. En nuestro caso tenemos 2 PICs, entonces hay que usarlo.
- También vamos a usar la ICW4. Esto se indica en la ICW1.

![teo7-6][6]

---

Diagramas de las ICWs. (**pag 102 pdf, diapo 84**).

**ICW1:** 

- A0 dice en qué dirección de entrada/salida del PIC se escribe este ICW.
- Bit IC4: indica si la necesitamos (1) ó no (0). **1**
- SINGL: cascada (0) ó single (1). **0**
- ADI: para un procesador viejo  de 8 bits, no se usa.
- LTIM: flanco (1) o nivel (0). **1**

¿Cómo detecta el PIC que lo voy a inicializar? 
El dispositivo detecta el comienzo de la secuencia de inicialización cuando se le escribe en el address 0 un byte cuyo bit D4 está en 1. Obviamente, esto pasa con ICW1, que es la primera palabra en la secuencia de inicialización.

El PIC máster está en la posición de E/S 0x20 y 0x21.
El segundo PIC está en la posición 0xA0 y 0xA1.
0x11 es el valor de nuestro ICW1 para inicializar. (ver bits).

**ICW2:**

- En los bits T vamos a guardar el tipo de la IR0. A las siguientes IRs, el PIC les va a asignar los tipos de interrupción consecutivos.

**ICW3:**

Tiene 2 formatos, uno para el máster y otro para el slave. El ICW3 hace la conexión lógica entre el máster y el slave. 

- Al máster le dice en qué líneas tiene conectado slaves y en qué lineas dispositivos.
- Y le dice al slave a qué línea del máster está conectado. Se lo decimos en los 3 bits menos significativos.

**ICW4:**

Indica en qué tipo de procesador estamos trabajando y el modo de fin de interrupción que usamos: automático (1) o normal (0). **0**

Luego hay 3 bits para poner en 0, que son de buffer y añidamiento.

---

Cuando termina la ICW4, el dispositivo PIC queda establecido y está en modo operacional. Si en algún momento querés modificar algún aspecto del PIC, tenés que mandar esta secuencia de inicialización nuevamente. 

---

**Modo operacional**

"En el modo operacional estamos ahora para trabajar con palabras que ya operen sobre la operación de esto."

OCW = Operational Control Word. Hay 3 OCWs.

**OCW1:** 

Nos permite trabajar sobre el registro de máscaras. Se pone en el puerto 0x21 o 0xA1. De ahí podemos escribir o leer el registro de máscaras.

**OCW2:** 

Importante porque nos permite mandar comandos. End of interrupt, automatic rotation, specific rotation. Vamos a usar los **fin de interrupción.**

Hay 2: específico o no específico.
El no específico resetea todo el ISR, el específico envía sólo el bit que quiere bajar. En L2, L1 y L0 se envía qué bit querés bajar.

**OCW3:**

Sirve para poder leer el IRR y el ISR, pero medio que no lo vamos a usar. Toda la parte de arriba va en 0. Requiere escribir para después leer. O sea escribís qué registro querés leer, y después lo leés. (IRR ó ISR).

---

---

## Hardware de una PC - Manejo de la E/S básica.

¿Por qué las cosas son como son?

Las PCs deben guardar compatibilidad con la primera. Esto nos lleva a tener un primer Mb de memoria fragmentado entre RAM y ROM y direcciones de E/S que debemos respetar siempre.

Hay una tabla de direcciones pre-asignadas para E/S. 

Programable Interval Timer.

Teclado. Break / Make. Scan, make y break code.

Video Monocromo (un solo color).


[1]: https://lh3.googleusercontent.com/NmQgdfZndh46mb9ZGuEQMJ9P-DnKZO67_UN71dXdrm8FYhNu8agSf_1x66JWDPPNyb8ZXXZ8Rr8WPsuuv9nJRFVUA0JPfxRebf_WqXnCDGWkMTxspi3p5F7mA7TlBDWHYOdWKOea1Fnz_rXRW2__BATvEvL1FjBHKRPki-ufRh634K2A2ww75cI6HKlIHUeimyEm0PUQMtpMADCqnLU7M_Jx0PVlj5v7CEADSO2JdakpaFcfHsF_zw6EkDhPLSHNM8y_ttoqwdU0Wi4glC22GavDCtbt8rN-jCTLDzpnDUw4h6GHLV75CZKvB9abGfRXze-eAs-ARuPZryKouqkhQp43LdDSgMrQ4HrOZ0blx6OSKMZLd1gKKDmHvmWtYA4npO4A564yl-W3F36GPyIhvF-KceWYSCBedrdrXpc8gRqxm8DccnG2Co1kdy8NbZFMUCBKHYP-tgVKOfMdQZ5nQiuR6v-TirH4n1W0LOPxgfIYF3oeZEcEe-fNDROOgmv5VHXcTpMnElIYaYAuT-oCiYgjbAFSoX5stjQgKGOD9QJ-w5c4zTF6WGgwRSBPlGoDJ24Ivf-bOL_om0ySbwbDKMw52AYV4PPFKaRUXz9uia56JWmpxwklHZMp50enWGG5eP0d827NluyBYH4nSzUCNSqyNkzcZXfkp9DoHFGPttspO1dla0tqlAO2yhLQRWfEhPYAj5pmT3Ans44quibQ0Gv_JlTVurWT2ffGIIVXSzHaNW2PsCxB5UqjTvTbv-U7NeiTehQhIkrJxOm9hSFQmkI4PxQvfn5wP3YMRNouYbk9zLnc1vXjXsSvdXUbLC-YbLmlC5RkXdgQzLsfk6yObz50SjV3r1J67Rkok-6ZnQ=w630-h226-no?authuser=3

[2]: https://lh3.googleusercontent.com/2Q_lRc8pZh5ud90HUBJdQTbOn9QK7lAuc-vlQHPSZO1Wgqb07S6GNUbebaz939g7JdAHB-VfPdB4v5soEGxFd7VSUmZE1U_9egffkukHWTfA6VH07OKcnknmjDwrK2_0pJFALoqIZA3grO1H4lovYMfvQucAh48KOyEzD_0P5UUrQjtRKu20iYMauHaYsznajhQj0ervF3HPBowhDmh5GFZb8XIUwGShqaQP0iDMXtOicUmYrX3_BG5WKkzhKPsR_tBa30LLQL9FKcg2Cm5-uF5FS6bkGiktq1XPmqcGjkF0Ql7WAU8R36nagWETkB0JMMLiuof0tK73BjAu_TZ6KBiVpigV5boVnblgygmjOyODVrvYxbHQRwdDjg1xDs7Hc2MabJPxGdFFUaK3lwPpCPLBplQIlua7MuEqnhafaHLqqlg3AbQRshG3ALUp0g68Rg6w1ughRYrVt7CTPpnyGr6_GgHrWFM0BAjlswMCpFNBvPYV2TYLB3r9VM-CjhUnrB69nrH9QdyQCyJaYF7mb05AYvv_Kd-2JBWnqYwy2ayeumMA-7g3do4jLm3wAnk4RARqFRHe98dLZj74dQyaMre6zX5YKbNluOKwjoz67Qq4ATsEqHg0XxmJeA5ETJ0rQiKVaZpdIIjMd6-xazHulFhn-egEFylrOjdsLeBSDqylpy5-_aLyME7A8cBN4p1QkU1CMQ7R3jyI_bJ1YUcrlb_hd7wiv-zfB7nsy0wmN2-gsZV84zyaS2wAwqQfl5rESEJZhoWQGyWB4ynG499sjYlc0dak8LBb5h6D-nwNo2HhGIfMbJNvHtNlCFneZUdPxm2pdVbCU05U7XcyizNl_Cnc-29ERw2tpYgsvJHrVg=w924-h614-no?authuser=3

[3]: https://lh3.googleusercontent.com/-utb3WI38pp2ILze1VESv2YN8gVeAz3b75hvFlQeo4JZflBoQPAI00lhWw4lgA5d3-OvuxHYv4AuVyIXt2afXuiBPalp3_XBC4uhxntZ1XhEkYSY1tkBL6Ffsw7QwMzvMuSOMR5cJIbJoPyNkbUuCmRIw264wq7lID21h1w9Pm7xk2eQfnpSdU4IcgUAz04lxOEn6pMgyVgNpU6GT2z2mftMhafH12k49Kqh5dJBaCB-gUYjfthwOWM6MQM2BLYyuUhRGAw_ZYf4RAMjNALmHmFSSALy8xzF79wp0hwATF-OzVNumpSk-He1Ow-V1jX5KrzGRX6KiLRJzbWKXJgRBMrHkZfmc15-jFzbJZ2bSmzqusEU1HL-EgVFZzm1IEZThnpYxPCEdG9PY6KDR6m73W-NkRIBuKhCIBiKbi4bVXFJo4P_YfKFzsorQl05CRBhHvRYZP-VYh-QVHS4d_-q51OVEiPJUQnx8JxfLvzyoVSRr1-HLqBHwYhJvkFmLZYta9NxrDIeDJW5CbHnp3ADQYglg6y-RxcbQO1BAta_Ol3_a9hBZPS1sadZJE6b1tKuXc4ie3AkAIlaAnR46_jLsOg6q_9v_QhL1mlRYEY6cpPsCvYLxKaqwOEMbUQX-N-C09yfFfOtfAHQh_6I9klAv-fGcxbrWRPKQgRXpyug3lh1dTpQL8lYuxKfwKyF4JtO7xJZpHPZS1zPieoQqDLlIYKZ0n1rTuyaupaNjcg9meqdCj7yIzfAW7dnJ4YLpHe8nPkjWPW7MQZypWVxAvQ78JQejh2pUm9DhgPW3YHYAF17wLip50oPjv8vQVtweRfePHSb08gpspjJc-parxBNdJTTNe6rNS4_52EiE9sInw=w834-h608-no?authuser=3

[4]: https://lh3.googleusercontent.com/SkUCiTkZ69d5xQiAW5TOUjjOxOMcSl8jL6t1T_CXsyyHJpikgMzsgPUZZHVlVA4yM_qGf81h7kLbft8NXH2it-U_H_AEXWm8Cm1GCkHLup06MTNEfCshuc-CcbtfKC5judbyYnImevNurcGAHtv03IcZQgGWdiWPdgFkpCWFKdP1KsjJnCGCtlLrYWs7WyZ8OBJZF0Oe6qbNnYqfGEYwC2ErqUhmac2SSib_a0jn6l--CRn1QKQMCUy9zW3fPtfT_nkkwIr57g6Qvje8DNyBIj8zaG1Eg4rq1KtMm9NFOm0Us9KcRkSLeVEU944SDI5_Nx-KVCi3Hoa7fGfNUb_SWlvH3mSkHGe1SkLjUoPkmKUBS5tldATgL81zFs_v1CCETqGP0RQ9NxG0-w2fYSU_Me_ywJPMAfQtYAd190yp_5-bm5Pm2l-VMrl7RVL48VtW-YRxhdLfkHjNl8Svjx2boG1vgHL-p1v34mJpf9JIpfozuSB9b07a7T1IH9yLLW6ocnHdB2GqyO2b9vz7LdfyLVZIPjFJ813L7xCGP2TaHwMr-dY5KWp9cob76Flu2FLXcM5-KNCU9gEFinpNxwwY17tRwsiYR_RFZQsLCngSq0wD0cNpQSlSFwNTWvHSXnQXyz_6FR6tzU6kZsyBzVeq7zVglXDpeAsCpi_L9mKrqPQSCXyzTg5iDIQNyww6zH9zX8mB9ExSmXzszSSfrcG1GEfMriw874P5RwEJ24QToJ7FvR5Tc9AmJAiVAQRhrEvH34772U1eShu9VvRJt1SyMJnhra4FjpHhV5ROn9SMjCm2b4VRYJzK8eJhA1zqD4Ww46zOtGbmh2q9mezFh-VnPsz7xWAxFXP1OdSt6Uxuzg=w858-h606-no?authuser=3

[5]: https://lh3.googleusercontent.com/3zp56T3a53cPp_elfHTuwm3mHH1BrglRAswJP2ezXexepevsPRr9zWcRGtZLjyFx9X99LQjGITCOK7T1bWVzsPirhvBPhQOd9s_6sNqxqibZnwgM2AanNoATS5A65lkmGTqfvpeX_Fi69oltAKFXRi_Ic76qPG6ORrhAJKQU2P0ROGsgqsxRjp3AfTNlbg4gHeG0egRvpXo2_gbeKSGKLQ4fQx1UyONRWek16nQchknnyqAxQm5e0bU5QGYU9mdywonulikoF-J4cd6uHnsko29bwuHB4egtfmbIW8CLixZzJoowi1Rp_h6JaLMjiFK8Ict1t9n7ol3n9bRuCVjhcGCN2J7TLOu-20WJzE_rPL7GMg7baSpSizj7MOIil-ETbl3aZH2rcm2CgoDsRnpFt1DH038E-a2qjM5CcMhIfMhxLbzkmhpBblg9Ms7XmIlDlVqrXrRHkDlTSO9wmdaNhi0G3dvCErOcl2KzZHcQoFHGMr4l5wkiw21Fdsq_3-Ze3153uAbWINHM_K4BZxyx1EJ9w7aWOGu_vtm3kwlMZHDhKUU_7lALb6No82XcKHGJTU7Rlo2Gh9oMg7IdO6fQ60Rxl6kdSZNl-pvxudzrO7ahNfUR-nWO-0Yx7SQfyhV6dYNb_aHn_SePjIsyBbdAt9LEC84MXH2_m8a_xlGQL9lNKgseOrzI6gKGf2W9_TnBaZwu4he2-0il6NzyT8sHZVrPoU3NyD45f8TSn1R4qLGClf3XINjorv8NNsD-OR7LSNKuwa8JGFYx5YKmX-ZOfhL39YzplbR8zYTT-FGN27I4GifXzqT-5eAZBYBdSvvxa8UTZrrNB3BNMCpYO7qHSK1z-1OM9YWe1Oqf_Hodkw=w906-h762-no?authuser=3

[6]: https://lh3.googleusercontent.com/IihfbekVC5rC-K_8ia4cWAxH8ux45N0E9N2vc52-oUpf6MaawDFkNf0S9kFOkYcj3TraxZolZdjBe9YPOrOxBcD7_GQMFv5zH1l9Oj2diUGRaVP0dql67PObHLac4SjJChny7005XGTem-QroZDewpqvXig76lryrXwCk2mVJVrWnGOBZqbjhkCNuvSx8F_iVVzcI92iB4lPgkBlzhlntbFmIOTKSLuTcuoeZkb5w-c0aON1T3iH_NgxScifS7x0PPBbx_rAsEfWb1zobNUTTQZ02PWbhpOraZMvqxYxA2eARS4Im2RRztXwWJMW7dh3iXZ03A3W1MTpIBheVcrm-jaaWU1Tetx0gxED6AgWEZQ72iGJxIYyZIgDDEi7LxD1dRVevPthg2xf-SaIAywbPTRNEtBqbDAzIpQxGPm9NZaDMWKLs3GaejE2OGEXevJjPkP-g_r38xER3QBQK_1zHYY0EiTjxH3qqGB5VvObreV2-caKYIYQrO6pEQBowxhJxrQk5RP_JvmnH8K1cULtCLdJZC0-YQW2pc_wW2aynJbIHbYwyFiQ-q8Je7yhyeF9rBhjBlAwSgddWSq4fmWU4Mf9VkbdM8qIK65n2FUEL6ijsa1KmAlmnRRtse1KPbaGxyv1E4ko1H7U0VM-URydZiwiYXi4a17qjGJaYetT9UgudUvi-txDQ9TUnAa6vgZueCI5bQaL2bVhOsn0ysCP5h0H-h-D0tJnX62hJZnn7wcJEMdM3HxxkI7rth81iPaCh-8rlU9iZsxGznCbtGaRx2MslcA2TjBCJyqfZ4FlZ2nlaf1BdvXcJNfMXGaaNs3PE-Nn7hW22hCUJjSH2Bj_ffeZhEJ2B_sMXJwhf2lYHA=w384-h638-no?authuser=3