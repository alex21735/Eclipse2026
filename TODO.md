# TODO — Herramienta de observabilidad de eclipses

Hoja de ruta para evolucionar la herramienta. Ordenado por prioridad.
Marca con [x] lo completado.

---

## 🔴 BUGS (crítico — hacer primero)

### [x] 1. La obscuración está hardcodeada al 12/08/2026 y a un punto fijo
**Problema.** Los tiempos de contacto `C1/C2/C3/C4` en `index.html` son valores
fijos (interpolados de Burriana/Castellón). Consecuencias:
- Cualquier fecha muestra la curva del 12/08/2026 (falsa para otros días).
- Al mover las coordenadas, la curva NO se recalcula (mal fuera de Castellón).

**Solución correcta (camino B): calcular circunstancias locales en el navegador.**
Integrar el *Solar Eclipse Calculator* de O'Byrne & McCann (motor del NASA/GSFC
JavaScript Solar Eclipse Explorer). Calcula posiciones topocéntricas Sol/Luna con
elementos besselianos y radios IAU (Sol 696000 km, Luna 1737.4 km), 100% en el
navegador, sin servidor.
- Fuente/base: https://eclipse.gsfc.nasa.gov/JSEX/JSEX-EU.html
- Requiere: elementos besselianos + ΔT de cada eclipse (tablas del Five Millennium Canon).
- Salida: C1, C2, C3, C4, máximo, magnitud, altura del Sol en cada contacto.
- Con esto la obscuración se recalcula para coordenadas y eclipse arbitrarios.

**Solución pragmática (camino A) si B es demasiado:** precargar elementos
besselianos SOLO del trío ibérico y calcular contactos para las coords del usuario
(los besselianos + coords ya bastan; no hace falta la lista de ciudades).

### [x] 2. Cargar el trío de eclipses ibéricos (2026 y 2027 hechos; 2028 anular pendiente)
- 2026-08-12 (total)
- 2027-08-02 (total)
- 2028-01-26 (anular)
Selector de eclipse que cargue los elementos besselianos correspondientes.
Nota: el de 2028 es ANULAR → la obscuración máxima NO llega a 100%; el código
debe manejar magnitud < 1 (anillo) y no forzar totalidad.

---

## ✅ HECHO recientemente
- Altitud automática por posición (SRTM) si el campo se deja vacío; manual si se rellena.
- Timestamp de consulta de datos y localidad cercana mostrados en la cabecera de resultados.
- Cálculo besseliano validado contra tiempos oficiales IGN (2026).

## 🟠 FEATURES DE ENTRADA (alta prioridad)

### [ ] 3. Selector de fecha real (date picker)
Sustituir el input de texto por `<input type="date">`. Al cambiar la fecha,
si coincide con un eclipse del trío, cargar sus elementos; si no, avisar
"no hay eclipse esta fecha" y ocultar la columna de obscuración.

### [x] 4. Buscar lugar por nombre (geocoding)
Campo de búsqueda de localidad → coordenadas. Usar la API de geocoding de
Open-Meteo (gratis, sin clave):
`https://geocoding-api.open-meteo.com/v1/search?name=Cabanes&count=5&language=es`
Devuelve lat/lon/nombre/altitud. Rellenar los campos automáticamente.

### [x] 5. Localidad más cercana (BigDataCloud reverse)
Al fijar unas coordenadas, mostrar el nombre del pueblo/ciudad más cercano.
Open-Meteo geocoding no hace reverse directo; alternativas:
- Nominatim (OpenStreetMap) reverse: requiere respetar su política de uso.
- Precargar lista de localidades de la franja y buscar la más cercana (haversine).

### [ ] 6. Selector de lugar con mapa interactivo
Mapa (Leaflet + tiles OSM) donde pinchar para fijar el observador.
- Leaflet es ligero y sin clave.
- Al hacer clic: actualizar lat/lon y recalcular.
- Bonus: dibujar la línea de visión al Sol (azimut) y marcar los 7 puntos de muestreo.
- Bonus: sombrear la franja de totalidad del eclipse seleccionado.

### [x] 7. Autoubicación (geolocation)
Botón "usar mi ubicación" → `navigator.geolocation.getCurrentPosition()`.
Solo funciona bajo HTTPS (GitHub Pages lo es). Pedir permiso.

---

## 🟡 FEATURES INTERACTIVAS (media prioridad)

### [ ] 8. Comparar varios modelos a la vez
Correr best_match + icon_eu + arome + ecmwf y mostrar lado a lado SOLO
nube baja + opacidad de columna, para ver la dispersión entre modelos
en la totalidad de un vistazo. (Idea ya discutida.)

### [ ] 9. Gráfica temporal
Curvas de obscuración, opacidad de nube y score vs tiempo (Chart.js o SVG propio).
Ver el cruce "totalidad vs cielo despejado" visualmente.

### [ ] 10. Altura de nubes configurable
Inputs para las 7 alturas de muestreo (ahora fijas 8/10/12, 3/5, 0.8/1.5 km).
A baja elevación el resultado es sensible a estas alturas.

### [ ] 11. Perfil de horizonte topográfico completo
Ahora solo se calcula el ángulo en el azimut solar. Dibujar el perfil de
horizonte en ±30° alrededor del ocaso, para ver si un pico cercano interfiere.

### [ ] 12. Permalink / estado en URL
Guardar coords/fecha/modelo en query params para compartir configuración.

---

## 🟢 PULIDO (baja prioridad)

### [ ] 13. Modo claro/oscuro
### [ ] 14. Responsive para móvil (la tabla necesita scroll horizontal; ver si se puede compactar)
### [ ] 15. Exportar la gráfica como PNG
### [ ] 16. i18n: español / inglés / catalán
### [ ] 17. Cachear respuestas de API en localStorage con TTL (evitar re-fetch al reordenar)
### [ ] 18. Tests: validar contactos calculados contra los datos oficiales del IGN

---

## Notas técnicas / deuda

- **Score**: pesos heurísticos (Rayleigh 0.04 con suelo 0.30; aero suelo 0.35;
  umbral topo 0.3°). Documentado en el HTML. Considerar calibrar si hay datos reales.
- **15 min nativos**: solo Centroeuropa; para España se interpola de datos horarios.
- **AROME**: solo NE peninsular. **ICON-D2**: no cubre España (excluido).
- **CORS en iOS**: el fetch a Open-Meteo puede fallar en Safari/Chrome iOS.
  Investigar si un proxy o los flags de la API lo resuelven.
- **Atribución**: mantener Open-Meteo (CC-BY), CAMS, SRTM, IGN. Ver README.
