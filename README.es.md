# Practilizate

**Observatorio de prácticas preprofesionales del Perú.**

Practilizate convierte avisos públicos de prácticas en un producto de datos reproducible: un modelo estrella validado en PostgreSQL, un dashboard en Power BI, un libro Excel, un reporte HTML portátil y un informe de calidad de datos.

> **Estado:** v0.1.3. El repositorio incluye el pipeline, el esquema PostgreSQL, los fixtures sintéticos, el proyecto bilingüe de Power BI (PBIP) y capturas agregadas. Los entregables Excel, HTML y CSV se generan en cada ejecución y **no** se versionan. Una muestra sintética generada desde los fixtures está adjunta al [release v0.1.3](https://github.com/tejedasebastian/Practilizate/releases/tag/v0.1.3).
> **English:** [README.md](README.md)

## Por qué Practilizate

Los estudiantes postulan sin datos comparables sobre sueldos, empresas, habilidades requeridas o fechas límite. Los centros de empleabilidad tampoco tienen una vista estructurada y actualizada del mercado.

Practilizate responde:

- ¿Qué habilidades se piden más por carrera y distrito?
- ¿Qué empresas publican más ofertas?
- ¿Qué rangos de subvención aparecen por rol y carrera?
- ¿Qué ofertas están por vencer?
- ¿Qué tan completa y confiable es la información publicada?

El análisis inicial de 73 ofertas era solo de Ingeniería de Sistemas en Lima. El producto está diseñado para **todas las carreras**.

## Qué incluye

- Ingesta multi-carrera desde practicas.pe.
- Normalización de empresa, ubicación, sueldo, fechas, carreras y habilidades.
- Reglas de calidad con informe visible.
- Modelo estrella en PostgreSQL con tablas puente de carrera y habilidad.
- Reporte Power BI con toggle español/inglés.
- Libro Excel en español.
- Reporte HTML offline con búsqueda, filtros y toggle español/inglés.
- Test E2E repetible con artefacto y manifiesto.
- Documentación bilingüe.

## Arquitectura

```text
practicas.pe
    |
    v
[ingest] caché + rate limit
    |
    v
[normalize] ofertas canónicas
    |
    v
[quality] reglas + informe
    |
    v
[load] modelo estrella PostgreSQL
    |
    +--> [export] dashboard_dataset.csv
    +--> [export] practilizate.xlsx
    +--> [export] practilizate.html
    +--> [powerbi] practilizate.pbip
```

## Demo

![Dashboard en español](docs/assets/dashboard-es.png)

![Dashboard en inglés](docs/assets/dashboard-en.png)

![Resumen ejecutivo en Excel](docs/assets/excel-summary.png)

![Informe de calidad de datos](docs/assets/quality-report.png)

![Reporte HTML portátil](docs/assets/html-report.png)

- Dashboard en español: `docs/assets/dashboard-es.png`
- Dashboard en inglés: `docs/assets/dashboard-en.png`
- Resumen Excel: `docs/assets/excel-summary.png`
- Informe de calidad: `docs/assets/quality-report.png`
- Reporte HTML portátil (vista agregada): `docs/assets/html-report.png`

¿Quieres los archivos reales? Descarga la muestra sintética del
[release v0.1.3](https://github.com/tejedasebastian/Practilizate/releases/tag/v0.1.3):

- `practilizate-synthetic.html` — reporte autocontenido.
- `practilizate-synthetic.xlsx` — libro Excel.
- `dashboard_dataset-synthetic.csv` — dataset del dashboard.

La muestra se genera desde `data/fixtures/` con `--as-of 2026-10-29` y no
contiene datos live.

## Fuente y atribución

Los datos provienen de avisos públicos de [practicas.pe](https://www.practicas.pe/).

- Practilizate **no está afiliado** a practicas.pe.
- Se usan datos públicos con fines educativos y de investigación.
- Solo se publican datos agregados de la fuente live; la muestra del release es sintética. No se incluyen HTML crudo ni datos personales.
- El CSV, el Excel y el HTML live quedan fuera de Git (`data/processed/`); las
  capturas públicas y la muestra del release solo contienen datos agregados o
  sintéticos.
- El recolector respeta `robots.txt`, límites de tasa y caché.
- Ver [DATA_SOURCES.md](DATA_SOURCES.md) y [NOTICE.md](NOTICE.md).

## Inicio rápido

```powershell
git clone https://github.com/tejedasebastian/Practilizate.git
cd Practilizate
python -m venv .venv
.venv\Scripts\Activate.ps1
pip install -e ".[dev]"
docker compose up -d db
Copy-Item .env.example .env
python -m practilizate run --source fixtures --as-of 2026-10-29
pytest tests/e2e
```

El modo `fixtures` es determinista y no llama al sitio en vivo.
Para el snapshot live:

```powershell
python -m practilizate run --source live --as-of 2026-10-29
```

Docker Compose publica PostgreSQL en el puerto **5433**; `.env.example` ya usa
ese puerto. La ingesta live usa caché, un límite de tasa configurable y el
filtro de carreras del sitio (65 carreras en la fecha del snapshot).

## Cómo usar Practilizate

**1. Revisar el resultado sin instalar nada**

Descarga la muestra sintética del
[release v0.1.3](https://github.com/tejedasebastian/Practilizate/releases/tag/v0.1.3)
y abre `practilizate-synthetic.html` en cualquier navegador: es un solo archivo
sin dependencias externas. Las capturas de `docs/assets/` muestran la vista
agregada. Para revisar la definición del reporte, abre
`powerbi/practilizate.pbip` en Power BI Desktop y guárdalo como `.pbix` si
necesitas un archivo único.

**2. Ejecutar el pipeline localmente**

Sigue el inicio rápido. El modo `fixtures` es determinista y sirve para
verificar la instalación. Usa `--source live` para avisos reales.

**3. Usar los entregables**

Una ejecución local escribe estos archivos en `data/processed/` (ignorado por
Git):

- `practilizate.html` — reporte portátil con búsqueda, filtros y toggle ES/EN.
- `practilizate.xlsx` — libro en español para estudiantes y centros de empleabilidad.
- `dashboard_dataset.csv` — snapshot para Power BI/analistas.

El repositorio en sí contiene:

- `powerbi/practilizate.pbip` — fuente del reporte bilingüe.
- Vista PostgreSQL `v_dashboard_ofertas` — análisis opcional en vivo (creada por `load`).

**4. Compartirlo en línea**

El reporte HTML es la forma más liviana de compartir el snapshot: envía un solo
archivo o súbelo a cualquier sitio estático; no necesita servidor, cuenta ni
conexión a internet. Para análisis interactivo, publica el PBIP en un workspace
de Power BI (requiere cuenta/licencia) o monta una aplicación sobre el
CSV/PostgreSQL. El repositorio público publica capturas y evidencia agregada;
el CSV, Excel y HTML live quedan locales.

**5. Automatizar una actualización**

Programa `python -m practilizate run --source live --as-of <fecha>` con Task
Scheduler o cron. El recolector usa caché y respeta el límite de tasa. El E2E
usa fixtures y nunca llama al sitio público.

## Power BI

Abre `powerbi/practilizate.pbip` en Power BI Desktop y guárdalo como `.pbix` si lo necesitas.

- v0.1 usa `data/processed/dashboard_dataset.csv` y los CSV de soporte para que sea portable.
- El modelo lee el snapshot mediante el parámetro `DataFolder`. Su valor por defecto es la ruta relativa al repositorio `data\processed`; en Power BI Desktop configúralo con la ruta absoluta de tu clon antes de actualizar (el conector de archivos no resuelve rutas relativas de forma fiable).
- Las seis páginas cubren resumen ejecutivo, habilidades, empresas y subvenciones, vencimientos, calidad y una guía bilingüe.
- El reporte incluye toggle ES/EN (tabla `Idioma` y medidas `SWITCH`), texto alternativo, colores accesibles, pie de fuente y botón `Limpiar filtros`.
- PostgreSQL queda como fuente opcional en vivo para analistas que prefieran SQL.

## Excel

El libro se genera en `data/processed/practilizate.xlsx` e incluye:

- `Portada`
- `Datos`
- `Resumen`
- `Habilidades`
- `Empresas`
- `Calidad`
- `Diccionario`

## Reporte HTML portátil

Cada ejecución también escribe `data/processed/practilizate.html`:

- Un solo archivo autocontenido: CSS y JavaScript inline, sin CDN ni llamadas de red.
- Búsqueda por cargo, empresa, carrera o habilidad; filtros por carrera, estado y disponibilidad de subvención.
- Tarjetas KPI, barras de demanda por carrera y habilidad, y ofertas con vencimiento y subvención.
- Toggle español/inglés y diseño listo para imprimir o exportar a PDF.
- Sin texto de requisitos, correos ni teléfonos; solo campos públicos del aviso.
- Se regenera desde un run existente con `python -m practilizate export --format html`.

## Testing

- Solo E2E. Sin tests unitarios.
- El E2E ejecuta el pipeline completo contra una base PostgreSQL desechable.
- Usa un escenario **sintético** medio-difícil: ofertas multi-carrera, duplicados, sueldo faltante, fechas inválidas, outliers y campos faltantes.
- El snapshot live se valida aparte; CI no consulta el sitio público.
- Cada ejecución E2E escribe un artefacto sintético repetible en `artifacts/e2e/`:

```text
artifacts/e2e/
├─ run_manifest.json
├─ quality_report.json
├─ normalized_offers.json
├─ database_load.json
├─ export_report.json
├─ practilizate.xlsx
├─ practilizate.html
├─ dashboard_dataset.csv
├─ offer_careers.csv
├─ offer_skills.csv
├─ quality_rules.csv
├─ run_metadata.csv
├─ dim_idioma.csv
├─ dim_estado.csv
└─ checksums.sha256
```

`artifacts/e2e/` se genera y está en `.gitignore`; CI lo sube como artefacto de
workflow. El release v0.1.3 adjunta una muestra sintética más pequeña (HTML,
XLSX, CSV) generada desde los fixtures, no el snapshot live.

## Resultados

> Snapshot live `live-20261029` (`--source live --as-of 2026-10-29`), obtenido
> del sitio público con límite de 1 s. Los conteos son de un momento dado.

| Métrica | Valor |
|---|---|
| Ofertas procesadas | 389 |
| Empresas | 83 |
| Carreras | 65 |
| Habilidades | 24 |
| Calidad de datos | 97.7% |
| Ofertas que vencen en 7 días | 330 |
| Tiempo del pipeline | 530.6 s (~8.8 min, PostgreSQL incluido) |

El informe de calidad marca 3 reglas con advertencias y ninguna regla crítica:
R06 (1 oferta con vencimiento anterior a la publicación), R14 (38 grupos
duplicados por título + empresa + vencimiento) y R15 (113 ofertas con requisitos
detallados pero sin habilidad del léxico). La subvención se publica en 156 de
389 ofertas; los promedios excluyen faltantes y valores atípicos mayores a
S/ 12,000. El libro Excel incluye una hoja de diccionario de datos y el reporte
HTML muestra los mismos indicadores agregados sin texto de requisitos.

## Estructura del proyecto

```text
Practilizate/
├─ README.md
├─ README.es.md
├─ DATA_SOURCES.md
├─ NOTICE.md
├─ LICENSE
├─ docs/assets/            # capturas versionadas
├─ src/practilizate/
├─ sql/
├─ tests/e2e/
├─ data/fixtures/          # fixtures sintéticos versionados
├─ data/processed/         # generado por runs (ignorado por Git)
├─ artifacts/e2e/          # generado por tests (ignorado por Git)
├─ powerbi/
├─ scripts/
├─ docker-compose.yml
├─ pyproject.toml
├─ .env.example
└─ .github/workflows/ci.yml
```

## Limitaciones

- v0.1 cubre Lima y prácticas preprofesionales.
- Los avisos pueden no publicar sueldo, fecha límite o requisitos detallados. En
  el snapshot live, 233 de 389 ofertas no publican subvención.
- El léxico de habilidades cubre 24 habilidades canónicas; 113 ofertas live
  tienen requisitos detallados pero ninguna coincidencia. Ampliar el léxico es
  una tarea de v0.2.
- El snapshot live es de un momento dado; 330 ofertas vencen dentro de 7 días
  de la fecha de corte porque muchas fechas límite se concentran.
- R14 marca 38 grupos duplicados por título + empresa + vencimiento; se mantienen
  visibles para auditoría en lugar de eliminarse en silencio.
- La estructura del sitio puede cambiar; los fixtures protegen el test suite.
- Es un observatorio, no un tracker de postulaciones.
- La calidad de datos se reporta, no se oculta.

## Licencia

Código: MIT. Ver [LICENSE](LICENSE) y [NOTICE.md](NOTICE.md).

Los datos y datasets derivados no están cubiertos por la licencia MIT. Ver [DATA_SOURCES.md](DATA_SOURCES.md).
