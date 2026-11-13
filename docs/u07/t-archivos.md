# 7.B Archivos y carpetas

## Rutas: `Path` y `PathBuf`

| Tipo | Es como... | Para qué |
|---|---|---|
| `&Path` | `&str` | Una ruta **prestada**, de solo lectura |
| `PathBuf` | `String` | Una ruta **propia**, que se puede construir y modificar |

```rust
use std::path::{Path, PathBuf};

fn main() {
    let ruta = Path::new("datos/clientes/ana.txt");

    println!("{:?}", ruta.file_name());        // Some("ana.txt")
    println!("{:?}", ruta.extension());        // Some("txt")
    println!("{:?}", ruta.parent());            // Some("datos/clientes")
    println!("{}", ruta.is_absolute());

    let mut nueva = PathBuf::from("datos");     // construir una ruta pieza a pieza
    nueva.push("clientes");
    nueva.push("luis.txt");
    println!("{}", nueva.display());
}
```

Salida:

```text
Some("ana.txt")
Some("txt")
Some("datos/clientes")
false
datos\clientes\luis.txt
```

## Comprobar, crear y borrar

```rust
use std::fs;
use std::path::Path;

fn main() {
    fs::create_dir_all("demo/sub").unwrap();     // crea «demo» y «demo/sub» de una vez, sin fallar si ya existen
    fs::write("demo/a.txt", "hola").unwrap();

    println!("{}", Path::new("demo/a.txt").exists());
    let meta = fs::metadata("demo/a.txt").unwrap();
    println!("{} bytes, es archivo: {}", meta.len(), meta.is_file());

    for entrada in fs::read_dir("demo").unwrap() {
        let entrada = entrada.unwrap();
        println!("{}", entrada.path().display());
    }

    fs::remove_file("demo/a.txt").unwrap();
    fs::remove_dir_all("demo").unwrap();          // borra la carpeta con todo su contenido
}
```

Salida:

```text
true
4 bytes, es archivo: true
demo\a.txt
demo\sub
```

## Controlar los errores con sentido

Cada operación de archivo devuelve un `Result`, y el tipo de error (`std::io::Error`) dice **qué pasó**:

```rust
use std::fs;
use std::io::ErrorKind;

fn main() {
    match fs::read_to_string("no_existe.txt") {
        Ok(texto) => println!("{texto}"),
        Err(e) if e.kind() == ErrorKind::NotFound => println!("No existe ese archivo"),
        Err(e) if e.kind() == ErrorKind::PermissionDenied => println!("Sin permiso para leerlo"),
        Err(e) => println!("Otro error: {e}"),
    }
}
```

Salida:

```text
No existe ese archivo
```

`ErrorKind` clasifica el error sin tener que comparar el texto del mensaje (que además cambia de un sistema operativo a otro).

## Errores frecuentes

| Error | Cómo evitarlo |
|---|---|
| Construir rutas concatenando texto (`carpeta + "/" + archivo`) | Usa `PathBuf::push` o `Path::join`: ponen el separador correcto en cada sistema |
| `create_dir` cuando la carpeta ya existe | Da error; usa `create_dir_all`, que no falla si ya está creada |
| Olvidar que `remove_dir_all` borra **todo** el contenido | Compruébalo dos veces antes de llamarlo con una ruta variable |
| Comparar el texto de un error en vez de su `ErrorKind` | El texto cambia según el sistema operativo; `ErrorKind` no |

## Para practicar

Haz los [ejercicios 7.2 de archivos y carpetas](archivos.md).
