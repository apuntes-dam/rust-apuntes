# 4.B Encapsulamiento

**Encapsular** es ocultar cómo está hecho un tipo por dentro y ofrecer solo una forma controlada de usarlo desde fuera. En Rust esto es, de hecho, el **comportamiento por defecto**.

## Visibilidad

A diferencia de Java o C#, en Rust **todo es privado si no dices lo contrario**:

| Quieres que... | Escribe |
|---|---|
| El campo o la función solo se use **dentro de este archivo/módulo** | nada (es lo normal) |
| Se pueda usar **desde fuera** del módulo | `pub` |
| Se pueda usar en **todo el crate**, pero no fuera de él (una librería) | `pub(crate)` |

```rust
struct CuentaBancaria {
    titular: String,
    saldo: f64,     // privado: nadie de fuera puede tocarlo directamente
}

impl CuentaBancaria {
    fn new(titular: &str, saldo_inicial: f64) -> CuentaBancaria {
        CuentaBancaria { titular: titular.to_string(), saldo: saldo_inicial.max(0.0) }
    }

    fn saldo(&self) -> f64 {        // «getter»: solo lectura
        self.saldo
    }

    fn ingresar(&mut self, cantidad: f64) {
        if cantidad > 0.0 {
            self.saldo += cantidad;
        }
    }

    fn retirar(&mut self, cantidad: f64) -> Result<(), String> {
        if cantidad > self.saldo {
            return Err(format!("saldo insuficiente: hay {:.2}, pides {:.2}", self.saldo, cantidad));
        }
        self.saldo -= cantidad;
        Ok(())
    }
}

fn main() {
    let mut cuenta = CuentaBancaria::new("Ana", 100.0);
    cuenta.ingresar(50.0);
    println!("{}", cuenta.saldo());

    match cuenta.retirar(1000.0) {
        Ok(()) => println!("retirado"),
        Err(e) => println!("Error: {e}"),
    }
    // cuenta.saldo = 1_000_000.0;   // no compilaría: el campo es privado fuera de este módulo
}
```

Salida:

```text
150
Error: saldo insuficiente: hay 150.00, pides 1000.00
```

No hace falta inventar la palabra «privado» para los campos: **ya lo son**. Un «getter» en Rust es simplemente un método que devuelve el valor; si el campo se puede modificar sin reglas, a veces ni siquiera hace falta: se marca `pub` directamente.

## Validar al crear: un `new` que puede fallar

En vez de lanzar una excepción (que en Rust no existe), un constructor que puede recibir datos inválidos devuelve un **`Result`**:

```rust
struct Rectangulo {
    base: f64,
    altura: f64,
}

impl Rectangulo {
    fn new(base: f64, altura: f64) -> Result<Rectangulo, String> {
        if base <= 0.0 || altura <= 0.0 {
            return Err(String::from("base y altura deben ser mayores que 0"));
        }
        Ok(Rectangulo { base, altura })
    }

    fn area(&self) -> f64 {
        self.base * self.altura
    }
}

fn main() {
    match Rectangulo::new(3.0, 4.0) {
        Ok(r) => println!("área: {}", r.area()),
        Err(e) => println!("Error: {e}"),
    }
    match Rectangulo::new(-1.0, 2.0) {
        Ok(r) => println!("área: {}", r.area()),
        Err(e) => println!("Error: {e}"),
    }
}
```

Salida:

```text
área: 12
Error: base y altura deben ser mayores que 0
```

Así, **si el `Rectangulo` existe, es válido**: no hace falta comprobar nada más después de crearlo. Esta idea («hazlo imposible de usar mal») es muy de Rust.

## Igualdad: `derive` en vez de escribirla a mano

En lugar de implementar `Equals` como en otros lenguajes, casi siempre basta con pedírselo al compilador:

```rust
#[derive(PartialEq, Debug)]
struct Punto { x: i32, y: i32 }

fn main() {
    let a = Punto { x: 1, y: 2 };
    let b = Punto { x: 1, y: 2 };
    let c = Punto { x: 9, y: 9 };

    println!("{}", a == b);   // true: compara por contenido, gracias a PartialEq
    println!("{}", a == c);
    println!("{a:?} {b:?}");  // Debug: para inspeccionar, no para mostrar al usuario
}
```

Salida:

```text
true
false
Punto { x: 1, y: 2 } Punto { x: 1, y: 2 }
```

`#[derive(PartialEq)]` genera la comparación campo a campo. `#[derive(Debug)]` genera una representación de depuración (`{:?}`); para un texto pensado para el usuario, se escribe `Display` a mano, como en [4.A](t-structs.md).

## Errores frecuentes

| Error | Cómo evitarlo |
|---|---|
| Hacer todos los campos `pub` «por si acaso» | Dejar privado lo que no haga falta exponer: así puedes cambiar la implementación sin romper a quien te usa |
| Un `new` que hace `panic!` con cualquier dato del usuario | Mejor un `Result`, salvo que el error sea realmente un fallo del programador |
| Reimplementar `==` a mano para cada struct | `#[derive(PartialEq)]` lo hace por ti, salvo que necesites una igualdad especial |
| Confundir `Debug` (`{:?}`, para programadores) con `Display` (`{}`, para el usuario) | Usa `derive(Debug)` para depurar y escribe `Display` para lo que vea una persona |

## Para practicar

Sigue con [U4.3 · POO II](poo-2.md).
