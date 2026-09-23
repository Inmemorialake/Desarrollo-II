# Product Backlog v1 — API Materiales/Fórmulas (Equipo 1)

23 de septiembre de 2026 · Andrés Gerardo González Rosero

## Contexto

Este backlog descompone las épicas EP-01, EP-02 y EP-03 en 6 historias de usuario (HU) para la API de Materiales/Fórmulas del Equipo 1, caso Cutit Saws (750021C Desarrollo de Software II, 2026-2).

**Marco:** ISO/IEC/IEEE 29148:2018; HU en modelo de Cohn validadas con INVEST; criterios de aceptación (CA) en Dado/Cuando/Entonces validados con SMART. Estimable (INVEST) y Achievable (SMART) quedan pendientes de validación por el equipo en todas las HU.

**Endpoints (fijos, sin agregar otros):** `POST /materiales` y `GET /materiales/{id}` (Caso de Estudio, §11).

**Integraciones:** consume Patent Sweep (Equipo 2); es consumida por Fabricación (Equipo 3) y por Patent Sweep.

**Decisiones derivadas del Paso 0:**

- Una fórmula se modela como un material con composición no vacía, porque no se agregan endpoints.
- La acción "actualizar" de EP-01 queda sin HU: exigiría un endpoint nuevo.
- El dictamen de patentes se expone dentro de la respuesta de `GET /materiales/{id}`.
- Si Patent Sweep falla, la fórmula queda en "pendiente de verificación" (confirmado por el equipo).
- Atributos, proporciones, contrato con Patent Sweep, efecto del dictamen y roles siguen abiertos: se registran como supuestos.

## Glosario de dominio

| Término | Definición operativa | Fuente |
| --- | --- | --- |
| Material (insumo) | Elemento físico usado en el laboratorio para fabricar cuchillas. Recurso de `POST /materiales` y `GET /materiales/{id}`. El caso no define sus atributos. | Procesos y Operación Actual; Caso de Estudio §11 |
| Material base | Material registrado sin composición (ver S3). | Supuesto S3 |
| Fórmula (receta) | Especificación predefinida de qué materiales y en qué proporción se aplican para fabricar una cuchilla. Se registra como material con composición no vacía (ver S3). | Procesos y Operación Actual; EP-02 |
| Composición | Conjunto de pares (material registrado, proporción) que forman una fórmula. | EP-02 |
| Cuchilla | Producto que sale del laboratorio; alcance de esta API. | Caso de Estudio §11 |
| Cadena | Producto ensamblado a partir de cuchillas; responsabilidad de Fabricación (Equipo 3). | Procesos y Operación Actual |
| Dictamen de riesgo de patente | Resultado que Patent Sweep devuelve sobre una fórmula; sus valores los define el Equipo 2. | EP-03; Caso de Estudio §11 |
| Pendiente de verificación | Estado de riesgo de una fórmula cuando Patent Sweep no respondió o respondió con error. | Respuesta del equipo al Paso 0 |

## Supuestos asumidos

Ocho supuestos cubren lo que el caso no define; cada HU indica cuáles usa.

| ID | Supuesto asumido | Origen |
| --- | --- | --- |
| S1 | Roles: "responsable del laboratorio de Cutit" (EP-01, EP-02) y "encargado de innovación/PI" (EP-03). | Lista de usuarios del prompt; el caso solo menciona "el laboratorio de Cutit". |
| S2 | Campos del material: nombre y tipo (obligatorios); composición (opcional). | Ejemplo HU-01 del prompt; el caso no define atributos. |
| S3 | Una fórmula se registra con `POST /materiales` como material con composición no vacía; un material base tiene composición vacía. | Restricción de no agregar endpoints. |
| S4 | Cada componente tiene un ID de material y una proporción numérica mayor que 0. Unidad y regla de suma (p. ej., 100 %) pendientes. | EP-02 ("proporciones exactas"). |
| S5 | Al registrar una fórmula se envían sus componentes a Patent Sweep y se guarda el dictamen tal como llega. Los materiales base no se verifican. Endpoint, payload y valores del dictamen se acuerdan con el Equipo 2. | Integración SOA de EP-03. |
| S6 | El dictamen es informativo: no bloquea el registro ni la consulta. | Efecto del dictamen sin definir; el caso solo dice "anticipar". |
| S7 | El valor del timeout hacia Patent Sweep se acuerda con el Equipo 2. | El estado "pendiente de verificación" sí está confirmado. |
| S8 | Códigos HTTP: 201 al crear, 200 al consultar, 400 por validación, 404 por recurso inexistente. | Propuesta del Paso 0, no objetada. |

## EP-01: Gestión del Catálogo de Materiales e Insumos

Dos HU cubren registro y consulta; la actualización queda fuera por la restricción de endpoints.

### HU-01: Registrar material base

**Fuente:** "la fabricación de las cuchillas en el laboratorio de Cutit [...] requiere el uso de materiales específicos" (Procesos y Operación Actual).

Como responsable del laboratorio de Cutit, quiero registrar un material base con su nombre y tipo, para que quede disponible como componente de las fórmulas de fabricación de cuchillas.

- **CA1 (Éxito):** Dado un `POST /materiales` con nombre y tipo y sin composición, cuando se procesa, entonces responde 201 con un identificador único y el material queda consultable en `GET /materiales/{id}`.
- **CA2 (Falla):** Dado un `POST /materiales` sin nombre o sin tipo, cuando se valida, entonces responde 400, indica qué campo falta y no crea ningún registro.

**Supuestos:** S1, S2, S3, S8. **Estimable / Achievable:** pendiente de validación por el equipo.

### HU-02: Consultar material por identificador

**Fuente:** endpoint mínimo "GET /materiales/{id}" (Caso de Estudio, §11 Tabla Formal de APIs).

Como responsable del laboratorio de Cutit, quiero consultar un material por su identificador, para verificar los datos con que quedó registrado.

- **CA1 (Éxito):** Dado un ID existente, cuando se hace `GET /materiales/{id}`, entonces responde 200 con nombre y tipo idénticos a los registrados.
- **CA2 (Falla):** Dado un ID que no existe, cuando se hace `GET /materiales/{id}`, entonces responde 404 y no devuelve datos de ningún material.

**Supuestos:** S1, S2, S8. **Estimable / Achievable:** pendiente de validación por el equipo.

## EP-02: Gestión de Fórmulas y Recetas de Fabricación

Dos HU: registrar la fórmula con su composición y exponerla a Fabricación, ambas sobre los endpoints existentes.

### HU-03: Registrar fórmula con su composición

**Fuente:** "centralice la gestión de materiales y fórmulas [...] garantizando trazabilidad" (Problemas del Negocio y Necesidades).

Como responsable del laboratorio de Cutit, quiero registrar una fórmula indicando qué materiales registrados la componen y en qué proporción, para estandarizar la receta con que se fabrica cada cuchilla.

- **CA1 (Éxito):** Dado un `POST /materiales` con nombre, tipo y una composición en la que cada componente referencia un material existente con proporción mayor que 0, cuando se procesa, entonces responde 201 con un ID y guarda exactamente esos componentes y proporciones.
- **CA2 (Falla):** Dado un componente que referencia un ID de material inexistente, cuando se valida, entonces responde 400 indicando el ID no encontrado y no crea la fórmula.
- **CA3 (Borde):** Dado un componente sin proporción o con proporción ≤ 0, cuando se valida, entonces responde 400 indicando el componente inválido y no crea la fórmula.

**Supuestos:** S1, S2, S3, S4, S8. **Estimable / Achievable:** pendiente de validación por el equipo.

### HU-04: Exponer la composición de una fórmula a Fabricación

**Fuente:** "Equipo 3 – Fabricación [...] Consume APIs: Materiales/Fórmulas" (Caso de Estudio, §11).

Como API de Fabricación (Equipo 3), quiero obtener la composición de una fórmula por su identificador, para validar mis registros de fabricación contra una receta registrada.

- **CA1 (Éxito):** Dado el ID de una fórmula existente, cuando se hace `GET /materiales/{id}`, entonces responde 200 con la lista de componentes (ID de material y proporción) idéntica a la registrada.
- **CA2 (Falla):** Dado un ID inexistente, cuando se consulta, entonces responde 404, de modo que Fabricación pueda distinguir una fórmula no registrada.

**Supuestos:** S3, S4, S8. **Estimable / Achievable:** pendiente de validación por el equipo.

## EP-03: Verificación de Riesgo de Patentes

Dos HU: verificar cada fórmula contra Patent Sweep al registrarla y exponer su estado de riesgo; ambas dependen del contrato con el Equipo 2.

### HU-05: Verificar riesgo de patente al registrar una fórmula

**Fuente:** "se realiza un barrido periódico de patentes [...] con el fin de anticipar riesgos de infracción o de competencia" (Procesos y Operación Actual); "Equipo 1 [...] Consume APIs: Patent Sweep" (Caso de Estudio, §11).

Como encargado de innovación/PI, quiero que cada fórmula registrada se someta a verificación en Patent Sweep, para anticipar riesgos de infracción antes de que pase a fabricación.

- **CA1 (Éxito):** Dado un registro de fórmula válido y Patent Sweep disponible, cuando se registra, entonces se envían sus componentes a Patent Sweep y el dictamen devuelto queda asociado a la fórmula sin modificaciones.
- **CA2 (Falla):** Dado que Patent Sweep no responde dentro del timeout acordado o responde con error, cuando se registra la fórmula, entonces se crea igual (201) con estado de riesgo "pendiente de verificación".
- **CA3 (Borde):** Dado un dictamen que indica riesgo, cuando se recibe, entonces el registro no se bloquea y el dictamen se almacena tal cual.

**Supuestos:** S1, S5, S6, S7. **Estimable / Achievable:** pendiente de validación por el equipo.

### HU-06: Consultar el estado de riesgo de una fórmula

**Fuente:** "con el fin de anticipar riesgos de infracción o de competencia" (Procesos y Operación Actual).

Como encargado de innovación/PI, quiero consultar el estado de riesgo de patente de una fórmula, para decidir si requiere revisión antes de fabricarse.

- **CA1 (Éxito):** Dada una fórmula con dictamen, cuando se hace `GET /materiales/{id}`, entonces la respuesta incluye el estado de riesgo idéntico al recibido de Patent Sweep.
- **CA2 (Borde):** Dada una fórmula cuya verificación falló, cuando se consulta, entonces la respuesta incluye el estado "pendiente de verificación".

**Supuestos:** S1, S5, S7. **Estimable / Achievable:** pendiente de validación por el equipo.

## Tabla resumen

6 HU y 14 CA en total; todas las HU tienen al menos un CA de éxito y uno de falla o borde.

| ID | Épica | HU | # CA | Supuestos |
| --- | --- | --- | --- | --- |
| HU-01 | EP-01 | Registrar material base | 2 | S1, S2, S3, S8 |
| HU-02 | EP-01 | Consultar material por identificador | 2 | S1, S2, S8 |
| HU-03 | EP-02 | Registrar fórmula con su composición | 3 | S1, S2, S3, S4, S8 |
| HU-04 | EP-02 | Exponer la composición a Fabricación | 2 | S3, S4, S8 |
| HU-05 | EP-03 | Verificar riesgo al registrar una fórmula | 3 | S1, S5, S6, S7 |
| HU-06 | EP-03 | Consultar el estado de riesgo | 2 | S1, S5, S7 |

## Autorrevisión y pendientes

Cinco puntos quedan abiertos tras la revisión INVEST/SMART; ninguno se puede cerrar sin el equipo o el profesor.

- **Brecha en EP-01:** "actualizar" no tiene HU porque exige un endpoint nuevo (`PUT` o `PATCH`). Consultarlo con el profesor.
- **Independencia (INVEST):** HU-04 y HU-06 necesitan datos creados por HU-03 y HU-05. Se mitiga con fixtures en pruebas; tenerlo en cuenta al priorizar.
- **Medible (SMART):** CA2 de HU-05 solo es medible cuando se fije el timeout.
- **Fórmulas anidadas:** no se sabe si una fórmula puede tener otra fórmula como componente.
- **Reintento de pendientes:** no hay mecanismo para reverificar fórmulas en "pendiente de verificación"; candidata a HU cuando exista el contrato con Patent Sweep.

### Agenda para la reunión con el Equipo 2 (Patent Sweep)

- [ ] Qué endpoint llamar: `POST /patentes/busqueda` o `GET /patentes/estado`.
- [ ] Qué payload enviar: IDs de material o la composición completa.
- [ ] Qué valores puede tener el dictamen.
- [ ] Cuál será el timeout.
- [ ] Para qué consumirá el Equipo 2 `GET /materiales/{id}` (evitar dependencia circular en el despliegue).

## Respuesta del validador

