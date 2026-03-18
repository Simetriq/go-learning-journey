# 02 - Slices en Go

> **Objetivo:** Dominar los slices, la estructura de datos más usada en Go. Entender su relación con los arrays subyacentes, operaciones comunes y patrones de uso.

---

## Tabla de Contenidos

1. [¿Qué es un slice?](#qué-es-un-slice)
2. [Crear slices](#crear-slices)
3. [Operaciones fundamentales](#operaciones-fundamentales)
4. [Slice de un array o slice](#slice-de-un-array-o-slice)
5. [Copia y referencias](#copia-y-referencias)
6. [Slices de slices (2D)](#slices-de-slices-2d)
7. [Patrones comunes](#patrones-comunes)
8. [Ejercicios propuestos](#ejercicios-propuestos)

---

## ¿Qué es un slice?

Un **slice** es una **vista dinámica** sobre un array subyacente. Internamente tiene tres campos:
- **Puntero** al primer elemento del array subyacente
- **Longitud** (`len`): cantidad de elementos visibles
- **Capacidad** (`cap`): espacio disponible desde el inicio hasta el final del array subyacente

```
Array subyacente: [10, 20, 30, 40, 50]
Slice s:           ↑___________↑
                   ptr  len=3   cap=5
```

---

## Crear slices

```go
package main

import "fmt"

func main() {
    // 1. Literal de slice (sin tamaño fijo, a diferencia de arrays)
    nombres := []string{"Alice", "Bob", "Carol"}
    fmt.Println("Literal:", nombres, "len:", len(nombres), "cap:", cap(nombres))

    // 2. make(tipo, longitud, capacidad)
    // Útil cuando sabes cuántos elementos tendrás
    puntuaciones := make([]int, 5)       // longitud=5, capacidad=5
    buffer := make([]byte, 0, 100)       // longitud=0, capacidad=100

    fmt.Println("make (puntuaciones):", puntuaciones)
    fmt.Println("make (buffer) len:", len(buffer), "cap:", cap(buffer))

    // 3. Slice nil (valor cero de un slice)
    var vacio []int
    fmt.Println("Slice nil:", vacio == nil) // true
    fmt.Println("len de nil:", len(vacio))  // 0 — válido!
}
```

---

## Operaciones fundamentales

```go
package main

import "fmt"

func main() {
    s := []int{1, 2, 3}

    // ── append: agrega uno o más elementos ──────────────────────────────
    s = append(s, 4)
    s = append(s, 5, 6, 7) // múltiples a la vez
    fmt.Println("Después de append:", s) // [1 2 3 4 5 6 7]

    // ── Expandir otro slice con ... ──────────────────────────────────────
    extra := []int{8, 9, 10}
    s = append(s, extra...) // el "..." expande el slice
    fmt.Println("Con extra:", s)

    // ── len y cap ───────────────────────────────────────────────────────
    fmt.Printf("len=%d, cap=%d\n", len(s), cap(s))
    // Go duplica la capacidad al crecer (aprox.), por eso cap > len

    // ── Eliminar un elemento por índice (índice 2) ──────────────────────
    i := 2
    s = append(s[:i], s[i+1:]...)
    fmt.Println("Sin índice 2:", s)

    // ── Insertar en posición (índice 1) ─────────────────────────────────
    s = append(s[:1], append([]int{99}, s[1:]...)...)
    fmt.Println("Con 99 en índice 1:", s)
}
```

---

## Slice de un array o slice

```go
package main

import "fmt"

func main() {
    base := []int{0, 10, 20, 30, 40, 50}

    // Sintaxis: s[inicio:fin] — el fin es EXCLUSIVO
    a := base[1:4]  // [10, 20, 30]
    b := base[:3]   // [0, 10, 20]   equivale a base[0:3]
    c := base[4:]   // [40, 50]      equivale a base[4:len(base)]
    d := base[:]    // copia la vista completa

    fmt.Println("a:", a)
    fmt.Println("b:", b)
    fmt.Println("c:", c)
    fmt.Println("d:", d)

    // ⚠️ IMPORTANTE: comparten el mismo array subyacente
    a[0] = 999
    fmt.Println("base después de modificar a:", base)
    // [0, 999, 20, 30, 40, 50] ← base también cambió!
}
```

---

## Copia y referencias

```go
package main

import "fmt"

func main() {
    original := []int{1, 2, 3, 4, 5}

    // copy() hace una copia INDEPENDIENTE
    copia := make([]int, len(original))
    n := copy(copia, original)
    fmt.Printf("Copiados %d elementos\n", n)

    copia[0] = 999
    fmt.Println("Original:", original) // [1 2 3 4 5] — sin cambios
    fmt.Println("Copia:", copia)       // [999 2 3 4 5]

    // Truco: clonar un slice en una línea
    clon := append([]int{}, original...)
    fmt.Println("Clon:", clon)
}
```

---

## Slices de slices (2D)

```go
package main

import "fmt"

func main() {
    // Matriz dinámica 4x4 con slices
    filas := 4
    cols := 4
    matriz := make([][]int, filas)

    for i := range matriz {
        matriz[i] = make([]int, cols)
        for j := range matriz[i] {
            matriz[i][j] = i*cols + j + 1
        }
    }

    fmt.Println("Matriz 4x4:")
    for _, fila := range matriz {
        fmt.Println(fila)
    }
}
```

---

## Patrones comunes

```go
package main

import (
    "fmt"
    "sort"
)

func main() {
    numeros := []int{5, 2, 8, 1, 9, 3, 7, 4, 6}

    // ── Ordenar ─────────────────────────────────────────────────────────
    sort.Ints(numeros)
    fmt.Println("Ordenado:", numeros)

    // ── Filtrar (filter pattern) ─────────────────────────────────────────
    pares := numeros[:0] // truco: reutilizar el backing array
    for _, n := range numeros {
        if n%2 == 0 {
            pares = append(pares, n)
        }
    }
    fmt.Println("Pares:", pares)

    // ── Stack (pila) con slices ──────────────────────────────────────────
    var pila []string

    // Push
    pila = append(pila, "primero")
    pila = append(pila, "segundo")
    pila = append(pila, "tercero")

    // Pop
    tope := pila[len(pila)-1]
    pila = pila[:len(pila)-1]
    fmt.Println("Popped:", tope, "| Pila:", pila)
}
```

---

## Ejercicios propuestos

### Ejercicio 1 — Eliminar duplicados
Dado el slice `{3, 1, 4, 1, 5, 9, 2, 6, 5, 3}`, escribe una función que retorne un nuevo slice con los elementos únicos (sin duplicados), manteniendo el orden de primera aparición.

### Ejercicio 2 — Rotar un slice
Escribe una función `rotar(s []int, k int) []int` que rote el slice `k` posiciones hacia la izquierda.  
Ejemplo: `rotar([1,2,3,4,5], 2)` → `[3,4,5,1,2]`

### Ejercicio 3 — Aplanar slices
Dado `[][]int{{1,2},{3,4,5},{6}}`, escribe una función que retorne `[]int{1,2,3,4,5,6}`.

### Ejercicio 4 — Frecuencia de elementos
Dado `[]string{"go","python","go","rust","python","go"}`, cuenta cuántas veces aparece cada lenguaje usando **solo slices** (sin maps por ahora).

<details>
<summary>💡 Pista para el Ejercicio 1</summary>

Puedes usar un slice auxiliar para llevar registro de los ya vistos, o iterar y verificar con una función de búsqueda lineal.

</details>
