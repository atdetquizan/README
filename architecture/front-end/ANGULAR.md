# ANGULAR ARCHITECTURE - EXTRUCTURA
> La presente documentacion se base en el siguiente articulo. [Link de Articulo](https://medium.com/intive-developers/approach-to-clean-architecture-in-angular-applications-hands-on-35145ceadc98)

La mejor manera de mapear el principio de capa en un proyecto y la estructura de código es mediante el uso de algún tipo de paquetes y agrupar todas las clases e interfaces relacionadas con la capa. Recuerde, solo las capas superiores pueden acceder a las inferiores, no al revés. Para garantizar la interoperabilidad entre las capas, también es una buena idea especificar interfaces (por ejemplo, repositorios) en una capa muy baja

- CORE: el paquete principal agrega entidades centrales, casos de uso y definiciones de interfaz, porque están estrechamente vinculadas. Los casos de uso acceden a las entidades directamente y también tienen que trabajar con interfaces dentro de la lógica empresarial, como guardar datos en repositorios. Dado que nuestro paquete principal, de acuerdo con la regla de dependencia, no debe tener ningún conocimiento sobre la lógica del repositorio real, pero debe saber cómo llamar a una interacción con un repositorio, debe representarse como una interfaz independiente.

- DATA: esta capa se ocupa de implementar las interfaces de datos reales y vincula la lógica de ahorro de datos específica del marco (por ejemplo, guardar datos en un almacenamiento local o enviarlos a una API) a un adaptador interactiva para nuestro caso de uso especificado en la capa subyacente.

- PRESENTATION: La capa de presentación contiene la estructura angular clásica con componentes y vistas como ya los conoce. Cada componente ahora puede hacer uso de los casos de uso previamente definidos para llenar los componentes con lógica empresarial.

- CONFIGURATION: en contextos no angulares, podríamos hacer uso de una capa de configuración, que proporciona una abstracción de la inyección de dependencias y otros elementos configurables.

> Diagrama de capas
<img src="https://miro.medium.com/max/1400/1*__dVBSYpbgHwdkIB4IiM-A.png" width="100%">


### Mapeadores y objetos de datos específicos de la capa
---
Hasta ahora todo va bien, ahora sabemos cómo se puede estructurar nuestro proyecto. Una última cosa a mencionar es la importancia de los objetos de datos en toda la arquitectura: dado que nuestro paquete central abstrae la lógica empresarial a través de casos de uso y entidades, cada capa de aplicación tiene que manejarlos. Pero los objetos de datos en la capa de datos o presentación no son en su mayoría los mismos que se utilizan en la lógica empresarial. Esto se debe a que las API a menudo proporcionan más información, los repositorios de datos necesitan alguna funcionalidad de mapeo para miembros de objetos o los objetos de datos de presentación necesitan algunos campos más para mostrar los datos según sea necesario. Para hacer frente a esto, se recomienda encarecidamente hacer uso de objetos de datos específicos de la capa y mapearlos desde y hacia las entidades centrales al transferir datos de casos de uso a repositorios o capas de presentación.

>Flujo de Core, Data, Presentation
<img src="https://miro.medium.com/max/1400/1*OFGygdA-FUx_MUkQeHyLOg.png" width="100%">


## Estructura del proyecto y Ejemplo
---
La aplicación de muestra es un calendario de cumpleaños para elefantes. Se mantiene muy simple para aclarar el uso de Arquitectura limpia. Toma datos de una API o un MockRepository incluido dentro de la aplicación y muestra todos los elefantes y sus cumpleaños en una tabla.

Un proyecto Angular podría estructurarse de la siguiente manera, partiendo de la estructura conocida generada por el angular-cli.

>Estructura de proyecto
<img src="https://miro.medium.com/max/1400/1*aaz5CiK3LgRqiyMDBQgEfQ.png" width="100%">

1. __Core Layer__: Primero, echemos un vistazo a nuestra capa Core. Como sabemos, debemos definir nuestras entidades centrales, casos de uso, interfaces de repositorio y mapeadores aquí. Para mantener la arquitectura limpia y reutilizable, considere agregar herencia para los casos de uso y mapeadores.
    
    - __Core Entities__: Las entidades de esta aplicación se mantienen muy simples, por lo que un ElephantModel contiene el nombre del elefante, su estado familiar (madre, padre, bebé…) y una fecha para su cumpleaños. Toda esta información es nuestra principal necesidad de aplicación.
        >```elephant.model.ts```
        ```typescript
        export interface ElephantModel {
            name: string;
            family: string;
            birthday: Date;
        }
        ```

    - __Usecase and Mapper Base__: 
        * Una interfaz Typecript es suficiente para mantener coherente el proceso de mapeo de todas las entidades a lo largo de todo el proyecto. Tiene solo dos funciones, una para mapear desde la capa de entidad central y otra para mapear a la capa de entidad central.
            >```mapper.ts```
            ```typescript
            export interface Mapper<I, O> {
                abstract mapFrom(param: I): O;
                abstract mapTo(param: O): I;
            }
            ```
        * El caso de uso consta de una función principal, que se llama cuando ejecutamos nuestro caso de uso y devuelve un observable RxJS. También definimos un parámetro de entrada S para pasar parámetros durante la ejecución del caso de uso.
            
            >```use-case.ts```
            ```typescript
            export interface UseCase<S, T> {
                execute(params: S): Observable<T>;
            }
            ```

    - __Repository Base and ElephantRepository__: Las operaciones de lectura y escritura se manejan en esta aplicación a través de repositorios. Tenga en cuenta que solo las interfaces se especifican allí para cada repositorio y que una interfaz de repositorio no tiene que ser un repositorio real. Esto significa que estas interfaces no necesitan comunicarse con una base de datos relacional o una tienda NoSQL, sino con una API tranquila, por ejemplo. Como ya se mencionó anteriormente, el uso de interfaces de repositorio para consultar API es una combinación perfecta, porque muchas API se basan en operaciones CRUD que se pueden representar perfectamente como un repositorio. 
    
        Nuestra interfaz real para nuestra API de cumpleaños de elefantes simple proporciona consultas para buscar un elefante por su ID y enumerar todos los elefantes que están almacenados en el repositorio.

        >```elephant.repository.ts```
        ```typescript
        export abstract class ElephantRepository {
            abstract getElephantById(id: number): Observable<ElephantModel>;
            abstract getAllElephants(): Observable<ElephantModel>;
        }
        ```
        *__Nota:__ Debido a que luego usaremos esta clase como una clase base para la inyección de dependencia con Angular, nuestro repositorio debe ser una clase abstracta . Esto es causado por Typecript, que no conoce las interfaces en tiempo de ejecución y la inyección de dependencia fallará. Las interfaces en Typecript solo están presentes en la verificación de código estático, pero se eliminan durante la compilación.*

    - __Usecases__: Finalmente, echemos un vistazo al núcleo de nuestro patrón de arquitectura: los casos de uso. En nuestra aplicación de muestra, nuestros casos de uso duplican más o menos la funcionalidad del repositorio, pero agregan algún nivel de abstracción en el medio. Recuerde, debido a la regla de dependencia, los casos de uso solo pueden usar capas que son más bajas que su capa actual; en este caso, eso es muy fácil porque solo tenemos nuestra capa central y nada debajo.

        Pero, ¿nuestro caso de uso necesita saber dónde puede encontrar los datos? No necesariamente. La única forma en que el caso de uso puede comunicarse con la fuente de datos es a través de la interfaz del repositorio, que proporcionaremos como una dependencia como esta:

        >```get-elephant-by-id.usecase.ts```
        ```typescript
        @Injectable({
            providedIn: 'root'
        })
        export class GetElephantByIdUsecase implements UseCase<number, ElephantModel> {

        constructor(private elephantRepository: ElephantRepository) { }

            execute(params: number): Observable<ElephantModel> {
                return this.elephantRepository.getElephantById(params);
            }
        }
        ```

        *__Nota:__ Como lector atento, puede que se pregunte por qué hay una anotación angular en una capa central donde, en teoría, solo debería ser mecanografiado sin ninguna dependencia externa a los marcos. La razón es que Angular solo tiene esta anotación @Injectable para proporcionar un módulo mediante inyección de dependencia. Como recordará, hablamos de una cuarta capa que se llamó configuración. Si Angular tuviera una funcionalidad como Spring Boot o Dagger, entonces la capa de configuración podría encargarse de nuestra inyección de dependencia. Pero por ahora, tenemos que ceñirnos a esta solución siempre que no queramos piratear el mecanismo de inyección de dependencia.*

2. __Data Layer:__ Primero, echemos un vistazo a nuestra capa Core. Como sabemos, debemos definir nuestras entidades centrales, casos de uso, interfaces de repositorio y mapeadores aquí. Para mantener la arquitectura limpia y reutilizable, considere agregar herencia para los casos de uso y mapeadores.
    - El repositorio actual que implementa la interfaz del repositorio definida en la capa central
    - La otra entidad elefante que el repositorio necesita para administrar y trabajar con los datos a nivel de base de datos o API
    - Un mapeador que traduce los objetos de datos de la representación de la capa de datos a la representación de la entidad central y viceversa.

    Este enfoque crea algunos gastos generales a través del código duplicado y puede parecer un poco extraño al principio. Sin embargo, desacoplar las entidades de lógica empresarial, las entidades de la capa de datos y las entidades de la capa de presentación puede resultar muy útil, ya que a menudo tienen campos diferentes o adicionales provocados por su uso.

    Hay muchos escenarios en los que la capa de abstracción puede resultar útil. En nuestra aplicación, la API, por ejemplo, entrega el cumpleaños de un elefante en milisegundos, pero nuestra lógica central o estructura de datos es más conveniente y adecuada con el formato de fecha, por lo que usar una entidad para ambos podría ser problemático. Entonces, nuestro mapeador simplemente convierte los formatos de hora de un lado a otro.
    - __Ejemplo de WebElephantRepository__

        >```elephant-web-repository-mapper.ts```
        ```typescript
        export class ElephantWebRepositoryMapper implements Mapper <ElephantWebEntity, ElephantModel> {
            mapFrom(param: ElephantWebEntity): ElephantModel {
                return {
                    name: param.name,
                    family: param.family,
                    birthday: new Date(param.birthday)
                };
            }

            mapTo(param: ElephantModel): ElephantWebEntity {
                return {
                    id: 0,
                    name: param.name,
                    family: param.family,
                    birthday: param.birthday.getTime()
                };
            }
        }
        ```
        
        >```elephant-web-entity.ts```
        ```typescript
        export interface ElephantWebEntity {
            id: number;
            name: string;
            family: string;
            birthday: number;
        }
        ```
        >```elephant-web-entity.ts```
        ```typescript
        
        @Injectable({
        providedIn: 'root'
        })
        export class ElephantWebRepository extends ElephantRepository {

            mapper = new ElephantWebRepositoryMapper();

            constructor(
                private http: HttpClient
            ) {
                super();
            }

            getElephantById(id: number): Observable<ElephantModel> {
                return this.http
                .get<ElephantWebEntity>('http://5b8d40db7366ab0014a29bfa.mockapi.io/api/v1/elephants/${id}')
                .pipe(map(this.mapper.mapFrom));
            }

            getAllElephants(): Observable<ElephantModel> {
                return this.http.get<ElephantWebEntity[]>('http://5b8d40db7366ab0014a29bfa.mockapi.io/api/v1/elephants')
                    .pipe(flatMap((item) => item))
                    .pipe(map(this.mapper.mapFrom));
            }
        }
        ```
        La implementación del repositorio utiliza el cliente http estándar de Angular y mapea las entidades desde el formato API con la ayuda de la clase mapeador al formato de entidad central de nuestras aplicaciones.

3. __Capa de presentación:__ Dado que ya hemos terminado con nuestra lógica empresarial central, los casos de uso y las implementaciones del repositorio, nuestra aplicación está lista para ejecutarse; solo tenemos que demostrar que la aplicación funciona. Esto lo maneja la capa de presentación. Esta capa puede ser tan angular como desee porque solo hace uso de todas las capas subyacentes y llama a los casos de uso aquí. Dado que definimos nuestros repositorios como inyectables, nuestros casos de uso saben automáticamente dónde buscar el repositorio correcto y, además, ¡el repositorio se puede intercambiar fácilmente a través de nuestra interfaz!

    Pero, ¿cómo sabe Angular qué repositorio queremos usar? Como vimos, tenemos dos repositorios implementados en este proyecto: mock y web. Simplemente tenemos que agregar una línea al módulo donde queremos proporcionar:

    ```js
    providers: [
        { provide: ElephantRepository, useClass: ElephantMockRepository }
    ]
    ```

    El parámetro "useClass" ofrece la posibilidad de especificar cualquier clase que implemente la interfaz ElephantRepository. ¡Así que simplemente reemplace "ElephantMockRepository" por "ElephantWebRepository" y nuestra aplicación estará lista para conectarse!

    El resto de la capa de presentación es realmente sencillo: cada componente que tiene que acceder a nuestra lista de elefantes necesita inyectar el caso de uso correcto y está listo para usar la lógica de negocios como lo haría con un servicio. Teóricamente, la capa de presentación también debería tener sus propias clases de entidad para mostrar datos en la interfaz de usuario.

    __Dentro de la presentacion aplicaremos el patrom MVP (Model, View y Presenter)__

    ---
    <img src="https://admin.indepth.dev/content/images/2019/11/1_DlYzzmP-izafLzzFq7Q1Gw.png">
    
    __MVP:__ es un patrón de diseño de software arquitectónico para implementar la interfaz de usuario (UI) de una aplicación. Lo usamos para minimizar la lógica compleja en clases, funciones y módulos ( artefactos de software ) que son difíciles de probar. En particular, evitamos la complejidad en los artefactos de software específicos de la interfaz de usuario, como los componentes angulares. 

    Al igual que __MVC__, el patrón del que se deriva, __MVP__ separa la presentación del modelo de dominio. La capa de presentación reacciona a los cambios en el dominio aplicando el patrón de observador.

    En el Patrón de observadores , un sujeto mantiene una lista de observadores a los que notifica cuando ocurre un cambio de estado. ¿Te suena familiar? Lo has adivinado, RxJS se basa en el patrón de observador.

    La vista no contiene ninguna lógica o comportamiento, excepto en forma de enlaces de datos y composición de widgets. Delega el control a un presentador cuando ocurren las interacciones del usuario.

    El presentador agrupa los cambios de estado de modo que el usuario que completa un formulario da como resultado un gran cambio de estado en lugar de muchos cambios pequeños, por ejemplo, actualiza el estado de la aplicación una vez por formulario en lugar de una vez por campo. Esto facilita deshacer o rehacer cambios de estado. El presentador actualiza el estado emitiendo un comando al modelo. El cambio de estado se refleja en la vista gracias a Observer Synchronization .

    <img src="https://admin.indepth.dev/content/images/2019/11/1_fHfe7hvtW_APS2oXF5regA.png">

    - Visualicemos cómo fluyen los datos y eventos a través de una tríada __MVP__.
    
        se ha producido un cambio de estado de la aplicación en un servicio. Se notifica al componente contenedor ya que se ha suscrito a una propiedad observable en el servicio.

        El componente contenedor transforma el valor emitido en una forma que es más conveniente para el componente de presentación. 
        
        Angular asigna nuevos valores y referencias a las propiedades de entrada enlazadas en el componente de presentación.

        El componente de presentación pasa los datos actualizados al presentador, que vuelve a calcular las propiedades adicionales utilizadas en la plantilla del componente de presentación.

        Los datos han terminado de fluir hacia abajo en el árbol de componentes y Angular muestra el estado actualizado en el DOM, mostrándolo al usuario en una lista.            
    <img src="https://media3.giphy.com/media/9XXV2l09EXNx354y7f/giphy.gif">

    - Flujo de eventos que comienza con la interacción del usuario y termina en un servicio

        La interacción del usuario es interceptada por el presentador que la traduce a una estructura de datos y la emite a través de una propiedad observable. El modelo de componente de presentación observa el cambio y emite el valor a través de una propiedad de salida.

        Angular notifica al componente contenedor del valor emitido en el evento específico del componente debido a un enlace de evento en su plantilla.

        Ahora que el evento ha terminado de fluir hacia arriba en el árbol de componentes, el componente contenedor traduce la estructura de datos en argumentos que se pasan a un método en el servicio.

        Después de un comando para cambiar el estado de la aplicación, un servicio a menudo emite el cambio de estado en sus propiedades observables y los datos fluyen una vez más por el árbol de componentes como se ve en la Figura
    <img src="https://media4.giphy.com/media/3vuam15RAFMHzM7rVS/giphy.gif">

__Conclusión ([Link referencia](https://medium.com/intive-developers/approach-to-clean-architecture-in-angular-applications-hands-on-35145ceadc98))__

>Por último, se resume los beneficios e inconvenientes que la Arquitectura Limpia tiene para ofrecer:

- __Pros:__
    - __La arquitectura es "limpia" y "gritando" :__ puede ver fácilmente lo que está sucediendo en el proyecto mirando la carpeta del caso de uso.
    - __Separación de preocupaciones:__ cada capa de aplicación tiene su propia responsabilidad, por lo que la funcionalidad y la lógica del marco se mezclan.
    - __Modularidad:__ dado que toda la lógica empresarial central está representada por interfaces (o también llamadas contratos), los detalles de implementación no forman parte de la lógica empresarial real. Es por eso que los datos que se almacenan simplemente se pueden intercambiar.
    - __Portabilidad:__ el módulo principal está escrito en TypeScript puro, por lo que, en teoría, la lógica central podría transferirse a otras aplicaciones para compartir la misma base de código (por ejemplo, backend).
    - __Capacidad de prueba :__ incluso si este artículo no cubrió (todavía) las pruebas, puede imaginar que probar interfaces simuladas y casos de uso sin efectos secundarios es mucho más conveniente.
    Aplicable universalmente: Una vez que se entienden los conceptos básicos de Arquitectura Limpia, se podría aplicar a cualquier tipo de aplicación, desde backend hasta móvil o, como acabamos de ver, desarrollo web. Una vez que todos los proyectos están igualmente estructurados, es mucho más fácil incorporarse.

- __Contras:__

    - __Gastos generales:__ la separación y la modularidad se compran a expensas de más clases e interfaces.
    - __Duplicación de código:__ el uso de diferentes entidades en cada capa es automáticamente algún tipo de duplicación de código. Por un lado, el desacoplamiento es excelente para ofrecer una capa de abstracción, pero por otro lado, se introduce una nueva fuente de fallas.
    - __Curva de aprendizaje empinada:__ para los principiantes, Clean Architecture hace cosas muy diferentes a las conocidas desde el marco real.

Herramientas
- >MVP [Link](https://lostechies.com/derekgreer/2008/11/23/model-view-presenter-styles/#the-encapsulated-presenter-style)
- >Generar Mocks con [Mockapi.io](https://mockapi.io/projects/5d922373741bd40014116a69)

Referencias
- >Toda la documentacion de clean code fue sacada del ariticulo siguiente [Link Articulo](https://medium.com/intive-developers/approach-to-clean-architecture-in-angular-applications-hands-on-35145ceadc98)

- >Documentacion del patron MVP [Link Articulo](
https://indepth.dev/model-view-presenter-with-angular/)
