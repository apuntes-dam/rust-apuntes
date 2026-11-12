# 1.5 Entorno: instalar, compilar y ejecutar

## Qué hace falta

| Pieza | Para qué |
|---|---|
| **rustup** | Instala y actualiza el compilador y sus herramientas |
| **rustc** | El compilador |
| **cargo** | Crea proyectos, compila, ejecuta, prueba y descarga librerías (viene con rustup) |
| **Un editor** | Visual Studio Code con la extensión *rust-analyzer*, o RustRover de JetBrains |

Todo se instala desde [rustup.rs](https://rustup.rs). En Windows también se puede instalar con `winget install Rustlang.Rustup`. En Windows hace falta además el enlazador de **Visual Studio** (las *Build Tools* de C++); el instalador lo ofrece si no está.

## Comprobar la instalación

```bash
rustc --version
cargo --version
rustup update        # actualiza a la última versión
```

## Crear y ejecutar un proyecto

```bash
cargo new hola
cd hola
cargo run
```

`cargo new` crea una carpeta con dos cosas importantes:

| Archivo | Qué es |
|---|---|
| `Cargo.toml` | La descripción del proyecto: nombre, versión y las librerías que usa |
| `src/main.rs` | Tu código; el programa empieza en `fn main` |

Y después de compilar aparece la carpeta `target/` (el resultado y los archivos temporales), que no se sube a Git (`cargo new` ya crea el `.gitignore`).

## Ejecutar un solo archivo

Para probar un ejercicio suelto no hace falta proyecto: el compilador genera un ejecutable y se lanza:

```bash
rustc hola.rs
./hola          # en Windows: .\hola.exe
```

## Compilar y publicar

```bash
cargo build                # compila (modo depuración, en target/debug)
cargo run                  # compila y ejecuta
cargo build --release      # compila optimizado para distribuir (target/release)
cargo check                # solo comprueba errores, sin generar el ejecutable: es lo más rápido
```

## Librerías de crates.io

```bash
cargo add rand
```

Añade la librería (un *crate*) al `Cargo.toml` y la descarga de [crates.io](https://crates.io) al compilar. Ya la puedes usar con `use rand::...;`.

## Herramientas que vienen incluidas

| Orden | Qué hace |
|---|---|
| `cargo fmt` | Da formato al código siguiendo el estilo oficial |
| `cargo clippy` | Avisa de código mejorable |
| `cargo test` | Ejecuta las pruebas |
| `cargo doc --open` | Genera y abre la documentación del proyecto |

!!! note "Errores frecuentes"
    * `cargo` no encontrado: reinicia el terminal tras instalar; rustup lo añade al `PATH` en `~/.cargo/bin`.
    * `linker 'link.exe' not found` en Windows: faltan las *Build Tools* de Visual Studio con la carga de trabajo de C++.
    * El compilador se queja de una variable sin usar: es un aviso, no un error; ponle `_` delante (`_x`) si es a propósito.
