# 3.5 XML

**XML** es otro formato de texto para guardar datos con **etiquetas**. La biblioteca estándar de Rust **no trae** soporte de XML; para **leer** se suele usar el crate **`roxmltree`**, que carga el documento como un árbol de solo lectura:

```bash
cargo add roxmltree
```

## Partes de un XML

| Parte | Ejemplo |
|---|---|
| **Elemento** | `<nombre>Juan</nombre>` |
| **Elemento raíz** | `<usuarios> ... </usuarios>` (uno solo, que contiene todo) |
| **Atributo** | `<usuario id="1">` |
| **Texto** | `Juan` (lo de dentro de las etiquetas) |

## Leer

En `Cargo.toml`:

```toml
[dependencies]
roxmltree = "0.20"
```

```rust
use roxmltree::Document;

fn main() {
    let texto = r#"<usuarios>
    <usuario><id>1</id><nombre>Juan</nombre><edad>30</edad></usuario>
    <usuario><id>2</id><nombre>Ana</nombre><edad>25</edad></usuario>
</usuarios>"#;

    let doc = Document::parse(texto).unwrap();
    let raiz = doc.root_element();
    println!("{}", raiz.tag_name().name());

    for usuario in raiz.children().filter(|n| n.is_element()) {
        // buscar un hijo por su nombre y leer su texto
        let campo = |nombre: &str| {
            usuario.children().find(|n| n.has_tag_name(nombre)).and_then(|n| n.text()).unwrap_or("")
        };
        let edad: u32 = campo("edad").parse().unwrap();
        println!("{} {} ({} años)", campo("id"), campo("nombre"), edad);
    }

    // buscar en todo el documento
    let nombres: Vec<&str> = doc.descendants().filter(|n| n.has_tag_name("nombre")).filter_map(|n| n.text()).collect();
    println!("{:?}", nombres);
}
```

Salida:

```text
usuarios
1 Juan (30 años)
2 Ana (25 años)
["Juan", "Ana"]
```

## Atributos

En `Cargo.toml`:

```toml
[dependencies]
roxmltree = "0.20"
```

```rust
use roxmltree::Document;

fn main() {
    let doc = Document::parse(r#"<tienda><producto codigo="A1" precio="2.5">Pan</producto><producto codigo="B2" precio="1.2">Leche</producto></tienda>"#).unwrap();
    let mut total = 0.0;
    for p in doc.descendants().filter(|n| n.has_tag_name("producto")) {
        let precio: f64 = p.attribute("precio").unwrap().parse().unwrap();
        println!("{} ({}) {precio}", p.text().unwrap(), p.attribute("codigo").unwrap());
        total += precio;
    }
    println!("Total: {total}");
}
```

Salida:

```text
Pan (A1) 2.5
Leche (B2) 1.2
Total: 3.7
```

## Escribir XML

`roxmltree` solo lee. Para producir XML sencillo se **construye el texto** con `format!`, **escapando** los caracteres especiales (`<`, `>`, `&`, `"`). Para documentos grandes existe `quick-xml`, que también escribe.

```rust
use std::fs;

fn escapar(s: &str) -> String {
    s.replace('&', "&amp;").replace('<', "&lt;").replace('>', "&gt;").replace('"', "&quot;")
}

fn main() {
    let usuarios = [(1, "Juan", 30), (2, "Ana & Luis", 25)];

    let mut xml = String::from("<usuarios>\n");
    for (id, nombre, edad) in usuarios {
        xml.push_str(&format!(
            "    <usuario>\n        <id>{id}</id>\n        <nombre>{}</nombre>\n        <edad>{edad}</edad>\n    </usuario>\n",
            escapar(nombre)
        ));
    }
    xml.push_str("</usuarios>\n");

    fs::write("datos_usuarios_nuevo.xml", &xml).unwrap();
    print!("{xml}");
}
```

Salida:

```text
<usuarios>
    <usuario>
        <id>1</id>
        <nombre>Juan</nombre>
        <edad>30</edad>
    </usuario>
    <usuario>
        <id>2</id>
        <nombre>Ana &amp; Luis</nombre>
        <edad>25</edad>
    </usuario>
</usuarios>
```

## Errores

Un XML mal formado (una etiqueta sin cerrar, un `&` suelto) da un `Err`:

En `Cargo.toml`:

```toml
[dependencies]
roxmltree = "0.20"
```

```rust
use roxmltree::Document;

fn main() {
    for texto in ["<a><b>hola</b></a>", "<a><b>hola</a>", "<a>5 & 6</a>", ""] {
        match Document::parse(texto) {
            Ok(_) => println!("{texto:?} -> bien formado"),
            Err(e) => println!("{texto:?} -> error: {e}"),
        }
    }
}
```

Salida:

```text
"<a><b>hola</b></a>" -> bien formado
"<a><b>hola</a>" -> error: expected 'b' tag, not 'a' at 1:11
"<a>5 & 6</a>" -> error: malformed entity reference at 1:6
"" -> error: the document does not have a root node
```

## ¿JSON o XML?

| | JSON | XML |
|---|---|---|
| Soporte en Rust | `serde_json` (comodísimo con `struct`) | `roxmltree` para leer, `quick-xml` para leer y escribir |
| Tamaño | Más compacto | Más verboso |
| Atributos y texto mezclado | No | Sí |
| Validación con esquema | JSON Schema (aparte) | DTD, XSD (ver la [web de XML](https://apuntes-dam.github.io/xml-apuntes/)) |

## Errores frecuentes

| Error | Cómo evitarlo |
|---|---|
| Suponer que `roxmltree` modifica o escribe | Es **solo lectura**: construye el texto aparte |
| Olvidar que los **espacios y saltos** entre etiquetas son nodos de texto | Filtra con `is_element()` al recorrer los hijos |
| Un `&` o `<` en un texto sin escapar | Genera XML mal formado: escapa siempre |
| `unwrap()` sobre un campo que puede faltar | Usa `unwrap_or`, `and_then` o `match` |

## Para practicar

Haz el [ejercicio 3.5 de XML](xml.md).
