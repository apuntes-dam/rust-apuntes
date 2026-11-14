# A1.B Colecciones sin bucles

Casi todo lo que haces con un `Vec` con un `for` entra en unas pocas operaciones: **quedarte con** unos elementos, **transformarlos**, **reducirlos** a un valor, **ordenarlos**, **agruparlos** y **preguntar** si alguno o todos cumplen algo. Dar nombre a cada operación hace que el código diga **qué** quieres, no **cómo** recorrer la colección.

## Las operaciones en Rust: iteradores

En Rust estas operaciones son métodos de **`Iterator`**, el trait que implementan `Vec`, arrays, `HashMap` y muchas más colecciones a través de `.iter()` (prestado), `.into_iter()` (dueño) o `.iter_mut()` (mutable):

| Qué quieres | Cómo se hace |
|---|---|
| Quedarte con los que cumplen | `.filter(condición)` |
| Transformar cada elemento | `.map(transformación)` |
| Reducir a un solo valor | `.sum()`, `.max()`, `.min()`, `.count()`, o `.fold(inicial, acumulador)` |
| Ordenar con un criterio | ordena un `Vec` ya recogido: `.sort()` / `.sort_by_key(criterio)`, o quédate con el mejor sin ordenar: `.max_by_key(...)` / `.min_by_key(...)` |
| Agrupar o contar por clave | un `HashMap`/`BTreeMap` con `.entry(clave).or_insert(0)` |
| ¿Alguno / todos? | `.any(...)` / `.all(...)` |
| Primero, o nada | `.next()` sobre el iterador (devuelve `Option<T>`) |
| Sin repetidos · los primeros n | ordenar y `.dedup()` · `.take(n)` |
| Guardar el resultado en una colección | `.collect()` (hace falta decir en qué: `Vec<_>`, `HashMap<_, _>`...) |

## Un ejemplo completo

Una pastelería guarda sus pedidos (producto, cantidad y precio por unidad). Con las operaciones anteriores se responde a todo sin un solo bucle explícito para calcular:

```rust
struct Pedido {
    producto: &'static str,
    cantidad: i32,
    precio: i32,
}

fn main() {
    let pedidos = vec![
        Pedido { producto: "tarta", cantidad: 2, precio: 18 },
        Pedido { producto: "galleta", cantidad: 12, precio: 1 },
        Pedido { producto: "tarta", cantidad: 1, precio: 18 },
        Pedido { producto: "magdalena", cantidad: 3, precio: 2 },
        Pedido { producto: "galleta", cantidad: 7, precio: 1 },
    ];

    // 1. transformar: el importe de cada pedido
    let importes: Vec<i32> = pedidos.iter().map(|p| p.cantidad * p.precio).collect();
    let textos: Vec<String> = importes.iter().map(|i| i.to_string()).collect();
    println!("importes: {}", textos.join(", "));

    // 2. reducir: el total
    let total: i32 = importes.iter().sum();
    println!("total: {}", total);

    // 3. filtrar y contar: pedidos de más de 10
    let caros = pedidos.iter().filter(|p| p.cantidad * p.precio > 10).count();
    println!("pedidos de más de 10: {}", caros);

    // 4. agrupar: unidades por producto
    use std::collections::BTreeMap;
    let mut por_producto: BTreeMap<&str, i32> = BTreeMap::new();
    for p in &pedidos {
        *por_producto.entry(p.producto).or_insert(0) += p.cantidad;
    }
    for (producto, unidades) in &por_producto {
        println!("  {}: {} unidades", producto, unidades);
    }

    // 5. ordenar: el pedido más caro primero
    let mas_caro = pedidos.iter().max_by_key(|p| p.cantidad * p.precio).unwrap();
    println!("el más caro: {} ({})", mas_caro.producto, mas_caro.cantidad * mas_caro.precio);

    // 6. preguntar
    println!("¿alguno de magdalena? {}", pedidos.iter().any(|p| p.producto == "magdalena"));
    println!("¿todos de más de 1 unidad? {}", pedidos.iter().all(|p| p.cantidad > 1));

    // 7. productos distintos, ordenados
    let mut productos: Vec<&str> = pedidos.iter().map(|p| p.producto).collect();
    productos.sort();
    productos.dedup();
    println!("productos: {}", productos.join(", "));
}
```

Salida:

```text
importes: 36, 12, 18, 6, 7
total: 79
pedidos de más de 10: 3
  galleta: 19 unidades
  magdalena: 3 unidades
  tarta: 3 unidades
el más caro: tarta (36)
¿alguno de magdalena? true
¿todos de más de 1 unidad? false
productos: galleta, magdalena, tarta
```

Cada bloque de la salida es una de las operaciones de la tabla. Compruébalo a mano: los importes son `2·18`, `12·1`, `1·18`, `3·2` y `7·1`; el total es `79`; y hay `19` galletas porque `12 + 7`.

## Lo que conviene saber

* **No modifican la colección original.** `.filter()`, `.map()` o `.sum()` consumen el **iterador**, no el `Vec`: la colección de partida queda igual mientras uses `.iter()` (que da referencias prestadas). Solo cambia si llamas a `.sort()` directamente sobre el `Vec`, que sí lo ordena en el sitio.
* **Son perezosos.** `.filter()` y `.map()` no hacen nada por sí solos: solo **describen** la cadena. Se explica en [A1.C](t-composicion.md).
* **`collect()` necesita saber el destino.** El mismo iterador puede recogerse en un `Vec<i32>`, un `HashSet<i32>` o un `String`; Rust lo deduce del tipo de la variable o de una anotación `::<Vec<_>>()`.

```rust
fn main() {
    let numeros = [5, 8, 12, 3, 20];
    let resultado: Vec<i32> = numeros.iter().filter(|&&n| n > 4).map(|n| n * 10).collect();
    let mut ordenado = resultado.clone();
    ordenado.sort();
    println!("{:?}", resultado);
    println!("{:?}", ordenado);
}
```

Salida:

```text
[50, 80, 120, 200]
[50, 80, 120, 200]
```

## Errores frecuentes

| Error | Cómo evitarlo |
|---|---|
| Usar `.iter()` cuando necesitas quedarte con los valores (no referencias) | Usa `.into_iter()`, o copia con `.cloned()`/`.copied()` tras el `.iter()` |
| Olvidar `.collect()` y esperar un `Vec` | Un iterador no es un `Vec`: termina la cadena con `.collect()`, `.sum()`, `.count()`... |
| No anotar el tipo que `.collect()` debe producir | Rust no siempre lo deduce: anota la variable (`let v: Vec<i32> = ...`) |
| Ordenar con `.sort()` esperando no tocar el original | `.sort()` ordena **en el sitio**; clona antes si necesitas conservar el orden original |

## Para practicar

Los ejercicios [A1.3 y A1.4](ejercicios.md) usan estas operaciones. Para ver la misma idea escrita en otros lenguajes: [Pasar de uno a otro](https://apuntes-dam.github.io/apuntes-lenguajes/pasar/).
