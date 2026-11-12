# 1.3 Tipos de datos

Rust es de **tipado estático**: cada variable tiene un tipo que se conoce al compilar. Casi nunca hay que escribirlo, porque el compilador lo deduce del valor, pero sigue estando ahí.

## Tipos básicos

| Tipo | Ejemplo | Nota |
|---|---|---|
| `i32` | `42` | entero con signo de 32 bits (el habitual) |
| `i64` | `42i64` | entero con signo de 64 bits |
| `u8` … `u64` | `255u8` | enteros **sin signo** (solo positivos) |
| `usize` | `3` | el tamaño de una colección o un índice |
| `f64` | `3.14` | decimal de 64 bits (el habitual) |
| `f32` | `3.14f32` | decimal de 32 bits |
| `bool` | `true` | solo `true` o `false` |
| `char` | `'A'` | un carácter Unicode (4 bytes) |
| `&str` | `"hola"` | texto literal, fijo |
| `String` | `String::from("hola")` | texto que puede crecer |

!!! warning "Dinero con enteros, no con `f64`"
    `f64` guarda los decimales en binario y no es exacto: `0.1 + 0.2` da `0.30000000000000004`. Para importes, guarda **céntimos en un entero** (`1999` céntimos) o usa una librería de decimales exactos (`rust_decimal`).

## Desbordamiento

Un `u8` va de 0 a 255. Si sumas 1 a 255, en modo de depuración el programa **para con un error** (no da la vuelta a 0 en silencio). Si quieres que dé la vuelta a propósito: `x.wrapping_add(1)`; si quieres detectarlo: `x.checked_add(1)` devuelve `None`.

## Cadenas

```rust
fn main() {
    let s = "Rust";
    println!("{}", s.len());                    // 4 (bytes, no letras)
    println!("{}", "hola".to_uppercase());      // HOLA
    println!("Hola, {s} y {}", s.len());        // interpolación
    println!("{}", "a,b,c".split(',').count()); // 3
    println!("{}", " texto ".trim());           // texto
    let mut saludo = String::from("Hola");
    saludo.push_str(", mundo");                 // String crece; &str no
    println!("{saludo}");
}
```

`len()` cuenta **bytes**: `"año".len()` es 4, porque la `ñ` ocupa dos. Para contar letras: `"año".chars().count()` da 3.

## Colecciones

```rust
use std::collections::{HashMap, HashSet};

fn main() {
    let mut lista = vec![3, 1, 2];                        // Vec: lista que crece
    let conjunto: HashSet<i32> = HashSet::from([1, 1, 2]); // sin repetidos
    let mapa = HashMap::from([("a", 1), ("b", 2)]);        // clave -> valor
    let fijo = [10, 20, 30];                               // array de tamaño fijo

    lista.push(4);
    println!("{:?}", lista);                 // [3, 1, 2, 4]
    println!("{}", conjunto.len());          // 2
    println!("{}", mapa["a"]);               // 1
    println!("{}", fijo.len());              // 3
}
```

`{:?}` muestra una colección entera (es el formato de **depuración**). Los tipos genéricos (`Vec<T>`, `HashSet<T>`, `HashMap<K, V>`) indican entre `< >` qué guardan.

## Conversiones

```rust
fn main() {
    let n: i32 = "42".parse().unwrap();          // texto -> número
    println!("{n}");                              // 42
    println!("{}", "abc".parse::<i32>().is_err()); // true (no para el programa)
    println!("{}", 42.to_string() + "!");         // 42!
    println!("{}", 3.9 as i32);                   // 3  (trunca)
    println!("{}", 3 as f64);                     // 3
    let grande: i64 = 5i32.into();                // conversión segura: i32 cabe en i64
    println!("{grande}");
}
```

- **Ampliar** (de un tipo pequeño a uno grande: `i32` a `i64`) se hace con `.into()` o `i64::from(x)`, y nunca pierde información.
- **Reducir** (de `f64` a `i32`, o de `i64` a `i32`) exige `as`, porque se puede perder información. `as` nunca para el programa: trunca o da la vuelta.
- `parse()` devuelve un `Result`: con `unwrap()` para el programa si el texto no es un número; con `is_err()` o `match` se comprueba sin fallar (unidad 2).

!!! note "Los decimales siempre con punto"
    A diferencia de C#, Rust no mira el idioma del equipo: `"3,5".parse::<f64>()` **falla** y `3.5` se escribe siempre con punto. Si el usuario escribe con coma, sustitúyela antes: `texto.replace(',', ".")`.

## `Option`: un valor que puede faltar

Rust no tiene `null`. Cuando un dato puede no existir, se usa `Option<T>`: vale `Some(dato)` o `None`, y el compilador obliga a tratar los dos casos. Por ejemplo, `lista.first()` devuelve `Option<&i32>`, y `apodo.unwrap_or("sin apodo")` da el valor o uno por defecto.
