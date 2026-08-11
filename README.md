# Observabilidad del eclipse — 12 de agosto de 2026

Herramienta de una sola página (HTML autocontenido, sin backend) para estimar la
**observabilidad del eclipse solar total del 12/08/2026** desde un punto dado,
integrando en cada intervalo de tiempo:

- **Geometría solar** exacta (azimut, elevación) por algoritmo NOAA.
- **Obscuración lunar** anclada a los tiempos de contacto del IGN.
- **Horizonte topográfico** en el azimut solar (¿tapa el relieve al Sol?).
- **Nubosidad en la línea de visión oblicua**: 7 rayos muestreados a distinta
  altura (alta/media/baja), cada uno a la distancia geométrica `d = h / tan(elev)`.
- **Extinción atmosférica**: masa de aire, visibilidad, AOD y polvo (calima).
- **Puntuación integrada** de visibilidad del disco (índice orientativo).

Diseñada originalmente para el **Prat de Cabanes-Torreblanca (Castellón)**, pero
funciona con cualquier coordenada dentro del alcance de los modelos.

## Uso

Abre `index.html` en un navegador de escritorio (Chrome/Firefox/Edge). Introduce
coordenadas, fecha, ventana horaria, intervalo (5/15/30/60 min) y modelo, y pulsa
**Calcular**. Los datos se descargan en vivo desde las APIs de Open-Meteo.

> **Nota:** en iOS (Safari/Chrome) el `fetch` a las APIs puede fallar por CORS.
> Úsalo preferentemente en un navegador de escritorio.

## Cómo leer la tabla

- **Obsc%** (obscuración): fracción del disco solar cubierta por la Luna. 100% = totalidad.
- **Opac** (columna de nube): `100·(1 − ∏(1 − cᵢ/100))` sobre los 7 rayos.
- **Score**: `100 · T_nube · T_aero · T_Rayleigh · f_topo`. Alto = disco visible.
  **Independiente de la obscuración**: mide si el Sol se ve a través de atmósfera
  y sin obstáculos, no si la Luna lo tapa.
- Filas resaltadas = totalidad. Filas rayadas = Sol por debajo del horizonte topográfico.

## Fuentes de datos y atribución

Este proyecto usa datos de terceros. Si lo reutilizas, mantén la atribución:

- **[Open-Meteo](https://open-meteo.com/)** — nubosidad, visibilidad, elevación
  (CC-BY 4.0). Agrega modelos de:
  - **DWD** (ICON, ICON-EU) — CC-BY 4.0
  - **Météo-France** (AROME) — datos abiertos
  - **ECMWF** (IFS) · **NOAA** (GFS)
- **[Copernicus CAMS](https://atmosphere.copernicus.eu/)** — aerosol optical depth, polvo.
- **SRTM / NASA** — modelo digital de elevación (vía Open-Meteo).
- **[IGN](https://eclipses.ign.es/)** — tiempos de contacto del eclipse.

## Limitaciones (léelas)

- **No es una predicción oficial.** Es un índice orientativo. Para decisiones,
  consulta **[AEMET](https://www.aemet.es/)** e **[IGN](https://eclipses.ign.es/)**.
- La **obscuración** usa una forma analítica anclada a esos tiempos; precisión de
  unos pocos %.
- Los datos de **nubosidad** son horarios (interpolados a intervalos menores; el
  paso nativo de 15 min solo existe en Centroeuropa).
- A baja elevación (~4°) un error de altura de nube se amplifica ×(1/tan elev)≈×14
  en distancia horizontal: las filas de última hora son las más inciertas.
- Los **pesos del score** (Rayleigh, umbral topográfico) son heurísticos.
- **AROME** solo cubre el NE peninsular; **ICON-D2** no cubre España.

## Licencia

Código bajo licencia **MIT** (ver cabecera de `index.html`). Los datos que consume
mantienen sus licencias respectivas (ver arriba).
