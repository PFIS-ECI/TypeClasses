Este archivo esta basado en [School of Haskell](https://www.schoolofhaskell.com/school/starting-with-haskell/introduction-to-haskell/5-type-classes)

# Type Clases

Cree un nuevo proyecto `stack`

```bash
$ stack new TypeClasses
$ stack build
```

# Polimorfismo Paramétrico

Los tipos de datos en Haskell depende del uso de las variables dentro del
cuerpo. 

Edite el archivo `src/Lib.hs`.
Modifique la línea:

```haskell
module Lib 
    ( someFunc
    ) where
```
para que quede:
```haskell
module Lib where
```

agregue la siguiente definición:

```haskell
imp p q = not p || q
```

ejecute el repl con `stack` y averigue el tipo de la función `imp`?

Cuando el tipo de datos de un parámetro no se puede deducir del cuerpo de la
función, utilizamos variables de tipo para indicar que puede ser utilizada con cualquier tipo de datos como en el caso de la función identidad:

Escriba en `Lib.hs` dos funciones diferentes que tengan tipo 
`a -> a -> a`?

```haskell
f1 :: a -> a -> a
f1 ...

f2 :: a -> a -> a
f2 ...
```

Para cada uno de los siguientes tipos de datos, determine un posible
comportamiento que pueda tener una función que tenga ese tipo:

* `a -> a`
* `a -> b`
* `a -> b -> a`
* `[a] -> [a]`
* `(b -> c) -> (a -> b) -> (a -> c)`
* `(a -> a) -> a -> a`

Al mirar el tipo de datos de una función se obtiene algo de información acerca de
la función.

# Polimorfismo AdHoc

En java hay un tipo de polimorfismo que depende del tipo de datos utilizado.

```java
public class AdHoc {

    public static Object adHoc(Object x,Object y) {
        if (x instanceof Integer && y instanceof Integer) {
            return (Integer)x + (Integer)y;
        } else if (x instanceof String && y instanceof String) {
            return (String)x +" "+ (String)y;
        } else {
            return x;
        }
    }

    public static void main(String args[]) {
        System.out.println(adHoc(2,3));
        System.out.println(adHoc("hello","world"));
        System.out.println(adHoc(true,false));
    }
}
```

Sin embargo, un código similar no se puede crear en Haskell, dado que no se
conocen los tipos de datos en el momento de la ejecución. En Haskell, una vez
que se han verificado que los tipos de datos son correctos, estos son eliminados
y no hacen parte del código compilado; por tanto no existe algo como la instrucción: `instanceof`.

Intente colocar el siguiente tipo de datos a la función `imp`
```haskell
imp :: a -> a -> a
imp p q = not p || q
```
y recargue el archivo en el repl.

El polimorfismo no solo se refiere a restricciones sobre los parámetros sino
tambien a garantias. 

# Otro punto de vista

Algunas veces es necesario decidir que hacer basado en el tipo de datos de la 
función y tener un tipo de datos muy general puede causar problemas.

Si se tiene el tipo de datos para la suma:
```haskell
(+) :: a -> a -> a
```
es demasiado general y se podria utilizar con tipos de datos que no la tienen definida, como:
```haskell
> True + False
> id + id
```

Pero es muy restringida si se tiene para un tipo de datos constante:
```haskell
(+) :: Int -> Int -> Int
```
No podría utilizarse con otros tipos de datos numéricos, como `Integer`, `Float`,
`Double`, etc.

Al solicitar el tipo de datos de `(+)` en Haskell se obtiene:
```haskell
> :t (+) 
(+) :: Num a => a -> a -> a
```

Que significa el `Num a` antes del `=>`?

Solicite en el repl los tipos de `(==), (<), show`.

# Type Classes

Se tiene que `Num`, `Eq`, `Ord` y `Show` son _tipos de clases_ y `(+)`, `(==)`,
`(<)` son polimórficos en tipos de clases. Intuitivamente los _tipos de clases_
corresponden a conjuntos de tipos de datos que tienen ciertas funciones
(operaciones) definidas para ellos.

Por ejemplo la clase `Eq`:

```haskell
class Eq a where
  (==) :: a -> a -> Bool
  (/=) :: a -> a -> Bool
```

La clase `Eq` esta definida con un parámetro de tipo `a`. Para un tipo `a` que
quiera ser una instancia de la clase `Eq`, debe definir dos funciones: `(==)` y
`(/=)`.

Por ejemplo, para que `Int` sea una instancia de `Eq`, se deben definir las
funciones `(==) :: Int -> Int -> Bool` y `(/=) :: Int -> Int -> Bool`. En
Haskell estas definiciones ya se realizaron y se puede consultar en el repl que
instancias estan definidas para `Int` con el comando de información:

```haskell
> :i Int
```

Cuando se mira el tipo de datos de `(==)`:

```
:t (==)
(==) :: Eq a => a -> a -> Bool
```

el `Eq a` se conoce con el nombre de restricción de tipo (_type constraint_) y
la restricción puede ser leida como para cualquier tipo `a` siempre que este sea
una instancia de `Eq`. Por lo tanto es un error llamar `(==)` con dos valores
donde su tipo no sea una instancia de `Eq`. El compilador utiliza el sistema
de tipos para determinar cual de las implementaciones debe ser usada. 

Por ejemplo, defina el tipo de datos `Nat` en `Lib.hs`:

```haskell
data Nat 
   = Zero
   | Succ Nat
```

y recarguelo en el repl
```haskell
> :r
...
> Zero == Zero
```

Podemos hacer que `Nat` sea una instancia de `Eq`

```haskell
instance Eq Nat where
    Zero == Zero = True
    Succ n == Succ m = n == m
    _ == _ = False
    
    n /= m = not (n == m)
```

En Haskell se pueden dar definiciones por defecto de algunas de las funciones
definidas en una clase, por ejemplo:

```haskell
class Eq a where
  (==) :: a -> a -> Bool
  (/=) :: a -> a -> Bool
  x /= y = not (x == y)
```

Sin embargo, una instancia particular puede sobreescribir la implementación
por defecto del `(/=)`.

De hecho la clase `Eq` en Haskell se define:

```haskell
class Eq a where
  (==), (/=) :: a -> a -> Bool
  x == y = not (x /= y)
  x /= y = not (x == y)
```
Permitiendo definir solo alguno de los operadores `(==)` o `(/=)`, la otra definición se toma por defecto.

# Type classes and Java interfaces

Las clases en Haskell y las interfaces de Java son similares a, ambas definen
conjuntos de tipos/clases que tienen un conjunto de operaciones común. Sin embargo, hay algunas diferencias.

* Una clase Java tiene que definir todas las interfaces que implementa,
  la instancia de una clase de tipo puede estar definida de manera independiente
  a la declaración del tipo correspondiente.
* Los tipos que pueden ser especificados en una clase de tipo son mas 
  generales y flexibles que las firmas que pueden ser dadas para los métodos
  de las interfaces.

