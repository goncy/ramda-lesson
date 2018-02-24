# Lección: Remando con Ramda
Primero que nada, Hola mundo!
Me llamo **Gonzalo Pozzo** y en el momento de escribir esto era Frontend Developer en [The Next Ad](https://www.thenextad.com/) (De no ser así, hola desde el pasado!).
Hoy les vengo a hablar de una de las librerias que más uso y más me gusta, [Ramda](http://ramdajs.com/).

## Intro
> _A practical functional library for JavaScript programmers._

### Por que Ramda?
Vamos a hacer items para simplificarlo, devs loves lists:
* Nunca muta datos (excelente para usar con librerias como React o Redux)
* Sin side-effects (siempre que reciban los mismos parametros, van a devolver el mismo resultado)
* Currying automatico (vamos a hablar de esto un poco mas abajo)
* Preparado para armar pipelines funcionales (también vamos a ver esto un poco mas abajo)

### Que es el currying?
Vamos a intentar de explicar esto con ejemplos, imaginemos que tenemos una función en JavaScript que suma 2 numeros (a, b), de la siguiente manera:

```javascript
const suma = (a, b) => a + b
```

* Que pasaría si yo hago `suma(1)`?
* Que pasaría si yo hago `suma(1, 2)`?

```javascript
suma(1) // -> NaN
suma(1, 2) // -> 3
```

[DEMO](http://bit.ly/2GG1HMZ)

Ahora vamos a implementar la misma función usando un helper de Ramda, específicamente, el helper `curry`, el cual nos permite crear nuestras propias funciones curriadas (esta palabra suena horrible, acostumbrate).

```javascript
const suma = R.curry((a, b) => a + b)
```

* Que pasaría si yo hago `suma(1)`?
* Que pasaría si yo hago `suma(1, 2)`?

```javascript
suma(1) // -> fn(n)
suma(1, 2) // -> 3
```

![01](./assets/baiabaia.jpg)

Pos que pasó ahi mijo?, primero veamos, a nivel implementación lo único que tuvimos que hacer, fue envolver el cuerpo de nuestra función con `R.curry`, pero a nivel funcionalidad pasó otra cosa, ahora `suma(1)` devuelve una función, vamos a ver como usar esa función:

```javascript
const suma = R.curry((a, b) => a + b)
const suma1 = suma(1)

suma1(2) // -> 3
suma1(3) // -> 4
```

[DEMO](http://bit.ly/2EPAriv)

> La función solo va a retornar un valor una vez que haya recibido todos los parametros que esperaba, mientras tanto, retornará una nueva función con los parametros anteriores implicitos.

That's all folks, eso es currying.

### Que es eso de preparado para armar pipelines funcionales?
Ahora que sabemos que es el currying y que todas las funciones de Ramda son curriadas, vamos a ver el uso de un helper, no importa si todavía no lo entendés, vamos a profundizar mas adelante.

Imaginemos una función super útil, que recibe un string de un nombre y apellido y le cambia el apellido por López (algo que hacemos todos los días, si no lo hacen no se de que trabajan).

Como somos cracks, ya sabemos como funciona el helper `R.curry` así que vamos a usarlo para un helper para ver como se utiliza en un caso real:

```javascript
const darApellido = R.curry((apellido, nombre) => `${nombre} ${apellido}`)
```

Así podriamos implementarla en JavaScript plano:

```javascript
function lopizadorVanilla(fullName) {
  return fullName.split(" ")[0] + " López"
}

lopizadorVanilla("Gonzalo Pozzo") // -> Gonzalo López
```

Y asi con Ramda

```javascript
const lopizadorRamda = R.pipe(
  R.split(" "),
  R.head,
  darApellido("López")// Podriamos usar R.invoker(1, "concat")(" López") pero let's keep it simple
)

lopizadorRamda("Gonzalo Pozzo") // -> Gonzalo López
```

Como podemos ver el resultado es el mismo, pero vamos a agregar algo de complejidad, no queremos un Charly López, por que no pinta, queremos que sea García.

```javascript
// JavaScript Plano
function lopizadorVanilla(fullName) {
  const nombre = fullName.split(" ")[0]
  return nombre === "Charly"
    ? darApellido("García", nombre)
    : darApellido("López", nombre)
}

lopizadorVanilla("Charly Pozzo") // -> Charly García
```

```javascript
// Ramda
const lopizadorRamda = R.pipe(
  R.split(" "),
  R.head,
  R.ifElse(
    R.equals("Charly"),
    darApellido("García"),
    darApellido("Lopez")
  )
)

lopizadorRamda("Charly Pozzo") // -> Charly García
```

[DEMO](http://bit.ly/2sSY7wI)

Bueno, cuál fue la diferencia al tener que cambiar la implementación de ambas funciones?, en la función de JavaScript plano tuvimos que cambiar la estructura del cuerpo de la función, mientras que en Ramda solo agregamos un "paso" al pipeline, ahora imaginá esto en un caso real, que preferís cambiar el cuerpo de una función compleja o agregar un paso (que hasta puede ser reutilizable en otras partes de la app) y solo meterlo al pipeline?.

## Helpers
Basta de chacharas, ahora que sabes masomenos de que va esto, vamos a ver algunos de los helpers

## 😱 Fin?
Bueno, hasta aquí llego mi amor, pero eso no significa que el tuyo también, la docu de Ramda es muy larga y hay +240 helpers! 🤯, así que imaginate todo lo que se puede hacer, se creativo, innova, mete magia, ritmo y sustancia y compartí lo que hagas. Cualquier duda preguntá y nunca, nunca dejes de aprender! (We re filosófico el Goncy)

## Docs
* 📚 [Ramda API Docs](http://ramdajs.com/docs/)
* ✏️ [Ramda REPL](http://ramdajs.com/repl?v=0.25.0)

## 📚 Más lecciones
* [Recompose](https://github.com/goncy/recompose-lesson)
* [Cypress](https://github.com/goncy/cypress-lesson)
---
*Si encontras un error, typo, cagada, moco o calificativo negativo, avisame o haceme un PR, gracias!*

**by [@goncy](http://github.com/goncy)**