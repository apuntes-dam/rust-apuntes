# 6.D Comentarios y documentación

## Un buen comentario explica el porqué

El código ya dice **qué** hace; un comentario útil explica **por qué** se hizo así, sobre todo cuando la razón no es evidente:

```rust
fn main() {
    let descuento = 0.30;   // comentario inútil: repite lo que ya se ve

    // Black Friday: el 30% se decidió en la reunión del 3 de noviembre, revisar cada año
    let descuento_bueno = 0.30;

    println!("{} {}", descuento, descuento_bueno);
}
```

Salida:

```text
0.3 0.3
```

## Documentar con `///`: lo que lee el editor y `cargo doc`

Rust tiene **comentarios de documentación** (`///`, encima del elemento) que son a la vez comentarios y datos: el editor los muestra al autocompletar, y `cargo doc` genera con ellos una página HTML navegable, como el Javadoc de Java o los XML doc de C#.

```rust
/// Calcula el precio final de un pedido con impuestos.
///
/// # Parámetros
/// - `subtotal`: el importe antes de impuestos.
/// - `tipo_iva`: el tipo de IVA en tanto por ciento (por ejemplo, `21.0`).
///
/// # Ejemplos
/// ```
/// let total = precio_final(100.0, 21.0);
/// assert_eq!(total, 121.0);
/// ```
fn precio_final(subtotal: f64, tipo_iva: f64) -> f64 {
    subtotal * (1.0 + tipo_iva / 100.0)
}

/// Un producto del catálogo.
struct Producto {
    /// Nombre visible para el cliente.
    nombre: String,
    /// Precio sin impuestos, en euros.
    precio: f64,
}

fn main() {
    println!("{}", precio_final(100.0, 21.0));
    let p = Producto { nombre: String::from("Teclado"), precio: 25.0 };
    println!("{}: {}", p.nombre, p.precio);
}
```

Salida:

```text
121
Teclado: 25
```

Algo propio de Rust: el bloque de código dentro de un comentario `///` (entre ``` ```` ```) **se compila y se ejecuta como prueba** con `cargo test`. Si cambias `precio_final` y el ejemplo deja de ser cierto, la prueba falla y te enteras: la documentación **no puede quedarse desactualizada** sin que algo avise.

## Cómo se hace en Rust

| Necesito... | Escribo |
|---|---|
| Documentar un elemento (función, struct, campo) | `///` justo encima |
| Documentar el módulo o el crate entero | `//!` al principio del archivo |
| Una sección dentro del comentario | `# Parámetros`, `# Ejemplos`, `# Panics` (markdown) |
| Un ejemplo que además se comprueba | Un bloque de código ` ``` ` dentro del `///` |
| Generar la documentación en HTML | `cargo doc --open` |
| Comentario normal (no genera documentación) | `//` o `/* ... */` |

## Qué documentar

* La **API pública** (todo lo `pub`): quien la usa no tiene por qué leer el código de dentro.
* **Cuándo puede fallar**: si una función hace `panic!` en algún caso, dilo en una sección `# Panics`.
* **Por qué**, no **qué**: el código ya dice qué hace línea a línea.
* Lo que **no** es evidente: una decisión de negocio, un número mágico, una limitación conocida.

## Errores frecuentes

| Error | Cómo evitarlo |
|---|---|
| Comentar cada línea obvia (`// suma uno`) | Son ruido: bórralos |
| Un `///` que describe el código y no **por qué** existe | Explica la razón, no la traducción línea a línea |
| Dejar un ejemplo de documentación desactualizado | No pasa inadvertido: `cargo test` lo ejecuta y falla si está mal |
| Documentar con `//` en vez de `///` sobre un elemento público | `cargo doc` no recoge los comentarios normales, solo los de documentación |

## Para practicar

Termina la unidad con los [ejercicios 6.11 al 6.13](documentacion.md).
