# U7.1 · Consola: entrada y salida estándar

<div class="ej-gate" data-unit="u07" data-nombre="U7 · Entrada/salida y GUI"></div>

Lectura desde teclado, salida con formato y la diferencia entre salida normal y salida de error.

## Ejercicio 7.1

**Tabla con formato.** Muestra una tabla de productos con columnas alineadas (nombre a la izquierda, precio con 2 decimales y unidades a la derecha) y una línea final con el total:

```text
Producto        Precio   Uds
Café con leche    1.40     3
Tostada           2.10     2
Total: 8.40 €
```

!!! note "En Rust"
    `println!("{nombre:<15}{precio:>6.2}{uds:>5}")` alinea a la izquierda, a la derecha y fija dos decimales.

## Ejercicio 7.2

**Lectura robusta.** Escribe una función que pida un número entero y **repita la pregunta** hasta que el usuario escriba un valor válido, con un mensaje de error claro. Reutilízala para pedir una edad entre 0 y 120. Usa una variable de control en lugar de `break`.

## Ejercicio 7.3

**Salida y salida de error.** Programa un script que escriba los resultados por la **salida estándar** y los avisos por la **salida de error**. Ejecútalo redirigiendo cada salida a un archivo distinto (`>` y `2>`) y comprueba qué hay en cada uno.

!!! note "En Rust"
    Salida de error: `eprintln!(...)`. Para ejecutarlo redirigiendo: `cargo run > resultados.txt 2> errores.txt`.
