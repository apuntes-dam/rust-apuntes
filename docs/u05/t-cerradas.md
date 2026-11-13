# 5.D Jerarquías cerradas con `enum`

Cuando el conjunto de «tipos relacionados» es **cerrado y conocido de antemano** (un semáforo solo puede estar en tres estados; una figura del juego solo puede ser de cuatro formas), un **`enum`** es mejor que un trait: el compilador **te obliga a tratar cada caso**.

```rust
enum Figura {
    Circulo { radio: f64 },
    Rectangulo { base: f64, altura: f64 },
    Triangulo { a: f64, b: f64, c: f64 },
}

impl Figura {
    fn area(&self) -> f64 {
        match self {
            Figura::Circulo { radio } => std::f64::consts::PI * radio * radio,
            Figura::Rectangulo { base, altura } => base * altura,
            Figura::Triangulo { a, b, c } => {
                let s = (a + b + c) / 2.0;
                (s * (s - a) * (s - b) * (s - c)).sqrt()
            }
        }
    }
}

fn main() {
    let figuras = vec![
        Figura::Circulo { radio: 2.0 },
        Figura::Rectangulo { base: 3.0, altura: 4.0 },
        Figura::Triangulo { a: 3.0, b: 4.0, c: 5.0 },
    ];
    for f in &figuras {
        println!("{:.2}", f.area());
    }
}
```

Salida:

```text
12.57
12.00
6.00
```

Comparado con el trait object de [5.B](t-polimorfismo.md): aquí **no hace falta `Box`**, porque el compilador conoce de antemano el tamaño máximo que puede ocupar cualquier variante. A cambio, **añadir una variante nueva** es tocar el `enum` (algo que un trait, con tipos abiertos, no te pide); y si más adelante otra parte del programa quiere añadir su propio tipo de figura sin tocar tu código, un trait es lo adecuado.

## ¿`enum` o `trait`?

| Pregunta | Si la respuesta es sí |
|---|---|
| ¿El conjunto de variantes es **fijo y lo controlas tú**? | `enum` |
| ¿Quieres que **otro código, en otro módulo o crate**, pueda añadir sus propios tipos? | `trait` |
| ¿Necesitas que el compilador te **avise si te dejas un caso**? | `enum` + `match` |
| ¿Los «tipos» comparten comportamiento pero tienen formas de datos muy distintas y van a crecer? | `trait` + `Box<dyn Trait>` |

## Un `enum` para estados (como un `sealed` con un `switch` exhaustivo)

```rust
#[derive(Debug, PartialEq)]
enum EstadoPedido {
    Pendiente,
    Enviado { numero_seguimiento: String },
    Entregado,
    Cancelado { motivo: String },
}

fn resumen(estado: &EstadoPedido) -> String {
    match estado {
        EstadoPedido::Pendiente => String::from("Esperando confirmación"),
        EstadoPedido::Enviado { numero_seguimiento } => format!("En camino (seguimiento: {numero_seguimiento})"),
        EstadoPedido::Entregado => String::from("Entregado"),
        EstadoPedido::Cancelado { motivo } => format!("Cancelado: {motivo}"),
    }
}

fn main() {
    let pedidos = vec![
        EstadoPedido::Pendiente,
        EstadoPedido::Enviado { numero_seguimiento: String::from("ES123456") },
        EstadoPedido::Cancelado { motivo: String::from("sin stock") },
    ];
    for p in &pedidos {
        println!("{}", resumen(p));
    }
}
```

Salida:

```text
Esperando confirmación
En camino (seguimiento: ES123456)
Cancelado: sin stock
```

Cada estado lleva **solo los datos que tienen sentido para él** (un pedido pendiente no necesita número de seguimiento): algo que en una jerarquía de clases obligaría a campos opcionales o a subclases extra.

## Errores frecuentes

| Error | Cómo evitarlo |
|---|---|
| Usar `enum` para algo que puede crecer desde fuera de tu código | Usa un `trait`: añadir una variante a un `enum` rompe todo el código que hace `match` sobre él |
| Usar `trait` + `Box<dyn ...>` para un conjunto de 3 o 4 casos que nunca van a cambiar | Un `enum` + `match` es más simple y más rápido |
| Un `match` con `_` que esconde un caso que deberías tratar | Quita el `_` mientras lo desarrollas: así el compilador te avisa de lo que falta |

## Para practicar

Cierra la unidad con los ejercicios 5.9 y 5.10 de [U5.1](herencia.md).
