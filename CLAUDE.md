# Ruta de la Libertad VR

## Contexto del proyecto

Aplicación de realidad virtual que corre en **PC con las gafas Meta Quest conectadas
por Quest Link o Air Link (PC VR)** — **no** es un APK independiente para Android por
ahora. Desarrollada en **Unity 6000.3.29f1 (Unity 6.3 LTS)** con la plantilla VR (XR
Interaction Toolkit + OpenXR). Proyecto académico para la materia de
**Interculturalidad** de la ESPE. Un build autónomo para Quest (Android) queda como
posible extra para después, sin prioridad ahora.

### Idea central

El usuario aparece en un museo virtual, la **"Sala de la Libertad"**. Un **soldado
guía** (personaje Mixamo, con audio pregrabado, sin IA) lo recibe y le indica qué
puerta abrir. Cada puerta lleva a una **foto 360° real** de un sitio histórico de
Quito, con puntos de información (texto + narración con subtítulos). El usuario
regresa por la misma puerta, responde una pregunta al guía y gana una **medalla**
en su **"Pasaporte de la Libertad"**, que desbloquea la siguiente puerta.

### Fase 1 — Quito (5 puertas, en este orden)

1. Museo de Cera (Centro Cultural Metropolitano)
2. Casa Museo de Sucre
3. Museo Templo de la Patria
4. Mirador de la Cima de la Libertad
5. Ruco Pichincha

### Alcance realista

- Solo se modela/decora la **sala del museo**.
- Los sitios históricos son **fotos 360°** (no entornos 3D modelados).
- El guía es un personaje de **Mixamo** con **audio pregrabado** (sin IA, sin
  diálogo dinámico).
- Preguntas de opción múltiple (no reconocimiento de voz ni IA generativa).

## Estado actual del proyecto (verificado vía MCP de Unity)

- **Unity:** 6000.3.29f1 (6.3 LTS).
- **Paquetes XR instalados:** XR Interaction Toolkit 3.5.1, OpenXR Plugin 1.17.1,
  Unity OpenXR Meta 2.5.1 (backend OpenXR de Meta, **no** es el Meta XR SDK
  completo), XR Hands 1.8.1, XR Plugin Management 4.6.1, XR Composition Layers
  2.6.1, AR Foundation 6.5.0.
- **Meta XR SDK (com.meta.xr.sdk.*, Building Blocks, OVRCameraRig/OVRManager):
  NO está instalado** ni disponible en el registro configurado. Todo el soporte
  Meta actual pasa por el backend OpenXR estándar de Unity.
- **Escena abierta:** `Assets/Scenes/SampleScene.unity` (también existe
  `BasicScene.unity`), ambas de la plantilla VR de Unity con contenido de demo/tutorial.
- **Jerarquía de SampleScene (8 raíces):** `XR Origin Hands (XR Rig)` (rig completo
  con manos, controladores, teletransporte, viñeta de confort — **reutilizable**),
  `Interactables`, `UI` (paneles espaciales de demo), `Environment` (geometría de
  demo), `Teleport Area Setup` (**reutilizable** como base de locomoción),
  `Lighting`, `EventSystem`, `Hands Permissions Manager`.
- Carpetas de plantilla ya presentes en `Assets/`: `XR/`, `XRI/`, `Settings/`,
  `VRTemplateAssets/`, `TextMesh Pro/`, `Samples/`, `CompositionLayers/`.

## Decisiones de arquitectura tomadas

- **SDK:** se usa el stack estándar **OpenXR + XR Interaction Toolkit** ya
  instalado (sin Meta XR SDK / Building Blocks / OVRManager). Si más adelante
  se necesita passthrough u otra feature exclusiva de Meta, esto se reevalúa.
- **Hardware objetivo:** **Meta Quest 2 como mínimo** (presupuesto de polígonos,
  texturas y post-procesado debe correr bien ahí; Quest 3/3S/Pro se benefician
  pero no son el piso de rendimiento).
- **Carga de las fotos 360°:** **una escena Unity por sitio**, cargada de forma
  **aditiva** desde la escena `Museo.unity` (ver estructura de carpetas en
  `Assets/_RutaLibertad/Scenes/Sitios360/`).
- **Arte:** no hay assets propios todavía. Se arranca con **placeholders**
  (primitivas para sala y puertas, personaje Mixamo genérico para el guía) y se
  reemplazan por arte final más adelante sin cambiar la lógica.
- **Plataforma:** **PC VR vía Quest Link / Air Link**, build target
  **StandaloneWindows64**. No se cambia a Android; el proyecto se prueba
  directamente en Play Mode con las gafas conectadas. Ya verificado: el loader
  OpenXR está asignado para Standalone y los perfiles de interacción
  `OculusTouchControllerProfile` y `MetaQuestTouchPlusControllerProfile` (más
  `HandTracking`/`MetaHandTrackingAim`) están habilitados para ese build target.
- **Idioma:** español, con **subtítulos siempre activos** (narración del guía y
  de los sitios 360°).
- **Pregunta fallida:** el guía da una **pista** y el jugador puede **reintentar
  sin límite** de intentos (no hay bloqueo ni penalización).
- **Medallas:** estilo **militar** (insignias/condecoraciones).
- **Pasaporte:** metáfora de **pasaporte con sellos** (un sello por sitio
  completado, en vez de solo un ícono de medalla).

## Progreso

- **Paso 0 (limpieza de plantilla) — hecho.** Se copió `Assets/Scenes/SampleScene.unity`
  a `Assets/_RutaLibertad/Scenes/Museo.unity` (ahora la escena principal) y se
  borraron los roots de demo `Interactables`, `UI` y `Environment`. El root
  `Teleport Area Setup` se renombró a `Sala` y quedó vacío de contenido de
  demo (se borraron `Teleport Area` y `Teleport Anchor` originales). Se
  conservan `XR Origin Hands (XR Rig)`, `Lighting`, `EventSystem` y
  `Hands Permissions Manager` tal como venían de la plantilla.
- **Paso 1 (Sala de la Libertad, geometría simple) — hecho.** Bajo el root
  `/Sala` en `Museo.unity`:
  - `Piso`: plano 12×10 m con componente `TeleportationArea` (colliders y
    `TeleportationProvider` del XR Rig ya enlazados) — toda la sala es
    caminable/teletransportable.
  - `Techo`: cubo a 4 m de altura.
  - `Paredes/`: 4 paredes (`Pared_Puertas` en Z+5 con las 5 puertas,
    `Pared_Entrada` en Z-5, `Pared_Izquierda` en X-6, `Pared_Derecha` en X+6).
  - `Puertas/`: 5 cubos placeholder nombrados y ordenados
    `Puerta_01_MuseoCera` … `Puerta_05_RucoPichincha`, embebidos en
    `Pared_Puertas`, espaciados cada 2 m.
  - `SpawnGuia`: punto vacío en (0,0,2) mirando hacia el jugador, entre la
    entrada y la pared de puertas.
  - Materiales placeholder creados en `Assets/_RutaLibertad/Art/Materiales/`
    (`Mat_Piso`, `Mat_Pared`, `Mat_Techo`, `Mat_Puerta`) y asignados.
  - El `XR Origin Hands (XR Rig)` no se movió (sigue cerca de
    `(-1.17, 0, 0.84)`, dentro de la sala).
- **Pendiente (no hecho todavía):** lógica de puertas (bloqueo/desbloqueo,
  `SitioSO`), transición a escenas 360°, guía (Mixamo + estados + audio),
  preguntas, Pasaporte con sellos. Ver el plan por pasos acordado en la
  conversación para el orden sugerido.

## Reglas de trabajo

1. **Código en C#.**
2. **Comentarios en español.** Solo donde aclaren un "por qué" no evidente
   (una restricción, una decisión de diseño, un workaround); no se documenta
   lo obvio.
3. **No cambiar la versión de Unity** (6000.3.29f1 / 6.3 LTS). No actualizar ni
   degradar paquetes XR sin confirmación explícita.
4. Mantener separado el contenido propio del proyecto (`Assets/_RutaLibertad/`)
   del contenido generado por la plantilla/paquetes (`XR/`, `XRI/`,
   `VRTemplateAssets/`, etc.), que se conserva como base del rig VR.
5. Preferir reutilizar y configurar el rig XR de la plantilla (XR Origin Hands,
   Teleport Area Setup, Lighting, EventSystem) antes de escribir sistemas propios
   de locomoción o interacción desde cero.
6. No introducir dependencias de IA (voz, generación de texto) — el guía usa
   audio pregrabado y lógica de estados simple.

## Estructura de carpetas propuesta en `Assets/`

```
Assets/
  _RutaLibertad/                     # Todo el contenido propio del proyecto
    Scenes/
      Museo.unity                    # Escena principal: Sala de la Libertad
      Sitios360/
        01_MuseoCera.unity
        02_CasaMuseoSucre.unity
        03_TemploPatria.unity
        04_CimaLibertad.unity
        05_RucoPichincha.unity
    Scripts/
      Core/                          # Estados de juego, GameManager, utilidades
      Puertas/                       # Lógica de puertas (bloqueo/desbloqueo, trigger)
      Transicion360/                 # Carga/descarga de escenas 360, fade
      Guia/                          # Comportamiento del guía, diálogos, estados
      Pasaporte/                     # Lógica de medallas y progreso
      Preguntas/                     # Lógica de preguntas de opción múltiple
      UI/                            # Scripts de interfaz (paneles, subtítulos)
    ScriptableObjects/
      Sitios/                        # Un asset por sitio: nombre, puerta, escena,
                                      # clips de narración, pregunta, medalla
      Preguntas/                     # Banco de preguntas por sitio
      Medallas/                      # Definición de medallas del pasaporte
    Prefabs/
      Guia/
      Puertas/
      Pasaporte/
      UI/
      Interaccion360/                # Puntos de información 360 (hotspots)
    Art/
      Modelos/
        SalaMuseo/
        Puertas/
        Guia/                        # Modelo Mixamo + animaciones
      Materiales/
      Texturas/
      Panoramas360/                  # Fotos equirectangulares, una carpeta por sitio
        01_MuseoCera/
        02_CasaMuseoSucre/
        03_TemploPatria/
        04_CimaLibertad/
        05_RucoPichincha/
    Audio/
      Narracion/                     # Audio pregrabado del guía y de cada sitio
      Musica/
      SFX/
    UI/
      Fuentes/
      Iconos/
      Subtitulos/                    # Archivos de subtítulos (si no van en código)

  # Carpetas existentes de la plantilla/paquetes (no mover, no renombrar):
  XR/, XRI/, Settings/, VRTemplateAssets/, TextMesh Pro/, Samples/, CompositionLayers/
  Scenes/                            # Escenas de demo de la plantilla (SampleScene,
                                      # BasicScene) — conservar como referencia o
                                      # eliminar cuando ya no se necesiten para probar el rig
```
