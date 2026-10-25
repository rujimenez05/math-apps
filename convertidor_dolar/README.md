# Convertidor de delimitadores matemáticos para eXeLearning

Herramienta web de una sola página que convierte los delimitadores de las fórmulas matemáticas de un proyecto de [eXeLearning](https://exelearning.net/) (`.elpx`) del formato `$…$` al formato `\(…\)` / `\[…\]`.

Todo el procesamiento se hace **en el navegador**: el proyecto no se envía a ningún servidor.

## ¿Para qué sirve?

Si has escrito las fórmulas de tu proyecto con signos de dólar, esta herramienta las reescribe con los delimitadores de LaTeX que usa eXeLearning:

| Tipo de expresión | Antes | Después |
|---|---|---|
| Bloque (en línea aparte) | `$$…$$` | `\[…\]` |
| En línea (dentro del texto) | `$…$` | `\(…\)` |

Ejemplo:

```text
Antes:   Sea $x^2$ y $$a+b=c$$
Después: Sea \(x^2\) y \[a+b=c\]
```

## Uso

1. Abre `index.html` en un navegador moderno (con conexión a Internet, ver [Limitaciones](#limitaciones)). También puedes usar la aplicación publicada en GitHub Pages si el repositorio la tiene activada.
2. Arrastra tu archivo `.elpx` a la zona de carga, o haz clic para elegirlo.
3. Pulsa **Convertir y preparar descarga**.
4. Guarda el archivo resultante, que se llama `nombre_convertido.elpx`.
5. Abre el resultado en eXeLearning y **revisa que las fórmulas y las actividades se ven y se pueden editar** antes de distribuirlo.

Una vez convertido el archivo, solo queda visible el botón **Empezar de nuevo**, para evitar convertir dos veces el mismo proyecto.

### Carpeta de guardado

- En **Chrome, Edge y otros navegadores basados en Chromium**, si seleccionas o arrastras el archivo, el diálogo de guardado se abre en la **misma carpeta que el original**.
- En **Firefox y Safari** los navegadores no permiten elegir la carpeta desde una página web, así que el archivo se descarga a la carpeta de descargas configurada en el navegador.

## Cómo funciona

Un `.elpx` es un archivo ZIP. La herramienta:

1. Abre el proyecto con [JSZip](https://stuk.github.io/jszip/).
2. Localiza el único `content.xml` y lo lee como UTF-8.
3. Convierte los delimitadores en el contenido HTML y también dentro de los bloques `jsonProperties` (JSON), volviendo a codificar cada cadena para que el JSON siga siendo válido. Solo cambia el **texto visible**: no toca las etiquetas ni sus atributos (por ejemplo, `onclick`), ni el contenido de `script`, `style`, `code`, `pre`, `kbd`, `samp` o `textarea`, ni los comentarios. Una fórmula no puede cruzar una etiqueta, así que nunca pasa de un párrafo a otro. Funciona igual si el HTML está en secciones CDATA, como texto escapado (proyectos de eXeLearning 3.0) o en atributos (`contentv3.xml` de los `.elp`), y solo se reescriben los valores que cambian.
4. Valida que cada bloque JSON y el XML final son correctos. Si alguna validación falla, **no genera ningún archivo**.
5. Vuelve a empaquetar el proyecto como un nuevo `.elpx`. El resto de ficheros del ZIP no se tocan.

El archivo original **nunca se modifica**.

## Qué se considera una fórmula y qué no (monedas)

Para no estropear textos con importes como `5$`, una expresión en línea `$…$` solo se convierte si cumple estas reglas:

- El `$` de apertura no va precedido de una letra, un número, otro `$` ni una barra invertida (`\$`).
- Hay un carácter que no es espacio **justo después** del `$` de apertura y **justo antes** del `$` de cierre.
- El `$` de cierre **no va seguido de un número** ni de otro `$`.
- La fórmula no cruza saltos de línea.

Las expresiones de bloque `$$…$$` se emparejan dentro del mismo párrafo y pueden ocupar varias líneas o llevar espacios junto a los `$$`.

| Texto | ¿Se convierte? |
|---|---|
| `$x^2$` | Sí |
| `$\left(1^{g}=100^{m}\right)$` | Sí |
| `cuesta 5$ y 10$` | No |
| `entre $5 y $10` | No |
| `de $5-$10` | No |
| `$ x $` (con espacios) | No |
| `$$ x $$` o `$$` en su propia línea | Sí |
| `<code>$x$</code>` | No (es código) |
| `onclick="$('a').click()"` | No (es un atributo) |

Al terminar, el estado indica cuántos signos `$` se han dejado sin cambios.

## Limitaciones

- **Las fórmulas con espacios junto a los `$` no se convierten.** `$ x $` hay que escribirlo como `$x$`.
- **Casos ambiguos:** un texto como `$5$` se interpreta como fórmula, porque es indistinguible de una. Lo mismo ocurre con dos importes en el mismo párrafo escritos de forma distinta, como `Pagas $5. Te devuelven 3$.`
- **Las fórmulas de los juegos no se convierten.** Los iDevices de juego (preguntas rápidas, adivina, crucigrama…) guardan sus datos cifrados, y la herramienta no los modifica.
- **Se presupone que los `$` del texto visible se usan como delimitadores matemáticos.** El código y los atributos no se tocan, pero un `$` escrito en el texto normal con la forma de una fórmula se convierte. Revisa el resultado.
- **Solo modifica `content.xml`.** Si hay fórmulas en otros ficheros del proyecto, no se tocan.
- **Requiere un único `content.xml`.** Si el proyecto tiene varios o ninguno, la herramienta se detiene con un mensaje de error.
- **Requiere codificación UTF-8** en `content.xml`.
- **Necesita conexión a Internet** para cargar JSZip desde una CDN (`cdn.jsdelivr.net`). Para usarla sin conexión, descarga `jszip.min.js` y cambia la etiqueta `<script src=…>` de `index.html` para que apunte al fichero local.
- La herramienta **no comprueba que las fórmulas resultantes sean LaTeX válido**, solo que el XML y el JSON siguen siendo correctos.
- Proyectos muy grandes consumen memoria del navegador, porque todo se procesa localmente.

## Privacidad

- El archivo se procesa **íntegramente en tu navegador**. No se sube a ningún servidor y no se almacena en ninguna parte.
- La herramienta no usa cookies, analítica ni almacenamiento local.
- La única petición de red es la carga de la biblioteca JSZip desde la CDN al abrir la página (la CDN puede registrar esa petición como cualquier otra descarga de un recurso web).

## Recomendaciones

- **Trabaja siempre con una copia** del proyecto y conserva el original.
- Tras la conversión, abre el resultado en eXeLearning, comprueba que las fórmulas se muestran bien y que las actividades se pueden editar.

## Compatibilidad

Navegadores modernos con soporte de expresiones regulares con *lookbehind* y propiedades Unicode (Chrome, Edge, Firefox y Safari recientes). La función de guardar en la misma carpeta que el original solo está disponible en navegadores Chromium.

## Estructura del repositorio

```text
index.html   Aplicación completa (HTML, CSS y JavaScript en un solo archivo)
README.md    Este documento
LICENSE      Texto completo de la Licencia Apache 2.0
```

## Licencia

Copyright 2026 Rubén Jiménez Jiménez

Este proyecto se distribuye bajo la [Licencia Apache 2.0](https://www.apache.org/licenses/LICENSE-2.0). Puedes usar, copiar, distribuir y modificar la aplicación, incluso con fines comerciales, siempre que:

- conserves el aviso de copyright y el texto de la licencia en las copias o versiones derivadas,
- indiques los cambios significativos que hayas hecho,
- y conserves el fichero `NOTICE`, si existe.

El texto completo está en el fichero [`LICENSE`](LICENSE). El software se ofrece «tal cual», sin garantías de ningún tipo.

## Aviso

Herramienta no oficial, sin relación con eXeLearning. Se ofrece «tal cual», sin garantías: revisa siempre el resultado antes de usarlo en materiales importantes.
