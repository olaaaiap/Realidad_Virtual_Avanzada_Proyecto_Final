# Quantum Kingdom — Shooter en realidad virtual con Unreal Engine

> Proyecto final de la asignatura **Realidad Virtual Avanzada**
> Autora de la parte individual: **Olaia Picabea Sainz** · Unreal Engine 5.5 · C++ · Blueprints · OpenXR

**Quantum Kingdom** es un juego de tipo shooter en primera persona desarrollado en Unreal Engine para las gafas de realidad virtual Meta Quest. El juego se ambienta en un entorno futurista pero medieval, y el jugador tiene que viajar al pasado para acabar con las criaturas medievales y así evitar la invasión extraterrestre que destruirá la tierra.

Este repositorio contiene la **versión individual** del proyecto. Parte de la demo desarrollada en grupo y añade mis mejoras: migración de la lógica a C++, nuevos enemigos con IA, un socket de arma en la muñeca, mejoras de rendimiento y mejoras de presencia en VR.

📄 [Leer el Game Design Document completo (PDF)](GDD_proyecto_final_RVAV_Olaia_Picabea.pdf)

🎬 [Ver video explicativo del proyecto que se entregó con el código](https://youtu.be/T0rUCFJFJO0)


## El juego

1. **Nave (tutorial).** El jugador empieza en una nave futurista. Una voz por megafonía le dirige a la sala de entrenamiento, donde practica disparando a dianas fijas y móviles.
2. **Portal temporal.** Desde la nave cruza un portal hecho con partículas Niagara que le lleva al pasado.
3. **Coliseo medieval (Level2).** En la arena tiene que eliminar las dianas mientras esquiva los fuegos y a los monstruos que aparecen, que le persiguen y le quitan vidas.
4. **Final.** La partida termina en una escena de victoria al eliminar todas las dianas, o de *game over* al perder las 5 vidas.

El contraste visual es parte del diseño. La nave usa materiales metálicos, luz fría y tonos azulados con elementos emisivos. El coliseo usa piedra, antorchas y braseros con iluminación cálida, y fuego hecho con Niagara.

### Controles (Meta Quest)

| Acción | Control |
|---|---|
| Moverse (*smooth locomotion*) | Joystick izquierdo |
| Girar (*snap turn*) y teletransporte | Joystick derecho |
| Coger y soltar el arma | Grip |
| Disparar | Gatillo |
| Interactuar con los menús | Disparando con la pistola de menú |

## Mi aporte individual

### 1. Migración de Blueprints a C++

He reescrito en C++ la lógica de los Blueprints del proyecto grupal. Las nuevas clases son los padres de los Blueprints originales, así que el comportamiento se mantiene y la lógica principal queda en código.

| Clase C++ | Blueprint | Responsabilidad |
|---|---|---|
| `AAPistol` | `Pistol` | Coger y soltar el arma, disparo con la mano izquierda o derecha, munición (`CurrentAmmo`, `MaxAmmo`), sonido y vibración háptica |
| `AAPistolMenu` | `BP_APistolMenu` | Hereda de `AAPistol`: pistola de munición infinita para interactuar con los menús |
| `AAProjectile` | `Projectile` | Proyectil con colisión: daña dianas y enemigos y activa los botones de la interfaz |
| `AATarget` | `BP_Target` | Dianas fijas o móviles, con movimiento oscilante configurable (amplitud y velocidad) |
| `AADamageSphere` | `BP_DamageSphere` | Zona de daño (los fuegos de la arena) que quita vida al jugador al entrar en ella |
| `AALevel2Manager` | `BP_Level2Manager` | Cuenta los objetivos eliminados en la arena y detecta el final de la partida |

### 2. Nuevos enemigos con inteligencia artificial

- **Spawner.** `BP_EnemyManager` crea pequeñas criaturas moradas animadas en puntos concretos de la arena cada cierto tiempo, en grupos de 1, 2 o 3 según el momento de la partida.
- **Behavior Tree de Unreal.** Los enemigos usan uno completo: `BT_Basic_Enemy`, `BB_Basic_Enemy`, `BP_AI_Controller_Basic`, un *Service*, un *Decorator* y las *Tasks* `WalkTo` y `ClearFocus`. Cuando el jugador se acerca, el enemigo le persigue y, si le toca, le quita una vida.
- **Se pueden eliminar.** Igual que las dianas, desaparecen al recibir un proyectil.


### 3. Socket de arma en la muñeca

La pistola va siempre en un socket en la muñeca del jugador. Se puede coger en cualquier momento y, al soltarla, vuelve sola a su sitio. Así el jugador no puede perder el arma y quedarse sin poder continuar la partida.

- El arma depende del nivel:
  - pistola de menú en `InitialShip`
  - pistola con munición en `Tutorial` y `Level2`
  - ninguna en las escenas de transición, victoria y *game over*
- Corregí un bug por el que el arma seguía disparando después de soltarla. Lo hice con un flag `bIsHeld`.

### 4. Rendimiento

- **Portal.** Eliminé la colisión de cada partícula de Niagara, porque el portal ya tiene un *box collider* general.
- **Fuegos.** Reduje su número en la arena a unos 5, quité las colisiones de las chispas y eliminé el componente *Light* de las llamas.
- **Blueprints.** Revisé los eventos *Tick* para quitar carga innecesaria y eliminé los *prints* de depuración.

### 5. Presencia en VR

- **Música adaptada a cada nivel:** ambiente para la nave, música de tensión en la arena, y temas propios para la victoria y el *game over*.
- **Pasos.** Suena un paso cada vez que el jugador recorre 110 unidades, para que no dé la sensación de estar levitando.
- **Háptica.** Vibración fuerte al recibir daño, junto con un sonido de dolor, y una vibración suave en la mano que dispara.
- **Teletransporte** en la arena, con el *NavMesh* configurado, para poder escapar de los enemigos.
- **Smooth locomotion rehecho.** La implementación anterior solo funcionaba en un ordenador.

### 6. Correcciones visuales

- **Vidas.** Los corazones del HUD solo se muestran en el nivel de combate (`Level2`).
- **Munición.** El contador de la pistola no se actualizaba desde C++. Lo arreglé guardando la referencia al widget (`CachedWidget`) y ajustando su *draw space*.
- **Manos.** La mano izquierda usaba el modelo de la derecha. La invertí escalando en Y con valor -1.

## Sistemas base del proyecto grupal

Además de mi parte, el repo incluye los sistemas creados en grupo. El proceso está documentado paso a paso en [`Content/Olaia/proceso.md`](Content/Olaia/proceso.md):

- `OlaiaVRPawn`: copia del VR Pawn del template, para no modificar el original.
- Sistema de munición en la pistola.
- Sistema de vidas, con daño y **3 segundos de invulnerabilidad** tras cada golpe.
- *Game over* al llegar a 0 vidas.
- Menú inicial con Play, Settings y Exit, y pantallas de victoria y *game over*.

## Diferencias con el GDD

El GDD describe un juego más ambicioso que lo que se pudo implementar en el tiempo de la asignatura, que fue además la primera experiencia del equipo con Unreal.

- **La arena.** Se basa en dianas, con apariciones de monstruos con IA, en lugar de oleadas completas.
- **El final.** No hay jefe final: la partida termina al eliminar todas las dianas.
- **Funcionalidades pendientes:** menú de ajustes, recarga, varias armas, *power-ups* y salto.

## Contenido del repositorio

- `Source/QuantumKingdom/`: clases C++ del juego (`Public/` y `Private/`).
- `Content/Olaia/`: mi trabajo en el editor:
  - `IA/`: Behavior Tree, Blackboard, AI Controller, Tasks, Services y Decorators.
  - `Audio/`: música y efectos de sonido.
  - Blueprints, `OlaiaVRPawn` y `proceso.md`.
- `Content/`: resto del contenido del proyecto (VR Template, modelos, armas, materiales, mapas…).
- `Config/`: configuración del proyecto de Unreal.
- `QuantumKingdom.uproject`: proyecto de Unreal Engine 5.5 con OpenXR.
- `GDD_proyecto_final_RVAV_Olaia_Picabea.pdf`: Game Design Document con el detalle de la parte grupal e individual.

## Lo que he aprendido

- Combinar C++ y Blueprints en Unreal: crear clases C++ como padres de Blueprints existentes y exponer propiedades y funciones al editor (`UPROPERTY`, `UFUNCTION`).
- IA de enemigos con Behavior Trees, Blackboards, AI Controllers y NavMesh.
- Interacción en VR con OpenXR: coger objetos, sockets, locomoción continua y teletransporte, háptica.
- Optimizar efectos de partículas (Niagara) y luces para mantener el rendimiento en VR.
- Diseñar para la presencia en VR: audio contextual, feedback háptico y coherencia visual del cuerpo del jugador.
