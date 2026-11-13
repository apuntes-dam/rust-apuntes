# 6.A Diseñar con traits y módulos

Sin herencia, «diseñar una jerarquía» en Rust es decidir **qué va en un `struct`**, **qué va en un `trait`** y **qué va en un módulo**. Un ejemplo completo: el sistema de habitaciones de un hotel.

## Un ejemplo completo: un hotel

```rust
trait Habitacion {
    fn precio_noche(&self) -> f64;
    fn capacidad(&self) -> u32;

    fn precio_total(&self, noches: u32) -> f64 {        // comportamiento compartido, por defecto
        self.precio_noche() * noches as f64
    }
}

struct Individual;
struct Doble { con_vistas: bool }
struct Suite { con_jacuzzi: bool }

impl Habitacion for Individual {
    fn precio_noche(&self) -> f64 { 40.0 }
    fn capacidad(&self) -> u32 { 1 }
}

impl Habitacion for Doble {
    fn precio_noche(&self) -> f64 { if self.con_vistas { 75.0 } else { 60.0 } }
    fn capacidad(&self) -> u32 { 2 }
}

impl Habitacion for Suite {
    fn precio_noche(&self) -> f64 { if self.con_jacuzzi { 150.0 } else { 120.0 } }
    fn capacidad(&self) -> u32 { 4 }
}

struct Reserva {
    habitacion: Box<dyn Habitacion>,
    noches: u32,
}

impl Reserva {
    fn total(&self) -> f64 {
        self.habitacion.precio_total(self.noches)
    }
}

fn main() {
    let reservas = vec![
        Reserva { habitacion: Box::new(Individual), noches: 2 },
        Reserva { habitacion: Box::new(Doble { con_vistas: true }), noches: 3 },
        Reserva { habitacion: Box::new(Suite { con_jacuzzi: true }), noches: 1 },
    ];
    for r in &reservas {
        println!("{} noches, capacidad {}: {:.2} €", r.noches, r.habitacion.capacidad(), r.total());
    }
    let ingresos: f64 = reservas.iter().map(|r| r.total()).sum();
    println!("Total: {ingresos:.2} €");
}
```

Salida:

```text
2 noches, capacidad 1: 80.00 €
3 noches, capacidad 2: 225.00 €
1 noches, capacidad 4: 150.00 €
Total: 455.00 €
```

`Reserva` **no necesita saber** qué tipo concreto de habitación tiene: le basta con que implemente `Habitacion`. Si mañana aparece `HabitacionFamiliar`, `Reserva` no cambia ni una línea.

## Herramientas para organizar el diseño

| Pregunta | Herramienta de Rust |
|---|---|
| ¿Varios tipos deben **prometer lo mismo**? | Un `trait` |
| ¿El comportamiento es **igual casi siempre**, con alguna excepción? | Un método del trait con cuerpo por defecto, sobrescrito donde haga falta ([5.C](../u05/t-default.md)) |
| ¿El conjunto de tipos es **cerrado y lo controlo yo**? | Un `enum` ([5.D](../u05/t-cerradas.md)) |
| ¿El código crece y quiero **agrupar lo relacionado**? | Un **módulo** (`mod reservas { ... }`), con lo privado oculto por defecto |
| ¿Dos tipos comparten datos, no comportamiento? | **Composición**: uno contiene al otro |

## Módulos: la forma de Rust de organizar en «paquetes»

```rust
mod hotel {
    pub struct Habitacion {
        pub precio_noche: f64,
    }

    impl Habitacion {
        pub fn new(precio_noche: f64) -> Habitacion {
            Habitacion { precio_noche }
        }
    }

    pub fn precio_total(h: &Habitacion, noches: u32) -> f64 {
        h.precio_noche * noches as f64
    }
}

fn main() {
    let h = hotel::Habitacion::new(60.0);
    println!("{}", hotel::precio_total(&h, 3));
}
```

Salida:

```text
180
```

Dentro de un módulo, todo es **privado por defecto**: `pub` decide, pieza a pieza, qué se ve desde fuera. En un proyecto real cada módulo suele ir en su propio archivo (`hotel.rs` o `hotel/mod.rs`).

## Preguntas antes de diseñar

1. ¿Esto es un **dato** (`struct`), un **comportamiento compartido** (`trait`) o un **conjunto cerrado de casos** (`enum`)?
2. ¿Necesito mezclar tipos distintos en la misma colección (trait object) o me basta con un tipo conocido (genérico)?
3. ¿Qué puede fallar, y quién debe decidir qué hacer si falla (`Result` que sube con `?`, o `panic!` porque es un error del programador)?
4. ¿Qué debe ser `pub` y qué puede quedarse privado?

## Errores frecuentes

| Error | Cómo evitarlo |
|---|---|
| Buscar una forma de «heredar» entre structs | No existe: usa composición o un trait compartido |
| Un trait enorme con 15 métodos que nadie necesita todos juntos | Parte en varios traits pequeños (ver [6.C](t-solid-2.md), segregación) |
| Hacer `pub` todo un módulo «para no pelearme con el compilador» | Expón solo la API que de verdad hace falta: así puedes cambiar el resto sin romper nada |

## Para practicar

Haz los [ejercicios 6.1 a 6.5](jerarquia.md).
