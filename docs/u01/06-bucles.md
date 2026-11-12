# 1.6 Bucles: `for`, `while` y `loop`

Un bucle repite un bloque de código. En Rust hay tres formas: `for`, `while` y `loop`.

## `for`: recorrer un rango o una colección

```rust
fn main() {
    for i in 1..=5 {
        println!("Número: {i}");
    }
}
```

| Parte | Qué hace |
|---|---|
| `1..=5` | **Rango** de 1 a 5, ambos incluidos |
| `1..5` | Rango de 1 a 4 (el final **no** se incluye) |
| `i` | La variable toma cada valor del rango |
| `{ ... }` | Lo que se repite |

No hay un `for` con inicio, condición y avance como en C o Java: en Rust el `for` siempre recorre algo (un rango, una lista, un texto...).

## `for` sobre una colección

```rust
fn main() {
    let nombres = vec!["Ana", "Luis", "Eva"];
    for nombre in &nombres {
        println!("{nombre}");
    }
    println!("{}", nombres.len());    // la lista sigue disponible gracias al &
}
```

Se recorre con `&nombres` (una **referencia**) para seguir usando la lista después; con `for nombre in nombres` la lista se **consume** y ya no se puede usar. No se puede modificar la colección mientras se recorre.

## Variantes del rango

```rust
fn main() {
    for i in (1..=5).rev() { print!("{i} "); }        // descendente: 5 4 3 2 1
    println!();
    for i in (1..=10).step_by(2) { print!("{i} "); }  // de dos en dos: 1 3 5 7 9
    println!();
    for i in 1..5 { print!("{i} "); }                 // sin llegar a 5: 1 2 3 4
    println!();
}
```

Para recorrer con índice, `enumerate()` da el índice y el elemento a la vez:

```rust
fn main() {
    let nombres = vec!["Ana", "Luis", "Eva"];
    for (i, nombre) in nombres.iter().enumerate() {
        println!("{i}: {nombre}");
    }
}
```

## Comparación rápida

| Característica | `for` | `while` | `loop` |
|---|---|---|---|
| Cuándo | Recorrer un rango o colección | Repetir mientras se cumpla algo | Repetir hasta que algo diga `break` |
| Condición | Implícita (se acaba lo recorrido) | Antes de cada vuelta | Ninguna: es infinito salvo `break` |
| `break` / `continue` | Sí | Sí | Sí (y `break` puede devolver un valor) |

## `while` y `loop`

```rust
fn main() {
    let mut cuenta = 3;
    while cuenta > 0 {            // comprueba antes
        println!("{cuenta}");
        cuenta -= 1;
    }

    let mut intentos = 0;
    loop {                        // se ejecuta al menos una vez
        intentos += 1;
        println!("Intento {intentos}");
        if intentos == 3 {
            break;
        }
    }
}
```

No hay `do-while`: su papel lo hace `loop` con un `break` al final. `loop` también puede **devolver un valor**: `let resultado = loop { ... break valor; };`.

## Cortar un bucle con una variable de control

Una forma sencilla de parar un bucle sin `break`: una variable booleana (una *bandera*) que el bucle consulta en su condición. Cuando pasa lo que buscas, la pones en `false`.

```rust
fn main() {
    let mut activo = true;
    let mut i = 1;

    while activo {
        println!("{i}");
        if i == 5 {
            activo = false;    // se cumple la condición: el bucle se detiene
        }
        i += 1;
    }
}
```

??? note "Opcional: `break` y `continue`"
    Existen y funcionan igual que en Java, pero no son imprescindibles: casi siempre se puede escribir lo mismo con una variable de control.

    ```rust
    fn main() {
        for i in 1..=10 {
            if i == 3 { continue; }   // salta esta vuelta
            if i == 6 { break; }      // sale del bucle
            println!("{i}");          // 1, 2, 4, 5
        }
    }
    ```
