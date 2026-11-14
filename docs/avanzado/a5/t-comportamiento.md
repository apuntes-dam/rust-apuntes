# A5.B Patrones de comportamiento

Los patrones de **comportamiento** se ocupan de **cómo colaboran** los valores, no de cómo se crean.

## Estrategia: una operación intercambiable

Una **estrategia** es "cómo hacer algo", pasado como valor para poder cambiarlo sin tocar quien la usa. En Rust, lo más directo es que el struct guarde una **closure** (o una función, que implementa el mismo trait `Fn`):

```rust
fn alfabetico(palabras: &[&str]) -> Vec<String> {
    let mut v: Vec<String> = palabras.iter().map(|s| s.to_string()).collect();
    v.sort();
    v
}

fn por_longitud(palabras: &[&str]) -> Vec<String> {
    let mut v: Vec<String> = palabras.iter().map(|s| s.to_string()).collect();
    v.sort_by_key(|s| s.len());
    v
}

struct Ordenador {
    estrategia: Box<dyn Fn(&[&str]) -> Vec<String>>,
}

impl Ordenador {
    fn ordenar(&self, lista: &[&str]) -> Vec<String> {
        (self.estrategia)(lista)
    }
}

fn main() {
    let palabras = ["pera", "uva", "manzana", "fresa"];

    let por_alfabeto = Ordenador { estrategia: Box::new(alfabetico) };
    let por_tamano = Ordenador { estrategia: Box::new(por_longitud) };

    println!("alfabetico: {}", por_alfabeto.ordenar(&palabras).join(", "));
    println!("por longitud: {}", por_tamano.ordenar(&palabras).join(", "));
}
```

Salida:

```text
alfabetico: fresa, manzana, pera, uva
por longitud: uva, pera, fresa, manzana
```

`Ordenador` no sabe **cómo** se ordena; solo sabe que su `estrategia` es algo que recibe una lista y devuelve otra. Cambiar el criterio es tan simple como construir otro `Ordenador` con otra función (o closure).

## Observador: avisar a quien esté suscrito

Un **observador** se suscribe para que le avisen cuando algo cambia, sin que el emisor conozca de antemano quién va a escuchar:

```rust
struct Termometro {
    observadores: Vec<Box<dyn Fn(i32)>>,
}

impl Termometro {
    fn nuevo() -> Self {
        Termometro { observadores: Vec::new() }
    }

    fn suscribir(&mut self, f: Box<dyn Fn(i32)>) {
        self.observadores.push(f);
    }

    fn fijar(&self, grados: i32) {
        for observador in &self.observadores {
            observador(grados);
        }
    }
}

fn main() {
    let mut termometro = Termometro::nuevo();
    termometro.suscribir(Box::new(|g| println!("pantalla: {}", g)));
    termometro.suscribir(Box::new(|g| {
        if g > 30 {
            println!("alarma: {}", g);
        }
    }));

    termometro.fijar(25);
    termometro.fijar(32);
}
```

Salida:

```text
pantalla: 25
pantalla: 32
alarma: 32
```

`Termometro` guarda una lista de closures y, en `fijar`, las llama **todas en orden**. Cada observador decide qué hacer con el valor (la pantalla siempre muestra, la alarma solo si pasa de 30); `Termometro` no sabe nada de pantallas ni de alarmas.

## Errores frecuentes

| Error | Cómo evitarlo |
|---|---|
| Un `if`/`match` largo que decide el comportamiento en vez de una estrategia intercambiable | Si el comportamiento varía de verdad en tiempo de ejecución, pásalo como closure o trait object |
| Suscribir un observador y olvidar que el orden de suscripción es el orden de aviso | Si el orden importa, documéntalo o suscribe en el orden correcto |
| Un observador que modifica el estado del emisor dentro de `fijar` | Puede causar sorpresas (y, con varios hilos, una condición de carrera); mantén los observadores con efectos simples |
| Usar `Box<dyn Fn>` cuando una enumeración cerrada de pocas estrategias sería más simple | Si las opciones son pocas y fijas, un `enum` con `match` puede ser más claro que closures |

## Para practicar

El ejercicio [A5.4](ejercicios.md) usa estrategias, y el [A5.5](ejercicios.md) un observador. Para ver la misma idea escrita en otros lenguajes: [Pasar de uno a otro](https://apuntes-dam.github.io/apuntes-lenguajes/pasar/).
