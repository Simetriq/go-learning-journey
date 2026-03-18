# 04 - Structs y Punteros en Go

> **Objetivo:** Definir tipos compuestos con structs, agregarles métodos y comprender el uso de punteros para modificar datos y evitar copias innecesarias.

---

## Tabla de Contenidos

1. [Structs: definición y uso](#structs-definición-y-uso)
2. [Métodos sobre structs](#métodos-sobre-structs)
3. [Structs anidados y composición](#structs-anidados-y-composición)
4. [Punteros](#punteros)
5. [Punteros a structs](#punteros-a-structs)
6. [Métodos con receptor puntero](#métodos-con-receptor-puntero)
7. [Interfaces básicas (preview)](#interfaces-básicas-preview)
8. [Ejercicios propuestos](#ejercicios-propuestos)

---

## Structs: definición y uso

Un **struct** agrupa campos de distintos tipos bajo un mismo nombre, similar a las clases en otros lenguajes (pero sin herencia).

```go
package main

import "fmt"

// Definición del tipo (convención: PascalCase para exportados)
type Persona struct {
    Nombre string
    Edad   int
    Email  string
}

func main() {
    // 1. Literal con campos nombrados (recomendado)
    p1 := Persona{
        Nombre: "Ana García",
        Edad:   29,
        Email:  "ana@ejemplo.com",
    }

    // 2. Literal posicional (frágil, evitar en producción)
    p2 := Persona{"Bruno López", 34, "bruno@ejemplo.com"}

    // 3. Con new() — retorna un puntero
    p3 := new(Persona)
    p3.Nombre = "Carlos Ruiz"
    p3.Edad = 45

    // Acceso a campos con punto
    fmt.Println(p1.Nombre, "tiene", p1.Edad, "años")
    fmt.Println(p2)
    fmt.Printf("p3: %+v\n", *p3) // %+v muestra nombre=valor
}
```

### Valor cero de un struct

```go
package main

import "fmt"

type Punto struct {
    X, Y float64 // múltiples campos del mismo tipo en una línea
}

func main() {
    var p Punto // X=0, Y=0
    fmt.Println("Punto cero:", p)

    // Los structs se comparan con ==  si todos sus campos son comparables
    p2 := Punto{0, 0}
    fmt.Println("Son iguales:", p == p2) // true
}
```

---

## Métodos sobre structs

En Go, los métodos se definen **fuera** del tipo, usando un **receptor**:

```go
package main

import (
    "fmt"
    "math"
)

type Circulo struct {
    Radio float64
}

type Rectangulo struct {
    Ancho, Alto float64
}

// Método con receptor por VALOR (no modifica el original)
func (c Circulo) Area() float64 {
    return math.Pi * c.Radio * c.Radio
}

func (c Circulo) Perimetro() float64 {
    return 2 * math.Pi * c.Radio
}

func (r Rectangulo) Area() float64 {
    return r.Ancho * r.Alto
}

func (r Rectangulo) Perimetro() float64 {
    return 2 * (r.Ancho + r.Alto)
}

func main() {
    c := Circulo{Radio: 5}
    r := Rectangulo{Ancho: 4, Alto: 6}

    fmt.Printf("Círculo  — Área: %.2f, Perímetro: %.2f\n", c.Area(), c.Perimetro())
    fmt.Printf("Rectángulo — Área: %.2f, Perímetro: %.2f\n", r.Area(), r.Perimetro())
}
```

---

## Structs anidados y composición

```go
package main

import "fmt"

type Direccion struct {
    Calle    string
    Ciudad   string
    Provincia string
}

type Empleado struct {
    Nombre    string
    Salario   float64
    Direccion Direccion // struct anidado
}

// Embedding (composición): promueve los campos del tipo embebido
type Gerente struct {
    Empleado          // campos de Empleado disponibles directamente
    Departamento string
}

func main() {
    emp := Empleado{
        Nombre:  "Diana Pérez",
        Salario: 85000,
        Direccion: Direccion{
            Calle:     "Av. Corrientes 1234",
            Ciudad:    "Buenos Aires",
            Provincia: "CABA",
        },
    }

    // Acceso anidado
    fmt.Println(emp.Nombre, "vive en", emp.Direccion.Ciudad)

    ger := Gerente{
        Empleado:     emp,
        Departamento: "Ingeniería",
    }

    // Gracias al embedding, accedemos directamente
    fmt.Println(ger.Nombre, "—", ger.Departamento) // sin ger.Empleado.Nombre
    fmt.Println("Ciudad:", ger.Ciudad)              // promovido desde Direccion
}
```

---

## Punteros

Un **puntero** almacena la **dirección de memoria** de un valor, en lugar del valor mismo.

```go
package main

import "fmt"

func main() {
    x := 42

    // & obtiene la dirección de memoria
    ptr := &x
    fmt.Println("Valor de x:", x)
    fmt.Println("Dirección de x:", ptr)   // algo como 0xc0000b4010
    fmt.Println("Valor via ptr:", *ptr)   // * desreferencia el puntero

    // Modificar a través del puntero
    *ptr = 100
    fmt.Println("x después:", x) // 100 ← x cambió!

    // Punteros y funciones
    duplicar := func(n *int) {
        *n = *n * 2
    }
    duplicar(&x)
    fmt.Println("x duplicado:", x) // 200
}
```

---

## Punteros a structs

```go
package main

import "fmt"

type Contador struct {
    Valor int
    Nombre string
}

// Sin puntero: modifica una COPIA, el original no cambia
func incrementarCopia(c Contador) {
    c.Valor++
}

// Con puntero: modifica el ORIGINAL
func incrementarOriginal(c *Contador) {
    c.Valor++ // Go permite c.Valor en vez de (*c).Valor — azúcar sintáctico
}

func main() {
    c := Contador{Valor: 0, Nombre: "visitas"}

    incrementarCopia(c)
    fmt.Println("Tras copia:", c.Valor) // 0 — sin cambios

    incrementarOriginal(&c)
    fmt.Println("Tras puntero:", c.Valor) // 1 ✓

    // new() crea un puntero a un struct inicializado en cero
    c2 := new(Contador)
    c2.Nombre = "descargas"
    incrementarOriginal(c2)
    fmt.Println("c2:", c2.Valor)
}
```

---

## Métodos con receptor puntero

Usa receptor puntero cuando:
1. Necesitas **modificar** el receptor.
2. El struct es **grande** (evitar copias costosas).

```go
package main

import "fmt"

type CuentaBancaria struct {
    Titular string
    Saldo   float64
}

// Receptor puntero: modifica el saldo real
func (c *CuentaBancaria) Depositar(monto float64) {
    if monto > 0 {
        c.Saldo += monto
    }
}

func (c *CuentaBancaria) Retirar(monto float64) error {
    if monto > c.Saldo {
        return fmt.Errorf("saldo insuficiente: tiene %.2f, solicitó %.2f", c.Saldo, monto)
    }
    c.Saldo -= monto
    return nil
}

// Receptor valor: solo lectura, no necesita modificar
func (c CuentaBancaria) String() string {
    return fmt.Sprintf("Cuenta[%s]: $%.2f", c.Titular, c.Saldo)
}

func main() {
    cuenta := &CuentaBancaria{Titular: "Elena Rodríguez", Saldo: 1000}

    cuenta.Depositar(500)
    fmt.Println(cuenta)

    if err := cuenta.Retirar(200); err != nil {
        fmt.Println("Error:", err)
    }
    fmt.Println(cuenta)

    if err := cuenta.Retirar(2000); err != nil {
        fmt.Println("Error:", err)
    }
}
```

---

## Interfaces básicas (preview)

Las interfaces en Go se satisfacen de forma **implícita** — si un tipo tiene los métodos requeridos, implementa la interfaz:

```go
package main

import (
    "fmt"
    "math"
)

// Interfaz: cualquier tipo que tenga Area() float64
type Figura interface {
    Area() float64
}

type Circulo struct{ Radio float64 }
type Cuadrado struct{ Lado float64 }

func (c Circulo) Area() float64   { return math.Pi * c.Radio * c.Radio }
func (c Cuadrado) Area() float64  { return c.Lado * c.Lado }

func imprimirArea(f Figura) {
    fmt.Printf("Área: %.2f\n", f.Area())
}

func main() {
    figuras := []Figura{
        Circulo{Radio: 3},
        Cuadrado{Lado: 4},
        Circulo{Radio: 1.5},
    }

    for _, f := range figuras {
        imprimirArea(f)
    }
}
```

---

## Ejercicios propuestos

### Ejercicio 1 — Lista enlazada simple
Define un struct `Nodo` con un campo `Valor int` y un puntero `Siguiente *Nodo`. Escribe funciones para:
- `insertar(lista **Nodo, valor int)` — inserta al inicio
- `imprimir(lista *Nodo)` — imprime todos los valores

### Ejercicio 2 — Biblioteca de libros
Crea un struct `Libro` con campos `Titulo`, `Autor`, `Año` y `Disponible bool`. Luego:
- Crea un slice de 5 libros.
- Escribe un método `Prestar()` que marque el libro como no disponible (retorna error si ya estaba prestado).
- Imprime los libros disponibles.

### Ejercicio 3 — Sistema de votación
Define un struct `Candidato` con `Nombre` y `Votos int`. Escribe una función `votar(c *Candidato)` y un programa que simule 10 votos distribuidos entre 3 candidatos, imprimiendo al ganador.

### Ejercicio 4 — Punto en 3D
Crea un struct `Punto3D {X, Y, Z float64}` y agrega métodos:
- `DistanciaAlOrigen() float64`
- `Trasladar(dx, dy, dz float64)` (receptor puntero)
- `String() string` para impresión legible
