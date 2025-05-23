# Preguntas sobre Lenguajes de Programación en Sistemas Embebidos

## 1. ¿Cuáles son los lenguajes en los que se puede programar sistemas embebidos?

A continuación, se listan algunos lenguajes comúnmente utilizados para programar sistemas embebidos:

- C
- C++
- Assembly
- Python (especialmente con MicroPython o CircuitPython)
- Rust
- Ada
- Lua (usado en sistemas embebidos ligeros)
- Java (limitado, más común en sistemas embebidos complejos como Android)
- Embedded C++
- Go (en algunos sistemas con suficiente capacidad)
- JavaScript (a través de entornos como Espruino)

---

## 2. ¿Qué ventajas y desventajas tienen dichos lenguajes comparados con C?

| Lenguaje       | Ventajas sobre C                                                                 | Desventajas sobre C                                                                 |
|----------------|-----------------------------------------------------------------------------------|--------------------------------------------------------------------------------------|
| **C++**        | Programación orientada a objetos, reutilización de código                         | Mayor consumo de recursos, compilación más lenta                                    |
| **Assembly**   | Control total del hardware, muy eficiente                                         | Muy difícil de mantener, dependiente de la arquitectura                             |
| **Python**     | Fácil de aprender, código más legible y rápido de escribir                        | Requiere más recursos (RAM/CPU), no apto para tiempo real estricto                 |
| **Rust**       | Seguridad en memoria sin garbage collector, buen rendimiento                      | Curva de aprendizaje más pronunciada, ecosistema más pequeño                        |
| **Ada**        | Alta confiabilidad, usado en sistemas críticos                                    | Menos popular, menos soporte comunitario                                            |
| **Lua**        | Ligero, fácil de integrar                                                         | Limitado para tareas complejas o de tiempo real                                     |
| **Java**       | Portabilidad, herramientas robustas                                               | Alto consumo de memoria y procesamiento                                             |
| **Go**         | Concurrencia moderna, sintaxis sencilla                                           | Aún emergente en sistemas embebidos, no tan optimizado para recursos limitados      |
| **JavaScript** | Familiaridad para desarrolladores web, fácil de usar con ciertos entornos         | Muy limitado a plataformas específicas como Espruino, no adecuado para bajo nivel   |

---

## 3. ¿Existe un ranking de lenguajes para sistemas embebidos?

Sí, existe un ranking actualizado anualmente por la revista **Embedded.com**. Puedes consultarlo en el siguiente enlace:

🔗 [https://www.embedded.com/survey-languages-used-in-embedded-systems/](https://www.embedded.com/survey-languages-used-in-embedded-systems/)

### Percepciones sobre el ranking

El ranking muestra que **C sigue siendo el lenguaje dominante** en el desarrollo de sistemas embebidos, debido a su eficiencia, control de hardware y bajo consumo de recursos. **C++** ocupa el segundo lugar, siendo usado en sistemas más complejos que requieren características orientadas a objetos. **Python** ha ganado terreno, especialmente en el prototipado rápido con placas como Raspberry Pi o ESP32, pero aún no es común en aplicaciones industriales críticas debido a su alto consumo de recursos. **Rust** está emergiendo como una alternativa segura y eficiente, aunque su adopción es todavía limitada.

Este tipo de rankings es útil para entender las tendencias del sector y evaluar qué lenguajes vale la pena aprender según el tipo de proyectos embebidos que se quieran desarrollar.

# Macros en C para Sistemas Embebidos

Este documento contiene una colección de macros útiles para manipular registros en sistemas embebidos, especialmente al trabajar con microcontroladores. Estas macros permiten establecer, verificar, alternar y escribir valores en registros de manera eficiente y legible.

---

## 1.  Aplicar una máscara para escribir en un registro

```c
#define SET_BITS(REG, MASK) ((REG) |= (MASK))
```
### Descripción:
Esta macro permite activar (poner en `1`) los bits especificados en la máscara sin afectar los demás bits del registro.

### Ejemplo de uso:

```c
#define PORTA (*(volatile uint8_t*)0x40049000)
SET_BITS(PORTA, 0x05); // Activa los bits 0 y 2 del puerto A
```

## 2. Verificar si un periférico está habilitado
```c
#define IS_PERIPHERAL_ENABLED(PCC_REG, BIT_POS) (((PCC_REG) & (1 << (BIT_POS))) ? 1 : 0)
```

### Descripción:
Permite verificar si un periférico (como un puerto) está habilitado al consultar su bit correspondiente en un registro de control como el PCC.

### Ejemplo de uso:
```c
#define PCC (*(volatile uint32_t*)0x40065000)
int estado = IS_PERIPHERAL_ENABLED(PCC, 13); // Verifica si el periférico en el bit 13 está habilitado
```
## 3. Alternar (toggle) un bit en un registro
```c
#define TOGGLE_BIT(REG, BIT_POS) ((REG) ^= (1 << (BIT_POS)))
```

### Descripción:
Cambia el estado de un bit específico: si estaba en 1 pasa a 0, y viceversa.

### Ejemplo de uso:
```c
TOGGLE_BIT(PORTB, 5); // Alterna el bit 5 del registro PORTB
```
## 4.  Macro adicional: Establecer un campo de bits
```c
#define WRITE_FIELD(REG, MASK, VALUE) ((REG) = ((REG) & ~(MASK)) | ((VALUE) & (MASK)))
```
### Descripción:
Limpia los bits especificados por la máscara en el registro y luego escribe el nuevo valor deseado en ese campo.

### Ejemplo de uso:
```c
#define CONFIG (*(volatile uint32_t*)0x40048000)
WRITE_FIELD(CONFIG, 0x3F, 0x20); // Escribe 0x20 en el campo de 6 bits (0-5) de CONFIG
```

# Análisis de errores y corrección en código C

Este documento analiza y corrige los errores presentes en un programa escrito en lenguaje C. Se identifican problemas comunes como bucles infinitos, accesos inválidos a memoria y condiciones mal formuladas.

---

## Código original

```c
#include <stdio.h>

int main() {
    int i;
    int num = 10;
    int array[5] = {1, 2, 3, 4, 5};
    int contador = 0;

    for (i = 1; i < 10; i--) {
        printf("Valor de i: %d\n", i);
    }

    for (i = 0; i <= 5; i++) {
        printf("Elemento del array: %d\n", array[i]);
    }

    while (num != 0) {
        printf("Valor de num: %d\n", num);
        num = num + 1;  
    }

    while (contador < 5) {
        printf("Valor de contador: %d\n", contador);
    }

    return 0;
}
```

## Errores identificados y soluciones
### 1. Bucle infinito en el primer for
```c
for (i = 1; i < 10; i--) {
```

**Problema**: i se decrementa mientras la condición i < 10 siempre es verdadera → bucle infinito.

**Solución**:

```c
for (i = 1; i < 10; i++) {
```
### 2. Acceso fuera de los límites del array
```c
for (i = 0; i <= 5; i++) {
    printf("Elemento del array: %d\n", array[i]);
}
```

**Problema**: El array tiene 5 elementos (0 a 4), pero se accede a array[5] → acceso inválido.

**Solución**:

```c
for (i = 0; i < 5; i++) {
    printf("Elemento del array: %d\n", array[i]);
}
```
### 3. Bucle infinito en while (num != 0)
```c
while (num != 0) {
    printf("Valor de num: %d\n", num);
    num = num + 1;
}
```
**Problema**: num incrementa indefinidamente, nunca llega a 0.

**Solución**:

```c
while (num != 0) {
    printf("Valor de num: %d\n", num);
    num = num - 1;
}
```
### 4. Bucle infinito en while (contador < 5)
```c
while (contador < 5) {
    printf("Valor de contador: %d\n", contador);
}
```
**Problema**: contador no se incrementa → condición siempre verdadera.

**Solución**:

```c
while (contador < 5) {
    printf("Valor de contador: %d\n", contador);
    contador++;
}
```
## Código corregido completo
```c
#include <stdio.h>

int main() {
    int i;
    int num = 10;
    int array[5] = {1, 2, 3, 4, 5};
    int contador = 0;

    for (i = 1; i < 10; i++) {
        printf("Valor de i: %d\n", i);
    }

    for (i = 0; i < 5; i++) {
        printf("Elemento del array: %d\n", array[i]);
    }

    while (num != 0) {
        printf("Valor de num: %d\n", num);
        num = num - 1;
    }

    while (contador < 5) {
        printf("Valor de contador: %d\n", contador);
        contador++;
    }

    return 0;
}
```
