# 2D Shell — el marco donde se dibuja cualquier juego 2D

2D Shell es la pantalla vacía que los juegos 2D rellenan: la imagen del lugar actual, el reloj y la barra de herramientas arriba, espacio para paneles y la esquina de notificaciones. Por sí sola no muestra nada. Los mods de contenido meten en ella sus propias capas: el Dialog Engine añade el cuadro de texto y las opciones de una novela visual, y el mod RPG añade el mapa de tiles transitable, de modo que una novela y un RPG de tiles comparten el mismo marco, la misma barra de herramientas y los mismos menús.

Lo incluye todo juego 2D: Cthangover, My Novel y RPG Plus.

![La misma carcasa en dos juegos: una escena de Cthangover con el cuadro de texto y las opciones, y la aldea de RPG Plus con el mapa de tiles; ambos tienen el reloj arriba a la izquierda y la barra de herramientas arriba a la derecha.](assets/two-games.jpg)

## Lo que ves en pantalla

![Una escena de Cthangover: el fondo de la cocina, el reloj y un contador de raciones arriba a la izquierda, la barra de herramientas arriba a la derecha, las opciones «Leave» y «Cook», y el cuadro de texto abajo.](assets/novel-scene.jpg)

En esta escena, la carcasa aporta:

- **la imagen del lugar** — la cocina, con su sombreado según la hora y sus luces (Time Engine);
- **la barra superior** — el reloj arriba a la izquierda, la barra de herramientas arriba a la derecha y, en medio, el indicador de guardado y, de noche, la lámpara portátil;
- **la esquina de notificaciones** bajo el reloj;
- **espacio para paneles** — la bolsa, el mapa o el diario se abren sobre la escena.

El cuadro de texto y las opciones vienen del Dialog Engine, y el contador de raciones de Cooking: son capas que esos mods añaden a la carcasa.

## Las capas, de atrás hacia delante

```text
BaseScene
└── Scene
    ├── ModFirstPanel        capas de mods por detrás de todo
    ├── ViewBox              el lugar: luces, fondo, primer plano, Lastground
    ├── (content layer)      capa de un mod de contenido, p. ej. el lienzo de diálogos o el mundo RPG
    ├── TopView              reloj, barra de herramientas, indicador de guardado, lámpara
    ├── ModBeforeUIPanel     paneles de la barra: bolsa, mapa, diario, menú…
    ├── NotificationPanel    mensajes en la esquina superior izquierda
    └── ModLastPanel         capas superpuestas sobre todo, p. ej. la ventana de la mesa de trabajo
```

Las capas posteriores se dibujan sobre las anteriores.

## Para autores de mods

**Añadir una capa de contenido con un parche de escena.** Un parche es un `.tscn` en la carpeta `patches/` de tu mod, en la misma ruta que la escena de la carcasa. Instancia la escena de la carcasa y añade o cambia nodos. El parche del Dialog Engine añade el lienzo de diálogos justo después del visor:

```text
patches/scenes/ui/base_scene.tscn
```

```text
[ext_resource type="PackedScene" path="res://mods/shell2d_base/scenes/ui/base_scene.tscn" id="1_target"]
[ext_resource type="PackedScene" path="res://scenes/ui/dialog_canvas.tscn" id="3_dialog"]

[node name="BaseScene" instance=ExtResource("1_target")]

[node name="DialogCanvas" parent="Scene" index="2" instance=ExtResource("3_dialog")]
```

El mod RPG añade su `RpgWorldHost` del mismo modo, así que un juego con ambos muestra el mapa de tiles y, encima, las líneas de diálogo.

**Añadir un panel sin parche.** Un panel de la barra o una capa superpuesta se declara en la sección `ui` del manifiesto con uno de los slots de la carcasa: `ModBeforeUIPanel`, `ModLastPanel` o el `Lastground` del visor. Los campos están en la guía del mod UI.

**Declara la dependencia.** Un mod que necesite el marco 2D lista `shell2d_base` en `depends`; la carcasa de por sí trae Time Engine y UI.

## Si algo falla

1. **Una pantalla negra con solo la barra de herramientas.** Ningún mod de contenido está rellenando la carcasa (falta el Dialog Engine o el mod RPG en el juego), o la escena no tiene fondo.
2. **Un panel aparece detrás de la escena.** Su manifiesto nombra un slot que se dibuja demasiado pronto; usa `ModBeforeUIPanel` o `ModLastPanel`.
