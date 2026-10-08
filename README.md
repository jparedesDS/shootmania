# Pulsar Elite

Prototipo jugable de un shooter de arena inspirado en el modo **Elite** de *ShootMania Storm*: un atacante contra tres defensores, turnos de 60 segundos y un poste que se abre a los 15.

La idea es la misma que siguió *Arena Breakout* con *Escape from Tarkov*: reproducir las **mecánicas** (que no están protegidas por derechos de autor) con nombre, mapa, arte y código propios. No se usa ningún asset, nombre, mapa ni sonido de Nadeo/Ubisoft.

## Cómo jugar

Abre `index.html` en un navegador de escritorio (Chrome, Edge o Firefox). Necesita conexión para descargar Three.js desde jsDelivr.

```bash
cd shootmania
python -m http.server 8000   # y abre http://localhost:8000
```

| Tecla | Acción |
| --- | --- |
| `W A S D` | Moverse |
| Ratón | Apuntar |
| Clic | Disparar (mantener para repetir) |
| `Espacio` | Saltar; junto a una pared en el aire, rebote |
| `Mayús` | Esprintar (gasta energía) |
| `Esc` | Pausa |

## Reglas implementadas (fieles a Elite)

| Elemento | Atacante | Defensores |
| --- | --- | --- |
| Blindaje | 3 | 1 |
| Arma | Cohete: 4 cargas, regenera 1 cada 0,85 s, daño en área de 2,7 m | Láser instantáneo, 1 carga, recarga 1,6 s |
| Placas cian | Cambia a láser (recarga 1,1 s) mientras pisa una | — |
| Gana el turno si… | Captura el poste (1,6 s en el anillo, abierto a los 15 s) o elimina a los tres | Elimina al atacante o se agotan los 60 s |

- Los equipos se alternan en el ataque y gana la partida el primero que llega a 5 turnos.
- Movimiento rápido al estilo Storm: esprint con energía, rebote en paredes, salto con cohete (el propio cohete empuja pero no hace daño).
- Bots con IA propia: navegación A* sobre una rejilla de alturas, línea de visión, tiempo de reacción, error de puntería, desplazamiento lateral, reacción al ruido de disparos y lógica de rotación para defender el poste. Tres niveles de dificultad.

Equilibrio medido en simulación bot contra bot (40 turnos por nivel):

| Nivel | Turnos que gana el atacante |
| --- | --- |
| Fácil | ~80 % |
| Normal | ~57 % |
| Difícil | ~17 % |

## Arquitectura del prototipo

Un único `index.html` con Three.js 0.160 (ES modules):

- **Mapa**: lista de cajas AABB (`buildMap`). Las mismas cajas sirven para el render, las colisiones, los rayos del láser y la rejilla de navegación.
- **Física**: paso fijo de 1/120 s, colisión eje a eje con subida automática de escalones de 0,6 m.
- **Armas**: el láser es un rayo instantáneo contra cajas y jugadores; el cohete es un proyectil con barrido continuo, explosión con comprobación de línea de visión y empuje propio.
- **IA**: clase `Brain` con percepción a 10 Hz, A* con penalización junto a paredes y suavizado de ruta.
- **Pruebas**: `window.__pulsar` expone `tick()` y el estado para simular partidas completas en modo acelerado (`game.autopilot = true` pone un bot en lugar del jugador).

## Hoja de ruta hacia un juego completo

El prototipo valida las reglas y la sensación de juego. Para llegar a algo comparable a Storm, en orden de prioridad:

1. **Multijugador en línea (3c3)**. Servidor autoritativo en Node.js o Go con WebSocket/WebTransport a 60 Hz, predicción en cliente con reconciliación, interpolación de enemigos y compensación de latencia para el láser (rebobinar posiciones al instante del disparo). Es el 70 % del trabajo pendiente.
2. **Motor definitivo**. Para un juego comercial, Godot 4 (libre, exporta a web y escritorio) o Unreal 5. El código de reglas y la IA de este prototipo se portan casi tal cual.
3. **Editor de mapas por bloques**, la seña de identidad de ShootMania: piezas en rejilla (rampas, plataformas, placas láser, cristal, tubos), validación de simetría y publicación de mapas por la comunidad.
4. **Más modos con el mismo núcleo**: Royal (todos contra todos con zona que se cierra), Joust (1c1), Siege, Battle y Time Attack.
5. **Competitivo**: emparejamiento por Elo/Glicko-2, ligas, repeticiones, modo espectador con cámara libre y estadísticas por jugador.
6. **Física de Storm completa**: rampas reales (no solo escalones), carga de salto, deslizamiento por raíles, armas extra (Arrow, Nucleus) y tubos de teletransporte.
7. **Arte y audio originales**: estilo visual propio (este prototipo usa neón sobre bloques), personajes, anunciador y música.

## Límites legales a respetar

- Las mecánicas, reglas y géneros no tienen copyright: se pueden reproducir.
- No se pueden usar las marcas *ShootMania*, *Storm* ni *TrackMania*, ni sus logotipos, mapas oficiales, modelos, texturas, sonidos o código de ManiaPlanet. «Elite» es una palabra genérica y puede formar parte de un nombre propio.
- La interfaz y la «presentación» tampoco deben imitarse hasta el punto de confundir al usuario (*trade dress*).
