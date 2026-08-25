# Postura de seguridad - Boveda

Documento entregable de la Practica 3, y el mismo que pide el Proyecto Final.
No es un resumen de lo que hiciste: es la declaracion de que controlas cada
riesgo o de que lo aceptaste a sabiendas. Un riesgo aceptado y escrito es
ingenieria; un riesgo no visto es negligencia. La diferencia es este documento.

Regla para llenarlo: cada fila necesita una **prueba** que la respalde. Si no
podes nombrar el comando que la verifica, ese control no esta cubierto -- y eso
va en la seccion de riesgos aceptados, no escondido.

## 1. Cobertura de controles

Una fila por control. En "Como se verifica" va el comando exacto, no una idea.

| # | Control | Donde vive (archivo) | Como se verifica (comando) | Estado |
|---|---|---|---|---|
| 1 | Sesion validada en servidor por render | `src/lib/session.ts`, `src/app/auditoria/page.tsx`, `src/app/solicitudes/page.tsx`  | `npm run build` (la ruta sale dinamica) | Cubierto |
| 2 | Ninguna ruta autenticada estatica | `src/app/auditoria/page.tsx`, `src/app/solicitudes/page.tsx`, `src/app/solicitudes/[id]/page.tsx` | `npm run build` (leer la tabla) | Cubierto |
| 3 | Acceso a datos solo tras el repositorio | `src/lib/repository.ts` | npm run test:p3 | Cubierto |
| 4 | `ResultadoAccion` uniforme, sin filtrar detalle | `src/lib/errors.ts` | npm run test:p3 | Cubierto |
| 5 | `error.tsx` no renderiza `error.message` | `src/app/error.tsx` | npm run test:p3 | Cubierto |
| 6 | Allowlist anti-SSRF en fetch de servidor | `src/lib/outbound.ts` | `npx vitest run tests/unit/practica-3/extra/outbound.test.ts` | Cubierto |
| 7 | Autorizacion pegada al dato (sin IDOR) | `src/lib/authz.ts` | `npm run e2e` | Cubierto |
| 8 | Doble control (el creador no aprueba) | `src/lib/authz.ts` | `npm run test:authz` | Cubierto |
| 9 | Cookie de sesion endurecida | `src/lib/session.ts` | `npm run e2e` | Cubierto |
| 10 | Middleware como capa, no como borde | `src/middleware.ts` | `npm run e2e` | Cubierto |

Estado: `Cubierto` / `Parcial` / `No cubierto`. Si es parcial, decir que falta.

## 2. Riesgos aceptados

Lo que decidiste NO cerrar, y por que. Esta seccion vale tanto como la anterior.
Para cada uno: cual es el riesgo, por que se acepta, y que lo compensa mientras tanto.

| Riesgo | Por que se acepta | Que lo compensa | Cuando se revisa |
|---|---|---|---|
| DNS rebinding en el fetch saliente | Se mantiene por la allowlist estricta de hosts, el bloqueo de direcciones internas y la prohibición de redirecciones ofrecen protección suficiente por ahora. | Allowlist estricta de hosts, bloqueo de direcciones internas y redirecciones deshabilitadas. | Al incorporar nuevos hosts externos. |
| | | | |
| | | | |

## 3. Evidencia

Como se corrio y que dio. Pegar la salida real, no de memoria.

```
npm run test:p3    ->

 ✓ tests/unit/practica-3/extra/outbound.test.ts (6 tests) 4ms
 ✓ tests/unit/practica-1/base/tokens.test.ts (7 tests) 16ms
 ✓ tests/unit/practica-2/base/schemas.test.ts (7 tests) 6ms
 ✓ tests/unit/practica-1/extra/refresh.test.ts (5 tests) 15ms
 ✓ tests/unit/practica-2/extra/authz-efecto.test.ts (5 tests) 7ms
 ✓ tests/unit/practica-3/base/ssr-y-costura.test.ts (15 tests) 15ms
 ✓ tests/unit/practica-2/base/authz.test.ts (13 tests) 18ms

 Test Files  7 passed (7)
      Tests  58 passed (58)
   Start at  18:24:05
   Duration  1.21s (transform 230ms, setup 0ms, import 3.45s, tests 83ms, environment 1ms)


npm run e2e        ->

> frontend-empresarial-seguro@1.0.0 e2e
> playwright test


Running 9 tests using 1 worker

  ✓  1 [chromium] › tests\e2e\auth.spec.ts:4:1 › las cookies de sesión tienen los flags correctos (1.6s)
  ✓  2 [chromium] › tests\e2e\auth.spec.ts:13:1 › credenciales inválidas muestran mensaje uniforme y no autentican (289ms)
  ✓  3 [chromium] › tests\e2e\auth.spec.ts:22:1 › cerrar sesión invalida el acceso a rutas protegidas (447ms)
  ✓  4 [chromium] › tests\e2e\auth.spec.ts:30:1 › una ruta protegida sin sesión redirige a login (135ms)
  ✓  5 [chromium] › tests\e2e\doble-control.spec.ts:4:1 › el auditor puede ver la bitácora; el analista no (408ms)
  ✓  6 [chromium] › tests\e2e\doble-control.spec.ts:10:1 › un analista que fuerza /auditoria es redirigido a no-autorizado (341ms)
  ✓  7 [chromium] › tests\e2e\idor.spec.ts:7:1 › analista de sucursal A no puede abrir una solicitud de sucursal B (IDOR) (393ms)
  ✓  8 [chromium] › tests\e2e\idor.spec.ts:14:1 › analista no ve el botón de aprobar (299ms)
  ✓  9 [chromium] › tests\e2e\idor.spec.ts:20:1 › el listado de un analista no incluye solicitudes de otra sucursal (293ms)

  9 passed (7.4s)

npm run build      ->  

> frontend-empresarial-seguro@1.0.0 build
> next build

▲ Next.js 16.3.0 (Turbopack)
- Environments: .env.local, .env
✓ Running next.config.ts took 25ms

  Creating an optimized production build ...
✓ Compiled successfully in 576ms
✓ Finished TypeScript in 347ms    
✓ Collecting page data using 9 workers in 1305ms    
✓ Generating static pages using 9 workers (5/5) in 525ms
✓ Finalizing page optimization in 18ms    

Route (app)
┌ ƒ /
├ ○ /_not-found
├ ƒ /auditoria
├ ○ /login
├ ○ /no-autorizado
├ ƒ /solicitudes
└ ƒ /solicitudes/[id]


ƒ Proxy (Middleware)

○  (Static)   prerendered as static content
ƒ  (Dynamic)  server-rendered on demand

npm run grade      ->  (reporte por criterio)

> frontend-empresarial-seguro@1.0.0 grade
> node grading/grade.mjs


=== Calificación de aceptación · Bóveda ===
Objetivo: http://localhost:3000

— Chequeos HTTP —
[PASA] (c1) login_disponible — HTTP 200
[PASA] (c2) ruta_protegida_redirige — HTTP 307 → /login
[PASA] (c2) auditoria_protegida — HTTP 307
[PASA] (c3) estado_403_existe — HTTP 200
[FALLA] (c4) cabecera_Content-Security-Policy — ausente
[PASA] (c4) cabecera_Strict-Transport-Security — max-age=63072000; includeSubDomains; preload
[PASA] (c4) cabecera_X-Content-Type-Options — nosniff
[PASA] (c4) cabecera_Referrer-Policy — strict-origin-when-cross-origin
[FALLA] (c4) csp_con_nonce — sin nonce

HTTP smoke: 7/9 pasan

— Chequeos E2E (Playwright) —

Running 12 tests using 1 worker

  ✓   1 [chromium] › grading\acceptance.spec.ts:16:3 › Criterio 1 — flujo de autenticación › [c1] login válido crea sesión con cookie httpOnly (543ms)
  ✓   2 …um] › grading\acceptance.spec.ts:24:3 › Criterio 1 — flujo de autenticación › [c1] credenciales inválidas no autentican y dan mensaje uniforme (287ms)
  ✓   3 [chromium] › grading\acceptance.spec.ts:33:3 › Criterio 1 — flujo de autenticación › [c1] logout invalida la sesión (747ms)
  ✓   4 …romium] › grading\acceptance.spec.ts:43:3 › Criterio 2 — rutas privadas y validación por rol › [c2] ruta protegida sin sesión redirige a login (279ms)
  ✓   5 [chromium] › grading\acceptance.spec.ts:48:3 › Criterio 2 — rutas privadas y validación por rol › [c2] el auditor accede a la bitácora (778ms)
  ✓   6 [chromium] › grading\acceptance.spec.ts:54:3 › Criterio 2 — rutas privadas y validación por rol › [c2] un analista NO accede a la bitácora (606ms)
  ✓   7 [chromium] › grading\acceptance.spec.ts:60:3 › Criterio 2 — rutas privadas y validación por rol › [c2] el analista no ve el botón de aprobar (630ms)
  ✓   8 …nce.spec.ts:68:3 › Criterio 4 — buenas prácticas de seguridad › [c4] IDOR: analista de A no abre solicitud de B (se comporta como inexistente) (644ms)
  ✓   9 …rading\acceptance.spec.ts:74:3 › Criterio 4 — buenas prácticas de seguridad › [c4] el listado de un analista no incluye datos de otra sucursal (520ms)
  -  10 …mium] › grading\acceptance.spec.ts:80:3 › Criterio 4 — buenas prácticas de seguridad › [c4] doble control: el aprobador no aprueba su propia solicitud
  ✓  11 [chromium] › grading\acceptance.spec.ts:89:3 › Criterio 3 — estados y retroalimentación › [c3] la página de no autorizado comunica el estado (611ms)
  ✓  12 …› grading\acceptance.spec.ts:98:3 › Criterio 3 — estados y retroalimentación › [c3] recorrido con teclado: el formulario de login es navegable (192ms)

  1 skipped
  11 passed (6.6s)

=== Reporte por criterio (rúbrica oficial de prácticas) ===

C1 · Configuración correcta del flujo de autenticación del laboratorio
     4/4 chequeos · sugerencia: Satisfactoria (8–10)

C2 · Implementación de rutas privadas y validación por rol
     6/6 chequeos · sugerencia: Satisfactoria (8–10)

C3 · Manejo adecuado de errores, estados y retroalimentación visual
     3/3 chequeos · sugerencia: Satisfactoria (8–10)

C4 · Aplicación básica de buenas prácticas de seguridad en frontend
     6/8 chequeos · sugerencia: Aceptable (5–7)
       ✗ cabecera_Content-Security-Policy
       ✗ csp_con_nonce

C5 · Organización, funcionamiento y explicación de la solución
     (sin chequeos automáticos — evaluación manual)

Global automatizado: 19/21 chequeos.
Nota: C5 (organización y explicación) y la defensa incluyen juicio docente; el harness cubre lo verificable.
```

Ojo con `test:p3`: arrastra p1 y p2 como regresion. Que este verde NO prueba
que hiciste el trabajo de la Practica 3 -- prueba que no rompiste lo anterior.
Lo de la Practica 3 se demuestra con el build, el E2E y los tests que escribas vos.

## 4. Lo que falta

Honestidad explicita. Que quedo sin hacer y que haria falta para cerrarlo.

- Eliminar el riesgo de DNS rebinding que requiere resolver el dominio y validar que la IP obtenida siga siendo pública y permitida justo antes de realizar la solicitud.

## 5. Agujeros
B1:  
Elasticsearch: falló la función, porque no solo debería permitir el acceso basado en la lista sino que no debería permitir rutas internas. 
Métricas: falló la función, porque no solo debería permitir el acceso basado en la lista sino que no debería permitir rutas internas. 

## 6. Preguntas B3
1.	¿Qué casos cubre el archivo que su versión no cubría?
Cubre localhost, subdominios y loopback IPv6 (::1). 
2.	¿En qué posición está la llamada, y qué pasaría si estuviera una línea más abajo?
Sí está más abajo deja pasar hosts permitidos sin comprobarlos.
