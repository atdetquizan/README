## CONTROL: PARA VALIDAR RUC PERSONA JURIDICA
---
```TYPESCRIPT
import { FormControl, Validators } from '@angular/forms';
import { RucValidator, RucOnlyPJValidator } from '../constants/validators.constant';

export class RucOnlyPJControl extends FormControl {
  /**
   * Mensajes de error según el tipo de validación
   */
  protected errorsMessages: { [key: string]: string} = {
    required: 'Este campo es requerido',
    rucInvalid: 'RUC inválido',
    maxlength: 'RUC inválido',
    minlength: 'RUC inválido'
  };

  /**
   * Retorna el texto de error
   */
  public get textError(): string {
    let message = '';

    for (const error in this.errors) {
      if (error && this.dirty) {
        message = this.errorsMessages[error];
      }
    }

    return message;
  }

  public constructor(required: boolean = true) {
    super('', []);

    this.createValidators(required);
  }

  /**
   * Indica si el control es inválido
   */
  public get isInvalid(): boolean {

    return this.touched && this.dirty && this.invalid;
  }

  private createValidators(required?: boolean) {

    const validators = [];

    validators.push(RucValidator);
    validators.push(RucOnlyPJValidator);
    validators.push(Validators.maxLength(11));
    validators.push(Validators.minLength(11));

    if (required) {
      validators.push(Validators.required);
    }

    this.setValidators(Validators.compose(validators));
  }
}
```