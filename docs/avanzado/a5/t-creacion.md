# A5.A Patrones de creación

!!! info "Para quién es esto"
    Es material **avanzado**: da por sabidas las unidades 4 y 5 (structs, traits y trait objects).

Un **patrón de diseño** es una solución conocida a un problema que se repite al diseñar. Los patrones de **creación** se ocupan de **cómo se construyen** los valores.

## Fábrica: construir sin que quien llama elija el tipo exacto

Una función fábrica decide **qué tipo concreto** crear a partir de un dato (aquí, un nombre), y devuelve un **trait object** (`Box<dyn Trait>`) para que quien la llama no necesite conocer los tipos concretos. En Rust, el caso «no sé construir eso» se expresa con `Result`, no con una excepción:

```rust
trait Figura {
    fn area(&self) -> i32;
}

struct Cuadrado {
    lado: i32,
}
impl Figura for Cuadrado {
    fn area(&self) -> i32 {
        self.lado * self.lado
    }
}

struct Rectangulo {
    base: i32,
    altura: i32,
}
impl Figura for Rectangulo {
    fn area(&self) -> i32 {
        self.base * self.altura
    }
}

fn crear_figura(nombre: &str) -> Result<Box<dyn Figura>, String> {
    match nombre {
        "cuadrado" => Ok(Box::new(Cuadrado { lado: 3 })),
        "rectangulo" => Ok(Box::new(Rectangulo { base: 2, altura: 5 })),
        _ => Err(format!("figura desconocida: {}", nombre)),
    }
}

fn main() {
    for nombre in ["cuadrado", "rectangulo", "triangulo"] {
        match crear_figura(nombre) {
            Ok(figura) => println!("{}: {}", nombre, figura.area()),
            Err(e) => println!("{}", e),
        }
    }
}
```

Salida:

```text
cuadrado: 9
rectangulo: 10
figura desconocida: triangulo
```

## Singleton: una sola instancia

Un **singleton** garantiza que solo existe una instancia de algo, y que siempre se obtiene la misma. En Rust, `std::sync::OnceLock` (de la biblioteca estándar, sin *crates* externas) se inicializa la **primera vez** que se pide y después siempre devuelve la misma referencia, de forma segura entre hilos:

```rust
use std::sync::{Mutex, OnceLock};

struct Registro {
    mensajes: Mutex<Vec<String>>,
}

// OnceLock se inicializa UNA sola vez, la primera vez que alguien lo pide
fn registro() -> &'static Registro {
    static INSTANCIA: OnceLock<Registro> = OnceLock::new();
    INSTANCIA.get_or_init(|| Registro { mensajes: Mutex::new(Vec::new()) })
}

fn main() {
    registro().mensajes.lock().unwrap().push("inicio".to_string());

    let otra_vez = registro();
    otra_vez.mensajes.lock().unwrap().push("fin".to_string());

    println!("mensajes: {}", registro().mensajes.lock().unwrap().len());
    println!("misma instancia: {}", if std::ptr::eq(registro(), otra_vez) { "sí" } else { "no" });
}
```

Salida:

```text
mensajes: 2
misma instancia: sí
```

`std::ptr::eq` compara **direcciones**: confirma que `registro()` y `otra_vez` son literalmente el mismo objeto en memoria, no solo dos objetos con el mismo contenido.

## Builder: construir paso a paso

Rust no tiene parámetros con nombre ni valores por defecto en las llamadas normales (a diferencia de Kotlin o Dart), así que para un valor con **muchos campos opcionales**, un **builder** encadenado es la forma idiomática:

```rust
struct Correo {
    para: String,
    asunto: String,
    copia: Vec<String>,
}

struct CorreoBuilder {
    para: String,
    asunto: String,
    copia: Vec<String>,
}

impl CorreoBuilder {
    fn nuevo(para: &str) -> Self {
        CorreoBuilder { para: para.to_string(), asunto: String::new(), copia: Vec::new() }
    }

    // cada método consume "self" y devuelve "Self": permite encadenar
    fn asunto(mut self, texto: &str) -> Self {
        self.asunto = texto.to_string();
        self
    }

    fn con_copia(mut self, direccion: &str) -> Self {
        self.copia.push(direccion.to_string());
        self
    }

    fn construir(self) -> Correo {
        Correo { para: self.para, asunto: self.asunto, copia: self.copia }
    }
}

fn main() {
    let correo1 = CorreoBuilder::nuevo("ana@mail.com").asunto("Hola").construir();
    println!("para: {} | asunto: {}", correo1.para, correo1.asunto);

    let correo2 = CorreoBuilder::nuevo("luis@mail.com")
        .asunto("Reunión")
        .con_copia("ana@mail.com")
        .con_copia("eva@mail.com")
        .construir();
    println!("para: {} | asunto: {} | cc: {}", correo2.para, correo2.asunto, correo2.copia.join(", "));
}
```

Salida:

```text
para: ana@mail.com | asunto: Hola
para: luis@mail.com | asunto: Reunión | cc: ana@mail.com, eva@mail.com
```

Cada método (`asunto`, `con_copia`) **consume** `self` (lo recibe por valor, no por referencia) y **devuelve `Self`**: eso es lo que permite encadenar llamadas. `con_copia` se puede llamar varias veces porque añade a un `Vec`, en vez de sustituir un valor.

## Errores frecuentes

| Error | Cómo evitarlo |
|---|---|
| Usar `panic!` para «no sé construir esto» | Devuelve `Result` o `Option`: es un caso esperable, no un error de programación |
| Un builder con métodos que reciben `&mut self` en vez de `self` | Con `self` por valor encadenas limpio; con `&mut self` necesitarías guardar cada paso en una variable |
| Un `OnceLock`/singleton para algo que en realidad necesitas poder sustituir en las pruebas | Si hace falta aislarlo al probar ([A4.C](../a4/t-aislar.md)), pásalo como parámetro en vez de hacerlo global |
| Crear un builder para un struct con dos o tres campos simples | Para pocos campos, un constructor normal (`Correo::nuevo(para, asunto)`) basta |

## Para practicar

Los ejercicios [A5.1, A5.2 y A5.3](ejercicios.md) usan fábrica, singleton y builder. Para ver la misma idea escrita en otros lenguajes: [Pasar de uno a otro](https://apuntes-dam.github.io/apuntes-lenguajes/pasar/).
