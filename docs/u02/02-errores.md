# 2.3 Errores: `Result`, `Option` y `panic!`

Rust **no tiene excepciones**. En su lugar separa dos situaciones:

| Situación | Cómo se trata | Ejemplo |
|---|---|---|
| Algo **puede fallar de forma normal** y el programa debe decidir qué hacer | Un valor **`Result<T, E>`** (o `Option<T>`) que el código está **obligado** a mirar | Convertir «abc» en número, abrir un archivo que no existe |
| Algo **no debería pasar jamás**: un fallo del programador | **`panic!`**: el programa se detiene con un mensaje | Un índice fuera de rango, `unwrap()` sobre un error |

La ventaja: el tipo de la función **dice** si puede fallar, y el compilador no te deja olvidarlo.

## `Result`: un valor o un error

`Result<T, E>` tiene dos variantes: **`Ok(valor)`** si salió bien y **`Err(error)`** si no. Se trata con `match`:

```rust
fn main() {
    for texto in ["42", "abc", "", "3.5", " 7 "] {
        match texto.parse::<i32>() {
            Ok(n) => println!("{texto:?} -> {n}"),
            Err(e) => println!("{texto:?} -> error: {e}"),
        }
    }
}
```

Salida:

```text
"42" -> 42
"abc" -> error: invalid digit found in string
"" -> error: cannot parse integer from empty string
"3.5" -> error: invalid digit found in string
" 7 " -> error: invalid digit found in string
```

Los mensajes de error (`invalid digit found in string`, `cannot parse integer from empty string`) vienen de la biblioteca estándar y **están en inglés**.

## `Option`: un valor que puede faltar

Cuando algo puede **no existir** sin que sea un error (el primer elemento de una lista vacía, una clave que no está en un mapa), se usa `Option<T>`: **`Some(valor)`** o **`None`**:

```rust
fn main() {
    let numeros = vec![4, 8, 15];
    println!("{:?} {:?}", numeros.first(), numeros.get(10));

    let vacio: Vec<i32> = Vec::new();
    println!("{:?}", vacio.iter().max());

    // unwrap_or da un valor por defecto si es None
    println!("{}", numeros.get(10).copied().unwrap_or(0));
}
```

Salida:

```text
Some(4) None
None
0
```

## Atajos: `unwrap`, `expect` y `unwrap_or`

| Método | Qué hace | Cuándo |
|---|---|---|
| `unwrap()` | Da el valor, o **para el programa** (`panic`) si es `Err` / `None` | Pruebas y programas de prueba, o cuando es imposible que falle |
| `expect("mensaje")` | Igual que `unwrap`, pero con **tu mensaje** | Cuando fallar sería un fallo del programa |
| `unwrap_or(x)` | El valor, o `x` si falla | Cuando hay un valor por defecto razonable |
| `unwrap_or_else(\|\| ...)` | El valor, o el resultado de calcular otro | Si el valor por defecto es caro de calcular |
| `is_ok()` / `is_err()` / `is_some()` | Pregunta, sin sacar el valor | Comprobaciones sueltas |

```rust
fn main() {
    let n: i32 = "42".parse().unwrap();
    let m: i32 = "abc".parse().unwrap_or(-1);
    println!("{n} {m}");

    println!("{}", "abc".parse::<i32>().is_err());
}
```

Salida:

```text
42 -1
true
```

## El operador `?`: pasar el error al que me llamó

Una función que puede fallar **devuelve un `Result`**. Con `?`, si una operación da `Err`, la función **termina en ese punto y devuelve ese error**; si da `Ok`, sigue con el valor:

```rust
use std::num::ParseIntError;

fn suma_de_dos(a: &str, b: &str) -> Result<i32, ParseIntError> {
    let x: i32 = a.trim().parse()?;     // si falla, la función devuelve ese error aquí mismo
    let y: i32 = b.trim().parse()?;
    Ok(x + y)
}

fn main() {
    println!("{:?}", suma_de_dos("20", " 22 "));
    println!("{:?}", suma_de_dos("20", "veinte"));

    match suma_de_dos("1", "x") {
        Ok(total) => println!("Total: {total}"),
        Err(e) => println!("No se pudo sumar: {e}"),
    }
}
```

Salida:

```text
Ok(42)
Err(ParseIntError { kind: InvalidDigit })
No se pudo sumar: invalid digit found in string
```

## Errores propios

Para dar a un fallo del programa un nombre y un mensaje propios, se define un tipo (normalmente un `enum`) y se implementa `Display` para su mensaje:

```rust
use std::fmt;

#[derive(Debug)]
enum ErrorAcceso {
    ContrasenaIncorrecta,
    UsuarioBloqueado(String),
}

impl fmt::Display for ErrorAcceso {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        match self {
            ErrorAcceso::ContrasenaIncorrecta => write!(f, "Incorrect Password!!"),
            ErrorAcceso::UsuarioBloqueado(quien) => write!(f, "El usuario {quien} está bloqueado"),
        }
    }
}

impl std::error::Error for ErrorAcceso {}

fn entrar(usuario: &str, clave: &str) -> Result<(), ErrorAcceso> {
    if usuario == "root" {
        return Err(ErrorAcceso::UsuarioBloqueado(usuario.to_string()));
    }
    if clave != "secreto" {
        return Err(ErrorAcceso::ContrasenaIncorrecta);
    }
    Ok(())
}

fn main() {
    for (u, c) in [("ana", "secreto"), ("ana", "1234"), ("root", "secreto")] {
        match entrar(u, c) {
            Ok(()) => println!("{u}: acceso concedido"),
            Err(e) => println!("{u}: Error: {e}"),
        }
    }
}
```

Salida:

```text
ana: acceso concedido
ana: Error: Incorrect Password!!
root: Error: El usuario root está bloqueado
```

`impl std::error::Error for ErrorAcceso {}` indica que es un error «de verdad» y permite devolverlo dentro de un `Box<dyn Error>` junto a otros tipos de error.

## Un patrón útil: pedir un dato hasta que sea válido

```rust
use std::io;

fn main() {
    loop {
        println!("Escribe un número entero positivo:");
        let mut linea = String::new();
        io::stdin().read_line(&mut linea).unwrap();

        match linea.trim().parse::<i32>() {
            Ok(n) if n > 0 => {
                println!("Gracias, has escrito {n}");
                break;
            }
            Ok(_) => println!("Tiene que ser positivo."),
            Err(_) => println!("Eso no es un número."),
        }
    }
}
```

Salida:

```text
Escribe un número entero positivo:
Eso no es un número.
Escribe un número entero positivo:
Tiene que ser positivo.
Escribe un número entero positivo:
Gracias, has escrito 17
```

Se repite con `loop` hasta que el valor es correcto; el `break` sale solo cuando todo está bien. Observa la **guarda** `Ok(n) if n > 0`, que mezcla `match` y una condición.

## Cuándo usar `Result` y cuándo `panic!`

* **`Result`** para lo que **puede pasar** aunque el programa esté bien escrito: datos del usuario, ficheros, red.
* **`panic!`** (o `unwrap`/`expect`) para un **fallo del programador**: una condición que, si se rompe, significa que el código está mal. En una biblioteca, casi nunca.
* Un `panic!` **no se captura** como una excepción: el hilo termina. Por eso en Rust se devuelven errores en lugar de lanzarlos.

```rust
fn dividir(a: i32, b: i32) -> Result<i32, String> {
    if b == 0 {
        return Err(String::from("no se puede dividir entre cero"));
    }
    Ok(a / b)
}

fn main() {
    println!("{:?}", dividir(7, 2));
    println!("{:?}", dividir(7, 0));

    let lista = vec![1, 2, 3];
    // println!("{}", lista[10]);   // esto sí es un panic: «index out of bounds»
    println!("{:?}", lista.get(10)); // la forma segura devuelve un Option
}
```

Salida:

```text
Ok(3)
Err("no se puede dividir entre cero")
None
```

## Errores frecuentes

* **Poner `unwrap()` en todas partes.** Es cómodo, pero cada `unwrap` es un `panic` esperando a un dato raro. En programas de verdad usa `?`, `match` o `unwrap_or`.
* **Tragarse el error.** `if let Ok(n) = x.parse::<i32>() { ... }` sin `else` descarta el fallo en silencio; al menos avisa.
* **Olvidar el `.trim()` antes de `parse()`**: la línea leída lleva el salto final y `"42\n".parse::<i32>()` falla.
* **Usar `?` en `main` sin cambiar su tipo.** `main` tiene que devolver `Result<(), ...>` para poder usar `?` dentro.
* **Confundir `Err(e)` con la causa real.** Muestra el mensaje con `{e}`; con `{e:?}` ves además el tipo.

## Para practicar

Haz los [ejercicios 2.3 de errores](excepciones.md). El objetivo es que **ningún error llegue al programa principal sin tratar** y lo aborte.
