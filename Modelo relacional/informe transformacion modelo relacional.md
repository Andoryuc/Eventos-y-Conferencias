# Informe de transformación del modelo entidad-relación al modelo relacional

**Proyecto:** Sistema de Gestión de Eventos — Modelo normalizado  
**Motor destino:** PostgreSQL  
**Fuente conceptual:** diagrama entidad-relación (notación de Chen)  
**Fuente lógica:** esquema DBML derivado del diagrama  
**Alcance de normalización declarado:** hasta quinta forma normal (5FN), bajo las dependencias funcionales y multivaluadas identificadas en el modelo conceptual

---

## 1. Objetivo

Documentar el paso del modelo entidad-relación al modelo relacional del sistema de gestión de eventos, dejar explícita la clasificación de cada atributo como clave primaria (PK), clave foránea (FK), clave única (UK) o atributo no clave, y justificar en qué forma normal queda cada relación y el esquema completo.

El diccionario de datos de cada tabla usa tres columnas:

| Atributo | Tipo de dato | Clave (PK / FK / UK) |
|---|---|---|

---

## 2. Lectura del modelo conceptual

### 2.1 Entidades y atributos

| Entidad | Identificador | Atributos simples | Atributos derivados | Atributos tratados como multivaluados en el modelo lógico |
|---|---|---|---|---|
| Organizacion | `id_organizacion` | nombre, email, telefono | — | — |
| Usuario | `id_usuario` | apellido, nombre, email, contraseña, rol | `nombre_completo` (nombre + apellido) | `telefono` (se separó para cumplir 1FN) |
| Ponente | `id_ponente` | — | — | especialidad, idioma |
| Categoria | `id_categoria` | nombre | — | — |
| Lugar | `id_lugar` | nombre, calle, ciudad, pais | `direccion` (calle + ciudad + pais) | — |
| Sala | `id_sala` | nombre, capacidad | — | — |
| Evento | `id_evento` | titulo, modalidad, descripcion, fecha_inicio, fecha_fin | `periodo` (fecha_fin − fecha_inicio) | — |
| Tipo_entrada | `id_tipo_entrada` | nombre, precio | — | — |
| Inscripcion | `id_inscripcion` | estado, fecha | — | — |
| Pago | `id_pago` | monto, estado, metodo | — | — |
| Sesion | `id_sesion` | titulo, tipo, fecha_hora_inicio, fecha_hora_fin | — | — |

En el diagrama, `fecha_hora_inicio` y `fecha_hora_fin` aparecen ligados a Evento. En el modelo relacional se trasladaron a Sesion, porque el horario real de celebración corresponde a cada sesión del programa y no al evento completo. El evento conserva solo el rango de fechas (`fecha_inicio`, `fecha_fin`). `cupo` aparece en el diagrama junto a la relación Ofrece; en el esquema lógico quedó como atributo de Evento (cupo global). Esa decisión se discute en la sección 6.

### 2.2 Relaciones y cardinalidades

| Relación | Participantes | Cardinalidad en el ER | Atributos de la relación |
|---|---|---|---|
| Pertenece_a | Usuario — Organizacion | N : 1 | — |
| Crea | Usuario — Evento | 1 : N | — |
| publica | Organizacion — Evento | 1 : N | — |
| Clasifica | Categoria — Evento | N : M | — |
| realiza | Lugar — Evento | N : N en el diagrama; implementada N : 1 en el DBML | — |
| Contiene | Lugar — Sala | 1 : N | — |
| Ofrece | Evento — Tipo_entrada | 1 : N | cupo |
| elige | Tipo_entrada — Inscripcion | 1 : N | — |
| Realiza | Usuario — Inscripcion | 1 : N | — |
| corresponde_a | Inscripcion — Evento | N : 1 | — |
| genera | Inscripcion — Pago | 1 : 1 | — |
| es | Usuario — Ponente | 1 : 1 (especialización) | — |
| Programa | Evento — Sesion | 1 : N | — |
| ocurre_en | Sesion — Sala | N : 1 | — |
| Imparte | Ponente — Sesion | N : M | — |

`Ponente` es especialización de `Usuario` (relación *es*). No todo usuario es ponente; todo ponente es exactamente un usuario.

---

## 3. Reglas de transformación aplicadas

1. **Entidad fuerte.** Cada entidad fuerte se convierte en una tabla. Su identificador subrayado pasa a ser PK, con `int` autoincremental.
2. **Atributo simple.** Se convierte en columna con un dominio acorde al significado (texto, fecha, decimal, entero).
3. **Atributo derivado.** No se almacena. `nombre_completo`, `direccion` y `periodo` se obtienen por consulta. Así se evita redundancia y anomalías de actualización.
4. **Atributo multivaluado.** Se extrae a una tabla propia cuya PK es la combinación (FK de la entidad, valor). Caso aplicado: teléfono de usuario, especialidad e idioma de ponente.
5. **Relación 1 : N.** La PK del lado 1 se copia como FK en el lado N. No se crea tabla intermedia.
6. **Relación 1 : 1.** Se resuelve con FK única en el lado opcional. `ponente.id_usuario` es FK y UK hacia `usuario`.
7. **Relación N : M.** Se crea una tabla asociativa. Su PK es la pareja de FK. Caso aplicado: `clasifica` e `imparte`.
8. **Especialización.** Se usa una tabla de subtipo con PK propia y FK única al supertipo (estrategia de tablas separadas). Conserva `id_ponente` porque el diagrama lo muestra como identificador propio, y enlaza 1 : 1 con `usuario`.
9. **Atributo de relación.** Si la relación es 1 : N, el atributo viaja a la tabla del lado N. `cupo` debería vivir en `tipo_entrada` si es cupo por tipo de entrada; el DBML lo dejó en `evento`. Se señala como decisión a revisar.
10. **Integridad referencial.** Toda FK declara la tabla y columna referenciadas. Las PK compuestas de las tablas asociativas impiden duplicar el mismo par.

---

## 4. Diccionario relacional

Convención de la tercera columna:

- **PK**: clave primaria.
- **FK**: clave foránea. Se indica la tabla y columna referenciadas.
- **UK**: restricción de unicidad (clave candidata o lado 1 de una 1 : 1).
- **—**: atributo no clave.

### 4.1 `organizacion`

Organización que publica eventos. Procede de la entidad Organizacion.

| Atributo | Tipo de dato | Clave (PK / FK / UK) |
|---|---|---|
| id_organizacion | int, autoincremental | **PK** |
| nombre | varchar(150), not null | — |
| email | varchar(255), not null | — (candidato a UK) |
| telefono | varchar(30), not null | — |

### 4.2 `usuario`

Persona del sistema. Procede de la entidad Usuario. `nombre_completo` no se almacena.

| Atributo | Tipo de dato | Clave (PK / FK / UK) |
|---|---|---|
| id_usuario | int, autoincremental | **PK** |
| nombre | varchar(100), not null | — |
| apellido | varchar(100), not null | — |
| email | varchar(255), not null | — (candidato a UK) |
| contrasena | varchar(255), not null | — |
| rol | varchar(50), not null | — |
| id_organizacion | int, not null | **FK** → organizacion.id_organizacion |

La FK materializa Pertenece_a (N : 1): muchos usuarios pertenecen a una organización. Es obligatoria, así que todo usuario queda adscrito a una organización.

### 4.3 `usuario_telefono`

Teléfonos del usuario. Tabla nueva para llevar el atributo a 1FN.

| Atributo | Tipo de dato | Clave (PK / FK / UK) |
|---|---|---|
| id_usuario | int, not null | **PK** compuesta y **FK** → usuario.id_usuario |
| telefono | varchar(30), not null | **PK** compuesta |

### 4.4 `ponente`

Subtipo de usuario (relación *es*, 1 : 1).

| Atributo | Tipo de dato | Clave (PK / FK / UK) |
|---|---|---|
| id_ponente | int, autoincremental | **PK** |
| id_usuario | int, not null | **FK** → usuario.id_usuario y **UK** |

La UK garantiza que un usuario sea como máximo un ponente.

### 4.5 `ponente_especialidad`

| Atributo | Tipo de dato | Clave (PK / FK / UK) |
|---|---|---|
| id_ponente | int, not null | **PK** compuesta y **FK** → ponente.id_ponente |
| especialidad | varchar(150), not null | **PK** compuesta |

### 4.6 `ponente_idioma`

| Atributo | Tipo de dato | Clave (PK / FK / UK) |
|---|---|---|
| id_ponente | int, not null | **PK** compuesta y **FK** → ponente.id_ponente |
| idioma | varchar(100), not null | **PK** compuesta |

Especialidad e idioma van en tablas distintas a propósito: son independientes entre sí. Juntarlos en una sola tabla `(id_ponente, especialidad, idioma)` produciría una dependencia multivaluada y violaría 4FN.

### 4.7 `categoria`

| Atributo | Tipo de dato | Clave (PK / FK / UK) |
|---|---|---|
| id_categoria | int, autoincremental | **PK** |
| nombre | varchar(100), not null | — (candidato a UK) |

### 4.8 `clasifica`

Tabla asociativa de Clasifica (Categoria N : M Evento).

| Atributo | Tipo de dato | Clave (PK / FK / UK) |
|---|---|---|
| id_evento | int, not null | **PK** compuesta y **FK** → evento.id_evento |
| id_categoria | int, not null | **PK** compuesta y **FK** → categoria.id_categoria |

### 4.9 `lugar`

Sede. `direccion` no se almacena; se compone con calle, ciudad y pais.

| Atributo | Tipo de dato | Clave (PK / FK / UK) |
|---|---|---|
| id_lugar | int, autoincremental | **PK** |
| nombre | varchar(150), not null | — |
| calle | varchar(255), not null | — |
| ciudad | varchar(100), not null | — |
| pais | varchar(100), not null | — |
| id_evento | int, not null | **FK** → evento.id_evento |

Esta FK implementa *realiza* como N : 1 (varios lugares pueden asociarse a un evento, y cada lugar queda ligado a un solo evento). El diagrama muestra *realiza* como N : N. Si una sede debe reutilizarse en varios eventos, esta FK sobra y hace falta una tabla asociativa `realiza(id_lugar, id_evento)`. Ver sección 6.

### 4.10 `sala`

| Atributo | Tipo de dato | Clave (PK / FK / UK) |
|---|---|---|
| id_sala | int, autoincremental | **PK** |
| nombre | varchar(100), not null | — |
| capacidad | int, not null | — |
| id_lugar | int, not null | **FK** → lugar.id_lugar |

Materializa Contiene (Lugar 1 : N Sala).

### 4.11 `evento`

| Atributo | Tipo de dato | Clave (PK / FK / UK) |
|---|---|---|
| id_evento | int, autoincremental | **PK** |
| titulo | varchar(200), not null | — |
| modalidad | varchar(50), not null | — |
| descripcion | text | — |
| fecha_inicio | date, not null | — |
| fecha_fin | date, not null | — |
| cupo | int, not null | — |
| id_organizacion | int, not null | **FK** → organizacion.id_organizacion |
| id_usuario | int, not null | **FK** → usuario.id_usuario |

- `id_organizacion` materializa *publica* (Organizacion 1 : N Evento).
- `id_usuario` materializa *Crea* (Usuario 1 : N Evento).
- `periodo` no se guarda; se calcula como `fecha_fin - fecha_inicio`.
- `cupo` quedó en el evento. Si el cupo del diagrama es atributo de Ofrece, debe moverse a `tipo_entrada`.

### 4.12 `tipo_entrada`

| Atributo | Tipo de dato | Clave (PK / FK / UK) |
|---|---|---|
| id_tipo_entrada | int, autoincremental | **PK** |
| nombre | varchar(100), not null | — |
| precio | decimal(10,2), not null | — |
| id_evento | int, not null | **FK** → evento.id_evento |

Materializa Ofrece (Evento 1 : N Tipo_entrada).

### 4.13 `sesion`

| Atributo | Tipo de dato | Clave (PK / FK / UK) |
|---|---|---|
| id_sesion | int, autoincremental | **PK** |
| titulo | varchar(200), not null | — |
| tipo | varchar(100), not null | — |
| fecha_hora_inicio | datetime, not null | — |
| fecha_hora_fin | datetime, not null | — |
| id_evento | int, not null | **FK** → evento.id_evento |
| id_sala | int, not null | **FK** → sala.id_sala |

- `id_evento` materializa Programa (Evento 1 : N Sesion).
- `id_sala` materializa ocurre_en (Sesion N : 1 Sala).

### 4.14 `imparte`

Tabla asociativa de Imparte (Ponente N : M Sesion).

| Atributo | Tipo de dato | Clave (PK / FK / UK) |
|---|---|---|
| id_sesion | int, not null | **PK** compuesta y **FK** → sesion.id_sesion |
| id_ponente | int, not null | **PK** compuesta y **FK** → ponente.id_ponente |

### 4.15 `inscripcion`

| Atributo | Tipo de dato | Clave (PK / FK / UK) |
|---|---|---|
| id_inscripcion | int, autoincremental | **PK** |
| estado | varchar(50), not null | — |
| fecha | date, not null | — |
| id_usuario | int, not null | **FK** → usuario.id_usuario |
| id_tipo_entrada | int, not null | **FK** → tipo_entrada.id_tipo_entrada |
| id_evento | int, not null | **FK** → evento.id_evento |

- `id_usuario` materializa Realiza (Usuario 1 : N Inscripcion).
- `id_tipo_entrada` materializa elige (Tipo_entrada 1 : N Inscripcion).
- `id_evento` materializa corresponde_a (Inscripcion N : 1 Evento).

Como `tipo_entrada` ya pertenece a un único evento, `inscripcion.id_evento` es redundante respecto de `tipo_entrada.id_evento`. Es la única dependencia transitiva relevante del esquema. Se discute en 3FN.

### 4.16 `pago`

| Atributo | Tipo de dato | Clave (PK / FK / UK) |
|---|---|---|
| id_pago | int, autoincremental | **PK** |
| monto | decimal(10,2), not null | — |
| estado | varchar(50), not null | — |
| metodo | varchar(50), not null | — |
| id_inscripcion | int, not null | **FK** → inscripcion.id_inscripcion y **UK** |

La UK materializa *genera* como 1 : 1: cada inscripción genera como máximo un pago, y cada pago corresponde a una inscripción.

---

## 5. Mapa de claves foráneas

| Tabla origen | Columna | Tipo de clave | Tabla destino | Relación ER de origen | Cardinalidad resultante |
|---|---|---|---|---|---|
| usuario | id_organizacion | FK | organizacion | Pertenece_a | N : 1 |
| usuario_telefono | id_usuario | PK + FK | usuario | atributo multivaluado | 1 : N |
| ponente | id_usuario | FK + UK | usuario | es | 1 : 1 |
| ponente_especialidad | id_ponente | PK + FK | ponente | atributo multivaluado | 1 : N |
| ponente_idioma | id_ponente | PK + FK | ponente | atributo multivaluado | 1 : N |
| lugar | id_evento | FK | evento | realiza | N : 1 (ver nota) |
| sala | id_lugar | FK | lugar | Contiene | N : 1 |
| evento | id_organizacion | FK | organizacion | publica | N : 1 |
| evento | id_usuario | FK | usuario | Crea | N : 1 |
| clasifica | id_evento | PK + FK | evento | Clasifica | N : M |
| clasifica | id_categoria | PK + FK | categoria | Clasifica | N : M |
| tipo_entrada | id_evento | FK | evento | Ofrece | N : 1 |
| sesion | id_evento | FK | evento | Programa | N : 1 |
| sesion | id_sala | FK | sala | ocurre_en | N : 1 |
| imparte | id_sesion | PK + FK | sesion | Imparte | N : M |
| imparte | id_ponente | PK + FK | ponente | Imparte | N : M |
| inscripcion | id_usuario | FK | usuario | Realiza | N : 1 |
| inscripcion | id_tipo_entrada | FK | tipo_entrada | elige | N : 1 |
| inscripcion | id_evento | FK | evento | corresponde_a | N : 1 |
| pago | id_inscripcion | FK + UK | inscripcion | genera | 1 : 1 |

---

## 6. Decisiones de diseño y diferencias respecto del diagrama

1. **Atributos derivados eliminados.** `nombre_completo`, `direccion` y `periodo` aparecen en el ER entre paréntesis o como composición calculable. No se persisten. Se recuperan así:
   - `nombre_completo = nombre || ' ' || apellido`
   - `direccion = calle || ', ' || ciudad || ', ' || pais`
   - `periodo = fecha_fin - fecha_inicio`
2. **Teléfono de usuario separado; teléfono de organización no.** El diagrama pinta un solo óvalo `telefono` en ambas entidades. El esquema lógico solo separó el del usuario. Es coherente con 1FN si se asume que un usuario puede tener varios teléfonos y una organización uno solo. Si la organización también puede tener varios, hay que crear `organizacion_telefono` con la misma estructura.
3. **Especialidad e idioma separados.** Aunque el diagrama muestra un óvalo por atributo, el modelo lógico los trata como multivaluados e independientes. Es la lectura que justifica las tablas `ponente_especialidad` y `ponente_idioma`, y es la que permite afirmar 4FN.
4. **Horario movido a sesión.** El ER asocia `fecha_hora_inicio` y `fecha_hora_fin` al evento. El relacional las coloca en `sesion`, que es donde tienen sentido operativo (cada charla tiene su franja). El evento conserva el rango de fechas.
5. **`cupo` ubicado en Evento.** En el diagrama está junto al rombo Ofrece. Si significa plazas por tipo de entrada, la columna debe estar en `tipo_entrada`, no en `evento`. Dejarlo en el evento no rompe la forma normal, pero cambia el significado.
6. **`realiza` implementada como N : 1.** Poner `id_evento` dentro de `lugar` impide que la misma sede participe en otro evento sin duplicar la fila de lugar (y, con ella, sus salas). Si el negocio reutiliza sedes, la transformación fiel del N : M es:

```text
realiza(id_lugar PK/FK, id_evento PK/FK)
```

y `lugar` no lleva `id_evento`.
7. **`inscripcion.id_evento` es redundante.** Está justificada en el diagrama por *corresponde_a*, pero ya es derivable: `inscripcion.id_tipo_entrada → tipo_entrada.id_evento`. Mantenerla exige un control extra (el evento de la inscripción debe coincidir con el evento del tipo de entrada). Quitarla deja el esquema en 3FN estricta sin perder información.
8. **Ciclo referencial.** `lugar` apunta a `evento`, `sala` apunta a `lugar`, `sesion` apunta a `sala` y a `evento`. No es una violación de forma normal, pero el orden de carga y de borrado debe respetar ese ciclo. Conviene definir `ON DELETE` de forma explícita.
9. **Contraseña.** Se guarda en `varchar(255)`, tamaño propio de un hash, no de la clave en claro. El informe asume que la aplicación persiste un hash.

---

## 7. Dependencias usadas en el análisis

Notación: `A → B` es dependencia funcional; `A ↠ B` es dependencia multivaluada.

### 7.1 Dependencias funcionales de las entidades

```text
id_organizacion → nombre, email, telefono
id_usuario      → nombre, apellido, email, contrasena, rol, id_organizacion
id_ponente      → id_usuario
id_usuario      → id_ponente          (solo si el usuario es ponente; UK)
id_categoria    → nombre
id_lugar        → nombre, calle, ciudad, pais, id_evento
id_sala         → nombre, capacidad, id_lugar
id_evento       → titulo, modalidad, descripcion, fecha_inicio, fecha_fin,
                  cupo, id_organizacion, id_usuario
id_tipo_entrada → nombre, precio, id_evento
id_sesion       → titulo, tipo, fecha_hora_inicio, fecha_hora_fin,
                  id_evento, id_sala
id_inscripcion  → estado, fecha, id_usuario, id_tipo_entrada, id_evento
id_pago         → monto, estado, metodo, id_inscripcion
id_inscripcion  → id_pago             (UK del lado pago)
```

Transitiva relevante:

```text
id_inscripcion → id_tipo_entrada → id_evento
```

Por tanto `id_inscripcion → id_evento` también se cumple por transitividad. `id_evento` no es primo en `inscripcion` si la única clave es `id_inscripcion`.

### 7.2 Dependencias multivaluadas

```text
id_usuario  ↠ telefono
id_ponente  ↠ especialidad
id_ponente  ↠ idioma
```

Especialidad no determina idioma, ni al revés. Son dos MVD independientes.

### 7.3 Dependencias de reunión que sí están implicadas por las claves

En `clasifica` y en `imparte` la única forma de reconstruir la relación es unir por el par de claves. No hay un tercer atributo independiente que obligue a descomponer más. No se identificó una dependencia de reunión no trivial fuera de las implicadas por las claves candidatas.

---

## 8. Formas normales

Se recorre cada forma en el orden clásico. Una relación está en la forma *k* solo si ya está en la forma *k − 1* y cumple la condición nueva.

### 8.1 Primera forma normal (1FN)

**Condición.** Todo dominio es atómico: no hay grupos repetitivos ni atributos compuestos o multivaluados dentro de una fila. Existe una clave que identifica cada tupla.

**Qué se hizo.**

- Se rechazó guardar teléfonos, especialidades o idiomas como listas dentro de una columna.
- `telefono` de usuario pasó a `usuario_telefono`.
- `especialidad` pasó a `ponente_especialidad`.
- `idioma` pasó a `ponente_idioma`.
- El atributo compuesto `direccion` se descompuso en `calle`, `ciudad` y `pais`.
- Cada tabla tiene PK simple o compuesta.

**Resultado.** Todas las tablas del esquema están en 1FN.

**Ejemplo de la anomalía evitada.** Si especialidad e idioma vivieran en la fila de ponente como texto repetido, no se podría insertar un idioma nuevo sin tocar especialidades, ni consultar “ponentes que hablan inglés” sin partir cadenas.

### 8.2 Segunda forma normal (2FN)

**Condición.** Está en 1FN y ningún atributo no primo depende de una parte propia de una clave candidata. Solo puede fallar en tablas con PK compuesta.

**Revisión de las PK compuestas.**

| Tabla | PK | Atributos no primos | ¿Dependencia parcial? |
|---|---|---|---|
| usuario_telefono | (id_usuario, telefono) | ninguno | No |
| ponente_especialidad | (id_ponente, especialidad) | ninguno | No |
| ponente_idioma | (id_ponente, idioma) | ninguno | No |
| clasifica | (id_evento, id_categoria) | ninguno | No |
| imparte | (id_sesion, id_ponente) | ninguno | No |

El resto de las tablas tiene PK de un solo atributo. En ellas 2FN se cumple de forma automática una vez cumplida 1FN, porque no existe “parte propia” de la clave.

**Resultado.** El esquema está en 2FN. No hizo falta una descomposición adicional: las tablas asociativas no arrastran atributos que dependan de una sola de las dos FK.

### 8.3 Tercera forma normal (3FN)

**Condición.** Está en 2FN y ningún atributo no primo depende transitivamente de una clave candidata. Equivalentemente: para toda DF no trivial `X → A`, o `X` es superclave, o `A` es primo.

**Cumplen 3FN sin observación.**

- `organizacion`, `usuario`, `categoria`, `sala`, `tipo_entrada`, `sesion`, `pago`, `ponente` y las cinco tablas de PK compuesta. En todas, el determinante de cada DF no trivial es la PK (o la UK, que también es superclave: `ponente.id_usuario`, `pago.id_inscripcion`).
- Quitar `nombre_completo`, `direccion` y `periodo` eliminó dependencias transitivas de atributos calculados. Por ejemplo, `id_usuario → nombre, apellido → nombre_completo`. Al no almacenar `nombre_completo`, esa cadena no genera anomalía de actualización.

**No cumple 3FN estricta.**

`inscripcion`, por esta cadena:

```text
id_inscripcion → id_tipo_entrada
id_tipo_entrada → id_evento          (DF que vive en tipo_entrada)
luego id_inscripcion → id_evento     y id_evento no es primo
```

`id_evento` depende de la clave a través de `id_tipo_entrada`. Consecuencias prácticas:

- Anomalía de actualización: cambiar el evento de un tipo de entrada no actualiza solo las inscripciones.
- Anomalía de inconsistencia: se puede insertar una inscripción cuyo `id_evento` no coincida con el evento del tipo de entrada elegido.

**Corrección para quedar en 3FN estricta.** Eliminar `inscripcion.id_evento` y obtener el evento por reunión:

```text
inscripcion ⋈ tipo_entrada
```

La relación *corresponde_a* sigue siendo recuperable. Si se desea conservar la columna por rendimiento, hay que añadir una restricción que iguale ambos eventos (por ejemplo, una FK compuesta `(id_tipo_entrada, id_evento)` hacia una UK `(id_tipo_entrada, id_evento)` de `tipo_entrada`). Con esa restricción la redundancia queda controlada, aunque la DF transitiva sigue existiendo.

**Resultado.** El esquema está en 3FN salvo la columna redundante `inscripcion.id_evento`. El resto de las tablas está en 3FN.

### 8.4 Forma normal de Boyce-Codd (BCNF)

**Condición.** Está en 3FN y, para toda DF no trivial `X → A`, `X` es superclave. BCNF no admite la excepción de 3FN en la que el atributo determinado es primo.

**Revisión.** No hay determinantes que no sean superclave, salvo la transitividad ya señalada en `inscripcion`.

- `email` podría ser clave candidata de `usuario` y de `organizacion` si se declara único. Aunque se declare, `email → rol` sigue teniendo como determinante una clave candidata, así que no rompe BCNF.
- `ponente.id_usuario` y `pago.id_inscripcion` son superclaves por la UK. Cumplen BCNF.
- Las tablas asociativas no tienen DF distinta de “la PK determina el vacío de atributos no primos”.

**Resultado.** Todas las tablas están en BCNF, excepto `inscripcion` mientras conserve `id_evento` junto a `id_tipo_entrada`. Al quitar esa columna, `inscripcion` también queda en BCNF, porque sus DF quedan determinadas por `id_inscripcion`.

### 8.5 Cuarta forma normal (4FN)

**Condición.** Está en BCNF y no contiene dependencias multivaluadas no triviales que no sean también dependencias funcionales. Si `X ↠ Y` y `X ↠ Z` son independientes, `Y` y `Z` no deben convivir en la misma tabla.

**Qué se hizo.**

- `usuario_telefono` aísla la MVD `id_usuario ↠ telefono`.
- `ponente_especialidad` aísla `id_ponente ↠ especialidad`.
- `ponente_idioma` aísla `id_ponente ↠ idioma`.

**Anomalía evitada.** Una tabla `ponente_habilidad(id_ponente, especialidad, idioma)` obligaría, para un ponente con 2 especialidades y 3 idiomas, a 6 filas si se quiere el producto, o a filas con nulos si no se quiere. Insertar un idioma nuevo replicaría especialidades. Separar las dos MVD deja cada hecho en una fila y la reunión natural recupera el producto solo cuando hace falta.

**Relaciones N : M.** `clasifica` e `imparte` no violan 4FN: cada una representa una sola relación binaria. No hay un segundo conjunto multivaluado independiente dentro de la misma tabla.

**Resultado.** Con la salvedad de BCNF en `inscripcion`, el esquema está en 4FN. Las MVD identificadas fueron descompuestas.

### 8.6 Quinta forma normal (5FN), o forma normal de proyección-reunión

**Condición.** Está en 4FN y toda dependencia de reunión está implicada por las claves candidatas. No debe ser posible descomponer una tabla en proyecciones y perder, o inventar, tuplas al reunirlas, salvo que esa reunión sea exactamente la que imponen las claves.

**Revisión.**

- No hay relaciones ternarias en el diagrama (ningún rombo conecta tres entidades). Clasifica, Imparte, Ofrece, Programa y el resto son binarias. Una relación binaria N : M ya está en 5FN cuando su tabla es el par de claves: la proyección sobre cada FK y la reunión por ambas recuperan exactamente las parejas, sin tuplas espurias.
- `sesion` tiene dos FK (`id_evento`, `id_sala`) más atributos propios. No es una relación ternaria descompuesta a medias: la sesión es entidad, y cada FK representa una relación  N : 1 distinta (Programa y ocurre_en). No existe una DF de reunión del estilo “evento-sala, evento-ponente, sala-ponente” dentro de `sesion`, porque el ponente no está en esa tabla; está en `imparte`.
- Unir `imparte`, `sesion` y `ponente` no crea una restricción cíclica oculta: un ponente imparte una sesión, la sesión pertenece a un evento y ocurre en una sala. Esas tres afirmaciones son independientes y se recuperan por reuniones sobre claves.
- Las tablas de atributos multivaluados son binarias (entidad, valor). Su única reunión significativa es con la entidad padre, implicada por la FK.

**Resultado.** No se identificó una dependencia de reunión no trivial fuera de las claves. Bajo las dependencias del modelo conceptual, el esquema alcanza 5FN en todas las tablas que ya están en 4FN. La única reserva para afirmar 5FN del esquema completo es cerrar antes la salvedad de 3FN/BCNF en `inscripcion`: 5FN presupone 4FN, y 4FN presupone BCNF.

### 8.7 Síntesis por tabla

| Tabla | 1FN | 2FN | 3FN | BCNF | 4FN | 5FN |
|---|---|---|---|---|---|---|
| organizacion | Sí | Sí | Sí | Sí | Sí | Sí |
| usuario | Sí | Sí | Sí | Sí | Sí | Sí |
| usuario_telefono | Sí | Sí | Sí | Sí | Sí | Sí |
| ponente | Sí | Sí | Sí | Sí | Sí | Sí |
| ponente_especialidad | Sí | Sí | Sí | Sí | Sí | Sí |
| ponente_idioma | Sí | Sí | Sí | Sí | Sí | Sí |
| categoria | Sí | Sí | Sí | Sí | Sí | Sí |
| clasifica | Sí | Sí | Sí | Sí | Sí | Sí |
| lugar | Sí | Sí | Sí | Sí | Sí | Sí |
| sala | Sí | Sí | Sí | Sí | Sí | Sí |
| evento | Sí | Sí | Sí | Sí | Sí | Sí |
| tipo_entrada | Sí | Sí | Sí | Sí | Sí | Sí |
| sesion | Sí | Sí | Sí | Sí | Sí | Sí |
| imparte | Sí | Sí | Sí | Sí | Sí | Sí |
| inscripcion | Sí | Sí | No, por `id_evento` | No | — | — |
| pago | Sí | Sí | Sí | Sí | Sí | Sí |

`lugar` está en 5FN como tabla, pero la FK `id_evento` puede ser una cardinalidad mal traducida (N : 1 en lugar de N : M). Eso es fidelidad al diagrama, no una violación de forma normal.

---

## 9. Recorrido de la transformación, relación por relación

| Elemento del ER | Regla | Resultado relacional |
|---|---|---|
| Entidad Organizacion | entidad fuerte → tabla | `organizacion` |
| Entidad Usuario | entidad fuerte → tabla; derivado `nombre_completo` eliminado | `usuario` |
| telefono de Usuario | multivaluado → tabla | `usuario_telefono` |
| Pertenece_a N : 1 | FK en el lado N | `usuario.id_organizacion` |
| Entidad Ponente + *es* 1 : 1 | subtipo con FK única | `ponente.id_usuario` UK |
| especialidad, idioma | dos MVD independientes → dos tablas | `ponente_especialidad`, `ponente_idioma` |
| Entidad Categoria | entidad fuerte → tabla | `categoria` |
| Clasifica N : M | tabla asociativa | `clasifica` |
| Entidad Lugar | entidad fuerte; `direccion` descompuesta y no almacenada | `lugar` |
| realiza | traducida como FK en Lugar | `lugar.id_evento` |
| Entidad Sala | entidad fuerte → tabla | `sala` |
| Contiene 1 : N | FK en el lado N | `sala.id_lugar` |
| Entidad Evento | entidad fuerte; `periodo` eliminado | `evento` |
| publica 1 : N | FK en el lado N | `evento.id_organizacion` |
| Crea 1 : N | FK en el lado N | `evento.id_usuario` |
| Entidad Tipo_entrada | entidad fuerte → tabla | `tipo_entrada` |
| Ofrece 1 : N | FK en el lado N | `tipo_entrada.id_evento` |
| cupo (atributo de Ofrece) | quedó en Evento | `evento.cupo` |
| Entidad Sesion | entidad fuerte; horario propio | `sesion` |
| Programa 1 : N | FK en el lado N | `sesion.id_evento` |
| ocurre_en N : 1 | FK en el lado N | `sesion.id_sala` |
| Imparte N : M | tabla asociativa | `imparte` |
| Entidad Inscripcion | entidad fuerte → tabla | `inscripcion` |
| Realiza 1 : N | FK en el lado N | `inscripcion.id_usuario` |
| elige 1 : N | FK en el lado N | `inscripcion.id_tipo_entrada` |
| corresponde_a N : 1 | FK en el lado N | `inscripcion.id_evento` |
| Entidad Pago | entidad fuerte → tabla | `pago` |
| genera 1 : 1 | FK única en el lado pago | `pago.id_inscripcion` UK |

El esquema relacional tiene 16 tablas: 11 proceden de entidades y 5 de atributos multivaluados o de relaciones N : M (`usuario_telefono`, `ponente_especialidad`, `ponente_idioma`, `clasifica`, `imparte`). Las relaciones 1 : N y la 1 : 1 no generaron tabla propia.

---

## 10. Conclusión

El modelo relacional se obtuvo aplicando las reglas de transformación de Chen: entidad fuerte a tabla, 1 : N mediante FK en el lado muchos, 1 : 1 mediante FK única, N : M mediante tabla asociativa, atributo derivado eliminado y atributo multivaluado extraído.

El esquema está en 1FN y en 2FN de forma completa. Está en 3FN, BCNF, 4FN y 5FN en todas las tablas salvo `inscripcion`, donde `id_evento` depende transitivamente de `id_tipo_entrada`. Separar especialidad e idioma, y separar el teléfono del usuario, es lo que sostiene 4FN. La ausencia de relaciones ternarias con restricción cíclica independiente es lo que sostiene 5FN.

Para que la afirmación “normalizado hasta 5FN” sea estricta sobre todo el esquema basta con eliminar `inscripcion.id_evento` (o sujetarla con una FK compuesta al par ya existente en `tipo_entrada`) y, si las sedes se reutilizan, sustituir `lugar.id_evento` por la tabla asociativa `realiza`. Con ese ajuste, cada hecho queda en un solo lugar, toda DF no trivial tiene una superclave como determinante, las MVD independientes están separadas y toda reunión necesaria está implicada por las claves.

---

## 11. Ajuste mínimo recomendado para cerrar 5FN

```text
inscripcion(
  id_inscripcion   PK,
  estado,
  fecha,
  id_usuario       FK → usuario.id_usuario,
  id_tipo_entrada  FK → tipo_entrada.id_tipo_entrada
)
-- id_evento se obtiene por tipo_entrada.id_evento
```

Opcional, solo si *realiza* debe seguir siendo N : M:

```text
lugar(
  id_lugar  PK,
  nombre, calle, ciudad, pais
)

realiza(
  id_lugar   PK/FK → lugar.id_lugar,
  id_evento  PK/FK → evento.id_evento
)
```
