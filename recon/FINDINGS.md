# Fase 0 — Reconocimiento (resultado: DETENIDO)

Fecha: 2026-10-09

## Qué se probó

| # | Cliente | URL | Resultado |
|---|---------|-----|-----------|
| 1 | curl (UA de Chrome, `es-CL`) | `https://www.viajesfalabella.cl/` | **403**, DataDome `hard_block` |
| 2 | curl | `https://viajes.falabella.com` | El proxy de salida del entorno lo rechazó (502); el dominio vigente es `viajesfalabella.cl` |
| 3 | Chromium con interfaz (Playwright + Xvfb), `es-CL`, `America/Santiago` | portada | **403** → redirige a `geo.captcha-delivery.com/captcha` |
| 4 | Ídem | `/shop/flights/results/roundtrip/CCP/RIO/2027-03-03/2027-03-09/2/1/0?kids=3` | **403**, mismo bloqueo |

Evidencia:
- `recon/fixtures/blocked-datadome-403.html`: respuesta sanitizada (se redactaron cid, cookie y hashes)
- `recon/datadome-block-chromium.png`: "El acceso está restringido temporalmente"

Cabeceras relevantes de la respuesta:

```
server: cloudflare
x-datadome: protected
x-datadome-isbot: 1
x-datadome-traffic-rule-response: hard_block
x-datadome-ruletype: AI Threats Detection
x-datadome-botname: [Babbage] Bot signature mutations (Protect)
```

## Conclusión

El sitio está detrás de **Cloudflare + DataDome**. Desde una IP de datacenter, DataDome entrega
captcha o bloqueo duro **antes** de cargar la SPA, incluso a un navegador real con interfaz.
Por eso no se pudo capturar ningún XHR del buscador: no hay endpoints, fixtures de resultados,
ni datos sobre puntos o equipaje.

Estrategia: **(c) no factible desde Cloudflare** según el diseño pedido.
- (a) `fetch` directo desde un Worker: un Worker no ejecuta el JS de DataDome, así que recibiría
  el mismo 403. (No se pudo desplegar la prueba: el entorno no tiene credenciales de Cloudflare.)
- (b) Browser Rendering: corre en IPs de datacenter de Cloudflare, que DataDome clasifica igual
  que el caso 3. Pasar el bloqueo requeriría resolver captchas o evadir la detección, y eso
  queda fuera de alcance.

Se cumple la condición de detención: "el sitio responde con captcha o bloqueo persistente".
