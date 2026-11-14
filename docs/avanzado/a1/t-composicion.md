# A1.C Componer, esperar e inmutabilidad

## Componer funciones

Si una función devuelve algo que otra función necesita, se pueden **encadenar**. Y como las funciones son valores, se puede escribir una función que **construya** la cadena: `componer(f, g)` devuelve una closure nueva que aplica `f` y luego `g`. Con una lista de funciones, `.fold(...)` las compone todas, que es una **tubería** (*pipeline*):

```rust
fn componer(f: impl Fn(i32) -> i32 + 'static, g: impl Fn(i32) -> i32 + 'static) -> impl Fn(i32) -> i32 {
    move |x| g(f(x))
}

fn main() {
    let doble = |n: i32| n * 2;
    let mas_uno = |n: i32| n + 1;
    let cuadrado = |n: i32| n * n;

    let doble_y_mas_uno = componer(doble, mas_uno);
    println!("doble y +1 de 5 = {}", doble_y_mas_uno(5));

    // Una tubería con cualquier número de funciones
    let funciones: Vec<Box<dyn Fn(i32) -> i32>> = vec![Box::new(doble), Box::new(mas_uno), Box::new(cuadrado)];
    let resultado = funciones.iter().fold(5, |acum, f| f(acum));
    println!("doble, +1 y cuadrado de 5 = {}", resultado);

    // El orden importa
    let al_reves: Vec<Box<dyn Fn(i32) -> i32>> = vec![Box::new(cuadrado), Box::new(mas_uno), Box::new(doble)];
    let resultado2 = al_reves.iter().fold(5, |acum, f| f(acum));
    println!("cuadrado, +1 y doble de 5 = {}", resultado2);
}
```

Salida:

```text
doble y +1 de 5 = 11
doble, +1 y cuadrado de 5 = 121
cuadrado, +1 y doble de 5 = 52
```

Fíjate en que el orden cambia el resultado: `((5·2)+1)²` es 121, pero `((5²)+1)·2` es 52. Con una tubería **vacía**, `.fold(x, ...)` simplemente devuelve `x` sin tocarlo: ya hace de función identidad (es lo que pide el ejercicio A1.5).

## Calcular solo lo necesario

Los iteradores de Rust son **perezosos**: `.map(...)` no calcula nada; solo **describe** el cálculo. Se ejecuta cuando alguien **consume** el iterador (con un `for`, `.collect()`, `.sum()`...), y solo entonces:

```rust
fn main() {
    use std::cell::Cell;
    let calculos = Cell::new(0);

    // Definir la cadena no calcula nada
    let consulta = (1..=5).map(|n| {
        calculos.set(calculos.get() + 1);
        n * n
    });
    println!("tras definirla: {} cálculos", calculos.get());

    // Un iterador normal solo se puede recorrer UNA vez (se consume); si necesitas
    // reutilizar el resultado, guárdalo con collect()
    let guardado: Vec<i32> = consulta.collect();
    println!("suma: {}, cálculos: {}", guardado.iter().sum::<i32>(), calculos.get());

    // Una secuencia infinita: solo se calcula lo que se pide
    let pares: Vec<i32> = (1..).filter(|n| n % 7 == 0).filter(|n| *n > 20).take(4).collect();
    println!("primeros múltiplos de 7 mayores que 20: {:?}", pares);
}
```

Salida:

```text
tras definirla: 0 cálculos
suma: 55, cálculos: 5
primeros múltiplos de 7 mayores que 20: [21, 28, 35, 42]
```

* Definir la cadena no hace ningún cálculo (`0 cálculos`).
* A diferencia de otros lenguajes, un iterador de Rust se **consume** al recorrerlo: no puedes recorrerlo dos veces sin volver a crearlo o guardarlo antes con `.collect()`. Es una protección más del compilador contra el error de "repetir sin darte cuenta" un cálculo caro.
* La pereza permite trabajar con **secuencias infinitas**: `(1..)` es un rango sin final, pero `.take(4)` pide únicamente cuatro valores y se detiene ahí. Nunca se calcula el quinto.

!!! warning "Efectos secundarios en una cadena"
    Si dentro de un `.map()` o `.filter()` haces algo más que calcular (escribir en pantalla, cambiar una variable), ese efecto ocurre cuando se consume el iterador, no donde escribiste la cadena. Mantén las closures de una cadena **puras**: que solo calculen un resultado a partir de sus parámetros.

## No modificar los datos

Rust es inmutable **por defecto**: `let` sin `mut` no se puede reasignar, y el compilador lo comprueba al compilar, no al ejecutar. Hay varias piezas relacionadas:

| Herramienta | Qué da |
|---|---|
| `let` (sin `mut`) | Una variable que no se puede reasignar ni modificar |
| `let mut` | Una variable que sí puede cambiar, pero solo a través de su **dueño** (o de una referencia `&mut`) |
| `..original` en un *struct literal* | Copia el resto de los campos; junto a un campo distinto, es la forma de Rust de "crear una copia con un valor cambiado" |
| *shadowing* (`let x = ...` otra vez) | Reutiliza el nombre para un valor (o tipo) nuevo, sin hacer falta `mut`; el valor anterior sigue existiendo hasta que alguien lo use |
| `.clone()` | Una copia independiente, para que los cambios posteriores del original no te afecten |

```rust
#[derive(Debug, Clone)]
struct Persona {
    nombre: String,
    edad: i32,
}

fn main() {
    // let (sin mut) es inmutable por defecto: el compilador lo impone
    let original = Persona { nombre: String::from("Ana"), edad: 20 };
    // ..original copia el resto de campos (como "with" en otros lenguajes)
    let mayor = Persona { edad: original.edad + 1, ..original.clone() };
    println!("{:?} -> {:?}", original, mayor);

    // una variable mut puede cambiar, pero solo su dueño puede cambiarla
    let mut datos = vec![1, 2, 3];
    datos.push(4);
    println!("datos tiene {} elementos", datos.len());

    // shadowing: "reutilizar" un nombre con un valor (y hasta un tipo) distinto, sin mut
    let datos = datos.len();
    println!("ahora «datos» es el número {}", datos);
}
```

Salida:

```text
Persona { nombre: "Ana", edad: 20 } -> Persona { nombre: "Ana", edad: 21 }
datos tiene 4 elementos
ahora «datos» es el número 4
```

## Errores frecuentes

| Error | Cómo evitarlo |
|---|---|
| Intentar reasignar una variable sin `mut` | El compilador lo rechaza: añade `mut` si de verdad necesita cambiar |
| Recorrer dos veces el mismo iterador | Se consume la primera vez; guárdalo con `.collect()` si lo necesitas dos veces |
| Hacer cosas con efectos dentro de un `.map()` | Mantén las closures puras |
| Confundir `mut` con `Cell`/`RefCell` | `mut` es para cuando **tú** tienes el control exclusivo; `Cell`/`RefCell` permiten mutar algo compartido, con comprobaciones en tiempo de ejecución |
| Componer funciones en el orden equivocado | Comprueba con un caso pequeño a mano |

## Para practicar

Los ejercicios [A1.5 y A1.6](ejercicios.md) usan estas ideas. Para ver la misma idea escrita en otros lenguajes: [Pasar de uno a otro](https://apuntes-dam.github.io/apuntes-lenguajes/pasar/).
