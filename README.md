# ConferenciaHub

Plataforma de eventos y conferencias

[![Estado](https://img.shields.io/badge/estado-primera%20entrega-blue)]()
[![Modelo](https://img.shields.io/badge/modelo-entidad%20relación-informational)]()
[![Licencia](https://img.shields.io/badge/uso-académico-success)]()

---

## Integrantes

| Nombre completo | Codigo estudiantil |
|---|---|
| Fredy Omar Avila Triana | 01251151005 |
| Andres Felipe Castilla Caselles | 01251151011 |

---

## Tema del proyecto

**Plataforma web de gestión de eventos y conferencias.**

El sistema permite publicar, organizar, difundir e inscribirse a eventos académicos y profesionales (congresos, conferencias, talleres, meetups y actividades híbridas o virtuales).

---

## Descripción inicial

ConferenciaHub nace para centralizar el ciclo de vida de un evento: desde que una organización lo crea hasta que el asistente se inscribe, asiste, califica las sesiones y obtiene su certificado.

Hoy esa gestión suele estar repartida entre formularios, hojas de cálculo, grupos de WhatsApp y páginas sueltas. La plataforma busca unificar:

- **Organizaciones y organizadores** que publican eventos.
- **Eventos** presenciales, virtuales o híbridos, con sede, cupos y categorías.
- **Agenda** de sesiones (keynotes, ponencias, talleres y paneles).
- **Ponentes** asociados a cada sesión.
- **Tipos de entrada** (gratuita, general, early bird, VIP, estudiante).
- **Inscripciones y pagos** con estado de la participación.
- **Patrocinadores**, materiales de apoyo, valoraciones y certificados de asistencia.

En esta primera entrega se documenta el **modelo entidad-relación**: entidades, identificadores principales, atributos, nombres de relación, tipo de relación (1:1, 1:N, N:M) y cardinalidad.

La carpeta `Primera Entrega` contiene:

- El diagrama entidad-relación.
- El archivo Markdown con la actividad de exploración.

---

## Objetivos del proyecto

1. Modelar de forma coherente usuarios, roles, organizaciones, eventos, sedes, sesiones y la inscripción.
2. Dejar explícitas las cardinalidades y las reglas de negocio (por ejemplo: un usuario se inscribe una sola vez a un mismo evento).
3. Servir de base para el diseño lógico y la implementación posterior de la base de datos.

---

## Alcance de la primera entrega

Incluye:

- Identificación de entidades y atributos.
- Claves primarias y relaciones con nombre.
- Cardinalidad y tipo de relación.
- Actividad de exploración del problema.

No incluye todavía:

- Script SQL de creación.
- Interfaz de usuario.
- Implementación de pagos reales.

---

## Estructura del repositorio

```text
.
├── README.md
└── Primera Entrega
    ├── modelo-entidad-relacion.png   # o .pdf / .drawio
    └── actividad-exploracion.md
