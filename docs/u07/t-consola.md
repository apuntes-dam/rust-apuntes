# 7.A Consola: entrada y salida

## Los tres flujos

| Flujo | En Rust | Para qué |
|---|---|---|
| Entrada estándar | `std::io::stdin()` | Lo que escribe el usuario |
| Salida estándar | `println!`, `print!` | La salida normal del programa |
| Salida de error | `eprintln!`, `eprint!` | Avisos y errores, **separados** de la salida normal |

Separar la salida normal de la de error permite redirigir cada una a su sitio: `cargo run > resultado.txt 2> errores.txt` guarda una en cada archivo.

```rust
use std::io;

fn main() {
    println!("Esto va a la salida normal");
    eprintln!("Esto va a la salida de error");

    print!("Sin salto de línea... ");
    io::Write::flush(&mut io::stdout()).unwrap();   // por si el terminal retiene la salida
    println!("ahora sí");
}
```

Salida:

```text
Esto va a la salida normal
Sin salto de línea... ahora sí
```

## Formato al mostrar

```rust
fn main() {
    let nombre = "Pan";
    let precio = 1.2;
    let unidades = 5;

    println!("{nombre:<10}{precio:>6.2}{unidades:>5}");    // alinear a la izquierda, derecha, con decimales
    println!("{:08.3}", 3.14159);                           // relleno con ceros
    println!("{:+}", 5);                                    // con signo siempre
    println!("{:#x} {:#b}", 255, 5);                         // hexadecimal y binario
}
```

Salida:

```text
Pan         1.20    5
0003.142
+5
0xff 0b101
```

## Argumentos de la línea de comandos

Un programa puede recibir datos al lanzarlo, sin esperar a que el usuario los escriba:

```rust
use std::env;

fn main() {
    let args: Vec<String> = env::args().collect();
    println!("{} argumentos", args.len());        // el primero siempre es el nombre del programa
    for (i, a) in args.iter().enumerate() {
        println!("{i}: {a}");
    }
}
```

Salida:

```text
1 argumentos
0: C:\Users\zyoan\AppData\Local\Temp\fx_ikpnfyvv\main.exe
```

Con `cargo run -- hola 42` los argumentos que llegan a `args` (sin contar el nombre del programa) son `hola` y `42`, siempre como texto.

## Errores frecuentes

| Error | Cómo evitarlo |
|---|---|
| Mezclar avisos con la salida normal | Los avisos van con `eprintln!`, no con `println!` |
| Esperar que `print!` muestre algo al momento | A veces el terminal retiene la salida hasta el siguiente salto de línea: fuerza con `io::stdout().flush()` |
| Pensar que `env::args()` incluye solo los argumentos | El primer elemento es **siempre** el nombre del programa |
| Un formato `{:.2}` que no redondea como esperas | Redondea al par más cercano en casos exactos (redondeo bancario); para el redondeo «de toda la vida» usa `.round()` antes |

## Para practicar

Haz los [ejercicios 7.1 de consola](consola.md).
