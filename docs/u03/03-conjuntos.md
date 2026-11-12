# 3.3 Conjuntos

Un **conjunto** guarda valores **sin repetir** y responde rápido a «¿está este valor?». Como los mapas, hay dos: **`HashSet<T>`** (sin orden) y **`BTreeSet<T>`** (ordenado).

## Operaciones

```rust
use std::collections::BTreeSet;

fn main() {
    let mut vistos = BTreeSet::new();
    println!("{}", vistos.insert(3));    // true: era nuevo
    println!("{}", vistos.insert(3));    // false: ya estaba
    vistos.insert(1);
    println!("{:?} {}", vistos, vistos.len());
    println!("{}", vistos.contains(&1));
    vistos.remove(&3);
    println!("{:?}", vistos);

    let a: BTreeSet<i32> = (1..=6).collect();
    let b: BTreeSet<i32> = (4..=9).collect();
    println!("unión:        {:?}", a.union(&b).collect::<Vec<_>>());
    println!("intersección: {:?}", a.intersection(&b).collect::<Vec<_>>());
    println!("diferencia:   {:?}", a.difference(&b).collect::<Vec<_>>());
    println!("simétrica:    {:?}", a.symmetric_difference(&b).collect::<Vec<_>>());
    println!("{} {}", a.is_subset(&b), a.is_disjoint(&b));
}
```

Salida:

```text
true
false
{1, 3} 2
true
{1}
unión:        [1, 2, 3, 4, 5, 6, 7, 8, 9]
intersección: [4, 5, 6]
diferencia:   [1, 2, 3]
simétrica:    [1, 2, 3, 7, 8, 9]
false false
```

## Quitar repetidos

Pasar una lista por un conjunto es la forma más corta de quitar repetidos:

```rust
use std::collections::{BTreeSet, HashSet};

fn main() {
    let compras = vec!["Nuria", "Jorge", "Nuria", "Eva", "Jorge", "Nuria"];

    let unicos: BTreeSet<_> = compras.iter().collect();
    println!("{:?} ({} distintos)", unicos, unicos.len());

    // conservando el orden de la primera aparición
    let mut vistos = HashSet::new();
    let sin_repetir: Vec<_> = compras.iter().filter(|n| vistos.insert(**n)).collect();
    println!("{:?}", sin_repetir);

    // frutas comunes y exclusivas
    let frutas1: BTreeSet<_> = ["manzana", "pera", "naranja", "plátano", "uva"].into_iter().collect();
    let frutas2: BTreeSet<_> = ["manzana", "pera", "durazno", "sandía", "uva"].into_iter().collect();
    println!("{:?}", frutas1.intersection(&frutas2).collect::<Vec<_>>());
    println!("{:?}", frutas1.difference(&frutas2).collect::<Vec<_>>());
    println!("{:?}", frutas2.difference(&frutas1).collect::<Vec<_>>());
}
```

Salida:

```text
{"Eva", "Jorge", "Nuria"} (3 distintos)
["Nuria", "Jorge", "Eva"]
["manzana", "pera", "uva"]
["naranja", "plátano"]
["durazno", "sandía"]
```

## ¿Vector o conjunto?

| Necesitas... | Usa |
|---|---|
| Conservar el **orden** y los **repetidos** | Vector |
| **Quitar repetidos** o comprobar pertenencia rápido | Conjunto |
| Operaciones de unión, intersección y diferencia | Conjunto |

## Errores frecuentes

| Error | Cómo evitarlo |
|---|---|
| Esperar orden en un `HashSet` | No lo hay: usa `BTreeSet` o ordena al final |
| Esperar que `union`, `intersection`... devuelvan un conjunto | Devuelven un **iterador** de referencias: recoge con `.collect()` |
| Guardar `f64` | No se puede (no tiene igualdad total): usa enteros o un tipo envoltorio |
| Que un conjunto «cambie» el orden de la lista original | La lista original no cambia; el conjunto es otra colección |

## Para practicar

Haz los [ejercicios 3.3 de conjuntos](conjuntos.md).
