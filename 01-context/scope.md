# Alcance del sistema — Simple Stock Flow

> **Fuente única:** [`spec/data-model.md`](../spec/data-model.md). El alcance se dibuja **con lo que el modelo cierra**: §12 lista expresamente lo que queda fuera y declara que «las tres [decisiones] están decididas y no se reabren».
> **Supuesto** = algo que no sale del modelo: **[SUPUESTO S-nn]**, en [`../supuestos.md`](../supuestos.md).
> El alcance evita el crecimiento descontrolado: es tan importante definir lo que el sistema **no** hace como lo que hace.

---

## 1. Dentro del alcance

### Funcionalidades

| # | Funcionalidad | Descripción | Agregado | Cita |
|---|---|---|---|---|
| 1 | Autenticación | Los usuarios internos inician sesión con nombre y contraseña; sin credencial, 401 | `User` | §2.5, §13 D-3 |
| 2 | Alta de vendedores | El administrador da de alta vendedores; el rol `admin` no se otorga en ejecución | `User` | §9.2, DP-04 |
| 3 | Administrador inicial | El arranque lo crea con credenciales de entorno | `User` | §9.2 |
| 4 | Catálogo de productos | Crear, modificar precio y stock, recategorizar y dar de baja productos con **cinco atributos** | `Product` | §2.2, DP-03 |
| 5 | Búsqueda de productos | Por texto parcial y categoría, solo activos, ordenados por nombre y paginados | `Product` | §6.1 Q1–Q3 |
| 6 | Categorías | Consultar las cinco categorías fijas | `Category` | §2.1, D-10 |
| 7 | Imagen del producto | Adjuntar, reemplazar o quitar; clave opaca, binario en almacenamiento externo | `Product` | §1, §7.1, D-08 |
| 8 | Registro de ventas | Ventas de una o más líneas; descuentan stock; inmutables | `Sale` | §2.3, §2.4 |
| 9 | Consulta de ventas | Una venta con sus líneas; listado por rango, paginado | `Sale` | §6.1 Q6, Q7 |
| 10 | Reporte de ventas | Agregación por producto sobre un rango; no se persiste | modelo de lectura | §1, §6.1 Q9, §11.1 |

Las historias de usuario que desarrollan cada una están en [`../04-requirements/user-stories.md`](../04-requirements/user-stories.md).

### Integraciones incluidas

| Sistema externo | Tipo | Propósito | Cita |
|---|---|---|---|
| **PostgreSQL 16** | Base de datos | Almacenamiento de las cinco tablas del esquema `sales` | «Verificado contra» |
| **Almacenamiento externo** | Almacén de objetos [SUPUESTO S-18: el modelo solo dice «almacenamiento externo»] | Binarios de imagen de producto, referidos por clave opaca | §1, §7.1, D-08 |

> **Proveedor del almacenamiento: no definido en el modelo.** No se nombra ninguno.

### Entornos

El modelo menciona solo el entorno donde se verificó: un contenedor de PostgreSQL (`simple-stock-flow-db-1`) levantado con `docker compose` desde el repositorio de infraestructura, con el servidor en **UTC** (§3, §10). **Otros entornos no se definen** [SUPUESTO S-07].

---

## 2. Fuera del alcance

Lo que el sistema **no construye** y por qué. Todo es **decisión cerrada** salvo que se indique.

| # | Fuera del alcance | Motivo | ¿Futuro? | Cita |
|---|---|---|---|---|
| 1 | Entidad cliente / comprador | No existe: la venta registra al operador, no al comprador | No definido | §1, §7 |
| 2 | Venta en varias monedas | El sistema es monomoneda por construcción; no se reintroduce | No | D-05 |
| 3 | Atributos de producto adicionales (descripción, SKU, código de referencia) | Cerrado por DP-03 | No | §1, §12 |
| 4 | Reporte desglosado por vendedor | DP-02; cruzaría datos personales | No | §6.3, §7.1 |
| 5 | Auditoría de cambios (`created_at`, `updated_at`, bitácora) | Sin requisito; decisión cerrada | **Solo si aparece una pregunta real de auditoría**; entonces vuelve al propietario | §8 |
| 6 | Mantenimiento de categorías (crear, renombrar, borrar) | Son datos sembrados de solo lectura | **Condición de revisión explícita:** si se abre, se revisa la unicidad por acentos y mayúsculas antes | §4.1, D-10 |
| 7 | Edición o borrado de ventas | Hecho consumado e inmutable | No | §2.3 |
| 8 | Borrado físico de productos | Baja lógica | No | §2.2, ADR-003 |
| 9 | Otorgar el rol `admin` en ejecución | DP-04: lo provisiona el despliegue | No | §11 H-3 |
| 10 | Pagos y tarjetas; datos de salud | No aplican | No | §7 |
| 11 | Analítica externa y exportación | No existen; el reporte no expone datos personales | No definido | §7.1 |
| 12 | Anonimización para analítica | Omisión consciente: no hay analítica | No definido | §7.1 |
| 13 | Contrato de la API, requisitos de negocio y decisiones técnicas del proyecto original | Viven en documentos que **no se entregan** | — | §12 |

### Lo que otro sistema o equipo maneja

| Funcionalidad | Quién | Por qué no nosotros |
|---|---|---|
| Almacenamiento de binarios de imagen | Almacenamiento externo | El sistema solo guarda una clave opaca; el binario no participa de la transacción de la base (§7.1) |
| Producción del hash de contraseña | Un puerto de hash (adaptador de seguridad) | El dominio no debe ver nunca la clave en claro (§2.5, D-09) |
| Provisión del rol `admin` | El despliegue, desde el entorno | Ningún usuario lo otorga en ejecución (DP-04) |

---

## 3. Decisiones de alcance aún abiertas

| # | Tema | Estado | Cita |
|---|---|---|---|
| H-2 | Política de limpieza de binarios de imagen huérfanos | **Abierta**; operativa, sin impacto en el entregable | §11 |
| CA-06.1 | Reescribir el criterio de `spec.md` «una fila por producto» para que no contradiga H-1 | **Decisión pendiente del propietario** | §11.1 |
| — | Qué rol puede gestionar catálogo, consultar ventas y reporte | **No definido** [S-03] | — |
| — | Volumen esperado de datos y usuarios | **No definido** | §10.4 |

---

## 4. Criterio para aceptar un cambio de alcance

Lo que el propio modelo usa para decidir qué entra:

1. **¿Hay un requisito que lo pida?** Sin requisito no hay alcance (§8, DP-03).
2. **¿Alguna consulta o puerto lo necesita?** Sin patrón de acceso no hay índice ni columna (§6, §8).
3. **¿Contradice una decisión cerrada?** Entonces vuelve a su dueño, no se resuelve con código (§8, §11.1).
