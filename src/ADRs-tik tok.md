# ADR-0001: Arquitectura de entrega de video con CDN y precarga predictiva para cumplir SLA de latencia de reproducción

**Estado:** Aceptado
**Fecha:** 2026-09-16
**Decisores:** Equipo de Arquitectura — Plataforma de video (TikTok)
**Contexto técnico relacionado:** Sistema de reproducción y distribución de videos

---

## Contexto y problema

Durante horas de alta demanda, con millones de usuarios conectados
simultáneamente, cuando un usuario desliza hacia el siguiente video el
sistema debe entregarlo y comenzar su reproducción en un máximo de 2
segundos en al menos el 95 % de las solicitudes. Servir los videos desde un
origen centralizado sin optimizaciones de distribución ni precarga genera
tiempos de espera crecientes conforme aumenta la concurrencia, lo que
rompe la experiencia de scroll continuo característica del feed y
compromete el atributo de calidad de **Rendimiento**.

## Fuerzas / restricciones en juego

* El estímulo (deslizar al siguiente video) es de alta frecuencia y ocurre
  en ráfagas impredecibles por usuario.
* La medida de respuesta (≤ 2 s en el 95 % de los casos) es un SLA duro,
  no una meta orientativa.
* El entorno es de carga alta pero sostenida (hora pico habitual), distinta
  de un pico viral puntual (ver ADR-0002).
* Los usuarios acceden desde redes muy heterogéneas (4G, 5G, WiFi de baja
  calidad).
* Se busca minimizar el consumo de datos/batería del cliente cuando sea
  posible.

## Opciones consideradas

1. **CDN multi-nivel + precarga predictiva + streaming adaptativo (ABR)**

   * + Reduce la distancia de red y los "cache miss" al servir desde el
     edge más cercano al usuario.
   * + La precarga del siguiente video probable (según el algoritmo de
     recomendación) puede lograr latencias percibidas cercanas a 0 s.
   * − Mayor costo de infraestructura (almacenamiento distribuido en
     múltiples PoPs) y de datos/batería en el cliente.

2. **Servir todo desde origen centralizado, sin CDN ni precarga**

   * + Arquitectura simple, menor costo de infraestructura.
   * − No puede cumplir el SLA de 2 s a escala global bajo alta
     concurrencia; la latencia crece con la distancia y la carga.

3. **Precarga agresiva de N videos siguientes (buffer amplio) sin ABR**

   * + Latencia de inicio prácticamente nula para los videos precargados.
   * − Alto consumo de datos y batería incluso para videos que el usuario
     nunca llega a ver; no resuelve el problema en redes lentas sin ABR.

## Decisión

Se adopta una arquitectura de entrega de video basada en **CDN multi-nivel
con edge caching, precarga predictiva del siguiente video y codificación
adaptativa (ABR)**, complementada con monitoreo continuo de latencia
p95/p99.

## Justificación

La combinación de CDN y precarga predictiva ataca directamente las dos
fuentes principales de latencia: la distancia de red hasta el contenido y
el tiempo de decisión/solicitud del siguiente video. El streaming
adaptativo evita que una red lenta bloquee el arranque, priorizando
comenzar a reproducir rápido sobre la resolución máxima en el primer
segundo. Esta combinación es la única de las opciones evaluadas que puede
sostener el SLA de ≤ 2 s en el 95 % de las solicitudes a la escala de
usuarios concurrentes descrita en el escenario.

## Consecuencias

**Positivas**

* Se cumple la medida de respuesta objetivo (≤ 2 s en el 95 % de las
  solicitudes) incluso en horas de alta demanda.
* La precarga predictiva mejora la percepción de fluidez del feed.
* El monitoreo p95/p99 permite detectar degradaciones antes de incumplir
  el SLA.

**Negativas / riesgos**

* Mayor costo de infraestructura por almacenamiento distribuido en
  múltiples ubicaciones de CDN.
* Consumo adicional de datos y batería en el cliente por la precarga,
  incluso de contenido no visto.
* Requiere invertir en observabilidad de latencia en tiempo real, lo que
  añade complejidad operativa.

**Seguimiento**

* Revisar periódicamente si conviene hacer la precarga configurable
  (por ejemplo, solo bajo WiFi) para reducir el consumo de datos móviles.
* Validar que el sistema de monitoreo de latencia definido aquí sirva
  también como señal de entrada para el auto-scaling del ADR-0002.

## Referencias

* Escenario de calidad — Rendimiento (documento de escenarios de calidad,
  02).
* ADR relacionado: ADR-0002 (Escalabilidad ante picos virales).

---

# ADR-0002: Auto-escalado dinámico y detección de contenido viral para soportar picos de hasta 10x la carga normal

**Estado:** Aceptado
**Fecha:** 2026-09-16
**Decisores:** Equipo de Arquitectura — Plataforma de video (TikTok)
**Contexto técnico relacionado:** Plataforma de distribución y reproducción de videos

---

## Contexto y problema

Cuando un video se vuelve viral, la cantidad de solicitudes de
reproducción puede aumentar hasta 10 veces respecto a la carga normal en
un período corto. La plataforma debe incrementar dinámicamente su
capacidad de procesamiento y distribución, manteniendo el inicio de
reproducción en ≤ 2 segundos para al menos el 95 % de las solicitudes, a
pesar de que la capacidad aprovisionada para carga normal es insuficiente
para ese volumen. A diferencia del ADR-0001 (carga alta pero previsible y
distribuida), aquí el riesgo se concentra en un único artefacto (el video
viral), lo que puede generar un "hotspot" sobre un origen o región
específica.

## Fuerzas / restricciones en juego

* El pico de carga es repentino, impredecible en el tiempo y
  desproporcionadamente concentrado en un solo contenido.
* La medida de respuesta exige mantener el mismo SLA de latencia
  (≤ 2 s en el 95 %) incluso bajo 10x la carga normal.
* Sobreaprovisionar permanentemente para 10x la carga es costoso y
  probablemente innecesario la mayor parte del tiempo.
* El escalado manual es demasiado lento frente a la velocidad de
  propagación de un video viral (minutos vs. horas de reacción humana).

## Opciones consideradas

1. **Auto-scaling horizontal basado en demanda + detección de "hot
   content" + degradación controlada**

   * + Responde a eventos virales sin intervención manual, replicando el
     contenido caliente hacia más nodos edge antes de saturar un origen.
   * + La degradación controlada (bajar bitrate inicial) prioriza cumplir
     la latencia de arranque sobre la calidad de imagen si la demanda
     supera incluso la capacidad auto-escalada.
   * − Existe una ventana de reacción del auto-scaling (no es
     instantáneo), con riesgo de incumplimiento del SLA en los primeros
     segundos/minutos de un pico extremo si no hay buffer de capacidad.

2. **Sobreaprovisionar permanentemente para 10x la carga normal**

   * + Elimina el riesgo de ventana de reacción del auto-scaling.
   * − Económicamente ineficiente, ya que esa capacidad estaría ociosa la
     mayor parte del tiempo.

3. **Escalado manual activado por el equipo de operaciones ante eventos
   virales**

   * + No requiere automatización compleja de detección de anomalías.
   * − Demasiado lento frente a la velocidad de propagación viral; el
     daño a la experiencia de usuario ya habría ocurrido antes de
     intervenir.

## Decisión

Se adopta una arquitectura de **auto-scaling horizontal basado en métricas
de demanda en tiempo real, con detección temprana de contenido viral
("hot content detection"), replicación dinámica de ese contenido en más
nodos edge, y degradación controlada de calidad como último recurso**.

## Justificación

Solo el auto-scaling automatizado puede reaccionar dentro de los tiempos
que exige la velocidad de propagación de un video viral. La detección
temprana de "hot content" permite anticipar y replicar el contenido antes
de que un solo origen o región se convierta en cuello de botella, y la
degradación controlada actúa como red de seguridad para garantizar que,
ante un evento aún mayor al previsto, se sacrifique calidad de imagen
antes que incumplir el tiempo de inicio de reproducción, que es la
prioridad establecida en el escenario de calidad.

## Consecuencias

**Positivas**

* Permite soportar hasta 10x la carga normal manteniendo la latencia de
  arranque dentro del SLA, sin intervención manual.
* La detección temprana de contenido caliente reduce el riesgo de
  cascada de fallos sobre un origen o región puntual.
* La degradación controlada evita el peor escenario (incumplir la
  latencia) incluso si el pico supera lo estimado.

**Negativas / riesgos**

* La ventana de reacción del auto-scaling puede generar incumplimientos
  puntuales del SLA en los primeros instantes de un pico extremo si no
  existe un buffer de capacidad base.
* Mantener capacidad de reserva y sistemas de detección de anomalías
  incrementa el costo operativo incluso en periodos sin eventos virales.
* La replicación dinámica de contenido "caliente" añade complejidad
  operativa (coordinación entre CDN, invalidación de caché, consistencia).

**Seguimiento**

* Ejecutar pruebas de carga periódicas que simulen picos de 10x para
  validar que el tiempo de reacción del auto-scaling no rompe el SLA.
* Evaluar, según el historial de eventos virales, si conviene mantener un
  buffer de capacidad base mayor al actual para reducir el riesgo de la
  ventana de reacción.

## Referencias

* Escenario de calidad — Escalabilidad (documento de escenarios de
  calidad, 03).
* ADR relacionado: ADR-0001 (Rendimiento — latencia de reproducción bajo
  carga alta).
