# 3.4 JSON

**JSON** es un formato de texto para guardar e intercambiar datos. Lo usan casi todas las aplicaciones y servicios web. A diferencia de Python o C#, **la biblioteca estándar de Rust no trae JSON**: se usa el *crate* **`serde_json`** (con `serde`, que hace la conversión entre estructuras y texto). Se añade con:

```bash
cargo add serde --features derive
cargo add serde_json
```

## Qué contiene un JSON

| Elemento | Ejemplo | En Rust (`serde_json::Value`) |
|---|---|---|
| Objeto | `{"nombre": "Ana"}` | `Value::Object` (un mapa) |
| Lista | `[1, 2, 3]` | `Value::Array` |
| Texto | `"hola"` | `Value::String` |
| Número | `30`, `1.5` | `Value::Number` |
| Booleano | `true` | `Value::Bool` |
| Nulo | `null` | `Value::Null` |

## Leer un JSON sin definir tipos: `Value`

En `Cargo.toml`:

```toml
[dependencies]
serde = { version = "1", features = ["derive"] }
serde_json = "1"
```

```rust
use serde_json::Value;

fn main() {
    let texto = r#"{"usuarios": [{"id": 1, "nombre": "Juan", "edad": 30}, {"id": 2, "nombre": "Ana", "edad": 25}]}"#;
    let datos: Value = serde_json::from_str(texto).unwrap();

    println!("{}", datos["usuarios"][0]["nombre"]);          // "Juan" (con comillas)
    println!("{}", datos["usuarios"][1]["edad"]);
    println!("{:?}", datos["usuarios"][0]["nombre"].as_str());   // Some("Juan")
    println!("{}", datos["usuarios"].as_array().unwrap().len());
    println!("{}", datos["no_existe"]);                      // null: no hace panic

    for u in datos["usuarios"].as_array().unwrap() {
        println!("{}: {}", u["id"], u["nombre"].as_str().unwrap());
    }
}
```

Salida:

```text
"Juan"
25
Some("Juan")
2
null
1: Juan
2: Ana
```

`Value` es cómodo para explorar, pero cada acceso devuelve un valor que puede no ser lo que esperas. Para datos con una forma conocida es mejor definir **tipos**.

## Leer y escribir con estructuras (lo recomendable)

Con `#[derive(Serialize, Deserialize)]`, `serde` convierte entre el JSON y tus `struct` **comprobando los tipos**:

En `Cargo.toml`:

```toml
[dependencies]
serde = { version = "1", features = ["derive"] }
serde_json = "1"
```

```rust
use serde::{Deserialize, Serialize};

#[derive(Serialize, Deserialize, Debug)]
struct Usuario {
    id: u32,
    nombre: String,
    edad: u32,
}

#[derive(Serialize, Deserialize, Debug)]
struct Datos {
    usuarios: Vec<Usuario>,
}

fn main() {
    let texto = r#"{"usuarios": [{"id": 1, "nombre": "Juan", "edad": 30}, {"id": 2, "nombre": "Ana", "edad": 25}]}"#;
    let mut datos: Datos = serde_json::from_str(texto).unwrap();
    println!("{:?}", datos.usuarios[0]);

    // modificar
    datos.usuarios[1].edad += 1;
    datos.usuarios.push(Usuario { id: 3, nombre: String::from("Eva"), edad: 41 });
    datos.usuarios.retain(|u| u.id != 1);        // borrar al usuario 1

    // escribir: compacto y bonito
    println!("{}", serde_json::to_string(&datos).unwrap());
    println!("{}", serde_json::to_string_pretty(&datos).unwrap());
}
```

Salida:

```text
Usuario { id: 1, nombre: "Juan", edad: 30 }
{"usuarios":[{"id":2,"nombre":"Ana","edad":26},{"id":3,"nombre":"Eva","edad":41}]}
{
  "usuarios": [
    {
      "id": 2,
      "nombre": "Ana",
      "edad": 26
    },
    {
      "id": 3,
      "nombre": "Eva",
      "edad": 41
    }
  ]
}
```

## Trabajar con archivos y controlar los errores

Leer y escribir un archivo puede fallar, y un JSON inválido también: todo son `Result`. Con `?` y un `Box<dyn Error>` se tratan todos juntos:

En `Cargo.toml`:

```toml
[dependencies]
serde = { version = "1", features = ["derive"] }
serde_json = "1"
```

```rust
use serde::{Deserialize, Serialize};
use std::error::Error;
use std::fs;

#[derive(Serialize, Deserialize, Debug)]
struct Usuario {
    id: u32,
    nombre: String,
    edad: u32,
}

#[derive(Serialize, Deserialize, Debug)]
struct Datos {
    usuarios: Vec<Usuario>,
}

fn cargar(ruta: &str) -> Result<Datos, Box<dyn Error>> {
    let texto = fs::read_to_string(ruta)?;          // puede fallar: el archivo no existe
    let datos = serde_json::from_str(&texto)?;      // puede fallar: JSON inválido
    Ok(datos)
}

fn main() {
    match cargar("datos_usuarios_orig.json") {
        Ok(mut datos) => {
            datos.usuarios.push(Usuario { id: 3, nombre: String::from("Eva"), edad: 41 });
            let salida = serde_json::to_string_pretty(&datos).unwrap();
            fs::write("datos_usuarios_nuevo.json", &salida).unwrap();
            println!("Guardados {} usuarios", datos.usuarios.len());
        }
        Err(e) => println!("Error: {e}"),
    }

    match cargar("no_existe.json") {
        Ok(_) => println!("¿?"),
        Err(e) => println!("Error: {e}"),
    }

    fs::write("roto.json", "{ \"usuarios\": [ ").unwrap();
    match cargar("roto.json") {
        Ok(_) => println!("¿?"),
        Err(e) => println!("Error: {e}"),
    }
}
```

Salida:

```text
Guardados 3 usuarios
Error: El sistema no puede encontrar el archivo especificado. (os error 2)
Error: EOF while parsing a list at line 1 column 16
```

Los mensajes de error (`The system cannot find the file specified`, `EOF while parsing a list`) **dependen del sistema y del crate y están en inglés**.

## Errores frecuentes

| Error | Cómo evitarlo |
|---|---|
| Un campo del JSON con otro tipo del esperado (`"edad": "30"`) | `serde` da un error al leer: controla el `Result` |
| Un campo que falta | Error al leer; si es opcional, decláralo `Option<T>` o con `#[serde(default)]` |
| `unwrap()` al leer datos de fuera | Un archivo malo hace `panic`: usa `match` o `?` |
| Nombres de campos distintos en el JSON (`userName`) | `#[serde(rename = "userName")]` o `#[serde(rename_all = "camelCase")]` |
| Olvidar `features = ["derive"]` | No compila `#[derive(Serialize, Deserialize)]` |

## Para practicar

Haz el [ejercicio 3.4 de JSON](json.md).
