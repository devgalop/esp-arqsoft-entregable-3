# Ejercicio de arquitecturas modulares

## Parte 1: Escenario inicial

### Descripción

FleetCore es el sistema operacional de una empresa de mensajería urbana. Por ahora opera en una sola ciudad, con un equipo pequeño y un solo proceso desplegado. El sistema gestiona tres tipos de actores:

- `Cliente`: solicita un envió indicando origen, destino y tipo de paquete.
- `Repartidor`: recibe la asignación, recoge el paquete y actualiza el estado de la entrega.
- `Operador`: supervisa el estado global y gestiona excepciones cuando una entrega falla.

Las cuatro capacidades del sistema en este momento:

- `Registrar una orden de envío` con origen, destino y cliente.
- `Asignar un repartidor disponible` a la orden — el repartidor debe estar dentro de un radio máximo y tener capacidad disponible.
- `Registrar los cambios de estado` de la orden: recogido, en tránsito, entregado, fallido.
- `Notificar al cliente` cuando el estado cambia.

[Para más detalle, ver el enunciado completo del ejercicio.](https://docs.google.com/document/d/1cuc2QB7oQfDiNET1j6G7cwuxRn0VS1fjITQGu4QtP1E/edit?tab=t.0)

### Diagrama C4 nivel 3

![Diagrama C4 nivel 3](/resources/ArquitecturaCapas.drawio.png)

### Justificación de la arquitectura

La arquitectura modular propuesta para FleetCore se basa en la separación de responsabilidades y la cohesión de los módulos. Cada módulo se encarga de una función específica, lo que facilita el mantenimiento y la escalabilidad del sistema.

- **Capa de Dominio**: En esta capa se encuentran las entidades del dominio, los servicios del dominio y los contratos de los repositorios. Esto permite que la lógica de negocio esté aislada de las preocupaciones técnicas, facilitando su evolución sin afectar otras partes del sistema.

  - `Entidades`: Representan los objetos del dominio, como `Orden`, `Repartidor` y `Cliente`.
  - `Servicios de dominio`: Contiene la lógica especifica del negocio, como la asignación de repartidores y la gestión de estados de las órdenes.
  - `Repositorios`: Define las interfaces para la persistencia de datos, permitiendo que la implementación pueda cambiar sin afectar la lógica de negocio.

- **Capa de Aplicación**: Esta capa actúa como un mediador entre el dominio y el mundo exterior. Contiene los casos de uso que orquestan la lógica del dominio para cumplir con los requisitos del sistema.

  - `Casos de uso`: Implementan la lógica de aplicación, coordinando las entidades y servicios del dominio para realizar tareas específicas, como registrar una orden o asignar un repartidor.
  - `Contratos de entrada`: Define las interfaces que los controladores deben implementar para interactuar con la capa de aplicación.
  - `Contratos de salida`: Define las interfaces que permiten la interacción con las aplicaciones externas, como notificaciones al cliente o integración con sistemas de terceros.
  - `DTOs`: Objetos de transferencia de datos que facilitan la comunicación entre capas, asegurando que solo se exponga la información necesaria.

- **Capa de Infraestructura**: Esta capa se encarga de las implementaciones concretas de los contratos definidos en las capas superiores. Aquí se encuentran los controladores, adaptadores y servicios externos.

  - `Adaptadores de entrada`: Implementan los controladores que reciben las solicitudes de los clientes y las traducen a casos de uso.
  - `Adaptadores de salida`: Implementan las interfaces para interactuar con sistemas externos, como servicios de notificación, geolocalización o bases de datos.

Esta arquitectura modular permite que cada parte del sistema evolucione de manera independiente, facilitando la incorporación de nuevas funcionalidades o la modificación de las existentes sin afectar el resto del sistema. Además, promueve la reutilización de código y la claridad en la organización del proyecto.

### Puertos de salida

- **Puertos de salida**: Estos puertos permiten que el sistema interactúe con servicios externos, como notificaciones al cliente, servicios de geolocalización y bases de datos para la persistencia de datos.

  - `ServicioNotificaciones`: Permite enviar notificaciones al cliente sobre el estado de su orden. Su implementación esta en la capa de infraestructura, pero su contrato se define en la capa de aplicación para mantener el desacoplamiento.
  - `ServicioGeolocalizacion`: Permite obtener la ubicación de los repartidores para asignaciones eficientes. Su implementación esta en la capa de infraestructura, pero su contrato se define en la capa de aplicación para mantener el desacoplamiento.
  - `RepositorioOrdenes`: Permite la persistencia y recuperación de órdenes en la base de datos. Su contrato se define en la capa de dominio, mientras que su implementación se encuentra en la capa de infraestructura.
  - `RepositorioRepartidores`: Permite la persistencia y recuperación de repartidores en la base de datos. Su contrato se define en la capa de dominio, mientras que su implementación se encuentra en la capa de infraestructura.
  - `RepositorioClientes`: Permite la persistencia y recuperación de clientes en la base de datos. Su contrato se define en la capa de dominio, mientras que su implementación se encuentra en la capa de infraestructura.

Al definir claramente estos puertos, se asegura una comunicación desacoplada entre las diferentes capas del sistema, facilitando la mantenibilidad y escalabilidad del mismo. Si en el futuro se desea cambiar la implementación de un servicio externo, como el sistema de notificaciones o geolocalización, solo será necesario modificar el adaptador de salida correspondiente sin afectar la lógica de negocio o los casos de uso.

### Análisis de asignación de repartidores

Si bien la regla de negocio para asignar repartidores depende de la geolocalización y la disponibilidad del repartidor, la implementación de esta lógica se encuentra en la capa de dominio, específicamente en el servicio de dominio `AsignacionRepartidor`. Este servicio se encarga de evaluar los repartidores disponibles y seleccionar el más adecuado para la orden de envío, considerando factores como la proximidad al origen, la capacidad disponible y el historial de entregas. El servicio del dominio hace uso del servicio de geolocalización a través de un puerto de salida para obtener la ubicación de los repartidores y realizar la asignación de manera eficiente. Esta separación permite que la lógica de negocio esté aislada y pueda evolucionar sin afectar otras partes del sistema.

El dominio es capaz de realizar esta asignación sin depender de la infraestructura, ya que el servicio de dominio puede interactuar con el servicio de geolocalización a través del puerto de salida definido en la capa de aplicación. Esto garantiza que la lógica de negocio se mantenga independiente de las implementaciones concretas, facilitando su evolución y mantenimiento a largo plazo.

### Comparativa Clean Architecture vs Hexagonal Architecture

En nuestro planteamiento, los contratos de entrada y salida que se encuentran en la capa de aplicación, pueden ser vistos como los puertos de la arquitectura hexagonal, mientras que los casos de uso y servicios del dominio pueden ser vistos como el núcleo de la arquitectura limpia. La capa de infraestructura, con sus adaptadores de entrada y salida, actúa como los adaptadores en la arquitectura hexagonal, permitiendo la interacción con el mundo exterior sin afectar la lógica de negocio.

---

## Parte 2: Cambian las condiciones del negocio

### Descripción de cambios

FleetCore opera ahora en tres ciudades. El sistema que diseñaron en Clase 1 sigue funcionando,
pero la organización y el sistema han crecido de maneras que generan fricción visible.

El equipo creció de uno a dos:

- `Equipo de Operaciones`: gestiona órdenes, asignación de repartidores y facturación.
- `Equipo de Experiencia`: gestiona rastreo en tiempo real, notificaciones y reporting para clientes corporativos.

El problema es visible desde hace tres meses:

- El `Equipo de Experiencia` necesita **desplegar el módulo de rastreo dos o tres veces por semana** — está iterando sobre el algoritmo de predicción de tiempos de entrega. Cada despliegue del sistema completo requiere coordinación con el `Equipo de Operaciones`, una ventana de mantenimiento de 20 minutos, y ha causado dos incidentes en facturación en el último mes por regresiones no relacionadas con rastreo.
- El **volumen de rastreo creció de manera diferencial**: Los repartidores envían su posición cada 10 segundos. El sistema procesa hoy alrededor de 800 escrituras por minuto solo por actualizaciones de posición, con picos de 2.400 durante las horas de mayor demanda. **El resto del sistema — órdenes, facturación, asignación — genera en total alrededor de 120 escrituras por minuto**. Los dos dominios están creciendo a velocidades completamente distintas.

- Apareció un SLA contractual: Un cliente corporativo firmó un **acuerdo que garantiza actualización de posición con no más de 30 segundos de latencia**. Ese SLA no existía hace seis meses — y ahora tiene consecuencias legales si se incumple.

El estado actual del sistema:

- Toda la lógica vive en un único API Backend con arquitectura limpia y hexagonal. Hay una sola base de datos PostgreSQL con las siguientes tablas principales:

  - ordenes — creadas por Operaciones, leídas por Experiencia para reporting.
  - repartidores — escritas por Operaciones (estado, capacidad), leídas por Experiencia (posición actual, identidad).
  - posiciones_repartidor — escritas exclusivamente por Experiencia cada 10 segundos.
  - eventos_rastreo — escritos exclusivamente por Experiencia con cada cambio de estado
  relevante.
  - facturas — escritas y leídas exclusivamente por Operaciones.
  - notificaciones — escritas por Experiencia, leídas por el servicio de envío externo.

### Pregunta 1

A partir del escenario, identifiquen qué partes del monolito deben convertirse en servicios independientes y cuáles deben permanecer juntas. Para cada decisión de separar, justifiquen usando al menos dos de estas variables: frecuencia de despliegue, presión de escala diferencial, modelo de equipos, requisitos de SLA. Para cada decisión de mantener unido, justifiquen por qué la presión no es suficiente todavía.

- **Respuesta**:

  - Es evidente que el **módulo de georeferenciación** debe convertirse en un servicio independiente, ya que su **escalado es completamente diferente al resto del sistema**. Mientras que el módulo de órdenes, asignación y facturación genera alrededor de 120 escrituras por minuto, el módulo de rastreo genera entre 800 y 2.400 escrituras por minuto dependiendo de la demanda lo cual **representa más de 6 veces la carga del resto del sistema**.

  - El **módulo de georeferenciación** también tiene una **frecuencia de despliegue mucho mayor que el resto del sistema**, esto genera fricción visible entre los equipos, ya que cada despliegue del sistema completo requiere coordinación entre ambos equipos y ha causado incidentes en facturación por regresiones no relacionadas con el módulo de rastreo. Al convertir el módulo de georeferenciación en un servicio independiente, se permite que el equipo de experiencia pueda iterar sobre el algoritmo de predicción de tiempos de entrega sin afectar al equipo de operaciones.

  - El **módulo de órdenes, asignación y facturación** pueden permanecer unidos en un mismo módulo ya que su **presión de escala es similar**, adicionalmente permanecer unidos **permite que el equipo de operaciones pueda gestionar de manera más eficiente las órdenes, asignación y facturación sin tener que preocuparse por la integración con un servicio externo**. Además, el módulo de órdenes, asignación y facturación no tiene una frecuencia de despliegue tan alta como el módulo de georeferenciación y adicionalmente **pertenece a un único equipo**, lo que facilita la coordinación y el mantenimiento.

  - El **módulo de notificaciones y reporting** pueden permanecer unidos en un mismo módulo ya que su **presión de escala es similar**, además, ambos módulos están **relacionados con la experiencia del cliente y pueden ser gestionados por el equipo de experiencia sin necesidad de una integración con un servicio externo**. Además, el módulo de notificaciones y reporting no tiene una frecuencia de despliegue tan alta como el módulo de georeferenciación y adicionalmente **pertenece a un único equipo**, lo que facilita la coordinación y el mantenimiento.

### Pregunta 2

Diseñen el nuevo C4 nivel 2 de FleetCore mostrando los servicios identificados, sus relaciones, y la estrategia de datos para cada uno. Señalen explícitamente qué tablas van con cada servicio y cuáles quedan en zona de decisión pendiente. El diagrama debe mostrar también cómo se comunican los servicios entre sí.

- **Respuesta**:

![Diagrama C4 nivel 2](/resources/escenario_2.drawio.png)

- **Módulo de órdenes, asignación y facturación**: Este módulo tiene acceso a las tablas `ordenes`, `repartidores` y `facturas`.
- **Módulo de notificaciones y reporting**: Este módulo tiene acceso a las tablas `notificaciones` y `eventos_rastreo`.
- **Módulo de georeferenciación**: Este módulo tiene acceso a la tabla `posiciones_repartidor`.

**NOTA**: Una decisión arquitectónica importante es la separación de las bases de datos. En este caso, se decidió que la georeferenciación tenga su propia base de datos para evitar la presión de escala diferencial y permitir una mayor independencia entre los módulos. Adicionalmente, mientras que la base de datos del módulo de órdenes, asignación y facturación puede ser una base de datos relacional como PostgreSQL, la base de datos del módulo de georeferenciación podría ser una base de datos NoSQL optimizada para consultas geoespaciales, como MongoDB. Esta decisión permite que cada módulo utilice la tecnología de base de datos que mejor se adapte a sus necesidades específicas.

### Pregunta 3

Para cada servicio identificado, justifiquen la estrategia de datos usando las cuatro variables de la sesión: quién escribe, frecuencia de cambio del esquema, requisitos de consistencia, y modelo de equipos. ¿En qué escenario (A, B o C) cae cada decisión? Para las tablas que están en zona gris, ¿cuál es el plan de migración y qué señal concreta los llevaría a separar en 12 meses?

- **Respuesta**:

  - **Módulo de órdenes, asignación y facturación**:
    - **Quién escribe**: Este módulo es escrito exclusivamente por el equipo de operaciones.
    - **Frecuencia de cambio del esquema**: La frecuencia de cambio del esquema es baja, ya que las tablas `ordenes`, `repartidores` y `facturas` tienen una estructura relativamente estable.
    - **Requisitos de consistencia**: Este módulo requiere una alta consistencia, especialmente en la tabla de `facturas`, ya que cualquier inconsistencia podría generar problemas legales o financieros.
    - **Modelo de equipos**: Este módulo es gestionado por un único equipo (equipo de operaciones), lo que facilita la coordinación y el mantenimiento.
    - **Escenario**: Este módulo tiene separación de lógica por servicio pero comparte la misma base de datos.

  - **Módulo de notificaciones y reporting**:
    - **Quién escribe**: Este módulo es escrito exclusivamente por el equipo de experiencia.
    - **Frecuencia de cambio del esquema**: La frecuencia de cambio del esquema es moderada, ya que las tablas `notificaciones` y `eventos_rastreo` pueden requerir cambios para adaptarse a nuevas funcionalidades o mejoras en la experiencia del cliente.
    - **Requisitos de consistencia**: Este módulo requiere una consistencia eventual, ya que las notificaciones y eventos de rastreo no necesitan ser procesados en tiempo real para garantizar la funcionalidad del sistema.
    - **Modelo de equipos**: Este módulo es gestionado por un único equipo (equipo de experiencia), lo que facilita la coordinación y el mantenimiento.
    - **Escenario**: Este módulo tiene la lógica de negocio separada por servicio y comparte la misma base de datos.

  - **Módulo de georeferenciación**:
    - **Quién escribe**: Este módulo es escrito exclusivamente por el equipo de experiencia.
    - **Frecuencia de cambio del esquema**: La frecuencia de cambio del esquema es baja, ya que la tabla `posiciones_repartidor` tiene una estructura relativamente estable.
    - **Requisitos de consistencia**: Este módulo requiere una consistencia eventual, ya que las actualizaciones de posición no necesitan ser procesadas en tiempo real para garantizar la funcionalidad del sistema, siempre y cuando se cumpla el SLA de actualización de posición con no más de 30 segundos de latencia.
    - **Modelo de equipos**: Este módulo es gestionado por un único equipo (equipo de experiencia), lo que facilita la coordinación y el mantenimiento.
    - **Escenario**: Este módulo tiene su propia base de datos.

  - **Plan de migración para tablas en zona gris**:
    - `repartidores`: Operaciones escribe (estado, capacidad) y Experiencia lee (posición, identidad). Hoy vive con Operaciones porque la escritura es su responsabilidad principal. El acceso de Experiencia se expone mediante REST, no acceso directo a la tabla. **Señal concreta para separar en 12 meses**: Si Experiencia necesita índices geoespaciales sobre repartidores que entren en conflicto con los índices transaccionales de Operaciones, se extrae la información de posición al servicio de Georeferenciación con sincronización por eventos.

    - `eventos_rastreo`: Experiencia escribe con cada cambio de estado relevante, pero Operaciones también los lee para su reporting. Hoy vive con Experiencia porque la escritura es su responsabilidad principal. El acceso de Operaciones se expone mediante REST, no acceso directo a la tabla. **Señal concreta para separar en 12 meses**: Si Operaciones hace joins regulares entre eventos_rastreo y ordenes para reportes de gestión, se replica la tabla hacia el schema de Operaciones con consistencia eventual, o se crea un servicio de reporting independiente que consolide ambas fuentes.

### Pregunta 4

Tomen la decisión más difícil de su diseño — la tabla o la relación entre servicios donde hubo más debate — y justifíquenla usando los números del Bloque 3: latencia, costo operacional y disponibilidad compuesta. ¿El precio de distribuir está justificado por la presión operacional del escenario? Si el SLA de rastreo baja a 10 segundos el próximo trimestre, ¿cambia alguna decisión de su diseño? ¿Cuál y por qué?

- **Respuesta**:

  La decisión más difícil fue la separación de la tabla `posiciones_repartidor` en un servicio independiente con su propia base de datos. Esta decisión se justifica por la presión operacional que representa el módulo de georeferenciación, ya que **genera una carga hasta 6 veces mayor** que el resto del sistema. Al tener su propia base de datos, se puede escalar de manera independiente y optimizar para consultas geoespaciales, lo que mejora la latencia y disponibilidad del servicio.

  Convertir la base de datos de georeferenciación en una tecnología **NoSQL optimizada para consultas geoespaciales**, como MongoDB, puede **mejorar significativamente la latencia de las consultas de ubicación**, lo que es crucial para cumplir con el SLA de actualización de posición con no más de 30 segundos de latencia. Además, esta separación permite que el equipo de experiencia pueda iterar sobre el algoritmo de predicción de tiempos de entrega sin afectar al equipo de operaciones, lo que mejora la eficiencia y reduce el riesgo de incidentes en facturación.

  Si el SLA de rastreo baja a 10 segundos el próximo trimestre, esta decisión se vuelve aún más justificada, ya que la presión operacional sobre el módulo de georeferenciación aumentaría aún más. En este caso, sería crucial mantener la separación para garantizar que el módulo de georeferenciación pueda cumplir con el nuevo SLA sin afectar al resto del sistema. Además, se podrían implementar optimizaciones adicionales en la base de datos y en la lógica del servicio para asegurar que se cumpla el nuevo requisito de latencia.

---

## Parte 3: Las áreas del negocio cambian y con ellas los requisitos

### Descripción de cambios

El sistema de dos servicios que diseñaron en Clase 2 sigue en producción, pero han aparecido tres presiones nuevas que hacen insostenible el estado actual:

Presión 1 — El equipo de Operaciones se dividió en tres.
El equipo creció y se especializó en tres sub-equipos: Órdenes, Asignación y Facturación. Cada sub-equipo trabaja en sprints independientes con prioridades distintas, pero comparten el mismo servicio y el mismo pipeline de CI/CD. Un bugfix en Facturación requiere coordinar el release con los otros dos sub-equipos, aunque el cambio sea de dos líneas. En el último mes hubo tres releases bloqueados por más de una semana por esta coordinación.

Presión 2 — El equipo de Asignación necesita ML.
El sub-equipo de Asignación va a reemplazar el algoritmo de asignación actual por un modelo de ML entrenado en datos históricos. El modelo requiere Python 3.11 con librerías específicas de ML (PyTorch, scikit-learn) que crean conflictos de versiones con otras dependencias del servicio de Operaciones. Llevan 6 semanas bloqueados sin poder introducir ninguna dependencia nueva.

Presión 3 — Facturación necesita aislamiento PCI-DSS.
Un auditor de seguridad determinó que para cumplir PCI-DSS, los datos de pago y facturación deben estar completamente aislados: base de datos propia con acceso restringido, audit trail independiente, y controles de acceso que el código compartido actual hace imposible de garantizar. El plazo del auditor es de 90 días.

Más el incidente abierto de Clase 2:
La tabla repartidores sigue siendo un punto de fricción. La semana pasada el equipo de Rastreo desplegó una migración de esquema en su ventana de mantenimiento que, por un error de coordinación, afectó la tabla repartidores y dejó el servicio de Operaciones en un estado inconsistente durante 4 horas. El post-mortem señala que el acoplamiento de esquema entre los dos equipos es insostenible.

[ver ejercicio completo](https://docs.google.com/document/d/1H5W-rL4uV3mZjuzwYE7e-RlgXqVi4qWJss860aEBBfs/edit?tab=t.0)

### Pregunta 1

¿Qué partes de Operaciones se convierten en microservicios independientes? Para cada microservicio propuesto justifiquen usando: Conway's Law (¿tiene equipo propio?), requisito de escala diferencial (¿escala de manera distinta al resto?), o requisito de aislamiento técnico o regulatorio. ¿Qué permanece junto y por qué?

- **Respuesta**:

  - El **servicio de facturación** debe convertirse en un microservicio independiente debido a los requisitos de aislamiento técnico y regulatorio impuestos por PCI-DSS. Al tener su propia base de datos con acceso restringido, audit trail independiente y controles de acceso específicos, se garantiza que los datos de pago y facturación estén completamente aislados del resto del sistema, lo que es crucial para cumplir con las normativas de seguridad.

  - El **servicio de asignación** también debe convertirse en un microservicio independiente debido a los requisitos técnicos relacionados con el uso de Python 3.11 y las librerías específicas de ML (PyTorch, scikit-learn). Al tener su propio servicio, el equipo de asignación puede gestionar sus dependencias sin generar conflictos con otras partes del sistema, lo que mejora la eficiencia y reduce el riesgo de bloqueos.

  - El **servicio de órdenes**  se convierte en un microservicio independiente debido a que el equipo de órdenes tiene prioridades distintas a los otros sub-equipos y necesita poder desplegar de manera independiente sin afectar a los otros módulos.

  - El **servicio de notificaciones** se convierte en un microservicio independiente debido a que este actúa como un servicio transversal que es activado por los demás servicios a través de eventos, lo que justifica su separación para evitar acoplamientos innecesarios entre los módulos.

  - El **servicio de reporting** se convierte en un microservicio independiente debido a que este también actúa como un servicio transversal que necesita de la interacción con otros servicios, lo que justifica su separación para evitar acoplamientos innecesarios entre los módulos.

  - El **servicio de georeferenciación** es un microservicio que tiene sus propias necesidades de escalado y despliegue, por lo que se mantiene como un microservicio independiente.

### Pregunta 2

Diseñen el nuevo C4 nivel 2 de FleetCore mostrando todos los microservicios identificados, el API Gateway, las bases de datos propias de cada servicio, y el modelo de comunicación entre ellos. Indiquen explícitamente qué llamadas son síncronas y cuáles son asíncronas, y por qué.

- **Respuesta**:
![Diagrama C4 nivel 2](/resources/escenario_3.drawio.png)

  - **Llamados asíncronos**:
    - Todos los llamados para notificaciones son asíncronos, gestionados a través de eventos, ya que las notificaciones no necesitan ser procesadas en tiempo real para garantizar la funcionalidad del sistema.
  - **Llamados síncronos**:
    - El proceso de creación de una orden debe ser síncrono, orquestado por un medidor de saga, para garantizar que se registren correctamente la orden, se asigne un repartidor disponible y se inicialice la factura. Esto es crucial para mantener la consistencia del sistema y asegurar que todas las partes involucradas en el proceso de creación de una orden estén coordinadas.
    - El proceso de asignación de repartidores también debe ser síncrono, ya que es una parte crítica del flujo de trabajo y requiere una respuesta inmediata para garantizar una experiencia de usuario fluida.
    - El proceso de facturación debe ser síncrono para garantizar que se generen correctamente las facturas y se cumplan los requisitos de PCI-DSS, lo que es crucial para la seguridad y el cumplimiento normativo.
    - La generación de reportes también debe ser síncrona, ya que los reportes pueden requerir datos en tiempo real para proporcionar información precisa y relevante a los usuarios. Esto implica que el mediador de saga debe coordinar la recopilación de datos de los diferentes servicios para generar los reportes de manera eficiente y oportuna.

### Pregunta 3

Cuando un cliente crea un pedido, el sistema debe: (1) registrar la orden, (2) asignar un repartidor disponible, y (3) inicializar la factura. Diseñen la saga para esta operación. Muestren la ruta feliz y todas las rutas de compensación. ¿Eligen orquestación o coreografía? Justifiquen. ¿Qué pasa si el paso de asignación falla después de que la orden ya fue creada?


### Pregunta 4

El incidente de 4 horas cerró el debate de Clase 2. Con cuatro equipos autónomos y cuatro servicios con bases de datos propias, diseñen la solución definitiva para la tabla repartidores. ¿Justifica ahora la sincronización por eventos? Argumenten usando: Conway's Law aplicada al estado actual del equipo, las cuatro variables de Clase 2, y el costo operacional del incidente. ¿Qué esquema de evento proponen y quién lo emite?