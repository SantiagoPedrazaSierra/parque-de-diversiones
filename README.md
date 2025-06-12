# Diseño de Base de Datos NoSQL para el Parque Nicole

## Integrantes
1. Santiago Pedraza
2. Nicolas Espitia


## Introducción
Este proyecto propone un diseño de base de datos NoSQL para el Parque Nicole utilizando MongoDB. El enfoque combina incrustación (embedding) y referencia (referencing) según la naturaleza de cada relación, optimizando para operaciones de lectura frecuentes y manteniendo la consistencia de datos.

## Análisis de Entidades y Propuesta de Modelo de Datos

### 1. Atracciones
**Colección:** `atracciones`

**Atributos:**
- `_id`: ObjectId (identificador único)
- `nombre`: String
- `tipo`: String (montaña_rusa, carrusel, show, etc.)
- `descripcion`: String
- `altura_minima`: Number (en cm)
- `capacidad`: Number
- `estado`: String (operativa, en_mantenimiento)
- `tiempo_espera_promedio`: Number (en minutos)
- `zona_id`: ObjectId (referencia a zona)

**Relaciones:**
- **Zona**: Uno a Muchos (Una atracción pertenece a una zona, una zona tiene muchas atracciones)
  - Estrategia: Referencia (`zona_id` en atracción)
  - Justificación: Las atracciones pueden cambiar de zona ocasionalmente, y las zonas pueden tener muchas atracciones. Referenciar es más flexible.

**Ejemplo JSON:**
```json
{
  "_id": ObjectId("665a1d3e8f7b1a2b3c4d5e6f"),
  "nombre": "Montaña Rusa Extrema",
  "tipo": "montaña_rusa",
  "descripcion": "La montaña rusa más alta del parque",
  "altura_minima": 140,
  "capacidad": 24,
  "estado": "operativa",
  "tiempo_espera_promedio": 30,
  "zona_id": ObjectId("665a1d3e8f7b1a2b3c4d5e70")
}
```

### 2. Zonas del Parque
**Colección:** `zonas`

**Atributos:**
- `_id`: ObjectId
- `nombre`: String
- `descripcion`: String
- `atracciones`: Array (de ObjectIds)

**Relaciones:**
- **Atracciones**: Uno a Muchos
  - Estrategia: Referencia (array de `_id`s de atracciones)
  - Justificación: Permite fácil consulta de todas las atracciones de una zona sin duplicar datos.

**Ejemplo JSON:**
```json
{
  "_id": ObjectId("665a1d3e8f7b1a2b3c4d5e70"),
  "nombre": "Zona Extrema",
  "descripcion": "Zona con atracciones para amantes de la adrenalina",
  "atracciones": [
    ObjectId("665a1d3e8f7b1a2b3c4d5e6f"),
    ObjectId("665a1d3e8f7b1a2b3c4d5e71")
  ]
}
```

### 3. Visitantes
**Colección:** `visitantes`

**Atributos:**
- `_id`: ObjectId
- `nombre`: String
- `apellido`: String
- `fecha_nacimiento`: Date
- `email`: String
- `historial_visitas`: Array (de fechas)
- `tickets`: Array (de ObjectIds)

**Relaciones:**
- **Tickets**: Uno a Muchos
  - Estrategia: Referencia (array de `_id`s de tickets)
  - Justificación: Un visitante puede tener muchos tickets a lo largo del tiempo, y los tickets tienen información independiente importante.

**Ejemplo JSON:**
```json
{
  "_id": ObjectId("665a1d3e8f7b1a2b3c4d5e72"),
  "nombre": "Juan",
  "apellido": "Pérez",
  "fecha_nacimiento": ISODate("1990-05-15"),
  "email": "juan.perez@example.com",
  "historial_visitas": [
    ISODate("2023-05-10"),
    ISODate("2023-08-15")
  ],
  "tickets": [
    ObjectId("665a1d3e8f7b1a2b3c4d5e73"),
    ObjectId("665a1d3e8f7b1a2b3c4d5e74")
  ]
}
```

### 4. Tickets
**Colección:** `tickets`

**Atributos:**
- `_id`: ObjectId
- `tipo`: String (diario, anual, VIP)
- `precio`: Number
- `fecha_compra`: Date
- `fecha_validez`: Date
- `visitante_id`: ObjectId

**Relaciones:**
- **Visitante**: Muchos a Uno
  - Estrategia: Referencia (`visitante_id` en ticket)
  - Justificación: Los tickets son entidades independientes con información valiosa por sí mismos.

**Ejemplo JSON:**
```json
{
  "_id": ObjectId("665a1d3e8f7b1a2b3c4d5e73"),
  "tipo": "diario",
  "precio": 50,
  "fecha_compra": ISODate("2023-05-10"),
  "fecha_validez": ISODate("2023-05-10"),
  "visitante_id": ObjectId("665a1d3e8f7b1a2b3c4d5e72")
}
```

### 5. Empleados
**Colección:** `empleados`

**Atributos:**
- `_id`: ObjectId
- `nombre`: String
- `apellido`: String
- `cargo`: String
- `horario`: String
- `atracciones_asignadas`: Array (de ObjectIds)
- `eventos_asignados`: Array (de ObjectIds)

**Relaciones:**
- **Atracciones**: Muchos a Muchos
  - Estrategia: Referencia (array de `_id`s)
  - Justificación: Un empleado puede trabajar en múltiples atracciones y viceversa.
  
- **Eventos**: Muchos a Muchos
  - Estrategia: Referencia (array de `_id`s)
  - Justificación: Similar a atracciones, relación muchos a muchos.

**Ejemplo JSON:**
```json
{
  "_id": ObjectId("665a1d3e8f7b1a2b3c4d5e75"),
  "nombre": "María",
  "apellido": "Gómez",
  "cargo": "Operador de Atracciones",
  "horario": "L-V 9am-6pm",
  "atracciones_asignadas": [
    ObjectId("665a1d3e8f7b1a2b3c4d5e6f"),
    ObjectId("665a1d3e8f7b1a2b3c4d5e71")
  ],
  "eventos_asignados": [
    ObjectId("665a1d3e8f7b1a2b3c4d5e77")
  ]
}
```

### 6. Eventos/Espectáculos
**Colección:** `eventos`

**Atributos:**
- `_id`: ObjectId
- `nombre`: String
- `descripcion`: String
- `horario`: String
- `ubicacion`: String (puede ser zona o atracción específica)
- `empleados`: Array (de ObjectIds)

**Relaciones:**
- **Empleados**: Muchos a Muchos
  - Estrategia: Referencia (array de `_id`s)
  - Justificación: Múltiples empleados pueden trabajar en un evento.

**Ejemplo JSON:**
```json
{
  "_id": ObjectId("665a1d3e8f7b1a2b3c4d5e77"),
  "nombre": "Espectáculo Nocturno",
  "descripcion": "Show de luces y fuegos artificiales",
  "horario": "20:00",
  "ubicacion": "Plaza Central",
  "empleados": [
    ObjectId("665a1d3e8f7b1a2b3c4d5e75"),
    ObjectId("665a1d3e8f7b1a2b3c4d5e76")
  ]
}
```

### 7. Mantenimiento
**Colección:** `mantenimientos`

**Atributos:**
- `_id`: ObjectId
- `fecha`: Date
- `descripcion`: String
- `costo`: Number
- `atraccion_id`: ObjectId
- `empleados`: Array (de ObjectIds)
- `estado_actual`: String (completado, en_progreso, programado)

**Relaciones:**
- **Atracción**: Muchos a Uno
  - Estrategia: Referencia (`atraccion_id`)
  - Justificación: Un mantenimiento pertenece a una atracción, pero una atracción puede tener muchos registros de mantenimiento.
  
- **Empleados**: Muchos a Muchos
  - Estrategia: Referencia (array de `_id`s)
  - Justificación: Varios empleados pueden participar en un mantenimiento.

**Ejemplo JSON:**
```json
{
  "_id": ObjectId("665a1d3e8f7b1a2b3c4d5e78"),
  "fecha": ISODate("2023-06-01"),
  "descripcion": "Revisión de seguridad anual",
  "costo": 1200,
  "atraccion_id": ObjectId("665a1d3e8f7b1a2b3c4d5e6f"),
  "empleados": [
    ObjectId("665a1d3e8f7b1a2b3c4d5e76")
  ],
  "estado_actual": "completado"
}
```

## Conclusiones y Desafíos

### Decisiones Clave:
1. **Incrustación vs Referencia**: Optamos por referencias en la mayoría de los casos debido a las relaciones complejas (muchos-a-muchos) y para evitar duplicación de datos.
2. **Normalización**: Mantuvimos un balance entre normalización y desempeño, evitando documentos excesivamente anidados.
3. **Consultas Frecuentes**: Diseñamos el esquema para optimizar las consultas más comunes (ej: atracciones por zona, tickets por visitante).

### Desafíos Encontrados:
1. **Relaciones Muchos-a-Muchos**: Como en empleados-atracciones y empleados-eventos, requirieron arrays de referencias en ambos lados o en uno según el caso de uso principal.
2. **Historial de Visitas**: Decidimos almacenar solo fechas en el visitante para no hacer el documento muy grande, pero podríamos considerar una colección separada si se necesita más detalle.
3. **Consistencia**: En un sistema distribuido, las referencias requieren aplicación de reglas de negocio para mantener consistencia.

### Mejoras Futuras:
1. Índices para campos de búsqueda frecuente (nombre de atracciones, email de visitantes).
2. Considerar fragmentación (sharding) para colecciones grandes como visitantes o tickets.
3. Implementar transacciones para operaciones complejas que afecten múltiples colecciones.

Este diseño proporciona una base sólida y flexible para el sistema de gestión del Parque Nicole, permitiendo escalabilidad y adaptabilidad a futuros requerimientos.