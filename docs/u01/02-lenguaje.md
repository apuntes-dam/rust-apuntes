# 1.2 Práctica con el lenguaje

## Estructura de un programa Rust

Todo programa empieza en la función `main`. Las funciones se declaran con `fn`, los parámetros llevan su tipo y el tipo de retorno va tras `->`:

```rust
// Programa completo: se guarda como main.rs
fn cuadrado(n: i32) -> i32 {
    n * n                      // sin punto y coma: es el valor que devuelve la función
}

fn main() {
    println!("Hola, Rust");
    println!("{}", cuadrado(5));   // 25
}
```

Reglas básicas: cada instrucción termina en `;`, los bloques van entre `{ }` y Rust **distingue mayúsculas de minúsculas**. La última expresión de un bloque, sin `;`, es su valor: así devuelven las funciones (también vale `return n * n;`). `println!` lleva `!` porque es una **macro**, no una función: genera el código que escribe en pantalla.

Las funciones y variables se escriben en `snake_case` (`calcular_area`), los tipos en `PascalCase` (`String`) y las constantes en `MAYUSCULAS`. El compilador avisa si no lo haces así.

## Variables

```rust
fn main() {
    let nombre = "Ana";          // el compilador deduce el tipo: &str (texto)
    let mut edad = 20;           // mut: se puede cambiar
    edad = 21;
    let altura: f64 = 1.68;      // tipo explícito
    let peso;                    // declarada, sin valor todavía
    peso = 62.5;                 // hay que darle valor antes de leerla
    println!("{nombre} tiene {edad} años, mide {altura} y pesa {peso}");
}
```

Las variables son **inmutables por defecto**: `let x = 5; x = 6;` no compila. Para cambiarlas hace falta `mut`. Es una de las ideas centrales de Rust: lo que puede cambiar se ve a simple vista.

!!! note "Sombreado (*shadowing*)"
    Se puede volver a declarar una variable con el mismo nombre, incluso con otro tipo: `let edad = "20"; let edad: i32 = edad.parse().unwrap();`. La segunda `edad` **sustituye** a la primera. Se usa mucho al convertir el texto leído del teclado.

## Constantes

| Forma | Cuándo se asigna | Ejemplo |
|---|---|---|
| `const` | En compilación; el tipo es obligatorio | `const IVA: f64 = 0.21;` |
| `static` | Una vez, para toda la ejecución; vive en una dirección fija | `static SALUDO: &str = "Hola";` |

Una `const` solo puede ser un valor que se conozca al compilar. Se escriben en mayúsculas y pueden declararse fuera de `main`.

## Literales

`42` (`i32`), `42u8` (`u8`, con el tipo como sufijo), `3.14` (`f64`), `3.14f32` (`f32`), `'a'` (`char`), `"hola"` (`&str`) y `true` (`bool`). Se pueden separar los miles con guion bajo: `1_000_000`. Hexadecimal `0xff`, binario `0b1010`.

## Operadores

```rust
fn main() {
    println!("{}", 7 + 2);          // 9
    println!("{}", 7 / 2);          // 3    (entero / entero = división entera)
    println!("{}", 7.0 / 2.0);      // 3.5  (siempre con punto)
    println!("{}", 7 % 2);          // 1
    println!("{}", 3 > 2 && 2 > 1); // true
    let mut x = 5;
    x += 2;                         // 7  (no existe x++)
    x += 1;                         // 8
    println!("{x}");
    let apodo: Option<&str> = None;
    println!("{}", apodo.unwrap_or("sin apodo"));   // sin apodo
    println!("{}", if x > 5 { "grande" } else { "pequeño" });   // if como expresión
}
```

Grupos: aritméticos (`+ - * / %`), relacionales (`== != < > <= >=`), lógicos (`&& || !`) y de asignación (`= += -= *=`). No hay `++` ni `--` ni operador `? :`: en su lugar, `if` es una **expresión** y devuelve un valor. No se pueden mezclar tipos: `7 / 2.0` no compila, hay que escribir `7.0 / 2.0` o `7 as f64 / 2.0`.

!!! tip "Comparar textos"
    En Rust `==` entre textos compara el **contenido**, como en Kotlin y Python.

## Entrada y salida

```rust
use std::io;

fn main() {
    println!("¿Cómo te llamas? ");
    let mut nombre = String::new();
    io::stdin().read_line(&mut nombre).unwrap();
    let nombre = nombre.trim();                      // quita el salto de línea final
    println!("Hola, {nombre}");                       // interpolación de variables
    println!("Tienes {} años", 20 + 1);               // las expresiones van fuera, en orden
}
```

`read_line` **añade** lo leído a un `String` que ya exista (por eso se crea vacío con `String::new()` y se pasa con `&mut`), y devuelve un `Result`: `.unwrap()` lo acepta y para el programa si falla. La línea incluye el salto final: `trim()` lo quita. `print!` no añade salto de línea y `println!` sí.

!!! note "Leer un número"
    Es lo mismo y después `parse()`: `let edad: i32 = texto.trim().parse().unwrap();`. El tipo de la variable le dice a `parse` qué número esperar. Si el texto no es un número, `unwrap()` para el programa con un mensaje; en la unidad 2 se ve cómo tratar ese error.

## Comentarios

```rust
// Comentario de una línea
/* Comentario
   de varias líneas */
/// Comentario de documentación: lo lee `cargo doc` y el editor
```
