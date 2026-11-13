# 6.B SOLID: S y O

**SOLID** son cinco ideas para diseñar código que se pueda mantener y ampliar sin romperlo. En Rust se aplican igual, pero con `struct`, `trait` y módulos en vez de clases e interfaces.

## S · Responsabilidad única

Cada tipo (y cada función) debería tener **una sola razón para cambiar**.

```rust
// Antes: un único struct que hace de todo
struct InformeMal {
    datos: Vec<f64>,
}
impl InformeMal {
    fn calcular_media(&self) -> f64 { self.datos.iter().sum::<f64>() / self.datos.len() as f64 }
    fn exportar_html(&self) -> String { format!("<p>Media: {:.2}</p>", self.calcular_media()) }
    fn guardar_en_archivo(&self, texto: &str) { let _ = std::fs::write("informe_mal.html", texto); }
}

// Después: cada responsabilidad en su sitio
struct Estadistica { datos: Vec<f64> }
impl Estadistica {
    fn media(&self) -> f64 { self.datos.iter().sum::<f64>() / self.datos.len() as f64 }
}

struct InformeHtml;
impl InformeHtml {
    fn generar(&self, media: f64) -> String { format!("<p>Media: {:.2}</p>", media) }
}

struct Almacen;
impl Almacen {
    fn guardar(&self, ruta: &str, texto: &str) -> std::io::Result<()> {
        std::fs::write(ruta, texto)
    }
}

fn main() {
    let est = Estadistica { datos: vec![4.0, 8.0, 6.0] };
    let html = InformeHtml.generar(est.media());
    println!("{html}");
    Almacen.guardar("informe.html", &html).unwrap();
    println!("guardado");
}
```

Salida:

```text
<p>Media: 6.00</p>
guardado
```

Separado así, cambiar el **formato** del informe (HTML a texto plano) no toca el cálculo de la media, y cambiar **dónde** se guarda no toca ninguna de las dos cosas.

## O · Abierto/cerrado

Un tipo debería estar **abierto a ampliarse** (añadir casos nuevos) y **cerrado a modificarse** (no tocar lo que ya funciona). Un `match` gigante que crece con cada caso nuevo rompe esta regla; un `trait` la respeta:

```rust
// Viola abierto/cerrado: cada descuento nuevo obliga a tocar esta función
fn calcular_descuento_mal(tipo: &str, precio: f64) -> f64 {
    match tipo {
        "estudiante" => precio * 0.10,
        "jubilado" => precio * 0.15,
        _ => 0.0,
    }
}

// La respeta: añadir un descuento nuevo es un `impl` más, sin tocar nada existente
trait Descuento {
    fn calcular(&self, precio: f64) -> f64;
}

struct DescuentoEstudiante;
impl Descuento for DescuentoEstudiante {
    fn calcular(&self, precio: f64) -> f64 { precio * 0.10 }
}

struct DescuentoJubilado;
impl Descuento for DescuentoJubilado {
    fn calcular(&self, precio: f64) -> f64 { precio * 0.15 }
}

struct DescuentoBlackFriday;      // AÑADIDO sin tocar las estructuras anteriores
impl Descuento for DescuentoBlackFriday {
    fn calcular(&self, precio: f64) -> f64 { precio * 0.30 }
}

fn aplicar(descuento: &dyn Descuento, precio: f64) -> f64 {
    precio - descuento.calcular(precio)
}

fn main() {
    println!("{:.2}", aplicar(&DescuentoEstudiante, 100.0));
    println!("{:.2}", aplicar(&DescuentoJubilado, 100.0));
    println!("{:.2}", aplicar(&DescuentoBlackFriday, 100.0));
}
```

Salida:

```text
90.00
85.00
70.00
```

`aplicar` no sabe (ni le hace falta saber) cuántos tipos de descuento existen: solo conoce el trait `Descuento`.

## Para practicar

Haz los [ejercicios 6.6 y 6.7](solid.md).
