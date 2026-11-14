# A4.A Qué es una prueba y cómo se escribe

!!! info "Para quién es esto"
    Es material **avanzado**: da por sabidas las unidades 1, 2 y 4 (funciones, errores y structs). Ya viste `#[test]` por encima en [1.4](../../u01/04-pruebas.md); aquí vas más a fondo.

Una **prueba automática** comprueba, sin que tú lo hagas a mano, que una función da el resultado que esperas. La receta de siempre es **preparar, actuar y comprobar**: prepara los datos de entrada, llama a la función, y compara el resultado con el esperado.

## Las pruebas vienen incluidas en el lenguaje

A diferencia de Dart, Java, Kotlin o Python, Rust no necesita una biblioteca externa de pruebas: `#[test]` y el propio `cargo test` (o `rustc --test`) ya saben ejecutarlas:

| Pieza | Qué es |
|---|---|
| `#[cfg(test)] mod tests { ... }` | Un módulo que solo se compila al probar (no entra en el binario final) |
| `#[test]` | Marca una función como prueba |
| `assert_eq!(obtenido, esperado)` | Falla si los dos valores no son iguales (y muestra ambos) |
| `assert!(condición)` | Falla si la condición es `false` |
| `#[should_panic(expected = "texto")]` | La prueba **pasa** si la función hace *panic* con ese texto (sin él, pasa con cualquier *panic*) |
| `use super::*;` | Trae al módulo de pruebas las funciones del archivo, para no repetir el camino completo |

## Un ejemplo

```rust
fn es_par(n: i32) -> bool {
    n % 2 == 0
}

fn dividir(a: i32, b: i32) -> i32 {
    if b == 0 {
        panic!("no se puede dividir por 0");
    }
    a / b
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn cuatro_es_par() {
        assert!(es_par(4));
    }

    #[test]
    fn tres_no_es_par() {
        assert!(!es_par(3));
    }

    #[test]
    fn diez_entre_dos_es_cinco() {
        assert_eq!(dividir(10, 2), 5);
    }

    #[test]
    #[should_panic(expected = "no se puede dividir por 0")]
    fn dividir_por_cero_falla() {
        dividir(1, 0);
    }
}
```

Salida:

```text

running 4 tests
test tests::cuatro_es_par ... ok
test tests::diez_entre_dos_es_cinco ... ok
test tests::tres_no_es_par ... ok
test tests::dividir_por_cero_falla - should panic ... ok

test result: ok. 4 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out
```

El resumen final (`test result: ok. 4 passed; ...`) es lo que imprime el propio ejecutable de pruebas: no has escrito tú ese recuento. Cambia `assert_eq!(dividir(10, 2), 5)` por un valor equivocado y vuelve a ejecutar: verás el mensaje de fallo con el valor **obtenido** y el **esperado**, y qué línea del archivo falló.

## Errores frecuentes

| Error | Cómo evitarlo |
|---|---|
| Olvidar `#[cfg(test)]` en el módulo | Sin él, el código de prueba se compilaría también en el binario final |
| Usar `assert!(a == b)` en vez de `assert_eq!(a, b)` | `assert_eq!` muestra los dos valores al fallar; `assert!` solo dice que falló |
| Un `#[should_panic]` sin `expected` | Pasa con **cualquier** panic, incluso uno por un motivo distinto al que querías probar; añade `expected` |
| Probar varias cosas distintas en una sola función de prueba | Una prueba, una idea: así, si falla, sabes exactamente qué se rompió |

## Para practicar

Los ejercicios [A4.1 y A4.2](ejercicios.md) usan estas ideas. Para ver la misma idea escrita en otros lenguajes: [Pasar de uno a otro](https://apuntes-dam.github.io/apuntes-lenguajes/pasar/).
