# Ejercicio de arquitecturas modulares.

## Descripción

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

## Diagrama C4 nivel 3

![Diagrama C4 nivel 3](/resources/ArquitecturaCapas.drawio.png)

## Justificación de la arquitectura

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

## Puertos de salida

- **Puertos de salida**: Estos puertos permiten que el sistema interactúe con servicios externos, como notificaciones al cliente, servicios de geolocalización y bases de datos para la persistencia de datos.

  - `ServicioNotificaciones`: Permite enviar notificaciones al cliente sobre el estado de su orden. Su implementación esta en la capa de infraestructura, pero su contrato se define en la capa de aplicación para mantener el desacoplamiento.
  - `ServicioGeolocalizacion`: Permite obtener la ubicación de los repartidores para asignaciones eficientes. Su implementación esta en la capa de infraestructura, pero su contrato se define en la capa de aplicación para mantener el desacoplamiento.
  - `RepositorioOrdenes`: Permite la persistencia y recuperación de órdenes en la base de datos. Su contrato se define en la capa de dominio, mientras que su implementación se encuentra en la capa de infraestructura.
  - `RepositorioRepartidores`: Permite la persistencia y recuperación de repartidores en la base de datos. Su contrato se define en la capa de dominio, mientras que su implementación se encuentra en la capa de infraestructura.
  - `RepositorioClientes`: Permite la persistencia y recuperación de clientes en la base de datos. Su contrato se define en la capa de dominio, mientras que su implementación se encuentra en la capa de infraestructura.

Al definir claramente estos puertos, se asegura una comunicación desacoplada entre las diferentes capas del sistema, facilitando la mantenibilidad y escalabilidad del mismo. Si en el futuro se desea cambiar la implementación de un servicio externo, como el sistema de notificaciones o geolocalización, solo será necesario modificar el adaptador de salida correspondiente sin afectar la lógica de negocio o los casos de uso.

## Análisis de asignación de repartidores

Si bien la regla de negocio para asignar repartidores depende de la geolocalización y la disponibilidad del repartidor, la implementación de esta lógica se encuentra en la capa de dominio, específicamente en el servicio de dominio `AsignacionRepartidor`. Este servicio se encarga de evaluar los repartidores disponibles y seleccionar el más adecuado para la orden de envío, considerando factores como la proximidad al origen, la capacidad disponible y el historial de entregas. El servicio del dominio hace uso del servicio de geolocalización a través de un puerto de salida para obtener la ubicación de los repartidores y realizar la asignación de manera eficiente. Esta separación permite que la lógica de negocio esté aislada y pueda evolucionar sin afectar otras partes del sistema.

El dominio es capaz de realizar esta asignación sin depender de la infraestructura, ya que el servicio de dominio puede interactuar con el servicio de geolocalización a través del puerto de salida definido en la capa de aplicación. Esto garantiza que la lógica de negocio se mantenga independiente de las implementaciones concretas, facilitando su evolución y mantenimiento a largo plazo.

## Comparativa Clean Architecture vs Hexagonal Architecture

En nuestro planteamiento, los contratos de entrada y salida que se encuentran en la capa de aplicación, pueden ser vistos como los puertos de la arquitectura hexagonal, mientras que los casos de uso y servicios del dominio pueden ser vistos como el núcleo de la arquitectura limpia. La capa de infraestructura, con sus adaptadores de entrada y salida, actúa como los adaptadores en la arquitectura hexagonal, permitiendo la interacción con el mundo exterior sin afectar la lógica de negocio.