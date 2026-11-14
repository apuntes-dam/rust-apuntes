# A4.C Aislar lo que no controlas

## El problema

Una función que lee la hora del sistema, tira un dado o llama a una red **no se puede probar de forma fiable**: cada ejecución daría un resultado distinto, o necesitaría conexión. La solución no es dejar de probarla: es **no llamar directamente** a esa fuente de incertidumbre desde dentro, sino **recibirla desde fuera**.

## Inyectar la dependencia como parámetro

En Rust, lo más simple casi siempre es pasar una **closure** como parámetro, en vez de una interfaz con una sola implementación real y otra falsa:

```rust
fn saludo(hora_actual: impl Fn() -> u32) -> &'static str {
    let h = hora_actual();
    if h < 12 {
        "buenos días"
    } else if h < 20 {
        "buenas tardes"
    } else {
        "buenas noches"
    }
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn a_las_8_es_de_dia() {
        assert_eq!(saludo(|| 8), "buenos días");
    }

    #[test]
    fn a_las_15_es_de_tarde() {
        assert_eq!(saludo(|| 15), "buenas tardes");
    }

    #[test]
    fn a_las_22_es_de_noche() {
        assert_eq!(saludo(|| 22), "buenas noches");
    }
}
```

Salida:

```text

running 3 tests
test tests::a_las_15_es_de_tarde ... ok
test tests::a_las_22_es_de_noche ... ok
test tests::a_las_8_es_de_dia ... ok

test result: ok. 3 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out
```

En el programa de verdad pasarías `saludo(|| hora_del_sistema())`; en una prueba, pasas una closure que devuelve **lo que tú decidas** (`|| 8`), sin tocar el reloj real. La función `saludo` no sabe ni le importa si el reloj es de verdad o no: solo necesita algo que implemente `Fn() -> u32`.

La misma idea sirve para cualquier cosa que no controles: una fecha, una llamada de red, un número aleatorio o un archivo. Si tu función los recibe como parámetro (una closure, o un trait con una implementación real y otra de prueba), puedes sustituirlos al probar.

## Errores frecuentes

| Error | Cómo evitarlo |
|---|---|
| Llamar directamente a la hora del sistema, una red o un archivo desde dentro de la función | Recíbelo como parámetro (closure o trait) y pasa la versión real solo en `main` |
| Crear una interfaz con un solo método únicamente para poder sustituirla al probar | Casi siempre basta una closure (`impl Fn(...) -> ...`); un trait merece la pena si hace falta más de un método relacionado |
| Probar una función que mezcla cálculo con efectos (leer el reloj, escribir en pantalla) a la vez | Separa el cálculo (fácil de probar) de los efectos (difíciles de probar) |
| Confiar en que «funciona en mi máquina» porque la hora actual dio la categoría esperada | Fija la entrada con una closure de prueba: el resultado no debe depender de cuándo ejecutes la prueba |

## Para practicar

Los ejercicios [A4.4 y A4.6](ejercicios.md) usan preparación previa y un reloj falso. Para ver la misma idea escrita en otros lenguajes: [Pasar de uno a otro](https://apuntes-dam.github.io/apuntes-lenguajes/pasar/).
