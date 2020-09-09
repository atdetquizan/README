** DISABLE CONTROL

- Angular le dice que es mejor que use las formas en que le brinda para deshabilitar / habilitar los controles de formulario.
Puede habilitar / deshabilitar un control de formulario utilizando las siguientes formas:
Cree una instancia nueva FormControlcon la disabledpropiedad establecida en true. FormControl({value: '', disabled: true}).
Llamando control.disable()o control.enable().
Pero a veces se pierde (😳) o necesita el método al que está acostumbrado (👆). Creemos una directiva que nos ayude a recrear esta funcionalidad.

```typescript
import { NgControl } from '@angular/forms';

@Directive({
  selector: '[disableControl]'
})
export class DisableControlDirective {

  @Input() set disableControl( condition : boolean ) {
    const action = condition ? 'disable' : 'enable';
    this.ngControl.control[action]();
  }

  constructor( private ngControl : NgControl ) {
  }

}
```
- Podemos obtener una referencia a la instancia de control de formulario a través de DI. Entonces solo es cuestión de llamar al método correcto según el Input()valor.
Ahora podemos usar nuestra directiva así:

```HTML
<input [formControl]="formControl" [disableControl]="disable">
<button (click)="disable = true">Disable</button>
<button (click)="disable = false">Enable</button>
```