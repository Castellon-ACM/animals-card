# Animals Card

Juego web de coleccionismo de cartas de animales reales.

Abres sobres, sacan especies reales fotografiadas, y montas tu álbum. Puedes vender cartas, mandarlas a gradear y seguir cómo sube o baja el valor de tu colección según el mercado.

## La idea

La rareza de cada carta no es un número inventado: es el estado real de conservación de la especie según la UICN.

| Estado | Rareza | Probabilidad |
|---|---|---|
| Preocupación menor (LC) | Común | 60% |
| Casi amenazada (NT) | Poco común | 23,5% |
| Vulnerable (VU) | Rara | 12% |
| En peligro (EN) | Épica | 3,8% |
| Peligro crítico (CR) | Legendaria | 0,6% |
| Extinta (EX) | Mítica | 0,1% |

Las dos cartas más raras de la serie son el tilacino y el dodo: las dos especies que ya no existen.

## Cómo se juega

- **Sobres**: 5 cartas, con una Vulnerable o mejor garantizada. Uno gratis cada 3 minutos, o comprado por 120 monedas.
- **Mercado**: cada hábitat (Océano, Selva, Polar, Sabana...) cotiza por su cuenta y se mueve cada 20 segundos. Vender en el momento bueno cambia mucho el resultado.
- **Gradeo**: cuesta 250 monedas. Depende del estado con que salió la carta más algo de suerte. Un 10 multiplica el valor por nueve; un 6 lo baja. Se puede perder dinero.
- **Álbum**: las 60 cartas de la Serie I. Lo que sacas alguna vez se queda registrado aunque lo vendas.

## Las fotos

Vienen de la API de [iNaturalist](https://www.inaturalist.org), filtradas para traer **solo** fotos con licencia CC0, CC BY o CC BY-SA, que son las que permiten uso comercial. Las CC BY-NC quedan excluidas a propósito porque prohíben monetizar.

Todas esas licencias obligan a citar al autor, y por eso el juego tiene una pestaña **Fotos** que lista la atribución de cada imagen cargada. **No la quites**: es una obligación legal de la licencia, no un adorno.

Si la API no responde, cada carta cae a una ilustración vectorial propia incluida en el código, así que el juego nunca se queda con huecos en blanco.

## Publicarlo

Es un único archivo sin dependencias. Para tenerlo online gratis:

1. Settings → Pages → Source: `main`, carpeta `/ (root)`.
2. Queda publicado en `https://castellon-acm.github.io/animals-card/`.

Guarda la partida en `localStorage`, así que cada jugador conserva su colección en su navegador.

## Monetización

En el código hay una función marcada:

```js
function showRewardedAd(onAdFinished){ ... }
```

Ahora mismo simula el anuncio. Para ganar dinero de verdad, sustituye el cuerpo por la llamada de tu red de anuncios (Adsterra, PropellerAds, AdinPlay u otra de vídeo recompensado) y llama a `onAdFinished()` solo cuando el anuncio haya terminado.

El equilibrio del juego se ajusta desde el bloque `CFG` al principio del script: precio del sobre, coste del gradeo, recompensa por anuncio y tiempos de espera.

## Estructura

Todo está en `index.html`:

- Sistema de ilustración vectorial (26 arquetipos: felino, cánido, cetáceo, rapaz...)
- Catálogo de las 60 especies con nombre científico, hábitat, estado y un dato real
- Dibujo de la carta en SVG (marco, número de colección, pips de rareza)
- Carga y caché de fotos de iNaturalist
- Lógica de juego: sobres, mercado, gradeo, álbum
