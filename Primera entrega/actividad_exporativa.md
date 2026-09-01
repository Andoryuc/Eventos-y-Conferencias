# Actividad de exploración

Plataforma de eventos y conferencias  
Modelo entidad-relación (notación Chen)

Este archivo describe el diagrama ER de la primera entrega.

---

## 1. Exploración del problema

Se modeló una plataforma para administrar eventos (congresos, conferencias y talleres).

Hechos que se sacaron del problema y que se ven en el dibujo:

- El **Usuario** se registra. Tiene un rol (organizador, ponente o asistente).
- El usuario **Pertenece_a** una **Organizacion**.
- El usuario **Crea** eventos y la organización **publica** eventos.
- El **Evento** puede ser presencial, virtual o híbrido.
- El evento **realiza** en un **Lugar**. El lugar **Contiene** **Sala**.
- El evento se **Clasifica** en **Categoria**.
- El evento **Programa** **Sesion**. La sesión **ocurre_en** una sala.
- El **Ponente** **es** un usuario e **Imparte** sesiones.
- El evento **Ofrece** **Tipo_entrada**.
- El usuario **Realiza** una **Inscripcion**. Esa inscripción **corresponde_a** un evento y **elige** un tipo de entrada.
- La inscripción **genera** un **Pago** si la entrada no es gratis.

---

## 2. Notación del diagrama

| Símbolo | Significado |
|---|---|
| Rectángulo | Entidad |
| Óvalo | Atributo simple |
| Óvalo padre con hijos | Atributo compuesto |
| Óvalo de doble línea | Atributo multivaluado |
| Nombre subrayado | Identificador principal |
| Rombo | Relación |
| 1, N, M | Cardinalidad en cada extremo |

Las claves foráneas no se dibujan. En Chen las indica la relación.

---

## 3. Entidades e identificadores

Hay 11 entidades. El identificador de cada una está subrayado.

| Entidad | Identificador | Qué es |
|---|---|---|
| Usuario | id_usuario | Persona registrada |
| Organizacion | id_organizacion | Grupo o empresa que publica el evento |
| Evento | id_evento | Congreso, conferencia o taller |
| Categoria | id_categoria | Tema del evento |
| Lugar | id_lugar | Sede física |
| Sala | id_sala | Auditorio o salón |
| Sesion | id_sesion | Charla o taller de la agenda |
| Ponente | id_ponente | Quien imparte la sesión |
| Tipo_entrada | id_tipo_entrada | Tarifa (general, estudiante, gratis…) |
| Inscripcion | id_inscripcion | Registro de un usuario a un evento |
| Pago | id_pago | Cobro de la inscripción |

Distribución en el dibujo:

- Izquierda: Usuario
- Arriba: Organizacion
- Centro: Evento
- Arriba derecha: Categoria, Lugar, Sala
- Derecha: Sesion
- Abajo: Ponente
- Abajo centro: Tipo_entrada, Inscripcion, Pago

---

## 4. Atributos de cada entidad

### Usuario
- id_usuario (identificador)
- nombre_completo (compuesto) → nombre, apellido
- email
- contraseña
- telefono (multivaluado)
- rol

### Organizacion
- id_organizacion (identificador)
- nombre
- email
- telefono

### Evento
- id_evento (identificador)
- titulo
- descripcion
- modalidad
- cupo
- periodo (compuesto) → fecha_inicio, fecha_fin

### Categoria
- id_categoria (identificador)
- nombre

### Lugar
- id_lugar (identificador)
- nombre
- direccion (compuesto) → calle, ciudad, pais

### Sala
- id_sala (identificador)
- nombre
- capacidad

### Sesion
- id_sesion (identificador)
- titulo
- tipo
- fecha_hora_inicio
- fecha_hora_fin

### Ponente
- id_ponente (identificador)
- especialidad (multivaluado)
- idioma (multivaluado)

### Tipo_entrada
- id_tipo_entrada (identificador)
- nombre
- precio

### Inscripcion
- id_inscripcion (identificador)
- fecha
- estado

### Pago
- id_pago (identificador)
- monto
- metodo
- estado

---

## 5. Compuestos y multivaluados

**Compuestos**

| Entidad | Padre | Hijos |
|---|---|---|
| Usuario | nombre_completo | nombre, apellido |
| Lugar | direccion | calle, ciudad, pais |
| Evento | periodo | fecha_inicio, fecha_fin |

**Multivaluados (óvalo de doble línea)**

| Entidad | Atributo | Por qué |
|---|---|---|
| Usuario | telefono | puede tener más de un número |
| Ponente | especialidad | puede manejar más de un tema |
| Ponente | idioma | puede hablar más de un idioma |

Categoria no va como óvalo doble: se resolvió con la relación Clasifica.  
Los ponentes de una sesión tampoco: se resolvió con Imparte.

---

## 6. Relaciones (nombre, tipo y cardinalidad)

Cardinalidades tomadas de los números 1, N y M del diagrama.

| Rombo | De | Card. | A | Card. | Tipo | Lectura |
|---|---|---|---|---|---|---|
| Pertenece_a | Usuario | N | Organizacion | 1 | 1:N | Varios usuarios pertenecen a una organización |
| Crea | Usuario | 1 | Evento | N | 1:N | Un usuario crea varios eventos |
| publica | Organizacion | 1 | Evento | N | 1:N | Una organización publica varios eventos |
| realiza | Evento | 1 | Lugar | N | 1:N | Un evento se realiza en uno o varios lugares |
| Contiene | Lugar | 1 | Sala | N | 1:N | Un lugar contiene varias salas |
| Clasifica | Evento | M | Categoria | N | N:M | Un evento tiene varias categorías y una categoría agrupa varios eventos |
| Programa | Evento | 1 | Sesion | N | 1:N | Un evento programa varias sesiones |
| ocurre_en | Sesion | N | Sala | 1 | N:1 | Varias sesiones ocurren en una sala |
| Ofrece | Evento | 1 | Tipo_entrada | N | 1:N | Un evento ofrece varios tipos de entrada |
| Realiza | Usuario | 1 | Inscripcion | N | 1:N | Un usuario realiza varias inscripciones |
| corresponde_a | Inscripcion | N | Evento | 1 | N:1 | Varias inscripciones corresponden a un evento |
| elige | Inscripcion | N | Tipo_entrada | 1 | N:1 | Varias inscripciones eligen un tipo de entrada |
| genera | Inscripcion | 1 | Pago | 1 | 1:1 | Una inscripción genera un pago |
| es | Usuario | 1 | Ponente | 1 | 1:1 | Un usuario es un ponente |
| Imparte | Ponente | N | Sesion | M | N:M | Varios ponentes imparten varias sesiones |

Resumen de tipos de relación:

- **1:1:** es, genera
- **1:N:** Pertenece_a, Crea, publica, realiza, Contiene, Programa, ocurre_en, Ofrece, Realiza, corresponde_a, elige
- **N:M:** Clasifica, Imparte

Usuario — Evento (inscribirse) es N:M en el negocio. En el diagrama se representó con la entidad Inscripcion porque esa relación tiene fecha y estado.

---

## 7. Reglas que se ven en el modelo

- Si el evento es virtual, realiza y ocurre_en pueden no usarse.
- Una Sesion pertenece a un solo Evento (Programa 1:N).
- Si el precio del Tipo_entrada es 0, Pago puede no existir. genera es 1:1.
- Un mismo Usuario puede organizar o asistir (atributo rol). Si además habla, se crea Ponente y se une con es.
- Clasifica es N:M para no convertir la categoría en atributo doble de Evento.
- Imparte es N:M para los paneles (varios ponentes en una sesión).

---

## 8. Conclusión

Con lo visto en clase (entidad, identificador, atributo simple, compuesto, multivaluado, relación y cardinalidad) el diagrama cubre el ciclo del evento: quién lo publica, dónde se hace, cómo se arma la agenda, quién habla y cómo se inscribe y paga.

No se modelaron certificado, materiales ni valoraciones. Se pueden agregar en otra entrega.