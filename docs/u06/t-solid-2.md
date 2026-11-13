# 6.C SOLID: L, I y D

## L · Sustitución de Liskov

Si un tipo **implementa un trait**, debe poder usarse **en cualquier sitio que espere ese trait**, sin sorpresas. Como Rust no tiene herencia de implementación, es más difícil violarlo sin querer, pero puede pasar si un método hace algo que el contrato del trait no promete:

```rust
trait Ave {
    fn volar(&self) -> String;
}

struct Aguila;
impl Ave for Aguila {
    fn volar(&self) -> String { String::from("el águila vuela alto") }
}

struct Pinguino;
impl Ave for Pinguino {
    fn volar(&self) -> String {
        panic!("¡los pingüinos no vuelan!");   // viola el contrato: quien espera un Ave no espera un panic
    }
}

fn hacer_volar(a: &dyn Ave) {
    println!("{}", a.volar());
}

fn main() {
    hacer_volar(&Aguila);
    // hacer_volar(&Pinguino);   // haría panic: Pinguino no debería prometer Ave si no puede volar

    println!("Mejor diseño: separa quién vuela de quién no");
}
```

Salida:

```text
el águila vuela alto
Mejor diseño: separa quién vuela de quién no
```

La solución, como en los demás lenguajes, es **no forzar el trait**: si `Pinguino` no puede cumplir de verdad lo que promete `Ave`, no debe implementar `Ave`. Se puede separar en dos traits (`Ave` y `AveVoladora`), de forma que `Pinguino` solo implemente el primero.

## I · Segregación de interfaces

Mejor **varios traits pequeños** que uno grande que obliga a implementar métodos que no tienen sentido para todos los tipos:

```rust
// Un trait gigante obliga a todos a implementar todo, tenga sentido o no
trait TrabajadorMal {
    fn trabajar(&self);
    fn comer(&self);
}
// Un robot no come: tendría que fingir un método vacío o hacer panic

// Mejor: un trait por capacidad
trait Trabaja {
    fn trabajar(&self) -> String;
}
trait Come {
    fn comer(&self) -> String;
}

struct Humano;
impl Trabaja for Humano { fn trabajar(&self) -> String { String::from("Humano trabajando") } }
impl Come for Humano { fn comer(&self) -> String { String::from("Humano comiendo") } }

struct Robot;
impl Trabaja for Robot { fn trabajar(&self) -> String { String::from("Robot trabajando") } }
// Robot no implementa Come: no lo necesita, y no está obligado a fingirlo

fn main() {
    let humano = Humano;
    let robot = Robot;
    println!("{}", humano.trabajar());
    println!("{}", humano.comer());
    println!("{}", robot.trabajar());
}
```

Salida:

```text
Humano trabajando
Humano comiendo
Robot trabajando
```

## D · Inversión de dependencias

Un tipo de alto nivel no debería depender de **un tipo concreto** de bajo nivel, sino de un **trait**: así se puede cambiar la implementación (por ejemplo, para hacer pruebas) sin tocar el tipo que la usa.

```rust
trait Notificador {
    fn enviar(&self, mensaje: &str);
}

struct NotificadorEmail;
impl Notificador for NotificadorEmail {
    fn enviar(&self, mensaje: &str) { println!("Correo: {mensaje}"); }
}

struct NotificadorFalso {       // para pruebas: no envía nada de verdad
    enviados: std::cell::RefCell<Vec<String>>,
}
impl Notificador for NotificadorFalso {
    fn enviar(&self, mensaje: &str) { self.enviados.borrow_mut().push(mensaje.to_string()); }
}

struct ServicioPedidos<'a> {
    notificador: &'a dyn Notificador,   // depende del TRAIT, no de NotificadorEmail
}

impl<'a> ServicioPedidos<'a> {
    fn confirmar(&self, pedido: &str) {
        self.notificador.enviar(&format!("Pedido {pedido} confirmado"));
    }
}

fn main() {
    let email = NotificadorEmail;
    let servicio = ServicioPedidos { notificador: &email };
    servicio.confirmar("A-100");

    let falso = NotificadorFalso { enviados: std::cell::RefCell::new(Vec::new()) };
    let servicio_prueba = ServicioPedidos { notificador: &falso };
    servicio_prueba.confirmar("A-200");
    println!("{:?}", falso.enviados.borrow());    // comprobar sin enviar nada real: ideal para un #[test]
}
```

Salida:

```text

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s
```

`ServicioPedidos` nunca menciona `NotificadorEmail`: solo conoce `Notificador`. Eso permite sustituirlo por `NotificadorFalso` en una prueba, sin mandar correos de verdad.

## Resumen de los cinco principios

| Principio | En Rust |
|---|---|
| **S**: responsabilidad única | Un `struct`, una razón para cambiar |
| **O**: abierto/cerrado | Un `trait` en vez de un `match` que crece sin parar |
| **L**: sustitución de Liskov | No implementes un trait que no puedes cumplir de verdad |
| **I**: segregación de interfaces | Varios `trait` pequeños, mejor que uno enorme |
| **D**: inversión de dependencias | Depende de un `trait`, no de un tipo concreto |

## Para practicar

Haz los [ejercicios 6.8 al 6.10](solid.md).
