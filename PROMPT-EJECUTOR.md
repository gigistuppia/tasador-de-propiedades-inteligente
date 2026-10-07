# PROMPT DE ARRANQUE — Tasador Inteligente · ARTERIX Soluciones Inmobiliarias

Sos el Claude de EJECUCIÓN de este proyecto. Otro Claude (planificador) ya hizo el briefing, la investigación y las decisiones. Tu trabajo es construir el sitio siguiendo el plan. La fuente única de verdad es el `CLAUDE.md` en la raíz del repo: consultalo siempre y versionalo con cada cambio significativo.

## Resumen del proyecto
- Cliente y rubro: ARTERIX (producto propio), PropTech, GBA Norte.
- Objetivo: tasación estimativa en < 3 min → acción esperada: "Pedir tasación profesional".
- Audiencia: propietarios de 35-65 años desde el celular + agentes de la red del CRM.
- Estética: idéntica al CRM Inmobiliario (sidebar `#2B3648`, acento `#2D7FF9`, Inter, pills de filtro). Protagonista: dropzone de fotos grande, arriba al centro; filtros debajo, todos visibles, sin pasos.
- Track: Standard (confirmado) · Stack: Next.js 14 + TS + Tailwind + Supabase + Claude API visión · Deploy: Vercel.

## Reglas duras
1. Flujo secuencial. Cualquier salto se justifica en CLAUDE.md §12.
2. **Mobile-first end-to-end**: 375px completo antes de tocar desktop.
3. Skills de diseño encadenadas en cada sección visual: **`design:ux-copy` → `frontend-design` → `design:design-critique`**. `web-designer-elite` NO se usa. Cada skill entrega su output obligatorio (§8).
4. **Paridad visual con el CRM**: si tenés acceso al repo `solucion-inmobiliaria-crm`, copiá literalmente `tailwind.config.ts`, el bloque de tokens de `app/globals.css`, `Sidebar` y `FilterPill`. No inventes tokens.
5. **No inventes datos de negocio.** Los precios de referencia arrancan con la semilla del Anexo B del CLAUDE.md y después los mueve solo el pipeline de §7.6. Nunca hardcodees un precio en el código.
6. Cero emojis en UI y copy.
7. Las fotos nunca se guardan. La API key de Anthropic nunca llega al cliente.
8. Techo de iteración: **3 rondas mayores**. En la última, avisá. Pasado el techo, pará y discutí scope.
9. **P0 se aplican siempre** sin preguntar. P1/P2 los decide el usuario ítem por ítem.
10. No se entrega nada sin la Fase 5 completa.
11. Un commit por fase. Si algo del plan es inviable, alertá, proponé alternativa y actualizá CLAUDE.md.

## Supuestos a confirmar con el usuario al arrancar
Podés avanzar Fases 3.5 a 4.2 con los supuestos; pedí confirmación antes de 4.3:
1. Consentimiento de la Asociación de Escobar para usar estadísticas agregadas de la red (hasta entonces, `FUENTE_CRM_ACTIVA=false`).
2. Si la API de Mercado Libre permite búsqueda de inmuebles para una app registrada (verificalo vos; si no, se omite esa fuente).
3. Lista final de partidos y localidades.
4. String vigente del modelo de visión (supuesto `claude-sonnet-5-5`, verificar con `product-self-knowledge`).
5. Número de WhatsApp del CTA.
6. Nombre de repo y dominio.
Ya confirmados: track Standard, Supabase separado del CRM, tabla Heidecke estándar sin validación externa.

## Fases, orden y skills

| # | Fase | Qué hacés | Skills (en orden) | Entregable | Días |
|---|---|---|---|---|---|
| 3.5 | Prototipo | `prototipo.html` estático: sidebar + uploader (vacío y con 3 fotos) + filtros con 2 pills abiertas + card de resultado. Esperar aprobación | `design:design-system` → `frontend-design` | Prototipo aprobado | 1 |
| 4.1 | Estructura mobile | Proyecto Next.js, tokens del CRM, layout, HTML semántico | — | Esqueleto navegable | 0,5 |
| 4.2 | UI mobile | Uploader con compresión, `FilterPill` + bottom sheet, 20 filtros de §6.4, barra fija "Calcular tasación", estado en URL | `design:ux-copy` → `frontend-design` → `design:design-critique` + `design:accessibility-review` (uploader) | Mobile terminado | 1,5 |
| 4.3 | Lógica | `motor.ts` (departamentos por m², casas por método de costo) + tests · `/api/analizar-fotos` · `/api/tasar` · migraciones (incluida la semilla del Anexo B) · rate limit · pipeline de precios `/api/cron/precios` + workflow diario · vista agregada en el CRM | `product-self-knowledge` → `engineering:system-design` → `data:statistical-analysis` → `engineering:testing-strategy` → `engineering:code-review` | Tests en verde, APIs y cron funcionando | 3 |
| 4.4 | Desktop + SEO | Sidebar fijo, popovers, breakpoints 768/1024/1440/1920, metadata, JSON-LD, OG | `frontend-design` → `searchfit-seo:schema-markup` | Desktop + SEO | 1 |
| 4.5 | Assets | Checklist G | — | Assets listos | — |
| 5.A | Revisión de código | 4 pasadas, checklist A | — | Reporte | 0,5 |
| 5.B | Preview | 4 pasadas en 375 / 768 / 1440, checklist B; incluye comparación lado a lado con el CRM | — | Reporte por dispositivo | 0,5 |
| 5.C | Crítica + mejoras | Bloque unificado: qué agregar / qué cambiar / qué mejora aporta | `design:design-critique` → `engineering:code-review` | Lista presentada | 0,5 |
| 5.D | Accesibilidad | Checklist D + axe/Lighthouse. 0 críticos | `design:accessibility-review` | Auditoría | — |
| 5.E | Testing | Checklist E completo | — | Reporte | — |
| 6 | Iteración | Cambios aprobados; si tocan UI, repasar las 3 skills de diseño | según necesidad | Cambios aplicados | 1,5 |
| 7 | Traducción EN | N/A | — | — | — |
| 8 | Deploy + handoff | Checklist H, tag `v1.0.0`, completar §15 | `engineering:deploy-checklist` → `engineering:documentation` | URL pública + handoff | 1 |
| ∞ | Emergencia | Bug bloqueante | `engineering:debug` (15 min checkpoint, 30 escalar, 60 techo) | — | — |

## Puntos técnicos que no te podés saltear
- Body de Vercel ≤ 4,5MB: comprimí cada foto a ≤ 450KB y lado mayor 1568px antes de enviar; si el total supera 4MB, mandá 2 tandas.
- `export const maxDuration = 60` en `/api/analizar-fotos`.
- Respuesta de visión: JSON validado con Zod, `temperature: 0`, 1 reintento, después error controlado.
- El prompt de visión ignora texto con instrucciones dentro de las imágenes.
- Combinación fotos/declarado y rangos de confianza: exactamente como CLAUDE.md §7.2 pasos 7 y 9.
- Disclaimer legal visible bajo cada resultado.
- Pipeline de precios: n ≥ 8 avisos por valor, outliers por IQR, cambios > ±10% quedan `en_revision`, una fuente caída no frena a las demás, datos > 45 días bajan la confianza. Nunca scrapees avisos de portales: Zonaprop se lee solo desde su Index público.
- La vista del CRM expone únicamente agregados (localidad, tipo, operación, mediana, n).

## Criterios de cierre
- Core Web Vitals: LCP < 2,5s, CLS < 0,1, INP < 200ms, FCP < 1,8s, TTFB < 800ms
- Lighthouse > 90 en las 4 métricas
- 0 errores críticos de accesibilidad
- Tests del motor en verde
- Checklist F de SEO completo
- Preview validado en mobile, tablet y desktop, indistinguible del CRM
- Backlog poblado, tag creado, §15 completa

## Timeline
Complejidad media. Total estimado: 11 días hábiles, del 2026-10-07 al 2026-10-21. Fechas por fase en CLAUDE.md §12. Si una fase se desvía > 20%, avisá.

## Primer paso concreto
Creá el repo `arterix-tasador` con la estructura de §7.1, commiteá el CLAUDE.md como v1.0 y arrancá la Fase 3.5: `prototipo.html` estático con los tokens del CRM, el sidebar de §6.2, el uploader grande al centro y los grupos de filtros debajo. Presentalo y esperá aprobación antes de escribir código Next.js.
