# MVP Manual QA Checklist

> Ejecutar en emulador despues de desplegar el build de Session 30.
> Marcar cada item con OK/FAIL y notas. Cualquier FAIL reproducible entra a Session 31.

## Setup

- [ ] Deploy `Vianium.Browser.App` x86 en Emulator 8.1 WVGA 4 inch 512MB.
- [ ] Abrir Settings y confirmar `pipeline.use_native_pipeline = true`.
- [ ] Confirmar `javascript.use_native_js = true`.
- [ ] Confirmar `emergency.force_legacy_pipeline = false`.
- [ ] En Debug Output, verificar una linea:

```text
[QA] Settings: native_pipeline=True native_js=True emergency=False
```

## EC1 — 5 sitios reales cargan sin crash

- [ ] Wikipedia mobile: `https://en.m.wikipedia.org/wiki/HTML` — texto y estructura visibles.
- [ ] ejemplo.com: `https://www.ejemplo.com/` — texto acentuado correcto.
- [ ] Blog simple: `https://danluu.com/` — articulos legibles.
- [ ] HN homepage: `https://news.ycombinator.com/` — posts visibles; si aparece Coming soon, marcar FAIL MVP.
- [ ] GitHub README: `https://github.com/oven-sh/bun` — repo read-only renderiza o falla con diagnostico claro.

## EC2 — Click en links navega

- [ ] Abrir `https://example.com/`.
- [ ] Tocar "Learn more".
- [ ] Confirmar navegacion a IANA sin abrir IE externo.
- [ ] Abrir Wikipedia.
- [ ] Tocar un link interno.
- [ ] Confirmar que la barra cambia y que el nuevo documento renderiza.

## EC3 — Scroll vertical

- [ ] Abrir Wikipedia HTML.
- [ ] Scroll hacia abajo varias pantallas.
- [ ] Confirmar que el contenido no se queda congelado.
- [ ] Abrir HN.
- [ ] Scroll hasta posts inferiores.
- [ ] Confirmar que no hay jank severo ni crash.

## EC4 — Imagenes `<img>`

- [ ] Abrir una pagina con imagenes livianas, por ejemplo BBC mobile o Wikipedia con imagen.
- [ ] Confirmar al menos 3 imagenes visibles.
- [ ] Confirmar que URLs relativas y absolutas cargan.
- [ ] Si una imagen falla, revisar logs de `XamlRenderSurface.DrawImage` / `BitmapImage`.

## EC5 — JS handlers basicos

- [ ] Abrir una pagina sintetica con `<button onclick="alert('test')">`.
- [ ] Tocar el boton.
- [ ] Confirmar que el handler se ejecuta.
- [ ] Abrir una pagina sintetica con `addEventListener('click', ...)`.
- [ ] Tocar el nodo.
- [ ] Confirmar console/log o efecto visual.

## EC6 — Tier 4 Coming soon fallback

- [ ] Navegar a `https://twitter.com/`.
- [ ] Confirmar pantalla "Coming soon", no canvas blanco.
- [ ] Navegar a `https://m.facebook.com/`.
- [ ] Confirmar pantalla "Coming soon", no crash.
- [ ] Confirmar que el mensaje menciona features pendientes del MVP.

## Logs check

Despues de cada navegacion, confirmar:

- [ ] `[PageNav.Html] url=...`
- [ ] `[PageNav.Pipeline] decision=...`
- [ ] `[PageNav.Native.Dom] parsed root=True elements=N`
- [ ] `[PageNav.Native.Render] painted N boxes`
- [ ] `=== [PageNav] COVERAGE REPORT url=... ===`
- [ ] `[PageNav.Coverage] timing fetch=... parse=... style=... layout=... render=... total=...`

## Performance sanity

- [ ] `example.com` carga bajo 1s despues de warm-up.
- [ ] Wikipedia carga bajo 5s en emulador.
- [ ] 10 navegaciones consecutivas no dejan memoria creciendo sin bajar.
- [ ] La UI responde al volver desde tab switcher.

## Issues for Session 31

Copiar por cada FAIL:

```text
Issue:
URL/scenario:
Expected:
Actual:
Severity: P0/P1/P2
Logs relevantes:
```

## Cierre

- [ ] No P0 abiertos.
- [ ] P1 documentados para bug-fix buffer.
- [ ] P2 movidos a polish post-MVP.
