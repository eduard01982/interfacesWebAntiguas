# Estilo frontend "Slackware" (diálogo ncurses)

Guía para reproducir en cualquier proyecto web el aspecto de los diálogos del
instalador de Slackware (programa `dialog` sobre ncurses): fondo azul, cajas
cian con sombra negra, títulos amarillos, listas con tecla rápida en rojo y
botones `< OK >` / `<Cancelar>`.

La implementación de referencia es [`slackware.html`](slackware.html).

---

## 1. Tipografía

Una sola fuente: **Share Tech Mono** (Google Fonts). No usar otras.

```html
<link rel="preconnect" href="https://fonts.googleapis.com">
<link href="https://fonts.googleapis.com/css2?family=Share+Tech+Mono&display=swap" rel="stylesheet">
```

```css
:root {
  --font: "Share Tech Mono", monospace;
  /* Escala con la ventana para que un diálogo de 64 columnas quepa en el móvil */
  --fs: min(17px, calc((100vw - 32px) / 42));
}
body {
  font-family: var(--font);
  font-size: var(--fs);
  line-height: 1.15;
  -webkit-font-smoothing: none;
}
```

Reglas:

- Todo es monoespaciado. Medir anchos y sangrías en `ch` y alturas en `em`,
  nunca en píxeles, para que se mantenga la rejilla de caracteres.
- Share Tech Mono solo tiene peso normal; el "negrita" (títulos, opciones de
  la lista) lo sintetiza el navegador y está bien así.
- Texto en una sola línea con `white-space: pre` cuando la alineación importa.

## 2. Paleta (VGA de 16 colores)

```css
:root {
  --azul:     #0000a8; /* fondo de pantalla y selección */
  --cian:     #00a8a8; /* fondo de los diálogos */
  --blanco:   #fcfcfc; /* texto normal, bordes con luz */
  --amarillo: #fcfc54; /* títulos y opciones de la lista */
  --rojo:     #a80000; /* tecla rápida, porcentaje */
  --negro:    #000000; /* sombras, bordes en sombra, texto de botones */
  --gris:     #a8a8a8; /* texto de ayuda */
  --verde:    #54fc54; /* flechas de desplazamiento ↑(-) ↓(+) */
}
body { background: var(--azul); color: var(--blanco); }
```

No introducir colores fuera de esta paleta ni degradados.

| Elemento                   | Fondo    | Texto                         |
|----------------------------|----------|-------------------------------|
| Pantalla                   | azul     | —                             |
| Título de fondo (backtitle)| azul     | cian                          |
| Diálogo                    | cian     | blanco                        |
| Título del diálogo         | cian     | amarillo, negrita             |
| Opción de lista            | cian     | amarillo, primera letra rojo  |
| Opción seleccionada        | azul     | blanco                        |
| Botón inactivo             | cian     | negro, `<` `>` blanco, tecla rápida roja |
| Botón activo               | azul     | blanco, tecla rápida subrayada|
| Porcentaje de la lista     | cian     | rojo                          |

## 3. Estructura

```html
<div class="backtitle">Mi Programa · Instalación</div>

<div class="dialog">
  <div class="dialog-title"><span>TÍTULO EN MAYÚSCULAS</span></div>
  <div class="prompt">Texto explicativo de 3-5 líneas.</div>

  <div class="listbox">
    <span class="arrow up">↑(-)</span>
    <ul>
      <li><span class="item"><b>O</b>pción uno</span></li>
      <li class="sel"><span class="item"><b>O</b>pción dos</span></li>
    </ul>
    <span class="arrow down">↓(+)</span>
    <span class="percent">48%</span>
  </div>

  <div class="sep"></div>
  <div class="buttons">
    <span class="btn active"><span class="br">&lt;</span>  <span class="hk">O</span>K  <span class="br">&gt;</span></span>
    <span class="btn"><span class="br">&lt;</span><span class="hk">C</span>ancelar<span class="br">&gt;</span></span>
  </div>
</div>
```

## 4. CSS de los componentes

### Diálogo con relieve y sombra

El borde es "en relieve": claro arriba/izquierda y oscuro abajo/derecha. La
sombra negra sólida se desplaza unas 2 columnas a la derecha y 1 fila abajo.

```css
.dialog {
  position: relative;
  width: 64ch;
  max-width: 100%;
  background: var(--cian);
  border-top: 0.12em solid var(--blanco);
  border-left: 0.12em solid var(--blanco);
  border-right: 0.12em solid var(--negro);
  border-bottom: 0.12em solid var(--negro);
  outline: 0.5ch solid var(--cian);          /* margen cian alrededor del borde */
  box-shadow: 2.5ch 1.2em 0 0 var(--negro);  /* sombra sólida, sin desenfoque */
  padding: 0.25em 1ch 0.5em;
}
.dialog-title {
  position: absolute; top: -0.62em; left: 0; right: 0;
  text-align: center;
}
.dialog-title span {
  background: var(--cian);   /* "corta" la línea del borde */
  color: var(--amarillo);
  font-weight: bold;
  padding: 0 1ch;
}
```

### Lista hundida

Bordes invertidos respecto al diálogo (oscuro arriba/izquierda, claro
abajo/derecha). Las flechas y el porcentaje se colocan sobre el borde con
fondo cian para "cortarlo".

```css
.listbox {
  position: relative;
  margin: 0.5em 1ch 0.6em;
  border-top: 0.12em solid var(--negro);
  border-left: 0.12em solid var(--negro);
  border-right: 0.12em solid var(--blanco);
  border-bottom: 0.12em solid var(--blanco);
  padding: 0.35em 0;
}
.arrow   { position: absolute; left: 3ch; background: var(--cian); color: var(--verde); line-height: 1; padding: 0 .3ch; }
.arrow.up   { top: -0.55em; }
.arrow.down { bottom: -0.55em; }
.arrow.off  { color: var(--cian); }   /* oculta la flecha sin mover nada */
.percent { position: absolute; right: 5ch; bottom: -0.55em; background: var(--cian); color: var(--rojo); line-height: 1; padding: 0 .3ch; }

ul { list-style: none; margin: 0; padding: 0; }
li { padding-left: 9ch; white-space: pre; cursor: pointer; }
li .item   { color: var(--amarillo); font-weight: bold; }
li .item b { color: var(--rojo); }                     /* tecla rápida */
li.sel .item, li.sel .item b { background: var(--azul); color: var(--blanco); }
```

### Separador y botones

```css
.sep { border-top: 0.12em solid var(--blanco); margin: 0.5em -1ch 0.4em; }
.buttons { display: flex; justify-content: center; gap: 8ch; white-space: pre; }
.btn { cursor: pointer; color: var(--negro); padding: 0 .2ch; }
.btn .hk { color: var(--rojo); font-weight: bold; }
.btn .br { color: var(--blanco); }
.btn.active { background: var(--azul); color: var(--blanco); }
.btn.active .hk { color: var(--blanco); text-decoration: underline; }
```

## 5. Comportamiento (teclado)

Igual que `dialog`:

| Tecla              | Acción                                          |
|--------------------|-------------------------------------------------|
| ↑ / ↓              | Mover la selección una línea                    |
| RePág / AvPág      | Mover una página (número de filas visibles)     |
| Inicio / Fin       | Primera / última opción                         |
| Letra              | Saltar a la siguiente opción que empiece por ella |
| Tab, ← / →         | Cambiar de botón                                |
| Intro              | Activar el botón seleccionado                   |
| Esc                | Cancelar                                        |

Lógica de desplazamiento: se muestran `VISIBLES` filas (11 en la referencia)
a partir de `inicio`; al mover la selección se ajusta `inicio` para que siga
visible. El porcentaje es `(inicio + VISIBLES) / total`. Las flechas `↑(-)` /
`↓(+)` se ocultan (clase `off`) cuando no hay más opciones en esa dirección.

> Ojo: no llamar `top` a una variable global en JavaScript, choca con
> `window.top` y rompe el script.

Los mensajes de confirmación son otro `.dialog` más pequeño (≈46ch), centrado
sobre la pantalla, con un único botón `< OK >`.

## 6. Reglas de estilo

- Títulos de diálogo en MAYÚSCULAS.
- Sin esquinas redondeadas, sin degradados, sin sombras difuminadas, sin
  animaciones.
- Todo centrado en la pantalla; el diálogo nunca supera el ancho de la ventana
  (`max-width: 100%`) y la fuente se reduce en móvil con `--fs`.
- `user-select: none` en el cuerpo para que se comporte como una interfaz de
  terminal, no como un documento.
