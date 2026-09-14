# Actividad: De la Historia de Éxito al Escenario de Calidad

## Empresa: TikTok

## ¿Por qué TikTok es exitosa?

TikTok es exitosa principalmente porque permite que los usuarios vean videos de manera rápida y fluida, incluso cuando existe una gran cantidad de usuarios conectados simultáneamente. Además, puede crecer y soportar enormes cantidades de contenido, usuarios e interacciones sin que el servicio deje de funcionar.

Por ello, los dos atributos de calidad que mejor explican su éxito son:

---

# 01. Atributos de calidad

## Atributo 1: Rendimiento (Performance)

### Justificación

TikTok necesita entregar videos y responder a las acciones del usuario en muy poco tiempo. Cuando una persona abre la aplicación, espera que el siguiente video aparezca rápidamente al deslizar la pantalla.

Por ejemplo, acciones como:

- Cargar un video.
- Pasar al siguiente video.
- Dar "Me gusta".
- Comentar.
- Compartir.
- Cargar recomendaciones.

deben ejecutarse con baja latencia.

### ¿Cómo contribuye al éxito?

Un buen rendimiento permite que el usuario tenga una experiencia fluida y no abandone la aplicación por esperas prolongadas.

### Indicador medible

Latencia de carga/reproducción inicial del video, por ejemplo, **≤ 2 segundos en el 95 % de las solicitudes**.

---

## Atributo 2: Escalabilidad

### Justificación

TikTok tiene que soportar millones de usuarios y una enorme cantidad de videos, reproducciones, comentarios, "Me gusta" y cargas de contenido.

El sistema debe poder aumentar sus recursos cuando aumenta la cantidad de usuarios sin que el rendimiento se degrade significativamente.

### ¿Cómo contribuye al éxito?

La escalabilidad permite que TikTok pueda atender grandes cantidades de usuarios simultáneamente, especialmente durante tendencias o videos virales.

### Indicador medible

El sistema debe poder incrementar su capacidad para atender un aumento de usuarios manteniendo la latencia dentro de los límites establecidos.

---

# 02. Escenario de calidad 1 — Rendimiento

## Problema: Carga lenta de videos

Un problema plausible para TikTok sería que una gran cantidad de usuarios intente reproducir videos simultáneamente y esto provoque retrasos en la carga.

## Escenario completo de 6 partes

| Parte | Descripción |
|---|---|
| **1. Fuente del estímulo** | Usuario de TikTok |
| **2. Estímulo** | El usuario desliza hacia el siguiente video para reproducirlo |
| **3. Entorno** | Hora de alta demanda, con millones de usuarios conectados simultáneamente |
| **4. Artefacto** | Sistema de reproducción y distribución de videos de TikTok |
| **5. Respuesta** | El sistema solicita, procesa y entrega el video utilizando los recursos disponibles |
| **6. Medida de respuesta** | El video debe comenzar a reproducirse en **≤ 2 segundos** en al menos el **95 % de las solicitudes** |

### Escenario redactado

Durante una hora de alta demanda, cuando un usuario desliza hacia el siguiente video, el sistema de reproducción de TikTok deberá entregar el contenido y comenzar su reproducción en un máximo de **2 segundos** en al menos el **95 % de las solicitudes**, evitando interrupciones perceptibles para el usuario.

### Atributo evaluado

**Rendimiento (Performance).**

### ¿Por qué es medible?

Porque podemos medir la:

**Latencia = tiempo desde que el usuario solicita el video hasta que comienza la reproducción.**

Por ejemplo:

- 1.2 segundos →  Cumple
- 1.8 segundos →  Cumple
- 2.0 segundos →  Cumple
- 3.5 segundos →  No cumple

---

# 03. Escenario de calidad 2 — Escalabilidad

## Problema: Video viral

Imaginemos que un video se vuelve viral y millones de usuarios intentan verlo al mismo tiempo.

Esto genera una carga mucho mayor que la habitual.

## Escenario completo de 6 partes

| Parte | Descripción |
|---|---|
| **1. Fuente del estímulo** | Millones de usuarios |
| **2. Estímulo** | Aumento repentino de reproducciones de un video viral |
| **3. Entorno** | Sistema funcionando bajo una carga excepcionalmente alta |
| **4. Artefacto** | Plataforma de distribución y reproducción de videos |
| **5. Respuesta** | El sistema incrementa dinámicamente sus recursos para atender la demanda |
| **6. Medida de respuesta** | Debe soportar un aumento de hasta **10 veces la carga normal**, manteniendo la latencia del video en **≤ 2 segundos** en el **95 % de las solicitudes** |

### Escenario redactado

Cuando un video se vuelva viral y la cantidad de solicitudes de reproducción aumente hasta **10 veces** respecto a la carga normal, la plataforma deberá incrementar dinámicamente su capacidad de procesamiento y distribución, manteniendo el inicio de reproducción en **≤ 2 segundos** para al menos el **95 % de las solicitudes**.

### Atributo evaluado

**Escalabilidad.**

### ¿Por qué es medible?

Porque podemos comparar:

- Carga normal → 100 %
- Carga viral → 1000 %

Y comprobar si el sistema mantiene el rendimiento establecido.

---

# 04. Mini-ADR — Decisión arquitectónica

## ADR: Uso de una arquitectura distribuida con CDN y escalamiento automático

### Contexto

TikTok debe entregar videos a una enorme cantidad de usuarios ubicados en diferentes regiones. Los videos pueden volverse virales rápidamente y generar millones de solicitudes simultáneas.

Si todos los usuarios solicitaran los videos desde un único servidor central, este podría saturarse y aumentar considerablemente la latencia.

Por esta razón, la arquitectura necesita distribuir el contenido y permitir aumentar la capacidad cuando exista mayor demanda.

### Decisión

Se decide utilizar una arquitectura distribuida basada en:

- Múltiples servidores/regiones.
- **CDN (Content Delivery Network)**.
- Escalamiento automático.

La CDN permite almacenar temporalmente contenido popular cerca de los usuarios, mientras que el escalamiento automático permite agregar recursos cuando aumenta la demanda.

### Funcionamiento simplificado

```text
Usuario
  ↓
CDN cercana
  ↓
Servidor de aplicación
  ↓
Servicios de TikTok
  ↓
Almacenamiento de videos
```

Cuando aumenta la demanda:

```text
Más usuarios → mayor carga → escalamiento automático → más recursos disponibles
```

---

# Consecuencias

## Ventajas

### 1. Mejor rendimiento

Los videos pueden entregarse desde servidores más cercanos al usuario, reduciendo la latencia.

### 2. Mayor escalabilidad

La plataforma puede aumentar sus recursos cuando existe un incremento de usuarios o reproducciones.

### 3. Mayor tolerancia a fallos

Si un servidor presenta problemas, otros servidores pueden continuar atendiendo las solicitudes.

### 4. Mejor experiencia del usuario

Los videos pueden comenzar a reproducirse rápidamente incluso durante períodos de alta demanda.

---

##  Desventajas / Trade-offs

### 1. Mayor complejidad

Una arquitectura distribuida es más compleja que utilizar un único servidor.

### 2. Mayor costo

Mantener múltiples servidores, CDN y mecanismos de escalamiento genera mayores costos de infraestructura.

### 3. Consistencia de datos

Al distribuir información entre diferentes servidores, algunos datos pueden tardar un poco en sincronizarse.

### 4. Mayor dificultad de administración

Se necesitan herramientas de monitoreo, automatización y administración de múltiples componentes.

---

# Entregable final

| Elemento | TikTok |
|---|---|
| **Empresa** | TikTok |
| **Atributo 1** | Rendimiento |
| **Justificación** | Los videos y acciones deben responder rápidamente para mantener una experiencia fluida |
| **Atributo 2** | Escalabilidad |
| **Justificación** | Debe soportar grandes aumentos de usuarios, reproducciones y contenido sin degradar significativamente el servicio |
| **Escenario 1** | Carga rápida de videos |
| **Métrica** | ≤ 2 segundos en el 95 % de las solicitudes |
| **Escenario 2** | Video viral |
| **Métrica** | Soportar hasta 10× la carga normal manteniendo ≤ 2 s en el 95 % |
| **ADR** | Arquitectura distribuida + CDN + escalamiento automático |
| **Trade-offs** | Mayor rendimiento y escalabilidad, pero mayor costo, complejidad y dificultad de administración |

---

# Conclusión para exponer

TikTok es exitosa porque ofrece una experiencia rápida y puede atender grandes cantidades de usuarios. Esto se relaciona directamente con los atributos de **rendimiento** y **escalabilidad**. Para lograrlo, una arquitectura distribuida con **CDN** y **escalamiento automático** permite reducir la latencia y aumentar la capacidad cuando existe una alta demanda. Sin embargo, esta decisión también genera *trade-offs*, como mayor costo y complejidad arquitectónica.
