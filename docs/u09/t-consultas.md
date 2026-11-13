# 9.B Consultas y modificaciones

## `SELECT`: consultar filas

En `Cargo.toml`:

```toml
[dependencies]
rusqlite = { version = "0.37", features = ["bundled"] }
```

```rust
use rusqlite::{params, Connection};

fn main() {
    let conexion = Connection::open_in_memory().unwrap();
    conexion.execute("CREATE TABLE producto (id INTEGER PRIMARY KEY, nombre TEXT NOT NULL, precio REAL NOT NULL)", []).unwrap();
    conexion.execute("INSERT INTO producto (nombre, precio) VALUES (?1, ?2)", params!["Teclado", 25.0]).unwrap();
    conexion.execute("INSERT INTO producto (nombre, precio) VALUES (?1, ?2)", params!["Monitor", 150.0]).unwrap();

    let mut consulta = conexion.prepare("SELECT id, nombre, precio FROM producto WHERE precio > ?1 ORDER BY precio").unwrap();
    let filas = consulta.query_map([50.0], |fila| {
        Ok((fila.get::<_, i64>(0)?, fila.get::<_, String>(1)?, fila.get::<_, f64>(2)?))
    }).unwrap();

    for fila in filas {
        let (id, nombre, precio) = fila.unwrap();
        println!("{id} {nombre} {precio:.2}");
    }
}
```

Salida:

```text
2 Monitor 150.00
```

`prepare` compila la consulta una vez; `query_map` la ejecuta y convierte cada fila con la función que le pasas. Cada `get::<_, Tipo>(columna)` puede fallar si el tipo no coincide, por eso devuelve un `Result` (con `?` dentro del cierre).

## `JOIN`: combinar tablas

En `Cargo.toml`:

```toml
[dependencies]
rusqlite = { version = "0.37", features = ["bundled"] }
```

```rust
use rusqlite::{params, Connection};

fn main() {
    let conexion = Connection::open_in_memory().unwrap();
    conexion.execute("CREATE TABLE usuario (id INTEGER PRIMARY KEY, nombre TEXT NOT NULL)", []).unwrap();
    conexion.execute("CREATE TABLE pedido (id INTEGER PRIMARY KEY, id_usuario INTEGER NOT NULL, total REAL NOT NULL)", []).unwrap();
    conexion.execute("INSERT INTO usuario (nombre) VALUES (?1)", params!["Ana"]).unwrap();
    conexion.execute("INSERT INTO pedido (id_usuario, total) VALUES (?1, ?2)", params![1, 42.50]).unwrap();
    conexion.execute("INSERT INTO pedido (id_usuario, total) VALUES (?1, ?2)", params![1, 10.0]).unwrap();

    let mut consulta = conexion.prepare(
        "SELECT usuario.nombre, pedido.total FROM pedido
         JOIN usuario ON usuario.id = pedido.id_usuario
         WHERE usuario.nombre = ?1"
    ).unwrap();
    let filas = consulta.query_map(["Ana"], |f| Ok((f.get::<_, String>(0)?, f.get::<_, f64>(1)?))).unwrap();
    for fila in filas {
        let (nombre, total) = fila.unwrap();
        println!("{nombre}: {total:.2}");
    }
}
```

Salida:

```text
Ana: 42.50
Ana: 10.00
```

## `UPDATE` y `DELETE`

En `Cargo.toml`:

```toml
[dependencies]
rusqlite = { version = "0.37", features = ["bundled"] }
```

```rust
use rusqlite::{params, Connection};

fn main() {
    let conexion = Connection::open_in_memory().unwrap();
    conexion.execute("CREATE TABLE producto (id INTEGER PRIMARY KEY, nombre TEXT NOT NULL, precio REAL NOT NULL)", []).unwrap();
    conexion.execute("INSERT INTO producto (nombre, precio) VALUES (?1, ?2)", params!["Monitor", 150.0]).unwrap();

    let filas_afectadas = conexion.execute("UPDATE producto SET precio = ?1 WHERE nombre = ?2", params![120.0, "Monitor"]).unwrap();
    println!("filas modificadas: {filas_afectadas}");

    let borradas = conexion.execute("DELETE FROM producto WHERE nombre = ?1", params!["No existe"]).unwrap();
    println!("filas borradas: {borradas}");                 // 0: no falla, solo no borra nada
}
```

Salida:

```text
filas modificadas: 1
filas borradas: 0
```

`execute` devuelve **cuántas filas** se vieron afectadas, no un error si no coincide ninguna: borrar algo que no existe no es un fallo.

## Errores frecuentes

| Error | Cómo evitarlo |
|---|---|
| `query_map` sin `prepare` antes | La consulta se prepara aparte; `query_map` la ejecuta con los parámetros |
| Pedir una columna con el tipo equivocado (`get::<_, i64>` en un `TEXT`) | El `Result` del cierre devuelve `Err`: compruébalo o deja que `?` lo propague |
| Esperar que `UPDATE`/`DELETE` fallen si no afectan a ninguna fila | No fallan: comprueban el número devuelto si necesitas saberlo |
| Un `JOIN` sin condición `ON` | Multiplica las filas (producto cartesiano): siempre con su `ON` |

## Para practicar

Haz los [ejercicios 9.2 de consultas y modificaciones](consultas.md).
