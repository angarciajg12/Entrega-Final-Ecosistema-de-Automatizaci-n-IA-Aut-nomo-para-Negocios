Proyecto Final – Automatización Inteligente para Gestión de Fallas de Equipos
Descripción del proyecto
Este proyecto implementa un sistema automatizado para la gestión de novedades y fallas de equipos utilizando n8n, Airtable, Inteligencia Artificial, Gmail y Telegram.

El objetivo principal es automatizar el proceso desde la recepción de una novedad hasta su clasificación, validación y posterior generación de una Orden de Trabajo.

El sistema combina automatización e Inteligencia Artificial con un mecanismo Human-in-the-Loop (HITL), permitiendo que determinadas decisiones críticas sean validadas por una persona antes de continuar con el proceso.

Problema a resolver
La gestión manual de novedades de equipos puede generar demoras, errores de clasificación y falta de seguimiento.

El sistema desarrollado permite:

Centralizar las novedades de los equipos.
Clasificar automáticamente las fallas mediante IA.
Determinar su nivel de urgencia.
Identificar si el equipo debe detenerse.
Generar un resumen técnico de la falla.
Solicitar aprobación humana cuando la situación lo requiere.
Crear automáticamente una Orden de Trabajo.
Registrar errores del proceso.
Notificar a los usuarios involucrados.
Tecnologías utilizadas
n8n – Orquestación y automatización de los procesos.
Airtable – Base de datos principal.
Inteligencia Artificial – Análisis y clasificación de fallas.
Gmail – Recepción y envío de notificaciones.
Telegram – Validación humana de alertas críticas.
VisionLink – Fuente de alertas automáticas de equipos.
GitHub – Documentación y almacenamiento de los workflows del proyecto.
Arquitectura general
El sistema posee dos flujos principales.

Flujo 1 – Novedades registradas en Airtable
El primer flujo comienza cuando se registra o modifica una novedad en Airtable.

Proceso:

Airtable detecta una nueva novedad.
n8n obtiene los datos.
Se validan los campos obligatorios.
Si falta información, se registra el error.
Si los datos son correctos, la información se envía al modelo de IA.
La IA analiza la descripción de la falla.
Se actualiza la novedad con los resultados.
Según la urgencia, se ejecuta una acción diferente.
Las fallas críticas requieren validación humana.
Si la falla es aprobada, se genera una Orden de Trabajo.
Flujo 2 – Alertas automáticas provenientes de VisionLink
El segundo flujo permite transformar automáticamente las alertas recibidas por correo electrónico en novedades dentro del sistema.

Proceso:

Gmail recibe una alerta de VisionLink.

n8n detecta el correo.

Se extraen automáticamente los datos relevantes:

Equipo.
Descripción de la falla.
Fecha de reporte.
Se busca el equipo correspondiente en Airtable.

Se crea automáticamente una nueva novedad.

La novedad continúa por el flujo principal de clasificación mediante IA.

De esta manera, las alertas automáticas pueden integrarse al mismo sistema que las novedades cargadas manualmente.

Clasificación mediante Inteligencia Artificial
La IA analiza la descripción de cada falla y devuelve información estructurada.

Ejemplo:

{
  "tipo_falla": "Mecánica",
  "urgencia": "Alta",
  "requiere_detener_equipo": true,
  "resumen": "Pérdida de capacidad de frenado durante la operación del equipo."
}
Los datos generados se almacenan nuevamente en Airtable.

Los principales campos generados por IA son:

Tipo de falla.
Urgencia.
Necesidad de detener el equipo.
Resumen técnico.
Clasificación de urgencia
Las novedades pueden clasificarse como:

Alta.
Media.
Baja.
Cada nivel ejecuta una acción diferente dentro del workflow.

Urgencia Alta
Las novedades críticas son enviadas a Telegram para solicitar validación humana.

Urgencia Media
Se genera una notificación automática por correo electrónico.

Urgencia Baja
Se registra la novedad y se envía la notificación correspondiente.

Human-in-the-Loop (HITL)
Uno de los componentes principales del proyecto es la incorporación de validación humana.

Cuando la Inteligencia Artificial detecta una falla de alta urgencia, n8n envía un mensaje mediante Telegram.

El mensaje contiene información como:

ID de la novedad.
Equipo.
Descripción.
Tipo de falla.
Urgencia.
Fecha de reporte.
El responsable puede aprobar o rechazar la recomendación.

El workflow permanece a la espera de esta respuesta antes de continuar.

Esto evita que una decisión crítica dependa únicamente de la Inteligencia Artificial.

Generación automática de Orden de Trabajo
Cuando una novedad crítica es aprobada, el sistema genera automáticamente una Orden de Trabajo en Airtable.

La Orden de Trabajo puede contener:

ID de OT.
Novedad de origen.
Prioridad sugerida por IA.
Prioridad confirmada.
Fecha planificada.
Técnico asignado.
Estado de la OT.
Responsable de aprobación.
Fecha de aprobación.
Posteriormente, cuando se completa la planificación, el sistema puede notificar al usuario que reportó originalmente la novedad.

Manejo de errores
El workflow incorpora mecanismos para detectar errores y evitar que información incompleta continúe dentro del proceso.

Antes de realizar la clasificación mediante IA se validan los campos obligatorios.

En caso de error, n8n genera un registro en una tabla específica de Airtable denominada:

Log_Errores

Esto permite mantener trazabilidad de los problemas ocurridos durante las ejecuciones.

Entre los errores controlados se encuentran:

Falta de equipo.
Falta de descripción.
Falta de usuario reportante.
Datos incompletos.
Fallos de API.
Errores en relaciones entre registros.
Procesamiento por lotes
El diseño del workflow contempla la posibilidad de procesar múltiples registros evitando sobrecargar los servicios externos.

El procesamiento controlado permite:

Administrar mejor las llamadas a APIs.
Reducir errores por límites de solicitudes.
Controlar el consumo de recursos.
Procesar grandes cantidades de novedades de manera ordenada.
Optimización del uso de IA
Para reducir el consumo de tokens y mejorar la eficiencia del sistema se busca mantener las instrucciones estáticas del prompt separadas de la información dinámica.

También puede utilizarse cache_control sobre instrucciones repetitivas para evitar reprocesar información que no cambia entre solicitudes.

Esto permite reducir el costo de utilización del modelo cuando el sistema procesa múltiples novedades bajo las mismas reglas.

Análisis de costos
El principal costo variable del sistema está asociado al uso del modelo de Inteligencia Artificial.

Cada ejecución utiliza una combinación de:

Tokens de entrada.
Tokens de salida.
Prompt del sistema.
Datos de la novedad.
Para disminuir los costos se aplican estrategias como:

Prompts concisos.
Respuestas estructuradas.
Reutilización de contexto.
Procesamiento por lotes.
Uso de caché cuando corresponda.
En escenarios donde las entradas reutilizadas pueden beneficiarse de caching, es posible reducir de manera significativa el costo asociado al procesamiento repetido, alcanzando aproximadamente un 50 % de reducción en determinados componentes del consumo, dependiendo del proveedor y modelo utilizado.

Base de datos
La solución utiliza Airtable como centro de control.

Las principales tablas son:

Novedades
Contiene la información original y los resultados del análisis mediante IA.

Ordenes_Trabajo
Contiene las órdenes generadas a partir de las novedades aprobadas.

Usuarios
Permite relacionar las novedades con los usuarios responsables.

Equipos
Contiene los equipos registrados en el sistema.

Log_Errores
Registra los problemas detectados durante las automatizaciones.

Estados del proceso
Una novedad puede atravesar diferentes estados:

Pendiente.
Procesado por IA.
En revisión Mantenimiento.
Aprobado.
OT creada.
En proceso.
Resuelto.
Cerrado.
Estos estados permiten conocer en todo momento en qué instancia se encuentra cada novedad.

Beneficios de la solución
La implementación permite:

Reducir tareas administrativas manuales.
Clasificar automáticamente fallas de equipos.
Priorizar situaciones críticas.
Mejorar los tiempos de respuesta.
Centralizar la información.
Mantener trazabilidad.
Incorporar supervisión humana.
Generar automáticamente órdenes de trabajo.
Integrar alertas provenientes de diferentes fuentes.
Seguridad
Las credenciales utilizadas por los diferentes servicios no se almacenan dentro del repositorio.

No deben publicarse:

API Keys.
Tokens de Telegram.
Credenciales de Airtable.
Credenciales de Gmail.
Tokens de acceso de n8n.
Contraseñas.
Las credenciales deben mantenerse únicamente dentro del sistema de credenciales de n8n o mediante variables de entorno.

Estructura del repositorio
proyecto-final-automatizacion-ia/
│
├── README.md
│
├── workflows/
│   ├── flujo_novedades_airtable.json
│   └── flujo_alertas_visionlink.json
│
├── evidencias/
│   ├── evidencia_01.png
│   ├── evidencia_02.png
│   ├── evidencia_03.png
│   ├── evidencia_04.png
│   └── evidencia_05.png
│
└── documentacion/
    └── proyecto_final.pdf
Conclusión
El proyecto demuestra cómo la automatización y la Inteligencia Artificial pueden utilizarse para mejorar un proceso real de gestión de mantenimiento.

La combinación de n8n + Airtable + IA + Gmail + Telegram permite crear un sistema capaz de recibir novedades, interpretarlas, clasificarlas, solicitar validación humana y generar acciones posteriores automáticamente.

La incorporación de Human-in-the-Loop permite mantener supervisión humana sobre las decisiones críticas, logrando un equilibrio entre automatización, eficiencia y control.
