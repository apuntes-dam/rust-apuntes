# 2.4 Depurar: encontrar los errores

**Depurar** es buscar y corregir los errores (*bugs*) de un programa. Los hay de tres clases:

| Tipo | Cuándo aparece | Ejemplo |
|---|---|---|
| **Compilación** | Al compilar | Un tipo que no encaja, una variable sin `mut`, un préstamo mal hecho |
| **Ejecución** | Con el programa en marcha | Un `panic!`: índice fuera de rango, `unwrap` sobre un error |
| **Lógicos** | Nunca avisan: el resultado es **incorrecto** | Un bucle que se queda corto |

En Rust el **compilador** atrapa muchísimos errores antes de ejecutar nada, y sus mensajes son muy descriptivos: **léelos enteros**, casi siempre dicen qué falla y cómo arreglarlo. Los errores lógicos son los difíciles, y para esos sirven las herramientas de esta página.

## Un error que no avisa

Este programa debería sumar los números del 1 al 5 (15), pero **compila, no falla y da otro resultado**:

```rust
fn main() {
    let mut total = 0;
    for i in 1..5 {          // ¡el rango 1..5 NO incluye el 5!
        total += i;
    }
    println!("{total}");     // esperaba 15
}
```

Salida:

```text
10
```

Se corrige con `1..=5`. Los errores de «uno de más o de menos» (*off by one*) son los más típicos de los bucles.

## Mensajes por pantalla: `println!` y `dbg!`

La técnica más simple: imprimir los valores en los puntos clave. Rust añade la macro **`dbg!`**, que muestra el **archivo, la línea, la expresión y su valor** (por la salida de **errores**, `stderr`) y devuelve el valor para seguir usándolo:

```rust
fn main() {
    let mut total = 0;
    for i in 1..=5 {
        total += dbg!(i * 2);
    }
    println!("{total}");
}
```

Salida:

```text
30
```

En el terminal, `dbg!` escribe cosas como `[src/main.rs:4:17] i * 2 = 2` (en la salida de errores, por eso no aparece en el bloque de arriba, que muestra solo la salida normal).

## El depurador

Un **depurador** ejecuta el programa **paso a paso** y deja mirar las variables en cada momento. En Rust se usa desde el editor:

* **VS Code**: la extensión *rust-analyzer* añade el botón **Debug** sobre `main` y sobre cada `#[test]`; hace falta la extensión *CodeLLDB* (en Windows, la de C++ de Microsoft).
* **RustRover** (JetBrains): depurador integrado.

Los pasos son siempre los mismos:

1. **Punto de ruptura** (*breakpoint*): pulsa en el margen, junto al número de línea; el programa se detiene ahí.
2. **Ejecuta en modo depuración** (no en modo normal).
3. **Avanza**: *Step over* (la línea entera), *Step into* (entra en la función que llama), *Step out* (sale de ella) y *Continue* (hasta el siguiente punto).
4. **Observa** el panel de variables: cómo cambian `i`, `total`, la lista...
5. En los depuradores se puede **evaluar expresiones** y poner **condiciones** a un punto de ruptura.

## Dónde ocurrió el `panic`

Cuando un programa se para con un `panic`, indica la **línea** y puedes pedir la **pila de llamadas** (qué función llamó a cuál) con una variable de entorno:

```bash
RUST_BACKTRACE=1 cargo run          # en PowerShell: $env:RUST_BACKTRACE=1; cargo run
```

## Las pruebas también depuran

Una prueba `#[test]` que **reproduce** el fallo es la mejor forma de arreglarlo y de que no vuelva (ver [1.4](../u01/04-pruebas.md)). `assert!` y `assert_eq!` dentro del propio programa comprueban lo que **debe** ser cierto y paran si no lo es:

```rust
fn media(valores: &[f64]) -> f64 {
    assert!(!valores.is_empty(), "no se puede calcular la media de una lista vacía");
    valores.iter().sum::<f64>() / valores.len() as f64
}

fn main() {
    println!("{}", media(&[4.0, 8.0, 6.0]));
}
```

Salida:

```text
6
```

## Depurador o mensajes en pantalla

| Mensajes (`println!`, `dbg!`) | Depurador |
|---|---|
| Rápidos de poner, funcionan en cualquier sitio | Control total: pausa, mira y cambia cosas |
| Hay que recompilar y limpiar después | No hay que tocar el código |
| Bien para ver un valor concreto | Mejor para entender **qué camino** sigue el programa |

## Errores típicos en los bucles

| Síntoma | Causa habitual |
|---|---|
| Falta el último elemento, o sobra uno | El límite está mal en un punto (`..` frente a `..=`) |
| El programa **no termina** | La variable de control nunca cambia, o el `loop` no tiene `break` alcanzable |
| El bucle no se ejecuta nunca | La condición es falsa desde el principio, o el rango está vacío (`5..1`) |
| El resultado sale solo con la primera vuelta | Falta actualizar un acumulador o está dentro del bloque equivocado |
| «borrow checker» se queja al tocar la lista mientras se recorre | No se puede modificar una colección mientras se recorre: guarda los cambios y aplícalos después |

## Para practicar

El ejercicio [2.4 de depuración](depurar.md) te pide implementar el algoritmo de la burbuja y **depurarlo** paso a paso.
