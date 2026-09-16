# ADR-0001: Optimizar el feed personalizado mediante caché distribuida

**Estado:** Propuesto  
**Fecha:** 2026-09-16  
**Decisores:** Julio Cesar Gonzales Castillo, Lilit Ameli Gutierrez Pancorbo, Luz Marina Apaza Gomes y José Antonio Chavez Gomez  
**Contexto técnico relacionado:** TikTok — API del feed «Para ti» y servicio de recomendación

---

## Contexto y problema

El feed «Para ti» debe entregar recomendaciones personalizadas con baja latencia durante periodos de alta demanda. El escenario de calidad considera 10 000 solicitudes concurrentes en una hora pico y establece una latencia p95 menor o igual a 500 ms, con al menos 99 % de respuestas sin error del servidor.

Calcular todas las recomendaciones de forma síncrona en cada solicitud incrementaría el tiempo de respuesta y podría convertir al servicio de recomendación en un punto de congestión. Una demora perceptible interrumpiría la navegación continua entre videos.

## Fuerzas / restricciones en juego

- El feed debe conservar la personalización para cada usuario.
- La API debe responder a 10 000 solicitudes concurrentes durante una hora pico.
- La latencia p95 debe ser menor o igual a 500 ms.
- Al menos 99 % de las solicitudes debe finalizar sin error del servidor.
- Una falla temporal del recomendador no debe impedir que el usuario reciba contenido.
- Las nuevas interacciones deben influir en las recomendaciones posteriores.

## Opciones consideradas

1. **Calcular las recomendaciones de forma síncrona en cada solicitud**
   - + Utiliza las señales más recientes del usuario.
   - + Evita almacenar lotes anticipados.
   - − La latencia del feed depende directamente del tiempo de cálculo del recomendador.
   - − Los picos de tráfico pueden saturar el servicio de recomendación.

2. **Utilizar una caché global de videos populares**
   - + Ofrece respuestas rápidas y sencillas de escalar.
   - + Puede continuar funcionando si el recomendador no está disponible.
   - − Reduce la personalización del feed.
   - − Puede mostrar contenido poco relevante para el usuario.

3. **Generar lotes personalizados anticipadamente y almacenarlos en una caché distribuida**
   - + Desacopla la entrega del feed del cálculo intensivo de recomendaciones.
   - + Conserva la personalización y reduce la latencia.
   - + Permite utilizar contenido popular como respaldo ante una falla.
   - − Aumenta el consumo de memoria y la complejidad de invalidación.
   - − Las recomendaciones pueden quedar desactualizadas durante un periodo breve.

## Decisión

Se adopta la generación asíncrona de lotes personalizados con almacenamiento temporal en una caché distribuida y una respuesta de respaldo basada en tendencias y región.

## Justificación

Esta alternativa permite responder rápidamente sin ejecutar el cálculo completo en cada solicitud. También conserva la personalización que caracteriza al feed «Para ti» y mantiene la continuidad del servicio cuando el recomendador presenta lentitud o una falla temporal.

## Consecuencias

**Positivas**

- Se reduce la latencia percibida por el usuario.
- Se distribuye la carga del recomendador fuera de la solicitud principal.
- El feed puede seguir respondiendo mediante contenido de respaldo.
- La solución favorece el cumplimiento de la latencia p95 menor o igual a 500 ms.

**Negativas / riesgos**

- Aumentan el consumo de memoria y el costo de la caché distribuida.
- Los lotes pueden contener recomendaciones temporalmente desactualizadas.
- Se requiere una estrategia de expiración, renovación e invalidación.
- El contenido de respaldo puede ser menos relevante que el personalizado.

**Seguimiento**

- Medir la latencia p95, la tasa de errores y la tasa de aciertos de la caché.
- Ejecutar una prueba de carga de 30 minutos con 10 000 solicitudes concurrentes.
- Verificar que al menos 99 % de las solicitudes termine sin error del servidor.
- Revisar periódicamente el tiempo de vida de los lotes y la calidad de las recomendaciones de respaldo.

## Referencias

- [Escenarios de calidad de TikTok](../atributos-y-escenarios-de-calidad.md)
- [Arquitectura de microservicios propuesta](../c4/arquitectura-microservicios.md)
- [TikTok Newsroom: Cómo TikTok recomienda videos #ParaTi](https://newsroom.tiktok.com/como-tiktok-recomienda-videos-foryou?lang=es-419)
- ADR relacionado: ADR-0002

---