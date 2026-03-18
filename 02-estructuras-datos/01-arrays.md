# 01 - Arrays en Go

> **Objetivo:** Comprender qué son los arrays en Go, cómo declararlos, inicializarlos y operar con ellos, entendiendo sus limitaciones frente a otras estructuras.

---

## Tabla de Contenidos

1. [¿Qué es un array?](#qué-es-un-array)
2. [Declaración e inicialización](#declaración-e-inicialización)
3. [Acceso y modificación](#acceso-y-modificación)
4. [Recorrer un array](#recorrer-un-array)
5. [Arrays multidimensionales](#arrays-multidimensionales)
6. [Limitaciones de los arrays](#limitaciones-de-los-arrays)
7. [Ejercicios propuestos](#ejercicios-propuestos)

---

## ¿Qué es un array?

Un **array** en Go es una secuencia de elementos del **mismo tipo** con un **tamaño fijo** determinado en tiempo de compilación. A diferencia de otros lenguajes, en Go el tamaño es parte del tipo: `[3]int` y `[5]int` son tipos distintos.

```go
// El tipo [3]int y [5]int son tipos DIFERENTES en Go
var a [3]int
var b [5]int
// a = b  // ❌ Error de compilación: tipos distintos
```

---

## Declaración e inicialización

```go
package main

import "fmt"

func main() {
    // 1. Declaración sin inicializar (se usa el valor cero del tipo)
    var numeros [3]int
    fmt.Println("Sin inicializar:", numeros) // [0 0 0]

    // 2. Declaración con valores literales
    colores := [3]string{"rojo", "verde", "azul"}
    fmt.Println("Colores:", colores)

    // 3. Go cuenta los elementos automáticamente con [...]
    primos := [...]int{2, 3, 5, 7, 11}
    fmt.Println("Primos:", primos)
    fmt.Println("Cantidad de primos:", len(primos)) // 5

    // 4. Inicialización por índice
    semana := [7]string{0: "Lunes", 6: "Domingo"}
    fmt.Println("Semana:", semana) // Los índices no asignados quedan como ""
}
```

---

## Acceso y modificación

```go
package main

import "fmt"

func main() {
    frutas := [4]string{"manzana", "banana", "cereza", "durazno"}

    // Acceso por índice (base 0)
    fmt.Println("Primera fruta:", frutas[0]) // manzana
    fmt.Println("Última fruta:", frutas[3])  // durazno

    // Modificación
    frutas[1] = "mango"
    fmt.Println("Después de modificar:", frutas)

    // len() retorna la longitud del array
    fmt.Println("Longitud:", len(frutas)) // 4
}
```

---

## Recorrer un array

```go
package main

import "fmt"

func main() {
    temperaturas := [5]float64{18.5, 22.3, 19.1, 25.0, 21.7}

    // Método 1: bucle for clásico
    fmt.Println("--- Bucle clásico ---")
    for i := 0; i < len(temperaturas); i++ {
        fmt.Printf("Día %d: %.1f°C\n", i+1, temperaturas[i])
    }

    // Método 2: range (más idiomático en Go)
    fmt.Println("--- Con range ---")
    for indice, valor := range temperaturas {
        fmt.Printf("Índice %d → %.1f°C\n", indice, valor)
    }

    // Método 3: range ignorando el índice
    suma := 0.0
    for _, t := range temperaturas {
        suma += t
    }
    fmt.Printf("Promedio: %.2f°C\n", suma/float64(len(temperaturas)))
}
```

---

## Arrays multidimensionales

```go
package main

import "fmt"

func main() {
    // Matriz 3x3 (tabla de multiplicar parcial)
    var matriz [3][3]int

    // Llenar la matriz
    for i := 0; i < 3; i++ {
        for j := 0; j < 3; j++ {
            matriz[i][j] = (i + 1) * (j + 1)
        }
    }

    // Mostrar la matriz
    fmt.Println("Tabla de multiplicar (3x3):")
    for _, fila := range matriz {
        fmt.Println(fila)
    }
    // Salida:
    // [1 2 3]
    // [2 4 6]
    // [3 6 9]
}
```

---

## Limitaciones de los arrays

> 💡 **Punto clave:** Los arrays en Go tienen **tamaño fijo** y se pasan **por valor** (se copian). Para trabajo dinámico, usaremos **slices** (siguiente sección).

```go
package main

import "fmt"

func duplicarPrimero(arr [3]int) {
    arr[0] = arr[0] * 2 // Modifica la COPIA, no el original
}

func main() {
    datos := [3]int{10, 20, 30}
    duplicarPrimero(datos)
    fmt.Println(datos) // [10 20 30] → sin cambios, se pasó una copia
}
```

---

## Ejercicios propuestos

### Ejercicio 1 — Suma y promedio
Declara un array de 6 enteros con los valores `{4, 8, 15, 16, 23, 42}`. Calcula e imprime la suma y el promedio.

### Ejercicio 2 — Invertir un array
Dado el array `{1, 2, 3, 4, 5}`, imprímelo en orden inverso **sin usar otro array**.

### Ejercicio 3 — Máximo y mínimo
Escribe un programa que encuentre el valor máximo y mínimo dentro de un array de 8 enteros de tu elección.

### Ejercicio 4 — Buscar un elemento
Dado un array de nombres `{"Ana", "Bruno", "Carlos", "Diana"}`, escribe un programa que indique si "Carlos" está en el array y en qué posición.

<details>
<summary>💡 Pista para el Ejercicio 2</summary>

Usa dos índices: uno desde el inicio (`i = 0`) y otro desde el final (`j = len(arr)-1`). Intercambia los valores mientras `i < j`.

</details>
