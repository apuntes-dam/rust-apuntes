# 9.A Conectar y crear tablas

## `rusqlite`

Rust no trae soporte de bases de datos en la biblioteca estándar. Para SQLite se usa el crate **`rusqlite`**, que incluye el propio motor de SQLite (con la característica `bundled`), así que no hace falta instalar nada en el sistema:

```bash
cargo add rusqlite --features bundled
```

## Conectar

En `Cargo.toml`:

```toml
[dependencies]
rusqlite = { version = "0.37", features = ["bundled"] }
```

```rust
use rusqlite::Connection;

fn main() {
    let conexion = Connection::open("tienda.db").unwrap();      // crea el archivo si no existe
    println!("conectado");
    conexion.close().unwrap();                                    // cierra antes de borrar (si no, el sistema lo tiene ocupado)

    std::fs::remove_file("tienda.db").unwrap();                  // limpieza, solo para este ejemplo
}
```

Salida:

```text
conectado
```

`Connection::open_in_memory()` abre una base de datos **en memoria**, que desaparece al terminar el programa: muy útil para pruebas.

## Crear una tabla

En `Cargo.toml`:

```toml
[dependencies]
rusqlite = { version = "0.37", features = ["bundled"] }
```

```rust
use rusqlite::Connection;

fn main() {
    let conexion = Connection::open_in_memory().unwrap();

    conexion.execute(
        "CREATE TABLE usuario (
            id INTEGER PRIMARY KEY AUTOINCREMENT,
            nombre TEXT NOT NULL,
            email TEXT NOT NULL UNIQUE
        )",
        [],
    ).unwrap();

    println!("tabla creada");
}
```

Salida:

```text
tabla creada
```

`execute` sirve para cualquier sentencia que **no devuelve filas**: `CREATE TABLE`, `INSERT`, `UPDATE`, `DELETE`.

## Claves foráneas

SQLite no activa las claves foráneas por defecto: hay que pedirlo:

En `Cargo.toml`:

```toml
[dependencies]
rusqlite = { version = "0.37", features = ["bundled"] }
```

```rust
use rusqlite::Connection;

fn main() {
    let conexion = Connection::open_in_memory().unwrap();
    conexion.execute("PRAGMA foreign_keys = ON", []).unwrap();

    conexion.execute("CREATE TABLE usuario (id INTEGER PRIMARY KEY, nombre TEXT NOT NULL)", []).unwrap();
    conexion.execute(
        "CREATE TABLE pedido (
            id INTEGER PRIMARY KEY,
            id_usuario INTEGER NOT NULL,
            total REAL NOT NULL,
            FOREIGN KEY (id_usuario) REFERENCES usuario(id)
        )",
        [],
    ).unwrap();

    conexion.execute("INSERT INTO usuario (nombre) VALUES (?1)", ["Ana"]).unwrap();
    let error = conexion.execute("INSERT INTO pedido (id_usuario, total) VALUES (?1, ?2)", rusqlite::params![99, 10.0]);
    println!("{}", error.is_err());     // no existe el usuario 99: la clave foránea lo impide
}
```

Salida:

```text
true
```

## Insertar con parámetros

Los datos **nunca** se meten en el SQL con `format!`: se pasan como **parámetros** (`?1`, `?2`...), y es la propia librería quien los escapa de forma segura (ver [9.C](t-buenas-practicas.md), inyección SQL).

En `Cargo.toml`:

```toml
[dependencies]
rusqlite = { version = "0.37", features = ["bundled"] }
```

```rust
use rusqlite::{params, Connection};

fn main() {
    let conexion = Connection::open_in_memory().unwrap();
    conexion.execute("CREATE TABLE usuario (id INTEGER PRIMARY KEY, nombre TEXT NOT NULL, edad INTEGER)", []).unwrap();

    conexion.execute("INSERT INTO usuario (nombre, edad) VALUES (?1, ?2)", params!["Ana", 30]).unwrap();
    conexion.execute("INSERT INTO usuario (nombre, edad) VALUES (?1, ?2)", params!["Luis", 25]).unwrap();

    let ultimo_id = conexion.last_insert_rowid();
    println!("último id: {ultimo_id}");
}
```

Salida:

```text
último id: 2
```

## Errores frecuentes

| Error | Cómo evitarlo |
|---|---|
| Construir el `CREATE TABLE` o el `INSERT` con `format!` a partir de datos del usuario | Usa siempre parámetros (`?1`, `params![...]`) |
| Olvidar `PRAGMA foreign_keys = ON` | SQLite las ignora por defecto: las claves foráneas no se comprueban |
| Abrir una conexión nueva por cada operación | Reutiliza la conexión; ver el pool en [9.C](t-buenas-practicas.md) |
| No comprobar el `Result` de `execute` | Un `INSERT` que viola una restricción (`UNIQUE`, `NOT NULL`) devuelve `Err`, no hace `panic!` solo |

## Para practicar

Haz los [ejercicios 9.1 de conexión y tablas](conexion.md).
