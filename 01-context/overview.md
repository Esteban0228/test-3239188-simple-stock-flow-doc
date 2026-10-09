# Descripción general — Simple Stock Flow

> **Fuente única:** [`spec/data-model.md`](../spec/data-model.md). Es el **único** insumo del reto: los demás documentos del proyecto original (`constitution.md`, `spec.md`, `plan.md`, `adr/`, `tasks.md`, `api-contract.md`) no se entregan (README del reto). Lo que no sale del modelo se marca **[SUPUESTO S-nn]** ([`../supuestos.md`](../supuestos.md)).
> **Idioma:** estos documentos están en español, igual que el modelo y el README del reto; los identificadores de código y de columna se conservan en inglés, como manda el glosario [SUPUESTO S-01].

---

## 1. ¿Qué es Simple Stock Flow?

Un sistema interno para **llevar el catálogo, el stock y las ventas** de un comercio pequeño. Lo opera personal del propio negocio —**no sus clientes**— y se apoya en una base PostgreSQL.

Lo que el modelo deja claro que hace:

- Mantiene un **catálogo de productos** con nombre, precio, stock, categoría e imagen opcional (§1, DP-03).
- **Registra ventas** de varias líneas que descuentan stock y quedan **inmutables** (§1, §2.3).
- Produce un **reporte de ventas por producto** sobre un rango de fechas (§1, §6.1 Q9).
- Distingue **dos roles** de usuario interno: `admin` y `seller` (§2.5).

> [SUPUESTO S-04] Que sea para un comercio tipo ferretería sale solo de las cinco categorías sembradas (*General, Herramientas, Electricidad, Fontanería, Pinturas*, §9.1). El modelo **no nombra** al negocio.

---

## 2. Problema que resuelve

### Antes del sistema

Cuando el inventario y las ventas dependen de que cada persona respete las reglas a mano:

- Dos ventas simultáneas pueden dejar el stock **negativo**.
- Renombrar, reprecificar o recategorizar un producto **reescribe el pasado** de los reportes.
- Una venta puede **editarse o borrarse** después de hecha.
- No queda claro **quién** registró cada operación.

### Con el sistema

- El stock **nunca baja de cero** y las ventas simultáneas no se pisan (§2.2, D-04).
- Cada línea de venta **congela** precio, nombre y categoría del momento (§2.4).
- Las ventas son **hechos consumados**: no se editan ni se borran (§2.3).
- Cada venta queda atribuida a un usuario (§2.3).

El detalle del problema está en [`../03-product/problem-framing.md`](../03-product/problem-framing.md).

---

## 3. Usuarios

| Rol | Descripción | Qué hace en el sistema |
|---|---|---|
| **Vendedor** (`seller`) | Operador interno que atiende ventas | Se autentica, consulta el catálogo y **registra ventas** (§1, §2.5) |
| **Administrador** (`admin`) | Operador interno con más privilegio | Da de alta vendedores (DP-04); mantiene el catálogo [SUPUESTO S-03] |
| **Responsable del despliegue** | Quien configura el entorno | Entrega las credenciales con las que el arranque crea al primer administrador (§9.2) |

**No hay clientes en el sistema.** No existe la entidad ni se guarda ningún dato personal de un comprador (§1, §7). El `username` del operador sí es un dato personal (§7).

---

## 4. Flujo principal

```text
Administrador                              Vendedor
     |                                        |
     v                                        v
Mantiene el catálogo                    Se autentica
(productos, precio, stock,                    |
 categoría de las cinco, imagen)              v
     |                                  Busca productos
     |                                        |
     |                                        v
     |                              Registra una venta (≥ 1 línea)
     |                                        |
     |                     +------------------+------------------+
     |                     |                                     |
     |                     v                                     v
     |            Stock baja por línea                Línea congela nombre,
     |            (una sola operación)               precio y categoría
     |                     |
     |                     v
     +------------> Reporte de ventas por producto
                    (rango de fechas, calculado en el motor)
```

Las partes de este flujo con su respaldo: catálogo §2.2; búsqueda §6.1 Q1; venta §2.3–§2.4; reporte §6.1 Q9 y §11.1.

---

## 5. El sistema en cifras (según el modelo)

| Dato | Valor | Cita |
|---|---|---|
| Entidades / tablas | 5: `category`, `product`, `sale`, `sale_item`, `user` | §2 |
| Raíces de agregado | 3: `Product`, `Sale`, `User` | §2 |
| Esquema | `sales`, en PostgreSQL 16 | «Verificado contra», §3 |
| Roles | 2: `admin`, `seller` | §2.5 |
| Categorías | 5, sembradas, sin mantenimiento | §9.1, D-10 |
| Migraciones aplicadas | 4 | §3.2 |
| Volumen al 2026-09-19 | 5 categorías · 1 usuario · 0 productos · 0 ventas · 0 líneas | §10.4 |

> **Nota.** El modelo afirma «22 columnas» en §3 y §12, pero la medición de §10.1 (2026-09-19) devuelve **21**. La diferencia probable es `product.deleted_at`, creada después de esa medición (T-09, §13 D-1) — es una lectura, no un dato del modelo. Ver OBS-01 en [`../05-architecture/verification.md`](../05-architecture/verification.md).

---

## 6. Estado del proyecto

El modelo describe un sistema **ya construido y en revisión**, con tareas pendientes explícitas:

- **Pendiente:** autoría de la venta con clave foránea a `user` (T-12), tres índices (T-13), guarda explícita de moneda (T-05) y bajar al motor cinco invariantes (T-20, parcial) (§2.3, §4, §6.2).
- **Hueco abierto:** política de limpieza de binarios de imagen huérfanos (H-2) (§11).
- **Decisión pendiente del propietario:** reescribir `spec.md` CA-06.1 para que no contradiga H-1 (§11.1).

Se detalla en [`../03-product/vision.md`](../03-product/vision.md) y [`../05-architecture/verification.md`](../05-architecture/verification.md).

---

## 7. Cómo leer esta documentación

| Orden | Carpeta | Qué responde |
|---|---|---|
| 1 | [`01-context/`](.) | Qué es y qué no es el sistema |
| 2 | [`02-domain/`](../02-domain/) | Entidades, reglas, eventos y glosario |
| 3 | [`03-product/`](../03-product/) | Problema y visión |
| 4 | [`04-requirements/`](../04-requirements/) | Historias de usuario y requisitos no funcionales |
| 5 | [`05-architecture/`](../05-architecture/) | Estilo, piezas y dónde vive cada regla; verificación de cierre |
| — | [`../supuestos.md`](../supuestos.md) | Todo lo que **no** sale del modelo |

> Estos documentos se **reconstruyeron hacia atrás**, de la arquitectura al contexto, a partir del modelo. Por eso el orden de lectura es el inverso al de escritura.
