# Fundamentos de Go 🥇

## ⚡ La Regla de Oro de Go

**TODO programa necesita:**

package main (el paquete principal)

func main() (el punto de entrada)

**En Go, TODO el código ejecutable debe estar dentro de una función.**

Este es probablemente el concepto más importante que debes entender cuando empiezas con Go. No hay código "suelto" como en otros lenguajes.

### ❌ Lo que NO puedes hacer (pero en otros lenguajes sí)

```go
package main

import "fmt"

// 🚫 ESTO NO FUNCIONA EN GO
fmt.Println("Hola")  // ¡ERROR! Código suelto fuera de función

// 🚫 ESTO TAMPOCO FUNCIONA
nombre := "Lucas"     // ¡ERROR! Asignación fuera de función
fmt.Println(nombre)   // ¡ERROR! Código suelto

func main() {
    // El código debe estar AQUÍ dentro
}
