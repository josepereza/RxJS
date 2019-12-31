boiler : https://github.com/bradtraversy/rxjs_boiler
latest version: https://github.com/Reactive-Extensions/RxJS/blob/master/doc/libraries/main/rx.complete.md
# RxJS
Tutorial de RxJS

¿Qué es la programación reactiva?
La programación Reactiva es programación orientada al manejo de streams de datos asíncronos y la propagación del cambio. Está es un poco la frase standard, pero nos deja un poco fríos al leerla, así que vamos a tratar de explicarlo con un poquito más de detalle.

Stream
Un stream es un flujo de datos y tradicionalmente los streams han estado ligados a operaciones I/O como lectura/escritura de ficheros o querys a base de datos. En RxJs, no es muy diferente ya que sea cual sea el origen de la información, será tratada como un stream.Aqui podemos ver stream muy simple que emite los valores contenidos en un Array:

const arrayStream$ = Rx.Observable.from([10,20,30]);  
// >> 10, 20, 30
Una de las máximas en la programación reactiva es que "todo es un stream" por lo que con RxJs, cualquier flujo de información será tratada como un stream. Eventos del ratón, Arrays, rangos de números, promesas, etc. Todo será un stream.

//Una letra es un stream
const letterStream$ = Rx.Observable.of(‘A’);  
// Un rango de números es un stream
const rangeStream$ = Rx.Observable.range(1,8);  
// Los valores de un Array son un stream
const arrayStream$ = Rx.Observable.from([1,2,3,4]);  
// Valores emitidos cada 100 ms son un stream
const intervalStream$ = Rx.Observable.interval(100);  
// Cualquier tipo de evento del ratón es un stream
const clicksStream$ = Rx.Observable.fromEvent(document, 'click');  
// La respuesta a un servicio basada en una promesa es un stream
const promiseStream$ = Rx.Observable.fromPromise(fetch(’/products'));  
En RxJs los streams están representados por "secuencias observables" o simplmente Observables, por lo que en RxJs todo, absolutamente todo es un Observable, lo que logicamente nos lleva al patrón Observer.

Un poco de patrón Observer
El patrón Observer juega un papel fundamental y explica a la perfección, el concepto de reactivo. El patrón Observer define un productor de información, nuestro stream y que en RxJs está representado por una secuencia Observable o simplemente Observable y un consumidor de la misma, que sería el Observer. Como hemos visto, en RxJs el Observable es nuestro stream y que nos sirve para prácticamente todo: eventos del ratón, rangos de números, promesas, etc., pero ¿como es el Observer?
```
//Observable
const myObservable$ = Rx.Observable.from([1,2,3]);

//Observer
const myObserver = {  
  next: x => console.log(`Observer next value: ${x}`),
  error: err => console.error(`Observer error value: ${x}`),
  complete: () => console.log(`Observer complete notification`),
};

myObservable$.subscribe(myObserver); 
```
Como vemos el Observer es un objeto que dispone de 3 métodos para recibir información acerca del Observable. Cada uno de estos métodos cumple una función y a través de cada uno de ellos recibiremos distintos tipos de notificaciones del Observable.

El Observer en sí mismo no devolverá ningún valor hasta que se active la comunicación entre ambas partes. Este mecanismo es la subscripción y nada se ejecutará hasta que establezcamos una subscripción. El método subscribe activa el Observable y habilita al Observer a recibir notificaciones del stream. No obstante, es posible establecer una subscripción pasando directamente funciones como argumentos, ya que internamente RxJs asignará cada uno de estos callbacks a los respectivos métodos del Observer, siguiendo el orden en el que son pasados, siendo el primero de ellos next, el segundo error y el tercero complete.
```
const arrayStream$ = Rx.Observable.from([1,2,3]);

arrayStream$  
  .subscribe(
     next => console.log(next),
     err => console.log(err),
     () => console.log('completed!')
);   
```
El resultado en ambos casos es el mismo, no hay ninguna diferencia, pero esta forma suele ser algo más habitual ya que además no es necesario establecerlos todos.

# Next
Es el primer y más importante callback, que recibiremos en la subscripción, ya que el resto son opcionales y notifica un valor del stream emitido por el Observable.
```
Rx.Observable.from([1,2,3])  
  .subscribe(next => console.log(next));  // -> 1, 2, 3
```
# Error
Se ejecuta cuando se ha producido un error o excepción.
```
Rx.Observable.from([1,2,3])  
  .subscribe(
     next => console.log(next),
     err => console.log(err)  // Se ha producido un error
);
```
# Complete
Complete es una notificación sin valor y se emite solo cuando el stream de datos ha finalizado:
```
Rx.Observable.from([1,2,3])  
  .subscribe(
     next => console.log(next),
     err => console.log(err),
     () => console.log('completed!')  // Se ejecuta cuando finaliza el stream, después del (3)
);
```
Es importante destacar que existen streams finitos y streams inifinitos, que nunca llegan a emitir un complete. Un stream sobre un evento, click de ratón, movimiento del ratón o similares, nunca llegaran a emitir un complete.

Estos 3 métodos han sido modificados en la versión 5 de RxJs para alinearlos con los métodos de la propuesta de Observable para ECMAScript ya que en versiones anteriores estos métodos eran: onNext() onComplete() y onError() y mucha de la documentación existente hace referencia a ellos pero pertenecen a versiones anteriores de RxJs.

Pull vs Push
Dentro del patrón Observer existen diversas implementaciones donde la iniciativa sobre quien comunica a quien puede variar. En un sistema basado en pull, será el consumidor de la información (Observer) el que tome la iniciativa y le preguntará al productor de la información (Observable) si existe nueva información del stream (Oye, ¿tienes nueva información?). En un sistema basado en push será el productor (Observable) el que informe al consumidor (Observer) (¡hey yo!, tengo nueva información), de tal forma que este se limita a "reaccionar" a las notificaciones del Observable.

Un poco de patrón Iterador
Otro de los patrones en los que se inspira RxJs, es el patrón iterador , que nos permite iterar contenedores de información como, por ejemplo, un Array, sin exponer su representación interna. Para ello se define un iterador -next-, que será el encargado de recorrer el contenedor de información, manteniendo el cursor o índice con la posición del último valor dado:
```
const simpleIterator = data => {  
  let cursor = 0;
  return {
    next: () => ( cursor < data.length ? data[cursor++] : false )
  };
}

var consumer = simpleIterator(['simple', 'data', 'iterator']);  
console.log(consumer.next()); // 'simple'  
console.log(consumer.next()); // 'data'  
console.log(consumer.next()); // 'iterator'  
console.log(consumer.next()); // false  
Si examinamos un Observable por dentro vemos claramente la influencia de estos dos patrones. Aquí hay un ejemplo muy sencillo de un Observable que simplemente itera los valores de un Array y emite los valores correspondientes utilizando el next. Una vez emitidos todos, finaliza el stream con complete.

const myCustomObservable = data => (  
  Rx.Observable.create(observer => {
    for (let i = 0; i < data.length; i++) {
      observer.next(data[i]);
    }
    observer.complete();
}));

const myStream$ = myCustomObservable([1,2,3]);

myStream$  
  .subscribe(x => console.log(`next ${ x }`));
  
  ```
Un poco de programación funcional
La programación funcional, es el otro gran pilar sobre el que sea asienta RxJs y su influencia es realmente grande. La programación funcional está de moda y eso que no es especialmente nueva. Lenguajes como Haskell peina canas ya, pero como los pantalones de pitillo, todo vuelve. Para hablar en extensión de la programación funcional harían falta varios posts y esto queda lejos del objetivo de éste, pero si es importante destacar al menos las características que más afectan a su uso en Rxjs:

Las funciones son ciudadanos de primera clase
Las funciones son las reinas del baile y permiten entre otras cosas el paso de funciones como argumentos de otras funciones. En la programación funcional todo está orientado al empleo de funciones.

Funciones puras
Son funciones que con los mismos argumentos siempre devolverán el mismo resultado. Esto puede parecer una obviedad, pero si en una función dependemos de algún elemento fuera de su ámbito o scope local, como variables globales o cosas así, la función deja de ser pura ya que existen elementos externos que pueden variar su resultado. Otra característica es que no tienen efectos colaterales (side effects) ya que no mutan datos, ni tienen acceso a ningún elemento externo. Una llamada a un console.log en una función, técnicamente lo convierte en un efecto colateral y por tanto dejaría de ser una función pura.

Funciones de orden superior
Las funciones de orden superior suelen ser muy utilizadas en la programación funcional. Son funciones puras, que reciben otras funciones como argumentos para la realización de cálculos. JavaScript está considerado un lenguaje funcional impuro ya que, si bien es un lenguaje imperativo, si tiene algunos elementos de la programación funcional descritos más arriba. Las funciones de orden superior como map, reduce o filter son un buen ejemplo de ello.

Vamos a ver un ejemplo de empleo de este tipo de funciones con un Array de números de los cuales queremos obtener la suma de todos los que sean pares:
```
const source = [0,1,2,3,4,5];

const result = source  
  .filter(x => x % 2 === 0) // -> [0, 2, 4]
  .reduce((acc, cur) => acc + cur) // -> 0 + 2 + 4 

console.log(result) // OUTPUT >> 6  

```
En el ejemplo anterior vemos estas características en funcionamiento. Tanto filter como reduce son funciones de orden superior que reciben otras funciones (x % 2 === 0) y que además devuelven un nuevo Array manteniendo inalterado el original. Dado que source es una constante si alguna de estas funciones mutara su valor, tendríamos un error.

RxJs tiene un fuerte componente de programación funcional y si bien no es necesario ser un total experto en programación funcional, si es importante tener claro al menos estos elementos. En RxJs vamos a trabajar de forma muy similar con un pipeline de funciones que recibirán (y devolverán) un Observable y que nunca mutarán el stream original.

****************************************

# Programación reactiva con RxJS
## rxjs

RxJS es una librería que implementa ReactiveX en javascript, permitiendo hacer uso de la programación reactiva en esta plataforma.

Gracias al uso de observables basados en eventos se facilita en gran medida la programación asíncrona, ofreciendo grandes ventajas respecto al uso de callback y Promise.

El funcionamiento básico de los observables se basa en un objecto llamado subject que contiene una lista de dependencias llamadas observers a las que notifica cuando se produce algún cambio, llamando a uno de sus métodos, de forma que reaccionen a ese cambio.

Los observables generan flujos de datos que se generan valores en diferentes momentos, y los observers subscritos a esos cambios reaccionan a los datos que se envían.

Estos son los conceptos básicos utilizados para este patrón:

    Observable: el objecto a cuyos cambios de estado se quieren subscribir otros objetos.
    Observer: el objeto que desea ser notificado cuando se producen cambios en el estado de algún observable.
    Subscription: se produce cuando los observers se conectan con los observables para recibir los valores que estos emiten. Estas subscripciones se pueden cancelar posteriormente.
    Subject: actúa como observer y observable a la vez. Puede emitir valores y subscribirse a observables.
    Operadores: permiten realizar operaciones con los datos enviados por el observer o el subject antes de ser enviados al observable. Se pueden encadenar.

En este ejemplo se crea un observable mediante el operador create.

import { Observable } from 'rxjs';

const observable = Observable.create((observer) => {
  observer.next('Hola');
  observer.next('mundo');
});

const subscripcion = observable.subscribe({
  next: x => console.log('Dato emitido: ' + x),
  error: err => console.error('Error: ' + err),
  complete: () => console.log('Terminado'),
});

El método subscribe del observable puede contener un único parámetro, que sería la función que se ejecuta cada vez que el observable emite un valor, o como en este ejemplo, contener tres funciones, la primera para cuando se emite un valor, la segunda en caso de error y la última para cuando se emite el último valor, en caso de no haberse producido ningún error antes

En este ejemplo se crea el observable a partir de un evento. input$ representa el flujo de datos que proviene de cualquier interacción que haga el usuario con elementoHtml.

const elementoHtml = document.querySelector('input[type=text]');

const input$ = Rx.Observable.fromEvent(elemento, 'input');

const subscripcion = input$.subscribe({
  next: event => console.log(`Has escrito: ${event.target.value}`),
  error: err => console.log(`Error: ${err}`),
  complete: () => console.log(`Completado`),
});

Mediante Subject, además de crearse la subscripción, es desde el propio Subject desde donde se emiten los valores.

let subject = new Rx.Subject();

var subscription = subject.subscribe(
    x => console.log('onNext: ' + x),
    e => console.log('onError: ' + e),
    () => console.log('onCompleted'));

subject.onNext(1);
// => onNext: 1

subject.onNext(2);
// => onNext: 2

subject.onCompleted();
// => onCompleted

subscription.dispose();

Existen varias versiones de Subject que modifican ligeramente su comportamiento habitual:

    AsyncSubject: tan solo emite el último valor producido desde el observable en el momento en que este se ha completado.
    BehaviorSubject: comienza emitiendo el último valor que se ha producido. Al crearlo se le puede pasar un valor por defecto, que será el que se emita al crear una subscripción en caso de no haberse emitido aún ningún valor.
    ReplaySubject: cuando se crea una subscripción a este tipo de subject se emiten todos los valores que se han producido hasta el momento, incluyendo los que se produjeron antes de crearse la subscripción

# Operadores

Una de las herramientas más importantes con las que cuenta RxJS es el uso de operadores.

Existen distintos tipos de operadores entre los que destacan los siguientes:

    Creación: se utilizan para crear observables a partir de otros elementos, como pueden ser arrays, eventos o promesas (from,fromEvent, of).
    Filtro: de entre los valores emitidos por los observables, filtran los que cumplen determinadas características y descartan el resto (debounceTime, distinctUntilChanged, filter, take, takeUntil).
    Transformación: se encargan de modificar los datos transmitidos a traves del flujo de datos producido desde los observables (bufferTime, concatMap, map, mergeMap, scan, switchMap).
    Gestión de errores: gestionan el comportamiento de la aplicación cuando se produce algún error. Permiten reintentar la operación que ha producido el error (catchError, retry, retryWhen).
    Combinación: se utilizan para juntar la información emitida desde distintos observables (combineLatest, concat, merge, startWith , withLatestFrom, zip).
    Utilidades: proporcionan utilidades básicas a los observables (tap, delay, toPromise).

Estos son alguno de los operadores más comunes:

   * create: se utiliza principalmente para pruebas. Se crea un observable pasando como parámetro una función con un observer.
   * from: crea un observable a partir de un arrays, una cadenas, Promises o iterables.
   * of: convierte los argumentos que se pasan a la función en valores que emite el observable creado y emite una notificación de completado al terminar.
   * interval: crea un observable que emite valores numéricos según el intervalo que se indique al crearlo.
   * timer: similar a interval, añade un primer parámetro que indica la pausa inicial antes de emitir un número. Si contiene un segundo parámetro, este se utiliza como interval, para generar valores periódicamente.
    toPromise: convierte un obserbable en una Promise.
    combineLatest: toma como parámetros distintos observables y emite datos cada vez que uno de ellos genera datos nuevos, combinando estos últimos datos con los más recientes del resto de observables que contiene.
    concat: según el orden en que se pasen los observables como parámetros, los ejecuta en orden, no pasando al siguiente hasta que no se ha completado el actual.
    merge: permite juntar dos flujos de datos en uno, ejecutándose simultáneamente.
    zip: espera a que todos los observables que lo componen emitan algún valor antes de generar un array combinandolos todos.
    filter: emite solo los valores que cumple una cierta condición.
    take: emite un número máximo de valores antes de completar el observable.
    skip: se salta el número indicado de los primero valores generados por el observable.
    delay: retrasa la emisión de valores durante el tiempo indicado.
    debounceTime: descarta los valores emitidos entre un cierto intervalo de tiempo. Si se le pasa por parámetro 1 segundo, no emitirá más de un valor por segundo.
    bufferTime: almacena todos los valores generados y los emite como array en el periodo indicado en el parámetro.
    map: permite aplicar cambios a cada uno de los valores emitidos.
    mergeMap: combina dos observables de forma que cada vez que el interno emite un valor, lo combina con el valor del observable externo.
    switchMap: combina dos observables reiniciando el observable interno cada vez que se produce un nuevo valor del externo.
    catchError: captura los errores del observable.
    retry: reintenta ejecutar el observable el número de veces indicado. Muy útil cuando es un error que puede ser temporal, como al ejecutar una petición ajax que puede fallar por falta de conexión.
    tap: permite efectuar acciones sobre los datos emitidos sin transformarlos.

Los operadores se pueden encadenar mediante el uso de la función pipe() de la siguiente forma:

import { filter, map } from 'rxjs/operators';

const numeros = of(1, 2, 3, 4, 5)
  .pipe(
    filter(n => n < 3),
    map(n => n * 2)
  );

numeros.subscribe(x => console.log(x));

