# 1.7 Propiedad (ownership)

Rust no tiene recolector de basura, pero tampoco obliga a liberar la memoria a mano: cada valor tiene un único **dueño**, y en cuanto ese dueño sale de su ámbito (el bloque `{ }` donde vive), Rust libera el dato automáticamente. El compilador comprueba esta regla al compilar, así que los errores de memoria (usar algo ya liberado, liberarlo dos veces...) no llegan a ejecutarse: **no compilan**.

## Mover vs copiar

Al asignar una variable a otra, Rust hace una de dos cosas, según el tipo:

| Tipo | Qué pasa al asignar | Ejemplos |
|---|---|---|
| Implementa `Copy` (tipos simples, de tamaño fijo y conocido) | Se **copia**: las dos variables quedan independientes y válidas | `i32`, `f64`, `bool`, `char`, tuplas de solo tipos `Copy` |
| No implementa `Copy` (el resto) | Se **mueve**: la variable de origen deja de ser válida | `String`, `Vec<T>`, casi cualquier `struct` |

```rust
fn main() {
    let a = 5;          // i32: implementa Copy
    let b = a;           // se copia; "a" sigue siendo válida
    println!("{a} {b}");

    let s1 = String::from("hola");   // String: no implementa Copy
    let s2 = s1.clone();             // copia explícita con clone()
    println!("{s1} {s2}");           // las dos siguen siendo válidas
}
```

Salida:

```text
5 5
hola hola
```

`a` sigue siendo válida después de `let b = a;` porque `i32` es `Copy`. `s1`, en cambio, no se copia sola al asignar: para quedarte con **dos** `String` independientes hace falta `.clone()`, que copia también el contenido (no solo la variable).

## Qué pasa si usas un valor después de moverlo

```rust
fn main() {
    let s1 = String::from("hola");
    let s2 = s1;          // s1 se MUEVE a s2
    println!("{s1}");      // error: s1 ya no es válida
}
```

**No compila:**

```text
error[E0382]: borrow of moved value: `s1`
 --> main.rs:4:16
  |
2 |     let s1 = String::from("hola");
  |         -- move occurs because `s1` has type `String`, which does not implement the `Copy` trait
3 |     let s2 = s1;          // s1 se MUEVE a s2
  |              -- value moved here
4 |     println!("{s1}");      // error: s1 ya no es válida
  |                ^^ value borrowed here after move
  |
help: consider cloning the value if the performance cost is acceptable
  |
3 |     let s2 = s1.clone();          // s1 se MUEVE a s2
  |                ++++++++

error: aborting due to 1 previous error; 1 warning emitted
```

El compilador no deja compilar este programa: en cuanto `s1` se mueve a `s2`, `s1` deja de ser válida. El propio mensaje de error sugiere la solución (`.clone()`).

## Propiedad y funciones

Pasar un valor a una función también mueve la propiedad, salvo que la función la devuelva:

```rust
fn mostrar(texto: String) {   // la función se queda con la propiedad
    println!("Recibido: {texto}");
}

fn devolver(texto: String) -> String {   // puedes recuperarla si la devuelves
    println!("Procesando: {texto}");
    texto
}

fn main() {
    let a = String::from("hola");
    mostrar(a);                 // "a" se mueve dentro de mostrar

    let b = String::from("adiós");
    let b = devolver(b);        // se recupera la propiedad (variable nueva, por el sombreado)
    println!("{b}");
}
```

Salida:

```text
Recibido: hola
Procesando: adiós
adiós
```

`mostrar` se queda con `a` para siempre: después de llamarla, `a` ya no se puede usar. `devolver`, en cambio, hace algo con `texto` y lo **devuelve**, así que `b` sigue siendo una variable válida después de la llamada. La alternativa más habitual para no tener que devolver nada es **prestar** el valor en vez de moverlo: se ve en la página siguiente.

## Constantes y variables estáticas

```rust
const IVA: f64 = 0.21;          // constante: se sustituye al compilar
static SALUDO: &str = "Hola";   // vive en una dirección fija durante toda la ejecución

fn main() {
    println!("{} {}", SALUDO, IVA);
}
```

Salida:

```text
Hola 0.21
```

| Forma | Vive | Se puede modificar |
|---|---|---|
| `const` | Se sustituye por su valor al compilar; no ocupa una dirección propia | No, nunca |
| `static` | Una sola dirección fija, durante toda la ejecución del programa | Con `mut`, pero exige `unsafe` |

Una `static mut` existe, pero casi nunca hace falta: modificar un dato global desde varias partes del programa (y, peor, desde varios hilos) es precisamente lo que la propiedad y el préstamo evitan. Si de verdad necesitas estado global mutable, las herramientas seguras son `std::sync::Mutex` o `std::sync::OnceLock`.
