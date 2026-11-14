# A4.B Errores y casos límite

## Probar que algo falla

Ya lo has visto con `#[should_panic]`: además del caso que funciona, conviene probar los **casos límite** (el primero y el último de cada rango, el valor justo antes de cambiar de categoría) y los que deben **fallar**.

## Una tabla de casos, en una sola prueba

Cuando varios casos comprueban **la misma idea** con entradas distintas, no hace falta una función `#[test]` por caso: un `for` sobre un array de `(entrada, esperado)` dentro de **una** prueba los recorre a todos, y si uno falla, el mensaje de `assert_eq!` dice **con qué entrada**:

```rust
fn nivel_bateria(porcentaje: i32) -> &'static str {
    if porcentaje < 0 || porcentaje > 100 {
        panic!("el porcentaje debe estar entre 0 y 100");
    } else if porcentaje < 20 {
        "crítico"
    } else if porcentaje < 50 {
        "bajo"
    } else {
        "normal"
    }
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn los_limites_de_cada_nivel() {
        let casos = [(0, "crítico"), (19, "crítico"), (20, "bajo"), (49, "bajo"), (50, "normal"), (100, "normal")];
        for (porcentaje, esperado) in casos {
            assert_eq!(nivel_bateria(porcentaje), esperado, "fallo con porcentaje {}", porcentaje);
        }
    }

    #[test]
    #[should_panic]
    fn un_porcentaje_fuera_de_rango_falla() {
        nivel_bateria(101);
    }
}
```

Salida:

```text

running 2 tests
test tests::los_limites_de_cada_nivel ... ok
test tests::un_porcentaje_fuera_de_rango_falla - should panic ... ok

test result: ok. 2 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out
```

El tercer argumento de `assert_eq!` (`"fallo con porcentaje {}", porcentaje`) es un mensaje **adicional** que se muestra solo si la comprobación falla: con una tabla de casos es casi obligatorio, porque si no, un fallo solo diría «no coinciden», sin decir en qué fila.

## Errores frecuentes

| Error | Cómo evitarlo |
|---|---|
| Olvidar los valores **justo en el límite** (el primero y el último de cada categoría) | Incluye siempre esos valores en la tabla de casos |
| Una tabla de casos sin mensaje adicional en `assert_eq!` | Añade qué entrada falló; si no, tocará adivinarlo |
| Una función `#[test]` por cada fila de una tabla pequeña | Agrúpalas en un `for` dentro de una sola prueba, salvo que cada caso merezca su propio nombre descriptivo |
| Probar solo el camino feliz | Añade siempre al menos una prueba de un caso que debe fallar |

## Para practicar

Los ejercicios [A4.3 y A4.5](ejercicios.md) usan validación y una tabla de casos. Para ver la misma idea escrita en otros lenguajes: [Pasar de uno a otro](https://apuntes-dam.github.io/apuntes-lenguajes/pasar/).
