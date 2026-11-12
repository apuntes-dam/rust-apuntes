# 3.1 Vectores y tuplas

Una **lista** guarda varios valores **en orden** y permite consultar por posición. En Rust hay tres formas parecidas:

| Tipo | Tamaño | Ejemplo |
|---|---|---|
| **Array** `[T; N]` | **Fijo**, se conoce al compilar | `let dias = [1, 2, 3];` |
| **Vector** `Vec<T>` | **Crece y se encoge** | `let mut v = vec![1, 2, 3];` |
| **Slice** `&[T]` | Una **vista** de una parte de un array o vector | `&v[1..3]` |

Casi siempre se usa `Vec<T>`.

## Operaciones básicas

```rust
fn main() {
    let mut numeros = vec![5, 3, 8];
    numeros.push(1);                     // añade al final
    numeros.insert(1, 9);                // inserta en la posición 1
    println!("{:?}", numeros);           // [5, 9, 3, 8, 1]

    println!("{} {}", numeros[0], numeros.len());
    println!("{:?} {:?}", numeros.first(), numeros.last());
    println!("{:?}", numeros.get(10));   // None: no hace panic

    let quitado = numeros.remove(1);     // quita por posición y lo devuelve
    println!("{quitado} {:?}", numeros);
    println!("{:?}", numeros.pop());     // quita el último (Option)

    println!("{}", numeros.contains(&3));
    numeros.sort();
    println!("{:?}", numeros);
    numeros.reverse();
    println!("{:?}", numeros);
}
```

Salida:

```text
[5, 9, 3, 8, 1]
5 5
Some(5) Some(1)
None
9 [5, 3, 8, 1]
Some(1)
true
[3, 5, 8]
[8, 5, 3]
```

`numeros[10]` fuera de rango **hace `panic`**; `numeros.get(10)` devuelve un `Option`. Para añadir hay que declararlo `mut`.

## Recorrer

```rust
fn main() {
    let asignaturas = vec!["Matemáticas", "Física", "Química"];

    for a in &asignaturas {                          // &: solo mira, el vector sigue siendo tuyo
        println!("Yo estudio {a}");
    }
    for (i, a) in asignaturas.iter().enumerate() {   // con la posición
        println!("{i}: {a}");
    }

    let mut notas = vec![5, 7, 9];
    for n in notas.iter_mut() {                      // &mut: puede cambiar cada elemento
        *n += 1;
    }
    println!("{:?}", notas);
}
```

Salida:

```text
Yo estudio Matemáticas
Yo estudio Física
Yo estudio Química
0: Matemáticas
1: Física
2: Química
[6, 8, 10]
```

`for a in asignaturas` (sin `&`) **consume** el vector: después ya no se puede usar. Se recorre con `&v` para mirarlo y con `&mut v` para cambiarlo.

## Copias, trozos y listas anidadas

```rust
fn main() {
    let original = vec![1, 2, 3, 4, 5];

    let copia = original.clone();            // copia independiente
    println!("{:?} {:?}", copia, &original[1..3]);

    let (izq, der) = original.split_at(2);   // dividir en dos trozos
    println!("{:?} {:?}", izq, der);

    // matriz: vector de vectores
    let mut matriz = vec![vec![0; 3]; 2];    // 2 filas, 3 columnas
    matriz[1][2] = 7;
    println!("{:?}", matriz);
    for fila in &matriz {
        println!("{}", fila.iter().map(|x| x.to_string()).collect::<Vec<_>>().join(" "));
    }
}
```

Salida:

```text
[1, 2, 3, 4, 5] [2, 3]
[1, 2] [3, 4, 5]
[[0, 0, 0], [0, 0, 7]]
0 0 0
0 0 7
```

## Tuplas

Una **tupla** agrupa valores de **tipos distintos**, de tamaño fijo. Se accede por posición (`.0`, `.1`) o se **desestructura**:

```rust
fn min_max(valores: &[i32]) -> (i32, i32) {
    let mut menor = valores[0];
    let mut mayor = valores[0];
    for &v in valores {
        if v < menor { menor = v; }
        if v > mayor { mayor = v; }
    }
    (menor, mayor)                 // una función puede devolver varios valores
}

fn main() {
    let persona = ("Ana", 30, 1.68);
    println!("{} {} {}", persona.0, persona.1, persona.2);
    let (nombre, edad, _) = persona;
    println!("{nombre} {edad}");

    let (menor, mayor) = min_max(&[50, 75, 46, 22, 80, 65, 8]);
    println!("Menor: {menor}, mayor: {mayor}");
}
```

Salida:

```text
Ana 30 1.68
Ana 30
Menor: 8, mayor: 80
```

## Transformar con iteradores

Los **iteradores** encadenan operaciones sin escribir el bucle: `map` transforma, `filter` filtra, y al final se «recoge» con `collect`, `sum`, `min`, `max`, `count`:

```rust
fn main() {
    let numeros = vec![1, 2, 3, 4, 5, 6];

    let pares: Vec<i32> = numeros.iter().filter(|n| *n % 2 == 0).cloned().collect();
    let cuadrados: Vec<i32> = numeros.iter().map(|n| n * n).collect();
    let suma: i32 = numeros.iter().sum();
    println!("{:?} {:?} {suma}", pares, cuadrados);
    println!("{:?} {:?}", numeros.iter().min(), numeros.iter().max());
    println!("{}", numeros.iter().any(|&n| n > 5));

    // ordenar decimales: f64 no tiene orden total, hay que decir cómo comparar
    let mut precios = vec![2.5, 0.8, 1.35];
    precios.sort_by(|a, b| a.partial_cmp(b).unwrap());
    println!("{:?}", precios);
    let media = precios.iter().sum::<f64>() / precios.len() as f64;
    println!("{media:.3}");

    // producto escalar de dos vectores
    let a = [1, 2, 3];
    let b = [-1, 0, 2];
    let escalar: i32 = a.iter().zip(b.iter()).map(|(x, y)| x * y).sum();
    println!("{escalar}");
}
```

Salida:

```text
[2, 4, 6] [1, 4, 9, 16, 25, 36] 21
Some(1) Some(6)
true
[0.8, 1.35, 2.5]
1.550
5
```

## Errores frecuentes

| Error | Cómo evitarlo |
|---|---|
| `v[10]` en un vector de 3 elementos | Hace `panic`. Comprueba `v.len()` o usa `v.get(10)` |
| Usar un vector después de `for x in v` | El bucle lo **consume**. Recorre con `&v` |
| Modificar el vector mientras se recorre | El compilador no lo permite (préstamo): guarda los cambios y aplícalos después |
| `v.sort()` con `f64` | No compila: usa `sort_by(\|a, b\| a.partial_cmp(b).unwrap())` |
| Olvidar `mut` al declarar | Sin `mut` no se puede hacer `push` ni `sort` |
| Una división entera sin querer en una media | `suma / v.len()` es entera: convierte antes con `as f64` |

## Para practicar

Haz los [ejercicios 3.1 de vectores](listas.md).
