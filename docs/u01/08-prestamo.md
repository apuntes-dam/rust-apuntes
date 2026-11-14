# 1.8 Préstamo y slices (borrowing)

Mover un valor cada vez que lo necesitas en otro sitio es incómodo: la mayoría de las veces solo quieres que una función **mire** o **cambie** un dato, no que se quede con él para siempre. Para eso existe el **préstamo**: tomar una referencia (`&valor` o `&mut valor`) sin tomar la propiedad. El dato sigue teniendo un único dueño; la referencia solo lo señala durante un tiempo.

## Un préstamo sencillo

```rust
fn longitud(texto: &String) -> usize {   // toma prestado: no se queda con la propiedad
    texto.len()
}

fn main() {
    let s = String::from("hola");
    let n = longitud(&s);     // préstamo: "&s"
    println!("{s} tiene {n} letras");   // "s" sigue siendo válida
}
```

Salida:

```text
hola tiene 4 letras
```

`longitud` recibe `&String` (una referencia), así que `s` sigue siendo válida después de llamarla: la función solo **mira** el dato, no se queda con él.

## La regla del verificador de préstamos

En un momento dado, sobre un mismo valor puedes tener **o bien** varias referencias inmutables (`&T`) a la vez, **o bien** una única referencia mutable (`&mut T`) — nunca las dos cosas mezcladas.

```rust
fn main() {
    let mut contador = 0;

    let r1 = &contador;
    let r2 = &contador;
    println!("dos préstamos inmutables a la vez: {r1} {r2}");

    let r3 = &mut contador;   // ya no quedan préstamos inmutables en uso: puede ser mutable
    *r3 += 1;
    println!("tras un préstamo mutable en solitario: {}", contador);
}
```

Salida:

```text
dos préstamos inmutables a la vez: 0 0
tras un préstamo mutable en solitario: 1
```

`r1` y `r2` conviven porque los dos son inmutables. Para el préstamo mutable (`r3`), el compilador comprueba que ya no quede ninguna referencia inmutable **en uso**: como `r1` y `r2` no se vuelven a usar después de su `println!`, Rust permite crear `r3` justo después (se llama *non-lexical lifetimes*: una referencia "vive" hasta su último uso, no hasta el final del bloque).

Si en cambio pides dos préstamos mutables a la vez, el compilador se queja:

```rust
fn main() {
    let mut contador = 0;
    let r1 = &mut contador;
    let r2 = &mut contador;      // segundo préstamo mutable: el primero sigue activo
    println!("{r1} {r2}");
}
```

**No compila:**

```text
error[E0499]: cannot borrow `contador` as mutable more than once at a time
 --> main.rs:4:14
  |
3 |     let r1 = &mut contador;
  |              ------------- first mutable borrow occurs here
4 |     let r2 = &mut contador;      // segundo préstamo mutable: el primero sigue activo
  |              ^^^^^^^^^^^^^ second mutable borrow occurs here
5 |     println!("{r1} {r2}");
  |                -- first borrow later used here

error: aborting due to 1 previous error
```

El mensaje es literal: «no se puede prestar `contador` como mutable más de una vez a la vez». Es la misma regla que evita, en otros lenguajes, que dos hilos modifiquen el mismo dato sin protección — aquí el compilador la comprueba en un solo hilo, antes incluso de ejecutar el programa.

## Slices: un préstamo de una parte

Un **slice** es una referencia a un **rango** dentro de una colección, sin copiar nada:

```rust
fn main() {
    let texto = String::from("Hola Rust");
    let primera = &texto[0..4];      // slice: "Hola" (no copia, solo referencia un rango)
    let resto = &texto[5..];         // hasta el final: "Rust"
    println!("{primera} / {resto}");

    let numeros = vec![10, 20, 30, 40, 50];
    let medio = &numeros[1..3];      // slice de vector: [20, 30]
    println!("{medio:?}");
}
```

Salida:

```text
Hola / Rust
[20, 30]
```

`&texto[0..4]` y `&texto[5..]` son `&str`: apuntan dentro del mismo `String`, no crean uno nuevo. Lo mismo con `&numeros[1..3]` sobre un `Vec`: el tipo es `&[i32]`, un slice de array/vector.

## Por qué las funciones deberían pedir `&str` o `&[T]`

```rust
fn primera_palabra(texto: &str) -> &str {   // &str acepta literales, Strings y slices
    texto.split_whitespace().next().unwrap_or("")
}

fn main() {
    let literal = "Hola mundo";
    let propio = String::from("Adiós mundo cruel");
    println!("{}", primera_palabra(literal));    // funciona con un literal &str
    println!("{}", primera_palabra(&propio));    // y con un String (se presta como &str)
}
```

Salida:

```text
Hola
Adiós
```

`primera_palabra` acepta `&str`, así que funciona igual con un literal (`"Hola mundo"`) y con un `String` prestado (`&propio`): un `String` se presta como `&str` automáticamente. Si la función hubiera pedido `String` directamente, pasarle `propio` se lo habría quitado a quien la llama. La convención en Rust es: si solo necesitas **leer**, pide prestado (`&str`, `&[T]`); si necesitas quedarte con el dato o modificarlo de forma duradera, pide el valor (`String`, `Vec<T>`) o una referencia mutable (`&mut`).
