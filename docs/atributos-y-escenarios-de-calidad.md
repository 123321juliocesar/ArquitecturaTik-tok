# TikTok: atributos y escenarios de calidad

Documento académico para los puntos 1 y 2 de la actividad «De la historia de éxito al escenario de calidad». Las cifras de los escenarios son **metas propuestas para el ejercicio**, no métricas oficiales de TikTok.

## 1. ¿Qué atributos de calidad sostienen su éxito?

### Atributo 1: rendimiento del feed personalizado

Una razón del éxito de TikTok es que cada usuario descubre rápidamente videos adaptados a sus intereses en el feed «Para ti». Esa experiencia depende de que la solicitud del siguiente lote de recomendaciones tenga **baja latencia**, incluso cuando muchos usuarios navegan a la vez. Si la respuesta tarda, el usuario espera entre videos y se rompe la continuidad del consumo. No hablamos de «que la aplicación sea rápida» en general, sino del tiempo de respuesta del feed personalizado, que es una función central del producto.

### Atributo 2: escalabilidad de la entrega de videos virales

Otra razón del éxito es que un video puede llegar a una audiencia amplia en poco tiempo. Esto genera picos repentinos de solicitudes para reproducir el mismo archivo. La plataforma necesita **escalar la entrega de video bajo demanda** para que ese contenido siga disponible sin ralentizar los demás. No hablamos de «soportar muchos usuarios» de forma genérica, sino de absorber el crecimiento súbito de reproducciones de un contenido viral.

Ambos atributos se complementan: el rendimiento describe el tiempo de respuesta del feed bajo una carga definida; la escalabilidad describe cómo se mantiene la entrega de videos cuando la demanda aumenta. La personalización del feed está explicada por [TikTok Newsroom](https://newsroom.tiktok.com/como-tiktok-recomienda-videos-foryou?lang=es-419). La relación entre viralidad, carga y capacidad es un análisis arquitectónico del equipo, no una descripción de la infraestructura interna de TikTok.

## 2. Escenarios de calidad de seis partes

Se sigue el orden de la diapositiva: **fuente del estímulo, estímulo, artefacto, entorno, respuesta y medida de respuesta**. El artefacto identifica los componentes concretos que reciben el estímulo, no toda la plataforma. Los valores elegidos sirven para evaluar una arquitectura hipotética con pruebas de carga; no describen acuerdos de servicio reales de TikTok.

### Escenario 1: rendimiento del feed «Para ti»

**Problema plausible:** durante una hora de alta actividad, la selección personalizada tarda demasiado y los usuarios perciben pausas entre videos.

| Parte | Descripción |
|---|---|
| 1. Fuente del estímulo | Usuarios autenticados que navegan el feed desde la aplicación móvil. |
| 2. Estímulo | Envían 10 000 solicitudes concurrentes para obtener el siguiente lote de videos de «Para ti». |
| 3. Artefacto | API del feed y servicio de recomendación de la arquitectura propuesta. |
| 4. Entorno | Producción, durante una hora pico en la región evaluada. |
| 5. Respuesta | La API consulta las recomendaciones y entrega un lote de identificadores y metadatos de videos elegibles, sin bloquear el desplazamiento del usuario. |
| 6. Medida de respuesta | En una prueba de carga de 30 minutos, latencia p95 ≤ 500 ms para la solicitud del feed y al menos 99 % de respuestas sin error del servidor. |

### Escenario 2: escalabilidad ante un video viral

**Problema plausible:** un video popular provoca un incremento abrupto de reproducciones y la capacidad disponible resulta insuficiente.

| Parte | Descripción |
|---|---|
| 1. Fuente del estímulo | Usuarios que intentan reproducir un mismo video viral. |
| 2. Estímulo | Las solicitudes de reproducción de ese video aumentan a 5 veces el nivel habitual en 15 minutos. |
| 3. Artefacto | Servicio de entrega de metadatos del video y capa de caché/CDN de la arquitectura propuesta. |
| 4. Entorno | Producción, con el aumento concentrado en una región y el resto del tráfico en su nivel habitual. |
| 5. Respuesta | La capa de distribución sirve el archivo desde caché y se amplía la capacidad del servicio de metadatos, manteniendo disponibles los demás videos. |
| 6. Medida de respuesta | En una prueba con demanda 5×, la capacidad adicional se habilita en ≤ 5 minutos; durante el pico, latencia p95 ≤ 700 ms para metadatos y al menos 99 % de respuestas sin error del servidor. |

## Cómo evaluar los escenarios

Se necesitaría una implementación o simulación de la arquitectura propuesta y una prueba de carga que reproduzca las condiciones indicadas. Las medidas permitirían decidir si el diseño cumple o si necesita cambios; este repositorio no afirma haber ejecutado esas pruebas.
