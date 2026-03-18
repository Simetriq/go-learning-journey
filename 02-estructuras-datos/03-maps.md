# 03 - Maps en Go

> **Objetivo:** Aprender a usar maps (diccionarios) para almacenar pares clave-valor, las operaciones disponibles y los patrones más comunes en Go.

---

## Tabla de Contenidos

1. [¿Qué es un map?](#qué-es-un-map)
2. [Crear maps](#crear-maps)
3. [Operaciones CRUD](#operaciones-crud)
4. [Verificar existencia de una clave](#verificar-existencia-de-una-clave)
5. [Iterar sobre un map](#iterar-sobre-un-map)
6. [Maps como contadores](#maps-como-contadores)
7. [Maps de slices](#maps-de-slices)
8. [Ejercicios propuestos](#ejercicios-propuestos)

---

## ¿Qué es un map?

Un **map** es una estructura de datos que asocia **claves únicas** con **valores**. Internamente usa una tabla hash, por lo que las operaciones de acceso, inserción y eliminación son O(1) en promedio.

```
map[string]int
  "manzana" → 5
  "banana"  → 3
  "cereza"  → 12
```

> ⚠️ Los maps **no tienen orden garantizado**. Si necesitas orden, debes ordenar las claves explícitamente.

---

## Crear maps

```go
package main

import "fmt"

func main() {
    // 1. Literal de map
    capitales := map[string]string{
        "Argentina": "Buenos Aires",
        "Brasil":    "Brasilia",
        "Chile":     "Santiago",
    }
    fmt.Println("Capitales:", capitales)

    // 2. Con make (recomendado cuando el tamaño es desconocido)
    edades := make(map[string]int)
    edades["Ana"] = 28
    edades["Bob"] = 34

    // 3. Map nil (¡cuidado! escribir en un map nil causa pánico)
    var nulo map[string]int
    fmt.Println("Es nil:", nulo == nil) // true
    // nulo["clave"] = 1  // ❌ panic: assignment to entry in nil map

    // La forma segura: siempre usar make o literal
    seguro := make(map[string]int)
    seguro["clave"] = 1 // ✅
    _ = seguro
}
```

---

## Operaciones CRUD

```go
package main

import "fmt"

func main() {
    inventario := map[string]int{
        "tornillos": 150,
        "tuercas":   200,
        "arandelas": 80,
    }

    // ── CREATE / UPDATE ──────────────────────────────────────────────────
    inventario["clavos"] = 300    // nueva clave
    inventario["tornillos"] = 175 // actualizar existente
    fmt.Println("Tras agregar:", inventario)

    // ── READ ─────────────────────────────────────────────────────────────
    cantidad := inventario["tuercas"]
    fmt.Println("Tuercas:", cantidad)

    // Clave inexistente retorna el valor cero del tipo (0 para int)
    fmt.Println("Pernos (no existe):", inventario["pernos"]) // 0

    // ── DELETE ───────────────────────────────────────────────────────────
    delete(inventario, "arandelas")
    fmt.Println("Tras eliminar:", inventario)

    // len() retorna la cantidad de pares
    fmt.Println("Total ítems:", len(inventario))
}
```

---

## Verificar existencia de una clave

```go
package main

import "fmt"

func main() {
    precios := map[string]float64{
        "café":   2.50,
        "té":     1.80,
        "jugo":   3.20,
    }

    // El idioma Go: valor, ok := map[clave]
    // "ok" es true si la clave existe, false si no
    if precio, ok := precios["café"]; ok {
        fmt.Printf("Café cuesta: $%.2f\n", precio)
    }

    if _, ok := precios["mate"]; !ok {
        fmt.Println("El mate no está en el menú")
    }

    // Patrón común: obtener con valor por defecto
    buscar := "agua"
    precio, ok := precios[buscar]
    if !ok {
        precio = 0.0 // valor por defecto
    }
    fmt.Printf("%s: $%.2f\n", buscar, precio)
}
```

---

## Iterar sobre un map

```go
package main

import (
    "fmt"
    "sort"
)

func main() {
    poblacion := map[string]int{
        "Buenos Aires": 3_120_000,
        "Córdoba":      1_454_000,
        "Rosario":      1_193_000,
        "Mendoza":        115_000,
    }

    // Iteración básica (orden NO garantizado)
    fmt.Println("── Iteración básica ──")
    for ciudad, hab := range poblacion {
        fmt.Printf("%-15s → %d habitantes\n", ciudad, hab)
    }

    // Iteración en orden alfabético
    fmt.Println("\n── Orden alfabético ──")
    claves := make([]string, 0, len(poblacion))
    for k := range poblacion {
        claves = append(claves, k)
    }
    sort.Strings(claves)

    for _, ciudad := range claves {
        fmt.Printf("%-15s → %d habitantes\n", ciudad, poblacion[ciudad])
    }
}
```

---

## Maps como contadores

Este es uno de los patrones más útiles en Go:

```go
package main

import (
    "fmt"
    "sort"
    "strings"
)

func contarPalabras(texto string) map[string]int {
    contador := make(map[string]int)
    palabras := strings.Fields(texto) // divide por espacios en blanco

    for _, palabra := range palabras {
        // Normalizar: minúsculas y sin puntuación básica
        palabra = strings.ToLower(strings.Trim(palabra, ".,!?;:"))
        contador[palabra]++ // si no existe, inicia en 0 y luego suma 1
    }
    return contador
}

func main() {
    texto := "el gato come y el perro come también el gato duerme"
    conteo := contarPalabras(texto)

    // Ordenar por clave para salida consistente
    claves := make([]string, 0, len(conteo))
    for k := range conteo {
        claves = append(claves, k)
    }
    sort.Strings(claves)

    fmt.Println("Frecuencia de palabras:")
    for _, palabra := range claves {
        fmt.Printf("  %-12s: %d\n", palabra, conteo[palabra])
    }
}
```

---

## Maps de slices

```go
package main

import "fmt"

func main() {
    // Agrupar estudiantes por nota (A, B, C, ...)
    notas := map[string][]string{
        "A": {},
        "B": {},
        "C": {},
    }

    resultados := []struct {
        nombre string
        nota   string
    }{
        {"Ana", "A"},
        {"Bruno", "B"},
        {"Carlos", "A"},
        {"Diana", "C"},
        {"Elena", "B"},
        {"Felipe", "A"},
    }

    for _, r := range resultados {
        notas[r.nota] = append(notas[r.nota], r.nombre)
    }

    for nota, alumnos := range notas {
        fmt.Printf("Nota %s: %v\n", nota, alumnos)
    }
}
```

---

## Ejercicios propuestos

### Ejercicio 1 — Anagrama
Escribe una función `sonAnagramas(a, b string) bool` que use un map para verificar si dos palabras son anagramas (contienen las mismas letras).  
Ejemplo: `"listen"` y `"silent"` → `true`

### Ejercicio 2 — Índice invertido
Dado un mapa de `autor → []libros`, construye el mapa inverso `libro → autor`.

### Ejercicio 3 — Top N palabras
Usando la función `contarPalabras` de los ejemplos, escribe una función que retorne las **3 palabras más frecuentes** de un texto dado.

### Ejercicio 4 — Cache de Fibonacci
Implementa una función `fibonacci(n int) int` que use un `map[int]int` como caché (memoización) para evitar recalcular valores ya computados.

<details>
<summary>💡 Pista para el Ejercicio 1</summary>

Itera sobre los caracteres de la primera palabra incrementando un contador en un map, luego itera sobre la segunda decrementando. Al final, todos los valores deben ser 0.

</details>
