# 1.1 Un programa

Un **programa** es una secuencia de instrucciones que resuelve un problema, y el **algoritmo** son los pasos, en orden, que van de los datos de entrada al resultado.

!!! info "La teoría general está aparte"
    Qué es un algoritmo, el ciclo de desarrollo y el **pseudocódigo** son iguales en todos los lenguajes, así que están en una sola página: [Fundamentos: algoritmos y pseudocódigo](https://apuntes-dam.github.io/apuntes-lenguajes/fundamentos/).

## Del pseudocódigo a Rust

El algoritmo del área de un rectángulo, en pseudocódigo:

```text
ALGORITMO areaRectangulo
  LEER base
  LEER altura
  area <- base * altura
  ESCRIBIR "Área:", area
FIN
```

y su traducción a Rust:

```rust
use std::io;

fn main() {
    let mut texto = String::new();

    println!("Base: ");
    io::stdin().read_line(&mut texto).unwrap();
    let base: f64 = texto.trim().parse().unwrap();

    texto.clear();
    println!("Altura: ");
    io::stdin().read_line(&mut texto).unwrap();
    let altura: f64 = texto.trim().parse().unwrap();

    let area = base * altura;
    println!("Área: {area}");
}
```

| Pseudocódigo | Rust |
|---|---|
| `LEER x` | `io::stdin().read_line(&mut texto)` (deja texto; se convierte con `texto.trim().parse()`) |
| `x <- expresión` | `let x = expresión;` |
| `ESCRIBIR x` | `println!("{x}")` |
| `SI ... SINO ... FIN SI` | `if` / `else` (ver [1.2](02-lenguaje.md)) |
| `PARA` / `MIENTRAS` | `for` / `while` (ver [1.6](06-bucles.md)) |

!!! warning "Errores típicos en Rust"
    Olvidar el `;` al final de cada instrucción, intentar cambiar una variable sin `mut` (no compila), mezclar un entero con un decimal en una operación (`7 / 2.0` no compila: hay que convertir con `as f64`) y olvidar el `.trim()` antes de `parse()`, porque la línea leída termina en salto de línea. El compilador de Rust explica cada error con detalle: léelo entero, casi siempre dice cómo arreglarlo.
