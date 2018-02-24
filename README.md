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
  darApellido("López")
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
![02](./assets/macriwiro.jpg)
> Let's get this party started

Basta de chacharas, ahora que sabes masomenos de que va esto, vamos a ver algunos de los helpers. Para empezar a ver los helpers es importante darse cuenta de algo, cuando nosotros trabajamos con helpers de Ramda, estos, en su mayoria, reciben el parametro a modificar como último parametro, por ejemplo:

```javascript
R.contains(3, [1, 2, 3]); // -> true
```

Esto es para que cuando nosotros creemos nuestros pipelines, funciones simples o hagamos composición de funciones, podamos hacerlo sin necesidad de tener el valor todavia disponible, por ejemplo:

```javascript
const contains3 = R.contains(3);

contains3([1, 2, 3]) // -> true
contains3([1, 2]) // -> false
```

### R.pipe
#### [DEMO](http://bit.ly/2CEUIBt)
Muchas veces vamos a querer crear un pipeline de funciones como vimos mas arriba, basicamente seria ejecutar una función sobre un valor y luego pasar ese resultado a otra función (y así sucesivamente en caso de ser necesario).

```javascript
const agregarFierro = frase => `${frase} 🔫`
const agregarJapish = frase => `${frase} japish japish`

const ritmoSustanciar = R.pipe(
  agregarFierro,
  agregarJapish
)

ritmoSustanciar('Ahora sos, cabeza de tortuga') // -> Ahora sos, cabeza de tortuga 🔫 japish japish
```

### R.assocPath
#### [DEMO](http://bit.ly/2CiAqBV)
Muchas veces queremos actualizar un valor que está muy adentro en un objeto, pero tenemos que modificarlo sin mutar el objeto (React o Redux por ejemplo), la solución habitual es ir haciendo `...spread` para cada property, un bajón, `R.assocPath` al rescate.

```javascript
const bar = {
  cabina: {
    pc: {
      musica: {
        sonando: 'Despacito - Luis Fonsi'
      }
    }
  }
}

const ponerRitmo = ritmo => R.assocPath(['cabina', 'pc', 'musica', 'sonando'], ritmo, bar)

ponerRitmo('Mala fama - Aguante la tuca') // -> {"cabina": {"pc": {"musica": {"sonando": "Mala fama - Aguante la tuca"}}}}
```

> Tip: También tenés `R.assoc` si el cambio que tenes que hacer es sobre un nivel solo.

### R.project
#### [DEMO](http://bit.ly/2Ftc7zP)
Algunas veces tenemos un array de objetos, cada uno de esos objetos tiene muchas propiedades que no usamos y no queremos tenerlas, por que vamos a mandar eso al server, o vamos a iterarlo o sea cual sea el motivo, no pinta tener esas properties, thanks Gaben we have `R.project`.

```javascript
const ranchada = [{
  nombre: 'Meta Guacha',
  ritmo: 'bien piola',
  hobbie: 'puntear',
  estudios: 'de sangre'
}, {
  nombre: 'Los gedes',
  ritmo: 're gede',
  hobbie: 'geder',
  estudios: 'uno'
}]

R.project(['nombre', 'ritmo'], ranchada) // -> [{"nombre": "Meta Guacha", "ritmo": "bien piola"}, {"nombre": "Los gedes", "ritmo": "re gede"}]
```

> Tip: Si en vez de un array queres obtener solo algunas properties de un objeto, podes usar `R.pick`

### R.partition
#### [DEMO](http://bit.ly/2CJjbpm)
Algunas veces necesitamos filtrar un array por una condición, pero tambien necesitamos un listado de los elementos que no cumplen esa condición, por lo que tenemos que terminar duplicando un filter, por ejemplo, imaginando que tenemos esta lista de canciones:

```javascript
const canciones = [{
  nombre: 'Mala Fama - Desnudos',
  escuchada: true
}, {
  nombre: 'Mala Fama - Duro Duro',
  escuchada: false
}, {
  nombre: 'Mala Fama - El Corazón',
  escuchada: true
}, {
  nombre: 'Mala Fama - El Pollerudo',
  escuchada: false
}]
```
Hariamos lo siguiente:

```javascript
const escuchadas = canciones.filter(cancion => cancion.escuchada)
const noEscuchadas = canciones.filter(cancion => !cancion.escuchada)

console.log(
  'escuchadas: ', escuchadas,
  'no escuchadas: ', noEscuchadas
)
```

`R.partition` to the rescue:
```javascript
const [escuchadas, noEscuchadas] = R.partition(R.prop('escuchada'), canciones)

console.log(
  'escuchadas: ', escuchadas,
  'no escuchadas: ', noEscuchadas
)
```

De esta manera, `R.partition` va a pasar cada elemento del array a `R.prop`, que va a devolver la prop `escuchada`, de cada elemento, la cual va a ser `true` o `false`, todos los elementos que sean true, iran al primer elemento del resultado, los que de false iran al segundo, asi que usando destructuring podemos obtenerlos fácilmente 💪

### R.evolve
#### [DEMO](http://bit.ly/2HI0pSZ)
Este es uno de mis favoritos, aveces tenés un objeto y querés ejecutar una función sobre algunas properties, sin cambiar las otras y muy importante, sin mutar el objeto. Don't worry be pillo y usa `R.evolve`

```javascript
const recital = {
  ranchada: 10,
  ritmo: 9,
  sustancia: 0,
  ubicacion: {
    direccion: 'Teatro Colón',
    provincia: 'BSAS'
  }
}

const publicitar = R.evolve({
  ritmo: R.multiply(1000),
  sustancia: R.always(Infinity),
  ubicacion: {
    direccion: 'Todo lado'
  }
})

publicitar(recital) // -> {"ranchada": 10, "ritmo": 9000, "sustancia": Infinity, "ubicacion": {"direccion": "Teatro Colón", "provincia": "BSAS"}}
```

### R.where
#### [DEMO](http://bit.ly/2ETTDru)
Las validaciones son difíciles cuando son sobre un objeto en vez de un valor, `R.where` es alto fit, nos deja al igual que `R.evolve` ejecutar una función para cada valor, pero en este caso, va a devolver `true` si todas las funciones devolvieron un truthy value y false si alguna devolvio un falsy value.

```javascript
const tanBionica = {
  estilo: 'pop',
  sustancia: 0,
  ritmo: 1
}

const malaFama = {
  estilo: 'cumbia',
  sustancia: 99,
  ritmo: 10
}

const tieneRitmoYSustancia = R.where({
  estilo: R.equals('cumbia'),
  sustancia: R.lt(10),
  ritmo: R.lt(5)
})

tieneRitmoYSustancia(tanBionica) // -> false
tieneRitmoYSustancia(malaFama) // -> true
```

## 😱 Fin?
![03](./assets/malamacri.jpg)
> Este lección fue bendecida por el mala fama cheto, tu código va a ser piola, nunca perderá el ritmo y siempre tendrá sustancia de la buena (japish)

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