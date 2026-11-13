# 4.C Constructores y enumerados

## Más de una forma de crear un valor

Rust no sobrecarga funciones por el número de parámetros, pero el mismo efecto se consigue con **varias funciones asociadas**, cada una con un nombre que dice qué hace, y con **valores por defecto** mediante el trait `Default`:

```rust
#[derive(Debug, Default)]
struct Pizza {
    tamano: String,
    extras: u32,
}

impl Pizza {
    fn new(tamano: &str, extras: u32) -> Pizza {
        Pizza { tamano: tamano.to_string(), extras }
    }

    fn margarita(tamano: &str) -> Pizza {      // constructor «con nombre», como una receta concreta
        Pizza::new(tamano, 0)
    }

    fn familiar() -> Pizza {                   // llama a otro constructor, como `: this(...)` en C#
        Pizza::new("familiar", 0)
    }
}

fn main() {
    let p1 = Pizza::new("mediana", 2);
    let p2 = Pizza::margarita("pequeña");
    let p3 = Pizza::familiar();
    let p4 = Pizza { extras: 1, ..Default::default() };   // valores por defecto + uno cambiado

    println!("{p1:?}");
    println!("{p2:?}");
    println!("{p3:?}");
    println!("{p4:?}");
}
```

Salida:

```text
Pizza { tamano: "mediana", extras: 2 }
Pizza { tamano: "pequeña", extras: 0 }
Pizza { tamano: "familiar", extras: 0 }
Pizza { tamano: "", extras: 1 }
```

`#[derive(Default)]` solo funciona si **todos los campos** tienen un valor por defecto razonable (`String` vacío, `0`, `false`...). Si no, se implementa `Default` a mano. La sintaxis `{ extras: 1, ..Default::default() }` crea un valor cambiando solo lo que te interesa.

## Enumerados

Un `enum` de Rust es mucho más que una lista de nombres: **cada variante puede llevar sus propios datos**. Es una de las herramientas más usadas del lenguaje:

```rust
#[derive(Debug, PartialEq)]
enum Color {
    Rojo,
    Verde,
    Azul,
    Personalizado(u8, u8, u8),     // esta variante lleva tres datos
}

impl Color {
    fn describe(&self) -> String {
        match self {
            Color::Rojo => String::from("rojo puro"),
            Color::Verde => String::from("verde puro"),
            Color::Azul => String::from("azul puro"),
            Color::Personalizado(r, g, b) => format!("rgb({r}, {g}, {b})"),
        }
    }
}

fn main() {
    let colores = [Color::Rojo, Color::Personalizado(10, 200, 30)];
    for c in &colores {
        println!("{c:?} -> {}", c.describe());
    }
    println!("{}", Color::Rojo == Color::Rojo);
}
```

Salida:

```text
Rojo -> rojo puro
Personalizado(10, 200, 30) -> rgb(10, 200, 30)
true
```

El `match` tiene que cubrir **todas** las variantes (o acabar con `_`): si más adelante añades `Color::Amarillo`, el compilador marca cada `match` que haya que actualizar. Es el sustituto, más seguro, de una jerarquía de clases cerrada (lo mismo que `sealed` en C# o un `enum class` con datos en Kotlin).

## Errores frecuentes

| Error | Cómo evitarlo |
|---|---|
| Intentar sobrecargar `new` con el mismo nombre y distintos parámetros | No se puede: usa nombres distintos (`margarita`, `familiar`...) |
| `#[derive(Default)]` en un struct con un campo sin valor por defecto evidente | Implementa `Default` a mano, o dale al campo un `Option<T>` con `None` por defecto |
| Olvidar una variante en un `match` sobre un `enum` | El compilador no compila hasta que la trates; es una ayuda, no un estorbo |
| Tratar un `enum` como si solo pudiera llevar un nombre | Aprovecha que cada variante puede llevar datos distintos: ahorra un `struct` aparte para cada caso |

## Para practicar

Sigue con los ejercicios de robots: [U4.4](robots-1.md) y [U4.5](robots-2.md).
