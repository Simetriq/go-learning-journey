# Tipos de Datos Básicos en Go

Este archivo contiene ejemplos prácticos de los tipos de datos fundamentales en Go: strings, números y booleanos.  

---

## 1. Strings (`string`)

Los strings en Go son inmutables y se declaran con comillas dobles.  
El paquete `strings` proporciona funciones útiles para manipularlos.

```go
package main

import (
    "fmt"
    "strings"
)

func main() {
    // Declaración
    saludo := "  Hola, Gophers!  "
    lenguaje := "Go"

    fmt.Println("=== STRINGS ===")
    fmt.Println("Saludo original:", saludo)
    fmt.Println("Lenguaje:", lenguaje)

    // Funciones útiles
    fmt.Println("\n--- Funciones útiles ---")
    fmt.Println("Longitud (len):", len(saludo))
    fmt.Println("Mayúsculas (ToUpper):", strings.ToUpper(saludo))
    fmt.Println("Minúsculas (ToLower):", strings.ToLower(saludo))
    fmt.Println("Sin espacios (TrimSpace):", strings.TrimSpace(saludo))

    // Split
    partes := strings.Split(strings.TrimSpace(saludo), ", ")
    fmt.Println("Dividido (Split):", partes)
    fmt.Println("Primera parte:", partes[0])
    fmt.Println("Segunda parte:", partes[1])

    // Concatenación
    mensaje := "Aprendiendo " + lenguaje
    fmt.Println("Concatenación:", mensaje)
}
```

---

## 2. Números: Enteros (`int`, `int8`, `int16`, `int32`, `int64`)

Go tiene múltiples tipos de enteros con diferentes tamaños y rangos.

```go
package main

import "fmt"

func main() {
    fmt.Println("=== ENTEROS ===")

    // Diferentes formas de declarar enteros
    var a int = 42
    b := 100 // inferencia de tipo -> int
    var c int8 = 127
    var d int64 = 1_000_000

    fmt.Println("a (int):", a)
    fmt.Println("b (int inferido):", b)
    fmt.Println("c (int8, máximo):", c)
    fmt.Println("d (int64):", d)

    // Operaciones aritméticas
    fmt.Println("\n--- Operaciones aritméticas ---")
    suma := a + b
    resta := a - b
    multiplicacion := a * b
    division := b / a

    fmt.Printf("%d + %d = %d\n", a, b, suma)
    fmt.Printf("%d - %d = %d\n", a, b, resta)
    fmt.Printf("%d * %d = %d\n", a, b, multiplicacion)
    fmt.Printf("%d / %d = %d\n", b, a, division)
}
```

---

## 3. Números: Decimales (`float32`, `float64`)

Para números con parte fraccionaria se usan `float32` y `float64`.  
Por defecto, los literales decimales son `float64`.

```go
package main

import "fmt"

func main() {
    fmt.Println("=== DECIMALES ===")

    // Declaración
    var precio float32 = 99.99
    pi := 3.141592653589793 // float64 por defecto
    radio := 2.5

    fmt.Println("precio (float32):", precio)
    fmt.Println("pi (float64):", pi)
    fmt.Println("radio:", radio)

    // Operaciones
    fmt.Println("\n--- Operaciones con decimales ---")
    circunferencia := 2 * pi * radio
    area := pi * radio * radio

    fmt.Printf("Circunferencia (2*pi*%.2f) = %.4f\n", radio, circunferencia)
    fmt.Printf("Área (pi*%.2f^2) = %.4f\n", radio, area)

    // Precisión limitada de float32 vs float64
    var f32 float32 = 1.0 / 3.0
    var f64 float64 = 1.0 / 3.0
    fmt.Println("\nPrecisión:")
    fmt.Printf("float32: %.20f\n", f32)
    fmt.Printf("float64: %.20f\n", f64)
}
```

---

## 4. Booleanos (`bool`)

Los booleanos solo pueden ser `true` o `false`. Se usan principalmente en condiciones y operaciones lógicas.

```go
package main

import "fmt"

func main() {
    fmt.Println("=== BOOLEANOS ===")

    // Declaración
    verdad := true
    falso := false

    fmt.Println("verdad:", verdad)
    fmt.Println("falso:", falso)
    fmt.Printf("Tipo de verdad: %T\n", verdad)

    // Operadores lógicos
    fmt.Println("\n--- Operadores lógicos ---")
    fmt.Printf("true && false = %v\n", verdad && falso) // false
    fmt.Printf("true || false = %v\n", verdad || falso) // true
    fmt.Printf("!true = %v\n", !verdad)                  // false

    // Comparaciones que devuelven booleanos
    edad := 25
    fmt.Println("\n--- Comparaciones ---")
    fmt.Printf("edad (%d) >= 18: %v\n", edad, edad >= 18)
    fmt.Printf("edad (%d) == 30: %v\n", edad, edad == 30)
    fmt.Printf("edad (%d) != 25: %v\n", edad, edad != 25)
}
```

---

## 5. Zero Value (equivalente a `None` en Python)

En Go no existe `None`. Si declaras una variable sin inicializarla, toma un **valor cero** por defecto según su tipo.

```go
package main

import "fmt"

func main() {
    fmt.Println("=== ZERO VALUES ===")

    var texto string
    var numero int
    var decimal float64
    var booleano bool
    var puntero *int

    fmt.Printf("string: %q\n", texto)
    fmt.Printf("int: %d\n", numero)
    fmt.Printf("float64: %f\n", decimal)
    fmt.Printf("bool: %v\n", booleano)
    fmt.Printf("puntero: %v\n", puntero) // nil
}
```

---

## 🔍 Comparativa Python vs Go

| Concepto | Python | Go |
|----------|--------|-----|
| String | `str` | `string` |
| Mayúsculas | `s.upper()` | `strings.ToUpper(s)` |
| Minúsculas | `s.lower()` | `strings.ToLower(s)` |
| Longitud | `len(s)` | `len(s)` |
| Dividir | `s.split(",")` | `strings.Split(s, ",")` |
| Entero | `int` (dinámico) | `int`, `int8`, `int16`, `int32`, `int64` |
| Decimal | `float` | `float64` (doble precisión) |
| Booleano | `bool` | `bool` |
| Nulo | `None` | Zero value (`""`, `0`, `false`, `null`) |