# CLAUDE.md — Sales Script App (NEPQ IUL)

## Descripción del proyecto

Aplicación web en un solo archivo HTML que funciona como guía de ventas en tiempo real para llamadas con leads. Se abrirá como PWA/web app desde Safari en Mac (guardada en el Dock). Combina el cuestionario NEPQ existente con scripts de transición por tipo de cliente. Todo el contenido fuente (HTML base + archivos caso_*.md) está incluido en el repositorio.

---

## Archivos del repositorio

```
/
├── index.html          ← archivo principal (el HTML base existente, modificado)
├── caso_1.md           ← script tipo C (sin ahorros)
├── caso_2.md           ← script tipo C (ahorra pero no le rinde)
├── caso_3.md           ← script tipo C/D (quiere retirarse, no tiene nada)
├── caso_4.md           ← script tipo D (tiene algo para el retiro pero le falta)
├── caso_5.md           ← script tipo B (quiere invertir, sin experiencia)
├── caso_6.md           ← script tipo B (quiere invertir, sin experiencia — variante)
└── CLAUDE.md           ← este archivo
```

El output final es **un solo `index.html` autocontenido** — todo CSS y JS inline, sin dependencias externas excepto Google Fonts.

---

## Mapeo de tipos de cliente → casos

| Tipo | Descripción del lead | Casos activos |
|------|----------------------|---------------|
| A | Objetivo no claro, hay que descubrirlo | Casos 1, 2, 3, 4, 5, 6 (todos) |
| B | Quiere invertir | Casos 5 y 6 |
| C | No ahorra / quiere ahorrar más | Casos 1 y 2 |
| D | Planea retiro | Casos 3 y 4 |
| E | Quiere comprar casa | *(ningún caso — excluir tipo E de la sección de casos)* |

Cuando el usuario selecciona un tipo, solo los casos que le corresponden deben ser visibles. Los demás desaparecen y el espacio se contrae (no solo se deshabilitan — se eliminan del flujo visual con animación de colapso).

---

## Cambios visuales y de diseño

### Fondo y tipografía
- Cambiar fondo de dark (`#0e0f11`) a **blanco** (`#ffffff`)
- Adaptar todos los colores de texto, bordes y fondos de componentes para funcionar sobre fondo claro
- Mantener las fuentes DM Sans y DM Mono (ya están cargadas desde Google Fonts)
- Aumentar legibilidad: tamaño de texto base mínimo 15px para preguntas, 13px para metadatos
- Asegurarse de que contraste de texto sea alto sobre fondo blanco

### Responsive / mobile
- La app debe verse bien en smartphone (viewport angosto ~375px)
- El selector de tipos ya tiene media query para 2 columnas en mobile — revisar y refinar
- Los casos desplegables deben ser fácilmente tapeables en móvil (mínimo 44px de altura en el header)
- Texto de los scripts debe ser legible sin zoom en iPhone

---

## Comportamiento de filtrado (preguntas)

### Comportamiento actual (mantener)
- Al seleccionar un tipo, los grupos con `data-types` que no incluyen ese tipo se marcan con clase `dimmed` (opacity baja, pointer-events none)

### Cambio requerido: colapso real
- En lugar de solo `dimmed`, los grupos que **no aplican** deben:
  1. Tener animación de colapso de altura (CSS transition en `max-height`)
  2. Desaparecer completamente del flujo visual (`display: none` después de la transición, o `max-height: 0` + `overflow: hidden`)
  3. El espacio entre los grupos que sí aplican debe contraerse limpiamente
- Los grupos que **sí aplican** deben permanecer completamente visibles sin cambios visuales de estado
- Al volver a "todos", todos los grupos reaparecen con animación de expansión

---

## Sección de Casos (nueva sección a agregar)

### Estructura
Agregar una nueva sección debajo de las preguntas NEPQ existentes, separada por un `<div class="sep">` y un label de sección tipo:
```
TRANSICIÓN → SCRIPT POR TIPO DE CLIENTE
```

### Cada caso es un acordeón desplegable con:
- Header con: nombre del caso, etiqueta del tipo al que corresponde (badge), chevron
- Body con el texto completo del script extraído del archivo `.md` correspondiente

### Filtrado de casos
- Igual que las preguntas: cuando se selecciona un tipo, solo los casos correspondientes son visibles; los demás desaparecen y el espacio se contrae
- Tipo A muestra todos los casos
- Tipo E no muestra ningún caso (la sección entera puede ocultarse o mostrar mensaje vacío)

### Sombreado de texto en los scripts (importante)
El texto de cada caso debe tener sombreado rotativo por fragmentos de frase para ayudar con modulación de voz durante la lectura. Reglas:

- Dividir el texto en fragmentos delimitados por **comas, puntos, o saltos de párrafo**
- Aplicar sombreado rotativo en ciclo: **verde → amarillo → rojo → verde → amarillo → rojo...**
- Los colores de fondo deben ser **muy suaves y translúcidos** — no dificultar la lectura, solo añadir una guía visual de ritmo:
  - Verde: `rgba(62, 207, 142, 0.12)` con borde izquierdo sutil `rgba(62, 207, 142, 0.4)`
  - Amarillo/ámbar: `rgba(245, 166, 35, 0.12)` con borde izquierdo `rgba(245, 166, 35, 0.4)`
  - Rojo: `rgba(242, 107, 107, 0.12)` con borde izquierdo `rgba(242, 107, 107, 0.4)`
- Aplicar el sombreado con `<span class="seg seg-green">`, `<span class="seg seg-amber">`, `<span class="seg seg-red">`
- Los spans deben ser inline y no romper el flujo natural del texto
- Nunca colorear un párrafo entero del mismo color — el ciclo debe cambiar dentro de cada párrafo
- Los títulos de sección dentro del script (ej. **EDUCACIÓN FINANCIERA EN NUESTROS PAÍSES**) van sin sombreado, en negrita, como separadores

### Calculadora de años (solo Caso 3)
El Caso 3 tiene una instrucción especial: incluir una mini calculadora interactiva integrada al comienzo del acordeón, con:
- **Input:** edad actual del cliente (número)
- **Input:** edad a la que quiere retirarse (número)
- **Output calculado automáticamente:** años que le quedan = edad retiro − edad actual
- Los valores calculados deben rellenarse automáticamente en el texto del script donde aparecen los placeholders `[Edad Actual]`, `[edad que dijo]`, y `[X]`
- Si los inputs están vacíos, los placeholders se muestran tal cual en el texto
- La calculadora debe aparecer visualmente integrada dentro del card del caso, no como elemento separado

---

## Comportamiento del selector de tipos (mejoras)

### Al seleccionar un tipo:
1. El botón seleccionado se activa visualmente (ya funciona)
2. Las preguntas que no aplican **desaparecen + espacio se contrae**
3. Los casos que no aplican **desaparecen + espacio se contrae**
4. La transición debe ser suave (~0.25s) para no ser abrupta durante una llamada en vivo

### Animación de colapso recomendada
Usar `max-height` transition para los grupos:
```css
.group { max-height: 2000px; overflow: hidden; transition: max-height 0.3s ease, opacity 0.25s; }
.group.hidden { max-height: 0; opacity: 0; pointer-events: none; margin: 0; padding: 0; }
```
Ajustar el valor máximo según necesidad real.

---

## Calculadora en Caso 3 — detalle técnico

```html
<!-- Dentro del acordeón del Caso 3, antes del texto del script -->
<div class="calc-block">
  <div class="calc-row">
    <label>Edad actual del cliente</label>
    <input type="number" id="calc-edad-actual" placeholder="ej. 38" oninput="actualizarCalc()">
  </div>
  <div class="calc-row">
    <label>Edad de retiro deseada</label>
    <input type="number" id="calc-edad-retiro" placeholder="ej. 65" oninput="actualizarCalc()">
  </div>
  <div class="calc-result" id="calc-resultado">— años hasta el retiro</div>
</div>
```

La función `actualizarCalc()` calcula la diferencia y reemplaza dinámicamente los spans con clase `placeholder-retiro`, `placeholder-edad`, `placeholder-anos` dentro del texto del Caso 3.

---

## Notas de implementación

- **Todo en un solo `index.html`** — no separar en CSS/JS externos. La app se guardará como archivo único en GitHub y se abrirá directamente en Safari.
- Mantener el sistema de copy-to-clipboard existente en las preguntas (click en pregunta → copia al portapapeles → toast).
- Los scripts de los casos NO necesitan función de copy (son para leer en voz alta, no para copiar).
- Probar que funcione offline una vez cargado en Safari (las fuentes de Google Fonts son la única dependencia externa; considerar incluir fallback stack sans-serif por si no hay internet).
- El archivo final no debe tener ningún `console.log` ni código de debug.
- Mantener los `data-types` attributes en todos los grupos de preguntas existentes — son la fuente de verdad para el filtrado.

---

## Lo que NO debe cambiar

- La lógica de copy-to-clipboard de las preguntas
- Los `data-types` de los grupos existentes
- El orden y contenido de las preguntas NEPQ
- Los badges de tipo en los headers de los grupos
- La sección de "Commitment Stage" (siempre visible para todos los tipos)
- La sección de "Puente hacia señalar el error" (siempre visible para todos los tipos)
