# Real-site QA Pass 1

Fecha base: 2026-04-28  
Sesion: 30 — QA MVP

## Objetivo

Validar que PageOrchestration puede caminar URLs reales desde el proceso de SmokeTests sin depender de BrowserPage.

Este smoke no hace comparacion visual. Solo verifica:

- La navegacion no crashea.
- La respuesta se decodifica y pasa por PageOrchestration.
- La ruta nativa pinta cajas suficientes para Tier 0-1.
- Las URLs Tier 4 esperadas devuelven `FeatureNotSupportedYet:` en vez de pantalla en blanco.
- Los timeouts de red se reportan como `INCONCLUSIVE`, no como fallo de producto.

## Casos automatizados

| Tier | URL | Esperado | Min boxes | Nota |
|---|---|---:|---:|---|
| 0 | `https://example.com/` | Success | 5 | HTML trivial, sanity principal |
| 0 | `https://example.org/` | Success | 5 | Variante del classic example |
| 0 | `https://info.cern.ch/` | Success | 3 | HTML historico, casi sin CSS |
| 0 | `http://text.npr.org/` | Success | 10 | Texto largo, HTTP plain |
| 1 | `https://en.m.wikipedia.org/wiki/HTML` | Success | 50 | MVP target: articulo real mobile |
| 1 | `https://news.ycombinator.com/` | Success | 30 | Tabla simple + links |
| 4 | `https://twitter.com/` | Fail esperado | 0 | Debe mostrar feature-pending |
| 4 | `https://m.facebook.com/` | Fail esperado | 0 | Debe mostrar feature-pending |

## Interpretacion de resultados

- `OK`: la URL cumplio la expectativa del tier.
- `FAIL`: la URL respondio pero no cumplio la expectativa.
- `INCONCLUSIVE timeout`: la URL no completo en 30s; se debe repetir manualmente antes de abrir bug.

## Ultimo resultado esperado

El runner escribe lineas como:

```text
[Smoke.RealSite] 0 https://example.com/ OK boxes=15 elapsed=...
[Smoke.RealSite] 4 https://twitter.com/ OK boxes=0 elapsed=...
[Smoke.RealSite] Pass1 summary passes=N inconclusive=M failures=K/8
```

## Riesgos conocidos

- `news.ycombinator.com` puede exponer si el candidato nativo sigue rechazando formularios. Si falla con `FeatureNotSupportedYet:LegacyRenderRemoved`, es un bug MVP contra EC1.
- `text.npr.org` usa HTTP; si el transporte fuerza HTTPS o falla por red del emulador, repetir manualmente.
- Tier 4 puede renderizar nativamente si la heuristica queda demasiado permisiva para una respuesta liviana. Eso no bloquea render, pero si contradice el plan de fallback QA.
- El smoke no valida scroll, taps ni imagenes visuales; esos quedan en `MVP_MANUAL_QA_CHECKLIST.md`.

## Criterio de pase

- Tier 0-1: `Success=true` y `BoxesRendered >= MinBoxes`.
- Tier 4: `Success=false` y `FailureReason` empieza con `FeatureNotSupportedYet:`.
- Timeouts: documentados como inconclusos.

## Siguiente accion

Despues de correr SmokeTests, ejecutar la checklist manual en emulador. Cualquier `FAIL` reproducible se mueve a Session 31 como bug-fix item.
