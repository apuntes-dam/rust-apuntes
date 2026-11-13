# 9.C Buenas prácticas

## Transacciones: todo o nada

Cuando varias operaciones deben completarse **juntas** (insertar un pedido y descontar el stock), una **transacción** asegura que, si algo falla a mitad, **nada** de lo anterior queda guardado:

En `Cargo.toml`:

```toml
[dependencies]
rusqlite = { version = "0.37", features = ["bundled"] }
```

```rust
use rusqlite::{params, Connection};

fn crear_pedido(conexion: &mut Connection, id_usuario: i64, total: f64, id_producto: i64, cantidad: i64) -> rusqlite::Result<()> {
    let tx = conexion.transaction()?;                 // empieza la transacción

    tx.execute("INSERT INTO pedido (id_usuario, total) VALUES (?1, ?2)", params![id_usuario, total])?;

    let stock: i64 = tx.query_row("SELECT stock FROM producto WHERE id = ?1", [id_producto], |f| f.get(0))?;
    if stock < cantidad {
        return Err(rusqlite::Error::QueryReturnedNoRows);   // al salir con Err, la transacción se deshace sola
    }
    tx.execute("UPDATE producto SET stock = stock - ?1 WHERE id = ?2", params![cantidad, id_producto])?;

    tx.commit()                                        // confirma: ahora sí se guarda todo
}

fn main() {
    let mut conexion = Connection::open_in_memory().unwrap();
    conexion.execute("CREATE TABLE pedido (id INTEGER PRIMARY KEY, id_usuario INTEGER, total REAL)", []).unwrap();
    conexion.execute("CREATE TABLE producto (id INTEGER PRIMARY KEY, stock INTEGER)", []).unwrap();
    conexion.execute("INSERT INTO producto (id, stock) VALUES (1, 5)", []).unwrap();

    println!("{:?}", crear_pedido(&mut conexion, 1, 20.0, 1, 2).is_ok());
    println!("{:?}", crear_pedido(&mut conexion, 1, 20.0, 1, 999).is_ok());   // no hay stock: se deshace entero

    let pedidos: i64 = conexion.query_row("SELECT COUNT(*) FROM pedido", [], |f| f.get(0)).unwrap();
    println!("pedidos guardados: {pedidos}");            // solo el primero: el segundo no dejó ni el INSERT
}
```

Salida:

```text
true
false
pedidos guardados: 1
```

Si `tx` se destruye (sale de ámbito) **sin** haber llamado a `commit()`, Rust la **deshace automáticamente** (*rollback*): no hace falta acordarse de deshacerla a mano en cada camino de error, basta con `?` y dejar que la función termine.

## Inyección SQL: por qué nunca se concatena

En `Cargo.toml`:

```toml
[dependencies]
rusqlite = { version = "0.37", features = ["bundled"] }
```

```rust
use rusqlite::{params, Connection};

fn buscar_mal(conexion: &Connection, nombre: &str) -> Vec<String> {
    // NUNCA hagas esto: el texto del usuario se mete directamente en el SQL
    let sql = format!("SELECT nombre FROM usuario WHERE nombre = '{nombre}'");
    let mut consulta = conexion.prepare(&sql).unwrap();
    consulta.query_map([], |f| f.get(0)).unwrap().map(|r| r.unwrap()).collect()
}

fn buscar_bien(conexion: &Connection, nombre: &str) -> Vec<String> {
    let mut consulta = conexion.prepare("SELECT nombre FROM usuario WHERE nombre = ?1").unwrap();
    consulta.query_map(params![nombre], |f| f.get(0)).unwrap().map(|r| r.unwrap()).collect()
}

fn main() {
    let conexion = Connection::open_in_memory().unwrap();
    conexion.execute("CREATE TABLE usuario (nombre TEXT)", []).unwrap();
    conexion.execute("INSERT INTO usuario (nombre) VALUES ('Ana')", []).unwrap();

    let ataque = "' OR '1'='1";                          // intenta que la condición sea siempre verdadera
    println!("{:?}", buscar_mal(&conexion, ataque));       // ¡devuelve a Ana, sin que el nombre coincida!
    println!("{:?}", buscar_bien(&conexion, ataque));      // vacío: trata el texto como un dato, no como SQL
}
```

Salida:

```text
["Ana"]
[]
```

Con parámetros, el texto del usuario **nunca** se interpreta como código SQL, pase lo que pase dentro de él.

## Un pool: reutilizar conexiones

Abrir una conexión no es gratis. En una aplicación que atiende muchas peticiones a la vez, conviene un **pool**: un grupo de conexiones ya abiertas, que se piden prestadas y se devuelven. El crate `r2d2` (con `r2d2_sqlite`) hace esto sobre `rusqlite`; la idea, con una sola conexión compartida de forma simple:

En `Cargo.toml`:

```toml
[dependencies]
rusqlite = { version = "0.37", features = ["bundled"] }
```

```rust
use rusqlite::{params, Connection};

fn main() {
    let conexion = Connection::open_in_memory().unwrap();     // en una app real, esta vendría de un pool
    conexion.execute("CREATE TABLE visita (id INTEGER PRIMARY KEY)", []).unwrap();

    for _ in 0..3 {
        conexion.execute("INSERT INTO visita DEFAULT VALUES", params![]).unwrap();   // reutiliza la MISMA conexión
    }
    let total: i64 = conexion.query_row("SELECT COUNT(*) FROM visita", [], |f| f.get(0)).unwrap();
    println!("{total}");
}
```

Salida:

```text
3
```

## El patrón DAO: una capa para acceder a los datos

**DAO** (*Data Access Object*) agrupa toda la lógica de una tabla en un sitio, para que el resto del programa no escriba SQL directamente:

En `Cargo.toml`:

```toml
[dependencies]
rusqlite = { version = "0.37", features = ["bundled"] }
```

```rust
use rusqlite::{params, Connection};

struct Usuario {
    id: i64,
    nombre: String,
}

struct UsuarioDao<'a> {
    conexion: &'a Connection,
}

impl<'a> UsuarioDao<'a> {
    fn crear(&self, nombre: &str) -> rusqlite::Result<i64> {
        self.conexion.execute("INSERT INTO usuario (nombre) VALUES (?1)", params![nombre])?;
        Ok(self.conexion.last_insert_rowid())
    }

    fn buscar_por_id(&self, id: i64) -> rusqlite::Result<Usuario> {
        self.conexion.query_row("SELECT id, nombre FROM usuario WHERE id = ?1", [id],
            |f| Ok(Usuario { id: f.get(0)?, nombre: f.get(1)? }))
    }

    fn listar(&self) -> rusqlite::Result<Vec<Usuario>> {
        let mut consulta = self.conexion.prepare("SELECT id, nombre FROM usuario")?;
        let filas = consulta.query_map([], |f| Ok(Usuario { id: f.get(0)?, nombre: f.get(1)? }))?;
        filas.collect()
    }
}

fn main() {
    let conexion = Connection::open_in_memory().unwrap();
    conexion.execute("CREATE TABLE usuario (id INTEGER PRIMARY KEY, nombre TEXT)", []).unwrap();
    let dao = UsuarioDao { conexion: &conexion };

    dao.crear("Ana").unwrap();
    dao.crear("Luis").unwrap();

    let ana = dao.buscar_por_id(1).unwrap();
    println!("{} {}", ana.id, ana.nombre);
    for u in dao.listar().unwrap() {
        println!("{}: {}", u.id, u.nombre);
    }
}
```

Salida:

```text
1 Ana
1: Ana
2: Luis
```

El resto del programa llama a `dao.crear(...)` o `dao.listar()`; si mañana cambias de SQLite a otro motor, solo tocas `UsuarioDao`.

## Errores frecuentes

| Error | Cómo evitarlo |
|---|---|
| Hacer varias operaciones relacionadas sin transacción | Si una falla a mitad, los datos quedan a medias; usa `conexion.transaction()` |
| Concatenar texto del usuario en el SQL | Siempre parámetros, nunca `format!` con datos externos |
| Abrir y cerrar una conexión por cada operación en una app con mucho tráfico | Usa un pool (`r2d2`) en vez de abrir una conexión nueva cada vez |
| Esparcir sentencias SQL por todo el programa | Agrúpalas en una capa DAO, una por tabla |

## Para practicar

Haz los [ejercicios 9.3 del pool, la seguridad, las transacciones y el DAO](buenas-practicas.md).
