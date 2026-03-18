# 01 - Funciones en Go: Ciudadanos de Primera Clase

> **Objetivo:** Comprender que en Go las funciones son valores de primera clase: pueden asignarse a variables, pasarse como argumentos y retornarse desde otras funciones. Dominar múltiples valores de retorno, funciones anónimas y closures.

---

## Tabla de Contenidos

1. [Repaso: funciones básicas](#repaso-funciones-básicas)
2. [Múltiples valores de retorno](#múltiples-valores-de-retorno)
3. [Retornos nombrados](#retornos-nombrados)
4. [Funciones como valores](#funciones-como-valores)
5. [Funciones anónimas](#funciones-anónimas)
6. [Closures](#closures)
7. [Funciones que retornan funciones](#funciones-que-retornan-funciones)
8. [Ejercicios propuestos](#ejercicios-propuestos)

---

## Repaso: funciones básicas

```go
package main

import "fmt"

// Firma: func nombre(parámetros) tipo_retorno
func saludar(nombre string) string {
    return "Hola, " + nombre + "!"
}

// Múltiples parámetros del mismo tipo: shorthand
func sumar(a, b int) int {
    return a + b
}

// Sin retorno
func imprimir(msg string) {
    fmt.Println(msg)
}

func main() {
    fmt.Println(saludar("Mundo"))
    fmt.Println("3 + 4 =", sumar(3, 4))
    imprimir("Go es genial")
}
```

---

## Múltiples valores de retorno

Esta es una de las características más distintivas de Go. Es el patrón estándar para manejar errores.

```go
package main

import (
    "errors"
    "fmt"
)

// Retorna cociente y error
func dividir(a, b float64) (float64, error) {
    if b == 0 {
        return 0, errors.New("no se puede dividir por cero")
    }
    return a / b, nil // nil significa "sin error"
}

// Retorna mínimo y máximo de un slice
func minMax(numeros []int) (int, int) {
    if len(numeros) == 0 {
        return 0, 0
    }
    min, max := numeros[0], numeros[0]
    for _, n := range numeros[1:] {
        if n < min {
            min = n
        }
        if n > max {
            max = n
        }
    }
    return min, max
}

func main() {
    // Capturar múltiples retornos
    resultado, err := dividir(10, 3)
    if err != nil {
        fmt.Println("Error:", err)
    } else {
        fmt.Printf("10 / 3 = %.4f\n", resultado)
    }

    // Usar _ para ignorar valores que no necesitamos
    _, err2 := dividir(5, 0)
    if err2 != nil {
        fmt.Println("Error:", err2)
    }

    // minMax
    datos := []int{8, 3, 15, 2, 9, 7}
    min, max := minMax(datos)
    fmt.Printf("Datos: %v\nMín: %d, Máx: %d\n", datos, min, max)
}
```

---

## Retornos nombrados

```go
package main

import "fmt"

// Los retornos nombrados actúan como variables declaradas al inicio
func calcularEstadisticas(datos []float64) (suma float64, promedio float64, cantidad int) {
    cantidad = len(datos)
    if cantidad == 0 {
        return // "naked return": retorna los valores actuales de las variables nombradas
    }

    for _, v := range datos {
        suma += v
    }
    promedio = suma / float64(cantidad)

    return // naked return con los valores calculados
}

func main() {
    datos := []float64{10, 20, 30, 40, 50}
    suma, promedio, n := calcularEstadisticas(datos)
    fmt.Printf("Suma: %.0f | Promedio: %.1f | N: %d\n", suma, promedio, n)

    s, p, cnt := calcularEstadisticas(nil)
    fmt.Printf("Vacío: suma=%.0f, prom=%.0f, n=%d\n", s, p, cnt)
}
```

> 💡 Los retornos nombrados son útiles para documentar el propósito de cada retorno, pero úsalos con moderación para no oscurecer el flujo del código.

---

## Funciones como valores

```go
package main

import (
    "fmt"
    "sort"
)

// Un tipo función puede declararse
type Transformador func(int) int

func aplicar(numeros []int, fn Transformador) []int {
    resultado := make([]int, len(numeros))
    for i, n := range numeros {
        resultado[i] = fn(n)
    }
    return resultado
}

func main() {
    numeros := []int{1, 2, 3, 4, 5}

    // Asignar funciones a variables
    doble := func(n int) int { return n * 2 }
    cuadrado := func(n int) int { return n * n }

    fmt.Println("Original:", numeros)
    fmt.Println("Dobles:  ", aplicar(numeros, doble))
    fmt.Println("Cuadrados:", aplicar(numeros, cuadrado))

    // Ordenar con función comparadora personalizada
    palabras := []string{"banana", "kiwi", "manzana", "uva", "cereza"}
    sort.Slice(palabras, func(i, j int) bool {
        return len(palabras[i]) < len(palabras[j]) // por longitud
    })
    fmt.Println("Por longitud:", palabras)

    // Slice de funciones
    operaciones := []func(int, int) int{
        func(a, b int) int { return a + b },
        func(a, b int) int { return a - b },
        func(a, b int) int { return a * b },
    }
    nombres := []string{"suma", "resta", "producto"}

    for i, op := range operaciones {
        fmt.Printf("%s(10, 3) = %d\n", nombres[i], op(10, 3))
    }
}
```

---

## Funciones anónimas

```go
package main

import "fmt"

func main() {
    // Función anónima asignada a variable
    cubo := func(n float64) float64 {
        return n * n * n
    }
    fmt.Println("Cubo de 3:", cubo(3))

    // IIFE (Immediately Invoked Function Expression)
    // Se define y ejecuta en el mismo lugar
    resultado := func(x, y int) int {
        return x*x + y*y
    }(3, 4) // ← se llama inmediatamente con argumentos 3 y 4
    fmt.Println("3² + 4² =", resultado)

    // Útil para encapsular lógica de inicialización
    config := func() map[string]string {
        m := make(map[string]string)
        m["host"] = "localhost"
        m["port"] = "8080"
        m["modo"] = "desarrollo"
        return m
    }()
    fmt.Println("Config:", config)
}
```

---

## Closures

Un **closure** es una función que "captura" variables del entorno donde fue creada. Recuerda y puede modificar esas variables aunque el scope original haya terminado.

```go
package main

import "fmt"

// contadorFactory retorna un closure que recuerda su estado
func contadorFactory(inicio int) func() int {
    cuenta := inicio // esta variable es "capturada" por el closure
    return func() int {
        cuenta++
        return cuenta
    }
}

func main() {
    // Cada llamada a contadorFactory crea su PROPIO estado independiente
    contadorA := contadorFactory(0)
    contadorB := contadorFactory(10)

    fmt.Println("A:", contadorA()) // 1
    fmt.Println("A:", contadorA()) // 2
    fmt.Println("A:", contadorA()) // 3
    fmt.Println("B:", contadorB()) // 11
    fmt.Println("B:", contadorB()) // 12
    fmt.Println("A:", contadorA()) // 4 (A sigue desde donde estaba)
}
```

### Closure práctico: acumulador con historial

```go
package main

import "fmt"

func nuevoAcumulador() (func(float64), func() float64, func() []float64) {
    var historial []float64

    agregar := func(v float64) {
        historial = append(historial, v)
    }

    total := func() float64 {
        suma := 0.0
        for _, v := range historial {
            suma += v
        }
        return suma
    }

    verHistorial := func() []float64 {
        return historial
    }

    return agregar, total, verHistorial
}

func main() {
    agregar, total, historial := nuevoAcumulador()

    agregar(10.5)
    agregar(20.0)
    agregar(5.5)

    fmt.Println("Historial:", historial())
    fmt.Printf("Total: %.1f\n", total())
}
```

---

## Funciones que retornan funciones

```go
package main

import (
    "fmt"
    "strings"
)

// Fábrica de multiplicadores
func multiplicadorPor(factor int) func(int) int {
    return func(n int) int {
        return n * factor
    }
}

// Middleware pattern: función que envuelve otra función
func conLog(nombre string, fn func(int) int) func(int) int {
    return func(n int) int {
        resultado := fn(n)
        fmt.Printf("[LOG] %s(%d) = %d\n", nombre, n, resultado)
        return resultado
    }
}

// Pipeline: encadenar transformaciones
func pipeline(valor string, fns ...func(string) string) string {
    for _, fn := range fns {
        valor = fn(valor)
    }
    return valor
}

func main() {
    // Multiplicadores
    doble := multiplicadorPor(2)
    triple := multiplicadorPor(3)
    porCinco := multiplicadorPor(5)

    fmt.Println(doble(7))   // 14
    fmt.Println(triple(7))  // 21
    fmt.Println(porCinco(7)) // 35

    // Con logging
    dobleConLog := conLog("doble", doble)
    dobleConLog(9)

    // Pipeline de strings
    resultado := pipeline(
        "  hola mundo  ",
        strings.TrimSpace,
        strings.ToUpper,
        func(s string) string { return strings.ReplaceAll(s, " ", "_") },
    )
    fmt.Println(resultado) // HOLA_MUNDO
}
```

---

## Ejercicios propuestos

### Ejercicio 1 — Map y Filter funcionales
Implementa las funciones:
- `mapSlice(s []int, fn func(int) int) []int`
- `filterSlice(s []int, predicado func(int) bool) []int`
- `reduce(s []int, inicial int, fn func(int, int) int) int`

Úsalas para: filtrar pares, cuadrar los resultados y sumar todo.

### Ejercicio 2 — Memoización genérica
Escribe una función `memoize(fn func(int) int) func(int) int` que retorne una versión cacheada de `fn`. Pruébala con Fibonacci.

### Ejercicio 3 — Función parcial
Escribe `parcial(fn func(int, int) int, a int) func(int) int` que "fije" el primer argumento de una función binaria.  
Ejemplo: `sumar5 := parcial(sumar, 5)` → `sumar5(3)` devuelve `8`.

### Ejercicio 4 — Decorador de validación
Escribe una función `validarPositivo(fn func(float64) float64) func(float64) float64` que envuelva cualquier función numérica retornando un error (usando `fmt.Println`) si el argumento es negativo.

<details>
<summary>💡 Pista para Ejercicio 2</summary>

Usa un `map[int]int` dentro del closure para guardar los resultados ya calculados. Si el valor está en el mapa, retórnalo directamente.

</details>
