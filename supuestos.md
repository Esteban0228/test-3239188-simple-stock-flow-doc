# Registro de supuestos — Simple Stock Flow

> **Regla del reto (README, regla 3):** cada afirmación debe poder rastrearse al modelo de datos; **si algo no sale del modelo, se marca como supuesto**.
> Este registro lista **todos** los supuestos de la documentación 01–05, con dónde se usan y cómo validarlos. En los documentos aparecen como **[SUPUESTO S-nn]**.
> Fuente: [`spec/data-model.md`](spec/data-model.md). Un supuesto **no es un hecho**: es una lectura razonable que debe confirmar quien tenga el dato.

---

## 1. Supuestos

| ID | Supuesto | Por qué hubo que suponer | Dónde se usa | Cómo validarlo |
|---|---|---|---|---|
| **S-01** | La documentación se escribe en **español**; los identificadores de código y de columna, en inglés | El modelo y el README del reto están en español; el glosario manda código y columnas en inglés (§1, artículo XI). El modelo no regula el idioma de la documentación | Todos | Confirmar con el propietario |
| **S-02** | El sistema es **un solo desplegable** | El modelo no declara cuántos procesos hay. Lo infiere el esquema único `sales` que «agrupa el sistema entero» (§3) y el repositorio único de la API (§12) | `05` overview, `02` eventos | Confirmar el estilo de despliegue |
| **S-03** | El **administrador** gestiona el catálogo; **ambos roles** consultan catálogo, ventas y reporte y registran ventas | El modelo solo dice qué hace cada rol en dos puntos: el `seller` registra ventas y el `admin` da de alta vendedores (§1, §9.2, DP-04). No dice quién gestiona catálogo ni quién ve reportes | `04` historias, `01` overview y scope, `03` problema | **Prioritario.** Definir la matriz de permisos antes de implementar autorización |
| **S-04** | El negocio es de tipo **ferretería** | Solo se infiere de las cinco categorías sembradas (§9.1). El modelo no nombra el sector | `01`, `03` | Confirmar el sector. El problema no depende de este supuesto |
| **S-05** | La prioridad **Alta / Media** de cada RNF | El modelo no asigna prioridades; se leyó por el peso que da a cada atributo | `04` RNF | Que el propietario las confirme o las cambie |
| **S-06** | Los **hechos de negocio** que podrían ser eventos de dominio | El modelo **no define ningún evento**; se derivaron de las operaciones que nombra por agregado | `02` eventos | Solo hace falta si aparece un consumidor real |
| **S-07** | **No se fijan** objetivos de rendimiento, disponibilidad, volumen, copias de seguridad, observabilidad ni otros entornos | El modelo no da ninguna cifra; no se inventan | `04` RNF §3, `01` scope, `05` | Pedir objetivos al propietario |
| **S-08** | La autenticación usa una **credencial portadora** (token) | Solo se infiere de «sin token → 401» (§13 D-3). El modelo no nombra mecanismo ni duración de sesión | `04` HU-AUTH, RNF-07, `05` | Revisar `api-contract.md` (no entregado) |
| **S-09** | El **cliente** (interfaz) no está definido | El modelo describe almacenamiento, no cliente alguno | `05` mapa de piezas, `04` RNF §3 | Confirmar si hay interfaz y de qué tipo |
| **S-10** | Los **cinco `CHECK`** de T-20 (`price > 0`, `quantity > 0`, nombre de categoría no vacío, `role`, nombre en minúsculas) se asumen **no bajados** al motor | §4 los marca «solo dominio · T-20»; §13 da por saldadas solo `sale_id NOT NULL`, el único compuesto y `FK-3` | `04` RNF-01, `03` visión | **Medir contra el motor** con las consultas de §10 |
| **S-11** | La generación de identificadores y la hora de la venta son puertos o servicios del dominio | El modelo dice que los pone el dominio, no cómo (§3, §5) | `05` §5.2 | Decidir en la implementación |
| **S-12** | La **transacción** de «descontar stock y añadir la línea» la abre el caso de uso | El modelo dice que es una sola operación (§2.3) pero no dónde se abre la transacción | `05` §6 | Revisar `plan.md` (no entregado) |
| **S-13** | El mensaje de un inicio de sesión fallido **no revela** cuál dato falló | Buena práctica; el modelo no la menciona | `04` HU-AUTH-001 | Confirmar o retirar |
| **S-14** | El arranque **no inventa credenciales por defecto** y es **idempotente** | El modelo dice que el arranque crea al administrador con credenciales de entorno y que hoy hay una fila (§9.2, §10.4) | `04` HU-AUTH-003 | Confirmar el comportamiento ante credenciales ausentes y arranques repetidos |
| **S-15** | Las frases de trabajo del usuario (JTBD) | No hay entrevistas ni descripción de uso en el material | `03` problema | Validar con quien opera el comercio |
| **S-16** | **No se describe la solución actual** del usuario | El modelo no dice cómo se lleva hoy el inventario | `03` problema | Investigar con el usuario |
| **S-17** | Las **métricas de éxito** y los **resultados clave** son una lectura nuestra | El modelo no define métricas ni metas de producto | `03` problema y visión | Que el propietario las confirme |
| **S-18** | El almacenamiento externo de imágenes es un **almacén de objetos** | El modelo solo dice «almacenamiento externo» y «clave opaca» (§1, D-08); no nombra tipo ni proveedor | `01` scope | Confirmar el proveedor y el tipo |

---

## 2. Lo que el modelo no entrega y no se pudo reconstruir

| Qué | Dónde lo cita el modelo | Consecuencia |
|---|---|---|
| `constitution.md`, `spec.md`, `plan.md`, `architecture.md`, `tasks.md`, `api-contract.md`, `adr/`, `traspaso/HANDOFF-TECNICO.md` | §«Rige bajo», §12, §11.1 | Se citan **solo** por lo que el modelo dice de ellos |
| Contenido de `D-01` y `D-02` | §«Rige bajo» | No se puede reconstruir |
| Detalle de A-1 (más allá de «el alta era anónima») | §9.2, §13 D-3 | Solo se usa lo que el modelo afirma |
| Estado real de T-05, T-10, T-11 y T-13 | §2–§6 | Se informa la lectura del modelo y su contradicción interna (ver [`05-architecture/verification.md`](05-architecture/verification.md)) |

---

## 3. Supuestos que conviene validar primero

Ordenados por cuánto cambiarían el resultado si estuvieran mal:

1. **S-03** — permisos por rol: cambia las historias de autorización y el RNF-07.
2. **S-10** — estado de los cinco `CHECK`: cambia qué reglas son del motor y cuáles siguen siendo deuda.
3. **S-02** — un solo desplegable: cambia el estilo arquitectónico.
4. **S-08** — mecanismo de autenticación: cambia el puerto de seguridad.
