# Funcion recursive

funcion para recorrer recursivamente un arreglo

```typescript
export interface IMapRecursive<T> {
    children?: T[];
}

export const MapRecursive = <U, T extends IMapRecursive<T>>(
    item?: T,
    fn?: (item: T, children?: U[]) => U,
): U => {
    const children: U[] =
        item.children && item.children.length
            ? item?.children.map((x) => MapRecursive(x, fn))
            : [];
    return fn(item, children);
};
```