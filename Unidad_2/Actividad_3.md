##  Preguntas de control

---

### 1. **¿Por qué usamos un temporizador (30 ms) para validar el estado del pulsador?**

El temporizador de 30 ms se utiliza para evitar lecturas erróneas causadas por los rebotes del pulsador. Al presionar un botón, los contactos mecánicos no se cierran de forma limpia, sino que "rebotan" y generan múltiples transiciones de señal en pocos milisegundos. Este comportamiento puede interpretarse como múltiples pulsaciones si no se controla. 

El retardo de 30 ms permite filtrar estos rebotes y asegurar que la lectura del estado del botón sea estable, es decir, que realmente ha cambiado de estado y no es solo ruido mecánico.

---

### 2. **¿En qué se diferencia una MEF de tipo Moore de una Mealy y por qué en antirrebote es más común Moore?**

- **MEF de Moore**: Las salidas dependen **solo del estado actual**.
- **MEF de Mealy**: Las salidas dependen del **estado actual y de las entradas**.

En los sistemas antirrebote, es más común usar **MEF de Moore** porque proporciona un comportamiento más **estable y predecible**: las salidas cambian únicamente cuando se cambia de estado, lo que ayuda a evitar respuestas indeseadas frente a entradas inestables (como las que ocurren durante el rebote). Esto simplifica el diseño y mejora la confiabilidad.

---

### 3. **¿Cómo podrías escalar este código para manejar múltiples pulsadores en simultáneo?**

Para escalar el código y controlar varios pulsadores al mismo tiempo, se recomienda:

- Crear una **estructura (struct)** que contenga el estado actual, `lastChangeTime` y otros datos relevantes de cada pulsador.
- Usar un **array de estructuras**, uno por cada botón.
- Modificar la función de actualización del estado (ej. `actualizarMEF`) para que reciba un puntero a la estructura correspondiente al pulsador a manejar.
- Llamar a esta función dentro de un bucle que recorra el array de pulsadores.

Este enfoque permite mantener un código limpio y modular, además de facilitar la extensión para más entradas.

---

### 4. **¿Por qué es importante actualizar el `lastChangeTime` cuando se pasa a un estado de debounce?**

`lastChangeTime` registra el momento en que ocurrió un cambio de estado del pulsador. Al ingresar al estado de *debounce*, es esencial actualizar esta variable para:

- **Medir correctamente el tiempo de estabilización** del botón.
- **Evitar que el sistema reaccione antes de que el contacto se estabilice** (es decir, antes de que termine el efecto del rebote).

Si no se actualiza `lastChangeTime`, el temporizador usado para esperar los 30 ms estaría desfasado y no garantizaría una validación precisa del nuevo estado del botón. Esto comprometería la eficacia del mecanismo antirrebote.
