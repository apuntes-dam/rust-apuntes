# 5.C Comportamiento por defecto

Un `trait` puede dar una **implementación por defecto** de un método: los tipos que lo implementan la heredan gratis, y solo escriben lo que de verdad cambia. Es el sustituto de las **clases abstractas** de otros lenguajes.

```rust
trait Vehiculo {
    fn velocidad_maxima(&self) -> u32;         // sin cuerpo: cada tipo DEBE darlo

    fn descripcion(&self) -> String {          // con cuerpo: es el valor «por defecto»
        format!("Vehículo con velocidad máxima de {} km/h", self.velocidad_maxima())
    }

    fn necesita_carnet(&self) -> bool {        // también por defecto
        self.velocidad_maxima() > 25
    }
}

struct Bicicleta;
struct Moto { cilindrada: u32 }

impl Vehiculo for Bicicleta {
    fn velocidad_maxima(&self) -> u32 { 25 }
}

impl Vehiculo for Moto {
    fn velocidad_maxima(&self) -> u32 { 180 }

    fn descripcion(&self) -> String {          // sobrescribe el valor por defecto
        format!("Moto de {} cc, hasta {} km/h", self.cilindrada, self.velocidad_maxima())
    }
}

fn main() {
    let bici = Bicicleta;
    let moto = Moto { cilindrada: 600 };

    println!("{} (carnet: {})", bici.descripcion(), bici.necesita_carnet());
    println!("{} (carnet: {})", moto.descripcion(), moto.necesita_carnet());
}
```

Salida:

```text
Vehículo con velocidad máxima de 25 km/h (carnet: false)
Moto de 600 cc, hasta 180 km/h (carnet: true)
```

`Bicicleta` **no** escribe `descripcion` ni `necesita_carnet`: usa los que trae el trait, que ya saben calcularse a partir de `velocidad_maxima()`. `Moto` sí sobrescribe `descripcion`, igual que un `override` en una clase abstracta.

## La diferencia con una clase abstracta

| Clase abstracta (C#, Java, Kotlin) | Trait con método por defecto (Rust) |
|---|---|
| Puede tener **campos** propios | Un trait **no tiene campos**: solo métodos |
| Un tipo hereda de **una sola** clase abstracta | Un tipo puede implementar **varios** traits a la vez |
| «Es un» | «Se comporta como», para cada trait que cumple |

Esto evita el problema clásico de la herencia múltiple (¿de cuál de las dos clases base hereda el campo si las dos tienen uno igual?): en Rust, si necesitas datos compartidos, van en el `struct`, y los traits solo aportan comportamiento.

## Errores frecuentes

| Error | Cómo evitarlo |
|---|---|
| Intentar poner un campo dentro de un `trait` | No se puede: los datos van en el `struct` |
| Pensar que hay que sobrescribir todos los métodos del trait | Solo los que no tengan cuerpo son obligatorios; los demás, solo si quieres cambiarlos |
| Un método por defecto que necesita datos que no tiene | Pide esos datos a través de **otro método** del propio trait (como `velocidad_maxima()` en el ejemplo), no accediendo a un campo que no existe |

## Para practicar

Termina [U5.1](herencia.md) con los ejercicios 5.6 al 5.8.
