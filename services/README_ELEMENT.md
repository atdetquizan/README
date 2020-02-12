# SERVICES

## ELEMENT
funciones para todo lo que se relaciona a elementos HTMLL

```
@Injectable({ providedIn: 'root' })
export class ElementService {    
  /**
   * Creates an instance of element service.
   */
  constructor() { }
  /**
   * Intersections visible
   * @param element 
   * @returns Observable<any> 
   */
  intersectionVisible(element: any): Observable<any> {
      //
  }
}
```
> intersectionVisible(element: any): Observable<any> 
```
/**
 * return Observable
 */
 return new Observable((subscriber) => {
    /**
    * El objeto intersectionObserverOptions pasado al constructor `IntersectionObserver()`
    * le deja controlar  las circunstancias bajo las cuales la función callback del
    * observer es invocada
    */
    const intersectionObserverOptions = {
        root: null,
        rootMargin: '150px',
        threshold: 1.0
    };
    /**
    * Declare class IntersectionObserver
    */
    const observer = new IntersectionObserver((entries: any) => {
        entries.forEach(entry => {
            /**
            * Are we in viewport?
            */
            if (entry.intersectionRatio > 0) {
            /**
                * subscriber
                */
            subscriber.next(entry.intersectionRatio);
            /**
                * Stop watching
                */
            observer.unobserve(element);
            }
        });
    }, intersectionObserverOptions);
    /**
    * Una vez usted ha creado el observer, necesita darle un elemento target para observar:
    */
    observer.observe(element);
});
```
### IntersectionObserver

Se implmento una funcion utilizando `IntersectionObserver` para detectar cuando un elemento esta visible en la pantalla haciendo scroll.

- `IntersectionObserver`: El API Intersection Observer le permite configurar una función callback que es llamada cada vez que un elemento, llamado target, intersecta con otro, bien el dispositivo viewport o un elemento específico.

El grado de intersección entre el elemento target y su elemento root es el intersection ratio.

### Opciones
- `root`: El elemento que es usado como viewport para comprobar la visibilidad de elemento target. Debe ser un elemento ascendiente del target. Por defecto se toma el viewport del navegador si no se especifica o si se especifica como null.
- `rootMargin`: Margen alrededor del elemeto root. Puede tener valores similares a los de CSS margin property, e.g. "10px 20px 30px 40px" (top, right, bottom, left). Los valores pueden ser porcentajes. Este conjunto de valores sirve para aumentar o encoger cada lado del cuadro delimitador del elemento root antes de calcular las intersecciones. Por defecto son todos cero.
- `threshold`: Es un número o un array de números que indican a que porcentaje de visibilidad del elemento target, la función callback del observer debería ser ejecutada. Si usted quiere que se detecte cuando la visibilidad pasa la marca del 50%, debería usar un valor de 0.5. Si quiere ejecutar la función callback cada vez que la visibilidad pase otro 25%, usted debería especificar el array [0, 0.25, 0.5, 0.75, 1]. El valor por defecto es 0 (lo que significa que tan pronto como un píxel sea visible, la función callback será ejecutada). Un valor de 1.0 significa que el umbral no se considera pasado hasta que todos los pixels son visibles.

### Métodos
- `IntersectionObserver.disconnect()`: Detiene el IntersectionObserverobjeto de observar cualquier objetivo.
- `IntersectionObserver.observe()`: Le dice al IntersectionObserverelemento objetivo que observe.
- `IntersectionObserver.takeRecords()`: Devuelve una matriz de IntersectionObserverEntryobjetos para todos los objetivos observados.
- `IntersectionObserver.unobserve()`: Le dice al IntersectionObserverpara que deje de observar un elemento objetivo particular.
