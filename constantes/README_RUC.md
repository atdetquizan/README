### CONSTANTE: VALIDAR RUC QUE NO COMIENCE CON ``10`` YA QUE ES DE PERSONA NATRUAL
---
```typescript
/**
 * Function that verifies that the RUC number does not start with 10 - Legal entity begins with 20
 * @requires RucValidator()
 */
export const RucOnlyPJValidator = (control: AbstractControl) => {
    const { value } = control;

    if (value) {
      const docNumber = value;
      const prefix = docNumber.substr(0, 2);

      if (prefix === '10') {
        return { rucInvalid: true };
      }
    }

    return null;
  };
};
```

### CONSTANTE: VALIDAR QUE EL RUC SEA VALIDO
---
```typescript
/**
 * Función que verifica si la fecha pertenece a un menor de edad
 *
 * @param rucNumber Fecha de nacimiento
 */
export const rucIsValid = (rucNumber?: string): boolean => {

  if (!rucNumber) {
    return false;
  }

  const iaux = new Array<number>();

  for (let iCont = 1; iCont <= 11; ++iCont) {
    iaux[iCont] = +rucNumber.substr(iCont - 1, 1);
  }

  iaux[1] = iaux[1] * 5;
  iaux[2] = iaux[2] * 4;
  iaux[3] = iaux[3] * 3;
  iaux[4] = iaux[4] * 2;
  iaux[5] = iaux[5] * 7;
  iaux[6] = iaux[6] * 6;
  iaux[7] = iaux[7] * 5;
  iaux[8] = iaux[8] * 4;
  iaux[9] = iaux[9] * 3;
  iaux[10] = iaux[10] * 2;

  let iA: number = 0;
  for (let iCont = 1; iCont <= 10; ++iCont) {
    iA = iA + iaux[iCont];
  }

  const iB: number = parseInt((iA / 11).toString());
  const iC: number = iA - (iB * 11);
  const iD: number = 11 - iC;

  let valid: boolean = false;
  if (iD === 10 && iaux[11] === 0) {

    valid = true;
  } else if (iD === 11 && iaux[11] === 1) {

    valid = true;
  } else if (iD === iaux[11]) {

    valid = true;
  }

  return valid;
};
```