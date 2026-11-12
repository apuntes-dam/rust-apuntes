# 1.4 Pruebas

Una **prueba unitaria** comprueba automáticamente que una función devuelve lo esperado. En Rust las pruebas vienen **incluidas en el lenguaje**: se marcan con `#[test]` y se ejecutan con `cargo test`, sin instalar nada.

## Crear el proyecto

```bash
cargo new calculadora --lib
cd calculadora
```

`--lib` crea una **biblioteca** (código para usar desde otros programas) con un archivo `src/lib.rs` que ya trae una prueba de ejemplo.

## La función y sus pruebas

En Rust lo habitual es escribir las pruebas **en el mismo archivo** que el código, en un módulo `tests` al final:

```rust
// src/lib.rs
pub fn suma(a: i32, b: i32) -> i32 {
    a + b
}

pub fn dividir(a: i32, b: i32) -> f64 {
    if b == 0 {
        panic!("b no puede ser 0");
    }
    a as f64 / b as f64
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn suma_dos_positivos() {
        assert_eq!(suma(2, 3), 5);
    }

    #[test]
    #[should_panic(expected = "b no puede ser 0")]
    fn dividir_por_cero_falla() {
        dividir(1, 0);
    }

    #[test]
    fn suma_varios_casos() {
        for (a, b, esperado) in [(1, 2, 3), (0, 0, 0), (-1, 1, 0)] {
            assert_eq!(suma(a, b), esperado);
        }
    }
}
```

```bash
cargo test
```

- `#[test]` marca una función como prueba. `#[cfg(test)]` hace que el módulo solo se compile al probar.
- `assert_eq!(obtenido, esperado)` comprueba un valor; `assert!(condición)` comprueba que algo es cierto.
- `#[should_panic]` comprueba que la función **para el programa** (`panic!`) con ese mensaje.
- No hay `[Theory]` como en xUnit: varios casos se recorren con un bucle dentro de la prueba.

!!! tip "Patrón AAA"
    **A**rrange (preparar), **A**ct (ejecutar), **A**ssert (comprobar).
