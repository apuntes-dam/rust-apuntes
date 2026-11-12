# 3.0 Cadenas

En Rust hay **dos tipos de texto**, y entender la diferencia evita casi todos los líos:

| Tipo | Qué es | Se puede cambiar | Ejemplo |
|---|---|---|---|
| **`&str`** («trozo de texto») | Una **vista** de texto que ya existe (los literales son `&str`) | No | `let s = "hola";` |
| **`String`** | Un texto **que posee** sus datos y puede crecer | Sí (si es `mut`) | `let mut s = String::from("hola");` |

Los dos son **UTF-8**: cualquier carácter (`ñ`, `é`, emojis) cabe, pero **no todos ocupan lo mismo**.

## Recorrer y buscar

```rust
fn main() {
    let texto = "murciélago";

    // longitud: len() cuenta BYTES; chars().count() cuenta letras
    println!("{} bytes, {} letras", texto.len(), texto.chars().count());

    // recorrer las letras (de principio a fin y al revés)
    for c in texto.chars().take(3) {
        print!("{c} ");
    }
    println!();
    let al_reves: String = texto.chars().rev().collect();
    println!("{al_reves}");

    // buscar: find devuelve un Option con la posición (en bytes)
    println!("{:?} {:?}", texto.find('c'), texto.find('z'));
    println!("{}", texto.contains("ciel"));
    println!("{} {}", texto.starts_with("mur"), texto.ends_with("go"));

    // posición y letra a la vez
    for (i, c) in texto.char_indices().filter(|(_, c)| *c == 'a' || *c == 'o') {
        println!("{c} en el byte {i}");
    }
}
```

Salida:

```text
11 bytes, 10 letras
m u r 
ogaléicrum
Some(3) None
false
true true
a en el byte 8
o en el byte 10
```

## Trozos de una cadena

Se pueden cortar con un **rango de bytes**: `&texto[2..5]`. Cuidado: si el corte cae **en medio de un carácter** de varios bytes, el programa hace `panic`.

```rust
fn main() {
    let texto = "murciélago";
    println!("{}", &texto[0..3]);      // mur
    println!("{}", &texto[3..]);       // ciélago  (la «é» ocupa 2 bytes)
    println!("{:?}", texto.get(0..3)); // get devuelve un Option: no hace panic
    println!("{:?}", texto.get(5..6)); // cae en mitad de la «é»: None

    // por letras, no por bytes:
    let primeras: String = texto.chars().take(4).collect();
    let desde_la_tercera: String = texto.chars().skip(2).collect();
    println!("{primeras} {desde_la_tercera}");
}
```

Salida:

```text
mur
ciélago
Some("mur")
None
murc rciélago
```

## Transformar

Casi todos los métodos **devuelven una cadena nueva** y no modifican la original:

```rust
fn main() {
    let s = "  Hola, Mundo  ";
    println!("[{}]", s.trim());
    println!("{} {}", s.to_uppercase(), s.to_lowercase());
    println!("{}", s.replace("o", "0"));

    // dividir y unir
    let csv = "ana,luis,eva";
    let partes: Vec<&str> = csv.split(',').collect();
    println!("{:?} {}", partes, partes.len());
    println!("{}", partes.join(" | "));

    // palabras separadas por cualquier número de espacios
    let frase = "uno   dos tres";
    println!("{:?}", frase.split_whitespace().collect::<Vec<_>>());

    // primera letra en mayúscula
    let palabra = "rust";
    let mut c = palabra.chars();
    let titulo = c.next().unwrap().to_uppercase().collect::<String>() + c.as_str();
    println!("{titulo}");
}
```

Salida:

```text
[Hola, Mundo]
  HOLA, MUNDO     hola, mundo  
  H0la, Mund0  
["ana", "luis", "eva"] 3
ana | luis | eva
["uno", "dos", "tres"]
Rust
```

## Construir una cadena

```rust
fn main() {
    let mut s = String::new();
    s.push_str("Hola");     // añade un &str
    s.push(',');            // añade un carácter
    s += " mundo";          // += también añade
    println!("{s}");

    let nombre = "Ana";
    let edad = 20;
    let frase = format!("{nombre} tiene {edad} años");   // format! construye un String
    println!("{frase}");

    println!("{}", "ab".repeat(3));
    println!("{:>8}|{:<8}|{:^8}|", "der", "izq", "cen");
    println!("{:08.3} {:+}", 3.14159, 5);
}
```

Salida:

```text
Hola, mundo
Ana tiene 20 años
ababab
     der|izq     |  cen   |
0003.142 +5
```

## Comparar y convertir

`==` compara el **contenido**. Para ignorar las mayúsculas, pasa los dos a minúsculas:

```rust
fn main() {
    let a = String::from("Rust");
    println!("{}", a == "Rust");                          // String con &str: funciona
    println!("{}", "Ana".to_lowercase() == "ANA".to_lowercase());
    println!("{}", "a" < "b");                            // orden alfabético por bytes

    // texto <-> número
    let n: i32 = "42".parse().unwrap();
    println!("{}", n + 1);
    println!("{}", 3.5.to_string() + "!");
    println!("{}", 'a'.is_alphabetic() && '7'.is_numeric());
    println!("{}", 'a' as u32);
    println!("{:?}", char::from_u32(98));
}
```

Salida:

```text
true
true
true
43
3.5!
true
97
Some('b')
```

## Unicode

`len()` es el número de **bytes**, no de letras: `"año".len()` es 4. Para contar lo que ve el usuario usa `chars().count()`. Y ni siquiera un «carácter» es siempre una letra visible (una letra con acento escrita con dos piezas combinadas, un emoji familiar…); para esos casos existe la librería `unicode-segmentation`. Para texto en español, `chars()` basta.

## Errores frecuentes

| Error | Cómo evitarlo |
|---|---|
| Escribir `texto[0]` | **No existe** indexar un `String` por posición: usa `texto.chars().nth(0)` o `&texto[0..1]` si es ASCII |
| Cortar con `&texto[a..b]` en mitad de un carácter | Provoca un `panic`: usa `get(a..b)` (`Option`) o `chars()` |
| Suponer que `len()` cuenta letras | Cuenta **bytes**: usa `chars().count()` |
| Esperar que `to_uppercase()` cambie la variable | Devuelve una cadena nueva: `let s = s.to_uppercase();` |
| Sumar dos `&str` con `+` | Solo se puede `String + &str`: `s1.to_string() + s2` o `format!("{s1}{s2}")` |
| Mover un `String` sin querer | `let b = a;` **mueve** `a` y ya no se puede usar; copia con `a.clone()` |

## Para practicar

Haz los [ejercicios 3.0 de cadenas](cadenas.md).
