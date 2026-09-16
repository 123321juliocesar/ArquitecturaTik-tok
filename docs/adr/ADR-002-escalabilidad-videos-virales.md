# ADR-0002: Distribuir videos virales mediante CDN y escalamiento automático

**Estado:** Propuesto  
**Fecha:** 2026-09-16  
**Decisores:** Julio Cesar Gonzales Castillo, Lilit Ameli Gutierrez Pancorbo, Luz Marina Apaza Gomes, Mildrely Laime Cervantes y José Antonio Chavez Gomez  
**Contexto técnico relacionado:** TikTok — servicio de metadatos y distribución de videos

---

## Contexto y problema

Un video viral puede multiplicar las solicitudes de reproducción en pocos minutos. El escenario de calidad considera un aumento de cinco veces sobre el tráfico habitual durante 15 minutos y exige habilitar capacidad adicional en un máximo de 5 minutos, mantener una latencia p95 menor o igual a 700 ms para los metadatos y lograr al menos 99 % de respuestas sin error del servidor.

Servir cada reproducción desde el almacenamiento de origen concentraría la carga, consumiría ancho de banda y aumentaría la latencia. Además, una cantidad fija de instancias del servicio de metadatos podría resultar insuficiente durante el pico.

## Fuerzas / restricciones en juego

- La plataforma debe absorber un incremento de cinco veces sobre la demanda habitual.
- La capacidad adicional debe estar disponible en un máximo de 5 minutos.
- La latencia p95 de los metadatos debe ser menor o igual a 700 ms.
- Al menos 99 % de las solicitudes debe finalizar sin error del servidor.
- La viralidad de un video no debe degradar la reproducción de otros contenidos.
- Los videos eliminados o restringidos deben dejar de distribuirse oportunamente.

## Opciones consideradas

1. **Entregar todos los videos desde el almacenamiento de origen**
   - + Mantiene una única fuente para los archivos multimedia.
   - + Simplifica la actualización o eliminación del contenido.
   - − Concentra el tráfico y el consumo de ancho de banda.
   - − Incrementa la latencia para usuarios alejados del origen.

2. **Mantener servidores regionales con capacidad fija**
   - + Ofrece capacidad disponible inmediatamente en cada región.
   - + Reduce la dependencia de un único origen.
   - − Mantiene recursos ociosos durante periodos de tráfico normal.
   - − No se adapta eficientemente a picos inesperados o localizados.

3. **Utilizar una CDN y escalamiento automático del servicio de metadatos**
   - + Distribuye los archivos desde ubicaciones cercanas a los usuarios.
   - + Reduce la carga y el ancho de banda del almacenamiento de origen.
   - + Ajusta la capacidad del servicio de metadatos según la demanda real.
   - − Aumenta la complejidad de configuración, supervisión e invalidación.
   - − Introduce costos de transferencia y dependencia del proveedor de CDN.

## Decisión

Se adopta una CDN para distribuir los archivos multimedia y escalamiento horizontal automático para el servicio de metadatos sin estado.

## Justificación

Esta alternativa permite atender la mayor parte de las reproducciones desde puntos cercanos a los usuarios y evita que el almacenamiento de origen reciba todo el tráfico. El escalamiento automático ajusta la capacidad dinámica a la demanda y evita mantener permanentemente recursos dimensionados para el pico máximo.

## Consecuencias

**Positivas**

- Se reduce la latencia de reproducción para usuarios de diferentes regiones.
- Disminuyen la carga y el ancho de banda del almacenamiento de origen.
- El servicio de metadatos puede ampliar su capacidad durante un evento viral.
- Los demás videos permanecen disponibles durante el incremento de demanda.

**Negativas / riesgos**

- Aumentan los costos de transferencia, caché y operación de la CDN.
- La primera solicitud regional puede presentar mayor latencia por ausencia de caché.
- El contenido eliminado podría permanecer temporalmente en algunos puntos de distribución.
- Una configuración incorrecta del escalamiento puede producir capacidad insuficiente o recursos ociosos.

**Seguimiento**

- Medir la latencia p95, la tasa de errores y la tasa de aciertos de la CDN por región.
- Ejecutar una prueba con una demanda cinco veces superior al nivel habitual.
- Verificar que la capacidad adicional quede disponible en un máximo de 5 minutos.
- Comprobar que la invalidación retire los videos eliminados de los puntos de distribución dentro del límite operativo establecido.

## Referencias

- [Escenarios de calidad de TikTok](../atributos-y-escenarios-de-calidad.md)
- [Arquitectura de microservicios propuesta](../c4/arquitectura-microservicios.md)
- ADR relacionado: ADR-0001

---
