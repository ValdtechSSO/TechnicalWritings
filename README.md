# TechnicalWritings

Repositorio para escribir, organizar y preparar articulos tecnicos, principalmente de programacion.

## Estructura sugerida

```text
.
|-- articles/
|   |-- drafts/
|   |-- published/
|-- shared/
|   |-- assets/
|   |-- snippets/
|-- templates/
|   |-- article-template.md
```

## Criterio

- `articles/drafts/`: articulos en progreso.
- `articles/published/`: articulos ya terminados o listos para publicar.
- `shared/assets/`: imagenes, diagramas o recursos reutilizables.
- `shared/snippets/`: fragmentos de codigo compartidos entre varios articulos.
- `templates/article-template.md`: plantilla base para arrancar nuevos articulos.

## Estructura recomendada por articulo

Cada articulo puede vivir en su propia carpeta para mantener juntos el texto, los assets y el codigo de ejemplo.

```text
articles/drafts/nombre-del-articulo/
|-- article.md
|-- assets/
|-- code/
```

Esto funciona bien para articulos de programacion porque evita mezclar imagenes o ejemplos entre temas distintos.

Si vas a publicar en varios idiomas, conviene que el articulo siga siendo la unidad principal y que cada idioma viva dentro de esa misma carpeta.

```text
articles/drafts/nombre-del-articulo/
|-- en/
|   |-- article.md
|-- es/
|   |-- article.md
|-- assets/
|-- code/
```

Asi compartes diagramas, imagenes y ejemplos de codigo entre idiomas sin duplicar contenido tecnico.

## Flujo simple

1. Crear el articulo dentro de `articles/drafts/`.
2. Escribir el contenido en `article.md`.
3. Guardar imagenes en `assets/`.
4. Guardar demos, scripts o proyectos pequenos en `code/`.
5. Mover la carpeta a `articles/published/` cuando el articulo este listo.
