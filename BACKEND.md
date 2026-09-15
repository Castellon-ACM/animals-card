# Animals Card — puesta en marcha del backend

El juego tiene **cuentas de usuario, base de datos e intercambios entre jugadores**. Eso necesita un servidor, y aquí se usa [Supabase](https://supabase.com) (capa gratuita: autenticación + PostgreSQL, sin pedir tarjeta).

## Por qué la lógica está en la base de datos

Si el navegador pudiera escribir directamente en la tabla de monedas, cualquiera se pondría un millón con la consola abierta. Por eso:

- Las políticas RLS solo permiten **leer**.
- Todo lo que cambia el juego (abrir sobres, vender, gradear, intercambiar) son funciones `SECURITY DEFINER` dentro de Postgres.
- El navegador solo puede *pedir* "ábreme un sobre". Qué sale y cuánto cuesta lo decide el servidor.

También por eso el tiempo de espera del sobre gratis y del anuncio se comprueba con el reloj del servidor, no con el del móvil: cambiar la hora del teléfono no sirve de nada.

## Pasos

### 1. Crear el proyecto en Supabase

En [supabase.com](https://supabase.com) → New project. Apunta la contraseña de la base de datos.

### 2. Cargar el esquema

SQL Editor → New query → pega el contenido de `schema.sql` entero → Run.

Crea las tablas, la seguridad y las funciones del juego. Se puede volver a ejecutar sin romper nada.

### 3. Sembrar el catálogo de especies

Las 1200 cartas no están escritas a mano: se descargan de iNaturalist con su **estado de conservación real**, que es lo que determina la rareza en el juego.

```bash
npm install @supabase/supabase-js

SUPABASE_URL="https://xxxxx.supabase.co" \
SUPABASE_SERVICE_KEY="tu_service_role_key" \
node seed-species.mjs
```

La `service_role` key está en Settings → API. **No la subas nunca a GitHub ni la pongas en el frontend**: salta toda la seguridad. Solo va en tu terminal.

Tarda un rato largo (unos 40-60 minutos): iNaturalist limita a unas 60 peticiones por minuto y el script lo respeta. Déjalo corriendo.

Si quieres otra cantidad: `TARGET=800 node seed-species.mjs`.

### 4. Activar el login por email

Authentication → Providers → Email. Para pruebas, desactiva "Confirm email" y así puedes registrarte sin pasar por el correo.

### 5. Conectar `app.html` con tu proyecto

Abre `app.html` y, al principio del `<script>`, cambia estas dos líneas por las tuyas (Settings → API):

```js
const SUPABASE_URL = 'https://TU-PROYECTO.supabase.co';
const SUPABASE_ANON_KEY = 'TU_ANON_KEY_AQUI';
```

La `anon key` **sí va en el frontend** a propósito (es pública por diseño); la seguridad la dan las políticas RLS y las funciones del servidor, no ocultar esa clave.

### 6. Publicarlo

Settings → Pages → Source: `main`, carpeta `/ (root)`. Sube `app.html` como `index.html` (o renómbralo) para que quede en la raíz del sitio.

## Sobre las fotos

Solo se aceptan licencias **CC0, CC BY y CC BY-SA**, que permiten uso comercial. Las CC BY-NC quedan descartadas aunque eso reduzca el número de especies disponibles, porque prohíben monetizar y el juego lleva anuncios.

Esas licencias obligan a citar al autor: la columna `photo_attr` guarda la atribución de cada foto, y la pestaña **Álbum → "Ver atribución de fotos"** la muestra. No la quites, es una obligación legal.

## Los intercambios, por dentro

- Al proponer un trueque, las cartas de ambos lados quedan **bloqueadas** (`locked = true`): no se pueden vender ni gradear mientras la oferta siga abierta.
- Al aceptar, el cambio de dueño ocurre en una única función; o se mueven todas las cartas de los dos lados, o ninguna.
- Antes de cerrar, el servidor comprueba que las cartas siguen donde deben — por si alguna cambió de manos mientras la oferta estaba pendiente.
- Puedes proponer un trueque solo de ida (por ejemplo, regalar una carta sin pedir nada a cambio): basta con dejar vacío el lado que no quieras usar.

## Lo que hay en cada archivo

| Archivo | Qué es |
|---|---|
| `schema.sql` | Tablas, seguridad RLS y toda la lógica del juego en Postgres |
| `seed-species.mjs` | Descarga las especies de iNaturalist y las carga en la base de datos |
| `app.html` | El juego completo: login, sobres, cartas, álbum, mercado e intercambios |

## Funciones disponibles desde el frontend

```js
supabase.rpc('open_pack', { p_free: false })      // abrir sobre (o p_free:true para el gratis)
supabase.rpc('sell_card', { p_card: cardId })     // vender
supabase.rpc('grade_card', { p_card: cardId })    // gradear
supabase.rpc('claim_ad_reward')                   // recompensa por anuncio
supabase.rpc('my_collection')                     // tus cartas con su valor ya calculado
supabase.rpc('my_trades')                         // tus intercambios, enviados y recibidos
supabase.rpc('player_cards', { p_username: 'x' }) // ver las cartas libres de otro jugador
supabase.rpc('create_trade',  { p_to_username, p_offer, p_request, p_note })
supabase.rpc('respond_trade', { p_trade, p_accept })
supabase.rpc('cancel_trade',  { p_trade })
supabase.rpc('species_rarity_counts')             // totales por rareza, para el álbum
supabase.rpc('tick_market')                       // hace avanzar el mercado (se llama solo)
```

## Ajustar el equilibrio

Los tiempos y costes reales están en la función `cfg()` dentro de `schema.sql`. El archivo `app.html` tiene una copia de esos mismos números arriba del todo (`const CFG = {...}`) solo para pintar las cuentas atrás en pantalla — si cambias uno, cambia el otro para que coincidan.
