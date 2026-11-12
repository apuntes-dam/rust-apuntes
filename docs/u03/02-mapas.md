# 3.2 Mapas

Un **mapa** guarda **pares clave → valor** y permite encontrar un valor por su clave. En Rust hay dos:

| Tipo | Orden | Cuándo |
|---|---|---|
| **`HashMap<K, V>`** | **Ninguno** (cambia entre ejecuciones) | Por defecto: es el más rápido |
| **`BTreeMap<K, V>`** | Ordenado por clave | Cuando quieres recorrerlo **en orden** |

## Crear, consultar, modificar y borrar

```rust
use std::collections::HashMap;

fn main() {
    let mut divisas = HashMap::new();
    divisas.insert("Euro", "€");
    divisas.insert("Dollar", "$");
    divisas.insert("Yen", "¥");

    println!("{:?}", divisas.get("Yen"));        // Some("¥")
    println!("{:?}", divisas.get("Peso"));       // None: no existe
    println!("{}", divisas["Euro"]);             // con [] hace panic si no existe
    println!("{}", divisas.contains_key("Yen"));
    println!("{}", divisas.len());

    divisas.insert("Euro", "EUR");               // insertar una clave existente la SOBRESCRIBE
    println!("{}", divisas["Euro"]);

    let quitada = divisas.remove("Dollar");
    println!("{:?} {}", quitada, divisas.len());

    match divisas.get("Peso") {
        Some(s) => println!("Peso -> {s}"),
        None => println!("La divisa Peso no está en el diccionario"),
    }
}
```

Salida:

```text
Some("¥")
None
€
true
3
EUR
Some("$") 2
La divisa Peso no está en el diccionario
```

`get` devuelve un **`Option`**: es la forma segura de consultar. Con `mapa["clave"]` el programa hace `panic` si la clave no existe.

## Recorrer

El orden de un `HashMap` **no está definido**: si necesitas orden, usa `BTreeMap` o ordena las claves.

```rust
use std::collections::{BTreeMap, HashMap};

fn main() {
    let creditos = HashMap::from([("Matemáticas", 6), ("Física", 4), ("Química", 5)]);

    let mut asignaturas: Vec<_> = creditos.iter().collect();
    asignaturas.sort();                               // para tener un orden fijo
    for (nombre, cred) in asignaturas {
        println!("{nombre} tiene {cred} créditos");
    }
    println!("Total: {}", creditos.values().sum::<i32>());

    // BTreeMap: ya sale ordenado por clave
    let ordenado: BTreeMap<_, _> = creditos.iter().collect();
    println!("{:?}", ordenado);
}
```

Salida:

```text
Física tiene 4 créditos
Matemáticas tiene 6 créditos
Química tiene 5 créditos
Total: 15
{"Física": 4, "Matemáticas": 6, "Química": 5}
```

## Contar con un mapa

La forma típica es la **API `entry`**: «dame la entrada de esta clave y, si no existe, créala con este valor»:

```rust
use std::collections::BTreeMap;

fn main() {
    let frase = "la casa de la pradera es la casa";
    let mut frecuencia: BTreeMap<&str, i32> = BTreeMap::new();

    for palabra in frase.split_whitespace() {
        *frecuencia.entry(palabra).or_insert(0) += 1;
    }
    println!("{:?}", frecuencia);

    // contar letras
    let mut letras: BTreeMap<char, usize> = BTreeMap::new();
    for c in "banana".chars() {
        *letras.entry(c).or_default() += 1;
    }
    println!("{:?}", letras);
}
```

Salida:

```text
{"casa": 2, "de": 1, "es": 1, "la": 3, "pradera": 1}
{'a': 3, 'b': 1, 'n': 2}
```

## Claves, valores y mapas de listas

```rust
use std::collections::BTreeMap;

fn main() {
    // cada clave guarda una lista
    let mut por_inicial: BTreeMap<char, Vec<&str>> = BTreeMap::new();
    for nombre in ["Ana", "Luis", "Alba", "Eva", "Laura"] {
        let inicial = nombre.chars().next().unwrap();
        por_inicial.entry(inicial).or_default().push(nombre);
    }
    for (inicial, nombres) in &por_inicial {
        println!("{inicial}: {}", nombres.join(", "));
    }

    // un mapa dentro de otro
    let mut clientes: BTreeMap<&str, BTreeMap<&str, String>> = BTreeMap::new();
    let ficha = clientes.entry("01234567L").or_default();
    ficha.insert("nombre", String::from("Luis González"));
    ficha.insert("preferente", true.to_string());
    println!("{:?}", clientes);
}
```

Salida:

```text
A: Ana, Alba
E: Eva
L: Luis, Laura
{"01234567L": {"nombre": "Luis González", "preferente": "true"}}
```

## ¿Mapa, vector o conjunto?

| Necesitas... | Usa |
|---|---|
| Buscar por **clave** | Mapa |
| Mantener el **orden** y repetir valores | Vector |
| Saber solo **si algo está** o quitar repetidos | Conjunto ([3.3](03-conjuntos.md)) |

## Errores frecuentes

| Error | Cómo evitarlo |
|---|---|
| Esperar un orden en un `HashMap` | No lo hay: usa `BTreeMap` u ordena las claves |
| `mapa["x"]` con una clave que no existe | Hace `panic`: usa `get`, que devuelve `Option` |
| `insert` de una clave que ya existe | Sobrescribe el valor **sin avisar** (devuelve el anterior): comprueba con `contains_key` o `entry` |
| Mezclar `&str` y `String` como clave | `HashMap<String, _>` se consulta con `get("texto")`, pero un `HashMap<&str, _>` no admite un `String` directamente: usa `get(texto.as_str())` |
| Un `f64` como clave | No compila: los decimales no se pueden usar como clave |

## Para practicar

Haz los [ejercicios 3.2 de mapas](mapas.md).
