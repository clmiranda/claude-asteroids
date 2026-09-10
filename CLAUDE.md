# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Descripción del proyecto

Clon del clásico arcade **Asteroids** implementado en HTML5 Canvas puro, sin dependencias ni bundler. Todo el juego vive en un único archivo (`game.js`).

## Cómo correr

No hay build ni tests. Para jugar/probar cambios, abre `index.html` directamente en el navegador, o sirve el directorio:

```bash
npx serve .
```

Luego visita `http://localhost:3000`. Al editar `game.js`, solo hace falta recargar la página (no hay paso de compilación).

## Arquitectura

Todo el estado y la lógica residen en `game.js`, organizado en secciones delimitadas por comentarios `// ── Sección ──`:

- **Input**: `keys` (estado continuo de teclas) y `justPressed` (detección de flanco de subida vía `pressed(code)`) se llenan desde los listeners `keydown`/`keyup` globales.
- **Clases de entidades**: `Bullet`, `Asteroid`, `Ship`, `Particle`. Cada una expone `update(dt)` y `draw()`, y se marca a sí misma con `this.dead = true` cuando debe eliminarse (patrón consistente en todo el juego).
- **Estado global del juego**: variables de módulo (`ship`, `bullets`, `asteroids`, `particles`, `score`, `lives`, `level`, `state`) en vez de una clase Game. `state` es una máquina de estados simple: `'playing' | 'dead' | 'gameover'`.
- **`update(dt)`**: rama según `state`, luego actualiza entidades, filtra las `dead` con `.filter()`, y resuelve colisiones bala↔asteroide y nave↔asteroide por fuerza bruta (O(n·m), aceptable dado el número bajo de entidades).
- **`draw()`**: limpia el canvas y dibuja en orden: partículas → asteroides → balas → nave → HUD → overlay de estado.
- **Loop principal**: `requestAnimationFrame` con `dt` en segundos, clamped a 0.05 (`loop()` al final del archivo).

### Convenciones a mantener

- El mundo es toroidal: todo movimiento posicional pasa por `wrap(v, max)` para envolver en los bordes del canvas (`W`×`H` = 800×600).
- Tamaños de asteroide van de 3 (grande) a 1 (pequeño); `RADII`, `SPEEDS` y `POINTS` son arrays indexados por tamaño. Al partirse, un asteroide genera dos de tamaño `size - 1` (`Asteroid.split()`); tamaño 1 no se parte.
- Las entidades nunca se eliminan in-place de los arrays: se marcan `dead = true` y se filtran al final del `update`.
- No hay clases de UI ni motor externo — HUD y overlays (`drawHUD`, `drawOverlay`) son funciones que dibujan directo sobre el `ctx` 2D.
- El código y los comentarios están en español; mantené ese idioma al editar `game.js` y `README.md`.
