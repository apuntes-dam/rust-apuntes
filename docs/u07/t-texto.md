# 7.C Ficheros de texto

## Leer

```rust
use std::fs;

fn main() {
    fs::write("notas.txt", "primera linea\nsegunda linea\n\nultima\n").unwrap();

    let contenido = fs::read_to_string("notas.txt").unwrap();      // todo de una vez
    println!("{} caracteres", contenido.chars().count());

    for (i, linea) in contenido.lines().enumerate() {               // línea a línea, sin el salto final
        println!("{i}: {linea:?}");
    }

    fs::remove_file("notas.txt").unwrap();
}
```

Salida:

```text
36 caracteres
0: "primera linea"
1: "segunda linea"
2: ""
3: "ultima"
```

Para archivos **grandes**, en vez de cargarlos enteros en memoria, se leen con un `BufReader`, línea a línea:

```rust
use std::fs::File;
use std::io::{BufRead, BufReader};

fn main() {
    std::fs::write("grande.txt", "a\nb\nc\n").unwrap();

    let archivo = File::open("grande.txt").unwrap();
    let lector = BufReader::new(archivo);
    let mut contador = 0;
    for linea in lector.lines() {
        let linea = linea.unwrap();     // cada línea es un Result (puede fallar por codificación)
        contador += 1;
        println!("{contador}: {linea}");
    }

    std::fs::remove_file("grande.txt").unwrap();
}
```

Salida:

```text
1: a
2: b
3: c
```

## Escribir y añadir

```rust
use std::fs::{self, OpenOptions};
use std::io::Write;

fn main() {
    fs::write("registro.txt", "primera entrada\n").unwrap();        // crea o SOBRESCRIBE

    let mut f = OpenOptions::new().append(true).open("registro.txt").unwrap();
    writeln!(f, "segunda entrada").unwrap();                         // añade al final, sin borrar lo anterior

    println!("{}", fs::read_to_string("registro.txt").unwrap());
    fs::remove_file("registro.txt").unwrap();
}
```

Salida:

```text
primera entrada
segunda entrada
```

`fs::write` siempre **sobrescribe** el archivo entero; para añadir hace falta `OpenOptions` con `.append(true)`.

## Cerrar el archivo

En Rust no hace falta cerrar nada a mano: cuando una variable de archivo (`File`, o el lector/escritor que lo envuelve) **sale de ámbito**, Rust lo cierra automáticamente (el *drop*). Es el mismo mecanismo que libera cualquier otro recurso, y evita el típico «se me olvidó cerrarlo».

## Errores frecuentes

| Error | Cómo evitarlo |
|---|---|
| Usar `fs::write` pensando que añade | Sobrescribe siempre: para añadir, `OpenOptions::new().append(true)` |
| Cargar un archivo enorme entero con `read_to_string` | Para archivos grandes, usa `BufReader` y procesa línea a línea |
| Olvidar que `.lines()` quita el salto de línea final de cada línea | Si lo necesitas, añádelo tú al volver a escribir |
| No comprobar la codificación | `read_to_string` falla si el archivo no es UTF-8 válido; para bytes «tal cual», usa `fs::read` (devuelve `Vec<u8>`) |

## Para practicar

Haz los [ejercicios 7.3 de ficheros de texto](texto.md).
