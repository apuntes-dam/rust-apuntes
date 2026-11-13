# 5.B Polimorfismo con trait objects

**Polimorfismo** es poder tratar valores de tipos distintos **de la misma forma**, porque comparten un comportamiento. En Rust hay dos caminos:

| Forma | Cuándo | Coste |
|---|---|---|
| **Generics** (`fn f<T: Trait>(x: T)`) | El compilador sabe el tipo concreto en cada llamada | Ninguno en tiempo de ejecución: se genera una versión por tipo |
| **Trait objects** (`Box<dyn Trait>`) | Necesitas una **lista con tipos distintos a la vez**, todos tratados por el mismo trait | Una pequeña indirección (una tabla de métodos) |

Para una `Vec` que mezcla `Circulo`, `Rectangulo` y `Triangulo`, hace falta un **trait object**: `Box<dyn Figura>`.

```rust
trait Figura {
    fn area(&self) -> f64;
    fn perimetro(&self) -> f64;
    fn nombre(&self) -> &str;
}

struct Circulo { radio: f64 }
struct Rectangulo { base: f64, altura: f64 }
struct Triangulo { a: f64, b: f64, c: f64 }

impl Figura for Circulo {
    fn area(&self) -> f64 { std::f64::consts::PI * self.radio * self.radio }
    fn perimetro(&self) -> f64 { 2.0 * std::f64::consts::PI * self.radio }
    fn nombre(&self) -> &str { "Circulo" }
}
impl Figura for Rectangulo {
    fn area(&self) -> f64 { self.base * self.altura }
    fn perimetro(&self) -> f64 { 2.0 * (self.base + self.altura) }
    fn nombre(&self) -> &str { "Rectangulo" }
}
impl Figura for Triangulo {
    fn area(&self) -> f64 {
        let s = self.perimetro() / 2.0;                     // fórmula de Herón
        (s * (s - self.a) * (s - self.b) * (s - self.c)).sqrt()
    }
    fn perimetro(&self) -> f64 { self.a + self.b + self.c }
    fn nombre(&self) -> &str { "Triangulo" }
}

fn main() {
    let figuras: Vec<Box<dyn Figura>> = vec![
        Box::new(Circulo { radio: 2.0 }),
        Box::new(Rectangulo { base: 3.0, altura: 4.0 }),
        Box::new(Triangulo { a: 3.0, b: 4.0, c: 5.0 }),
    ];
    for f in &figuras {
        println!("{}: área {:.2}, perímetro {:.2}", f.nombre(), f.area(), f.perimetro());
    }
    let total: f64 = figuras.iter().map(|f| f.area()).sum();
    println!("área total: {total:.2}");
}
```

Salida:

```text
Circulo: área 12.57, perímetro 12.57
Rectangulo: área 12.00, perímetro 14.00
Triangulo: área 6.00, perímetro 12.00
área total: 30.57
```

`Box<dyn Figura>` significa «un valor en el montículo (*heap*), de **algún** tipo que implementa `Figura`, no sé cuál exactamente». El `Vec` solo necesita que todos compartan el trait.

## Cuándo usar genéricos en vez de trait objects

Si **no** necesitas mezclar tipos distintos en la misma colección, un genérico es más simple y más rápido:

```rust
trait Figura {
    fn area(&self) -> f64;
}

struct Cuadrado { lado: f64 }
impl Figura for Cuadrado {
    fn area(&self) -> f64 { self.lado * self.lado }
}

fn describir<F: Figura>(f: &F) -> String {     // genérico: un único tipo concreto por llamada
    format!("área: {:.2}", f.area())
}

fn main() {
    let c = Cuadrado { lado: 4.0 };
    println!("{}", describir(&c));
}
```

Salida:

```text
área: 16.00
```

## Errores frecuentes

| Error | Cómo evitarlo |
|---|---|
| `Vec<dyn Figura>` sin `Box` | `dyn Figura` no tiene un tamaño fijo: necesita ir detrás de un puntero, casi siempre `Box<dyn Figura>` |
| Usar `Box<dyn Trait>` para todo, incluso con un único tipo conocido | Si no mezclas tipos, un genérico (`<T: Trait>`) es más simple |
| Llamar a un método que no está en el trait, sobre un `dyn Trait` | Solo ves los métodos del trait: si necesitas algo específico del tipo concreto, no uses un trait object para eso |

## Para practicar

Sigue con [U5.1](herencia.md), ejercicios 5.4 y 5.5 (interfaces y polimorfismo).
