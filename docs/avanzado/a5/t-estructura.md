# A5.C Estructura y cuándo usar patrones

Los patrones de **estructura** se ocupan de **cómo se combinan** los valores para formar algo más grande.

## Decorador: añadir comportamiento envolviendo

Un **decorador** envuelve un valor que cumple el mismo trait, y añade algo antes o después de delegarle el trabajo. Como todos implementan el mismo trait, se pueden **anidar** en cualquier orden:

```rust
trait Texto {
    fn contenido(&self) -> String;
}

struct Simple {
    valor: String,
}
impl Texto for Simple {
    fn contenido(&self) -> String {
        self.valor.clone()
    }
}

struct Corchetes {
    interior: Box<dyn Texto>,
}
impl Texto for Corchetes {
    fn contenido(&self) -> String {
        format!("[{}]", self.interior.contenido())
    }
}

struct Repetir {
    interior: Box<dyn Texto>,
}
impl Texto for Repetir {
    fn contenido(&self) -> String {
        let c = self.interior.contenido();
        format!("{}-{}", c, c)
    }
}

fn main() {
    let base1: Box<dyn Texto> = Box::new(Simple { valor: "ok".to_string() });
    let v1 = Repetir { interior: Box::new(Corchetes { interior: base1 }) };
    println!("{}", v1.contenido());

    let base2: Box<dyn Texto> = Box::new(Simple { valor: "ok".to_string() });
    let v2 = Corchetes { interior: Box::new(Repetir { interior: base2 }) };
    println!("{}", v2.contenido());
}
```

Salida:

```text
[ok]-[ok]
[ok-ok]
```

Fíjate en que **el orden importa**: `Repetir(Corchetes(Simple))` primero pone corchetes y luego repite el resultado completo (`[ok]-[ok]`), mientras que `Corchetes(Repetir(Simple))` primero repite y luego rodea todo con un solo par de corchetes (`[ok-ok]`). Cada decorador solo conoce el `Box<dyn Texto>` que envuelve, nunca el tipo concreto de dentro.

## Cuándo (no) usar un patrón

Un patrón resuelve un problema concreto; aplicado donde no hace falta, añade indirección sin beneficio:

| Patrón | Úsalo cuando... | No lo uses si... |
|---|---|---|
| Fábrica | Construir depende de un dato que solo se sabe en tiempo de ejecución | Solo hay un tipo posible: llama a su constructor directamente |
| Singleton | De verdad solo puede (o debe) existir una instancia (un registro global, una configuración) | Podrías pasar el valor como parámetro: es más fácil de probar y de razonar |
| Builder | El valor tiene varios campos opcionales o un orden de construcción que validar | Dos o tres campos simples: un constructor normal basta |
| Estrategia / Observador | El comportamiento varía de verdad en tiempo de ejecución, o varias partes deben reaccionar a un cambio | Hay una sola forma de hacerlo: no añadas la indirección "por si acaso" |
| Decorador | Quieres combinar comportamientos de formas distintas sin una clase por combinación | Una sola combinación fija: un struct normal con esos campos es más simple |

!!! tip "La regla general"
    Un patrón es un **nombre para una forma de resolver algo**, no un objetivo en sí mismo. Si tu código ya es simple y claro sin él, añadirlo solo para «usar un patrón» suele empeorarlo. Elige el patrón cuando **reconozcas el problema** que resuelve, no al revés.

## Errores frecuentes

| Error | Cómo evitarlo |
|---|---|
| Anidar decoradores en el orden equivocado | Comprueba con un caso pequeño: el orden de envoltura es el orden en que se aplican los efectos |
| Un decorador que cambia el tipo de trait que expone | Debe implementar el **mismo** trait que envuelve, para poder anidarse |
| Elegir un patrón antes de tener el problema | Escribe primero el código simple; introduce el patrón cuando el problema que resuelve aparezca de verdad |
| Mezclar varios patrones sin necesidad en un ejemplo pequeño | Cada patrón por separado; combínalos solo cuando el diseño real lo pida |

## Para practicar

El ejercicio [A5.6](ejercicios.md) usa un decorador. Para ver la misma idea escrita en otros lenguajes: [Pasar de uno a otro](https://apuntes-dam.github.io/apuntes-lenguajes/pasar/).
