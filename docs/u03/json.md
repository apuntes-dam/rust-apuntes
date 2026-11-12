# U3.4 · JSON

<div class="ej-gate" data-unit="u03" data-nombre="U3 · Estructuras de datos"></div>

Leer, modificar y guardar datos en un archivo JSON.

## Ejercicio 3.4.1

**Gestión de usuarios en JSON.** Parte de este archivo `datos_usuarios_orig.json`:

```json
{
    "usuarios": [
        {"id": 1, "nombre": "Juan", "edad": 30},
        {"id": 2, "nombre": "Ana", "edad": 25}
    ]
}
```

Programa estas funciones:

* `mostrarDatos(datos)`: muestra `ID: <id>, Nombre: <nombre>, Edad: <edad>` por cada usuario entre las líneas `--- Contenido Actual del JSON ---` y `--- Fin del Contenido ---`; si no hay usuarios, avisa.
* `inicializarDatos()`: copia `datos_usuarios_orig.json` en `datos_usuarios.json`. Debe controlar que el origen **no exista** y que tenga un **JSON inválido**, con mensajes de error claros. Si va bien: `Datos inicializados desde '...' a '...'`.
* `cargarJson()` y `guardarJson()`.

En `main`: limpia la consola, inicializa, carga y muestra los datos y haz una pausa. Después, mostrando los datos y haciendo una pausa tras cada paso: **actualizar la edad** de un usuario, **insertar** uno nuevo, **eliminar** otro y **guardar** el archivo.

!!! note "En Rust"
    Usa el crate `serde_json` (con `serde` y `#[derive(Serialize, Deserialize)]`) y `std::fs` (`read_to_string`, `write`). Un JSON inválido o un archivo inexistente dan un `Err`: trátalo con `match` o con `?`.
