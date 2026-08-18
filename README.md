# ¿DÓNDE HAY SALSA?
**La cartelera salsera de México** — V1, 100% frontend

Cartelera de eventos de salsa y ritmos latinos. Arranca en Puerto Vallarta, pero
**ninguna parte del código asume que solo existe esa ciudad**.

Sin servidor, sin base de datos, sin build, sin npm. Solo HTML, CSS y JavaScript.
Abres `index.html` y funciona.

---

## 1. Las 4 pantallas

| Pantalla | Archivo | Qué hace |
|---|---|---|
| Home / cartelera | `index.html` | Hero, buscador, 4 filtros, chips rápidos de fecha, cartelera, ciudades, mapa (preview), CTA organizadores |
| Detalle del evento | `evento.html?id=…` | Flyer grande, datos completos, "QUIERO IR", cómo llegar, compartir (WhatsApp / Facebook / copiar / nativo), organizador, eventos relacionados, barra fija en móvil |
| Ciudad | `ciudad.html?c=puerto-vallarta` | Misma cartelera filtrada, con su propio título y descripción |
| Publicar evento | `publicar.html` | Formulario validado, vista previa del flyer, y botón que te manda el evento por WhatsApp con todos los datos ya escritos |

Extras que ya vienen resueltos: SEO básico por página (title, description, Open Graph)
y datos estructurados `schema.org/Event`, búsquedas compartibles por URL
(`index.html?ciudad=cdmx&fecha=hoy`), estados vacíos, accesibilidad (foco visible,
`aria-live`, menú con `aria-expanded`) y respeto a `prefers-reduced-motion`.

---

## 2. Estructura

```
dondehaysalsa/
├── index.html · evento.html · ciudad.html · publicar.html
└── assets/
    ├── css/
    │   ├── tokens.css        ← Colores, tipografía, espacios. AQUÍ cambias la identidad visual
    │   ├── components.css      Navbar, botones, tarjetas, filtros, formularios
    │   └── pages.css           Hero, ciudades, mapa, página de evento, publicar
    ├── js/
    │   ├── core/
    │   │   ├── config.js       Nombre, ciudad default, tu WhatsApp, aviso de ejemplo
    │   │   ├── format.js       Fechas en español, precios, slugs, avisos, utilidades
    │   │   └── dataSource.js   Filtros, orden y búsqueda. Es el "cerebro" de la cartelera
    │   ├── data/
    │   │   ├── events.js     ← AQUÍ agregas tus eventos
    │   │   ├── cities.js     ← AQUÍ agregas ciudades
    │   │   └── categories.js   Tipos de evento, ritmos y rangos de fecha
    │   ├── components/
    │   │   ├── layout.js       Navbar + footer (una sola fuente de verdad para todo el sitio)
    │   │   ├── eventCard.js    La tarjeta de evento
    │   │   └── filters.js      Filtros y chips rápidos
    │   └── pages/              home.js · event.js · city.js · publish.js
    └── img/
        ├── flyers/             12 flyers de ejemplo en SVG (generados, sin derechos de terceros)
        ├── og.svg              Imagen que se ve al compartir el sitio
        └── favicon.svg
```

**Por qué está separado así:** las páginas nunca leen los arreglos de datos
directamente, siempre pasan por `DHS.data` (en `dataSource.js`). Eso mantiene la
lógica de filtros en un solo lugar y hace que agregar contenido no rompa nada.

---

## 3. Los 3 archivos que vas a tocar

### Agregar un evento → `assets/js/data/events.js`

Copia un bloque completo, cámbiale los datos:

```js
{
  id:'social-viernes-vallarta',          // debe ser único; se usa en la URL
  nombre:'Social Salsero del Viernes',
  descripcion:'Clase abierta y después pista libre con DJ…',
  fecha:'2026-09-21',                    // fecha fija (o usa dias:3 para "dentro de 3 días")
  hora:'21:00',
  lugar:'Salón Caribe',
  direccion:'Av. México 1234, Col. 5 de Diciembre',
  ciudad:'puerto-vallarta',              // slug que exista en cities.js
  precio:150,                            // 0 = Gratis
  tipo_evento:'social',                  // slug de categories.js
  ritmos:['salsa','bachata'],            // slugs de categories.js
  flyer:'assets/img/flyers/mi-flyer.jpg',
  link_boletos:'',                       // vacío = el botón manda al WhatsApp del organizador
  latitud:20.6280, longitud:-105.2350,   // opcional, para "Cómo llegar"
  organizador_id:1,                      // id de la lista de organizadores, arriba en el mismo archivo
  destacado:true                         // true = aparece primero en la cartelera
}
```

### Agregar una ciudad → `assets/js/data/cities.js`

Una línea con `activa:true` y esa ciudad aparece sola en los filtros, en "Explora
por ciudad", en el footer y tiene su propia página. Con `activa:false` sale como
"Próximamente".

### Cambiar la identidad visual → `assets/css/tokens.css`

Todo el color de la plataforma sale de `--c-accent` (el coral) y `--c-gold`.
Cambia esas dos variables y cambia el sitio entero.

**Antes de publicar:** en `assets/js/core/config.js` pon tu WhatsApp real en
`contacto.whatsapp` y cambia `mostrarAvisoEjemplo` a `false` para quitar la barra
amarilla de arriba.

---

## 4. Cómo probarlo

Abre `index.html` en el navegador. Ya está.

Para verlo como servidor local (recomendado, se comporta igual que en internet):

```bash
cd dondehaysalsa
python3 -m http.server 8000
# → http://localhost:8000
```

Checklist rápido:

1. Filtra por **Hoy** y por **Este fin de semana** → cambia el contador.
2. Busca `bachata` → filtra por nombre, ritmo, lugar y organizador.
3. Entra a un evento → prueba **Copiar enlace** y **WhatsApp**.
4. Achica la ventana a 390px de ancho → menú hamburguesa y barra fija de "QUIERO IR".
5. Publica un evento de prueba → aparece cómo quedó y el botón de WhatsApp.

---

## 5. Publicar en GitHub Pages

```bash
cd dondehaysalsa
git init
git add .
git commit -m "V1 ¿Dónde Hay Salsa?"
git branch -M main
git remote add origin https://github.com/TU_USUARIO/dondehaysalsa.git
git push -u origin main
```

En GitHub: **Settings → Pages → Source: Deploy from a branch → main / (root) → Save**.
En 1-2 minutos queda en `https://TU_USUARIO.github.io/dondehaysalsa/`.

Cuando tengas dominio propio, se agrega en esa misma pantalla en **Custom domain**.

---

## 6. Cómo funciona el formulario sin servidor

Una página estática no puede guardar nada en ningún lado. Así que el formulario
hace las dos cosas que sí puede hacer, y que además son las útiles:

1. **Arma el evento y te lo muestra** con el diseño real de la cartelera
   (se guarda en el navegador de quien lo llenó, solo ahí, como vista previa).
2. **Genera un mensaje de WhatsApp** con todos los datos ya redactados, listo para
   enviártelo. Tú lo pegas en `events.js` y queda publicado de verdad.

Es manual, y a propósito: con 5 o 20 eventos por semana funciona perfecto y te
deja controlar qué se publica. Cuando el volumen te gane, ese es el momento —
y no antes — de meterle backend.

---

## 7. Siguientes pasos, en orden de valor

1. **Reemplazar el contenido de ejemplo por eventos reales de Puerto Vallarta.**
   Es lo único que separa esto de ser un sitio de verdad. Todo lo demás ya está.
2. **Flyers reales.** Los SVG que vienen son de relleno. Pide los flyers a los
   organizadores en formato vertical (4:5) y guárdalos en `assets/img/flyers/`.
   Comprímelos antes de subirlos (tinypng.com) para que el sitio siga siendo rápido.
3. **Mapa real** con Leaflet + OpenStreetMap. Es gratis, son ~20 líneas, y el
   contenedor `#mapa` ya está listo. Google Maps cobra por carga.
4. **Página de organizador** (`organizador.html?id=…`) con todos sus eventos.
   Es lo que te permite después venderles un perfil destacado.
5. **PWA**: un `manifest.json` y el sitio se instala en el celular como app.
   Para una cartelera que se consulta el viernes en la noche, eso vale mucho.

---

## 8. Sobre el contenido

Todos los eventos, organizadores y flyers incluidos son **ficticios**. Los flyers
son SVG generados para este proyecto: no hay fotografías ni material de terceros.
Antes de lanzar, reemplaza el contenido de ejemplo por eventos reales con
autorización de sus organizadores.
