# Introduction

TODO

# Union Types

Occasionally, you'll run into a library that expects or gives back a `number` or `string`.
For instance, take the following function:

```TypeScript
/**
 * Takes a string and adds "padding" to the left.
 * If 'padding' is a string, then 'padding' is appended to the left side.
 * If 'padding' is a number, then that number of spaces is added to the left side.
 */
function padLeft(value: string, padding: any) {
    // ...
}
```

The problem with `padLeft` is that its `padding` parameter is typed as `any`.
That means that we can call it with an argument that's neither a `number` nor a `string`, but TypeScript will be okay with it.

```TypeScript
let indentedString = padLeft("Hello world", true); // passes at compile time, fails at runtime.
```

Instead of `any`, we can use a *union type* for the `padding` parameter:

```TypeScript
/**
 * Takes a string and adds "padding" to the left.
 * If 'padding' is a string, then 'padding' is appended to the left side.
 * If 'padding' is a number, then that number of spaces is added to the left side.
 */
function padLeft(value: string, padding: any) {
    // ...
}

let indentedString = padLeft("Hello world", true); // errors during compilation
```

A union type describes a value that can be one of several types.
We use the vertical bar (`|`) to separate each type, so `number | string | boolean` is the type of a value that can be a `number`, a `string`, or a `boolean`.

If we have a value that has a union type, we can only access members that are common to all types in the union. 
 
```TypeScript
interface Bird {
    fly();
    layEggs();
}

interface Fish {
    swim();
    layEggs();
}

function getSmallPet(): Fish | Bird {
    // ...
}

let pet = getSmallPet();
pet.layEggs(); // okay
pet.swim();    // errors
```

Union types can be a bit tricky here, but it just takes a bit of intuition to get used to.
If a value has the type `A | B`, we only know for *certain* that it has members that both `A` *and* `B` have.
On the other hand, in the example above `Bird` has a member named `fly`.
We can't be sure whether a variable typed as `Bird | Fish` has a `fly` method.
If the variable is really a `B` at runtime, then calling `pet.fly()` will fail.  

# Type Guards and Differentiating Types

Union types are useful for modeling situations when values can overlap in the types they can take on.
What happens when we need to know specifically whether we have a `Fish`?
A common idiom in JavaScript to differentiate between two possible values is to check for the presence of a member.
As we mentioned, you can only access members that are guaranteed to be in all the constituents of a union type.

```TypeScript
let pet = getSmallPet();

// Each of these property accesses will cause an error
if (pet.swim) {
    pet.swim();
}
else if (pet.fly) {
    pet.fly();
}
```

To get the same code working, we'll need to use a type assertion:

```TypeScript
let pet = getSmallPet();

if ((<Fish>pet).swim) {
    (<Fish>pet).swim();
}
else {
    (<Bird>pet).fly();
}
```

## User Defined Type Guards

Notice that we had to use type assertions several times.
It would be much better if once we performed the check, we could know the type of `pet` within each branch.

It just so happens that TypeScript has something called a *type guard*.
A type guard is some expression that performs a runtime check that guarantees the type in some scope.
We can write a type guard using a regular function:

```TypeScript
function isFish(pet: Fish | Bird): pet is Fish {
    return (<Fish>pet).swim !== undefined;
}
```

Any time `isFish` is used on a variable name, TypeScript will know that variable has that specific type. 

```TypeScript
// Both calls to 'swim' and 'fly' are now okay.

if (isFish(pet) {
    pet.swim();
}
else {
    pet.fly();
}
```

Notice that TypeScript not only knows that `pet` is a `Fish` in the `if` branch;
it also knows that in the `else` branch, you *don't* have a `Fish`, so you must have a `Bird`.

## `typeof` type guards

TODO

## `instanceof` type guards

TODO

# Type Aliases

Type aliases create a new name for a type.
Type aliases are sometimes similar to interfaces, but can name primitives, unions, tuples, and any other types that you'd otherwise have to write by hand.

```TypeScript
type XCoord = number;
type YCoord = number;

type XCoord = { x: XCoord };
type XYCoord = { x: XCoord; y: YCoord };
type XYZCoord = { x: XCoord; y: YCoord; z: number };

type Coordinate = XCoord | XYCoord | XYZCoord;
type CoordList = Coordinate[];

let coord: CoordList = [{ x: 10, y: 10}, { x: 0, y: 42, z: 10 }, { x: 5 }];
```

Aliasing doesn't actually create a new type - it creates a new *name* to refer to that type.
So `10` is a perfectly valid `XCoord` and `YCoord` because they both just refer to `number`.
Aliasing a primitive is not terribly useful, though it can be used as a form of documentation.

Just like interfaces, type aliases can also be generic - we can just add type parameters and use them on the right side of the alias declaration:

```TypeScript
type Container<T> = { value: T };
```

We can also have a type alias refer to itself in a property:

```TypeScript
type Tree<T> = {
    value: T;
    left: Tree<T>;
    right: Tree<T>;
}
```

However, it's not possible for a type alias to appear anywhere else on the right side of the declaration:

```TypeScript
type Yikes = Array<Yikes>; // error
```

## Interfaces vs. Type Aliases

As we mentioned, type aliases can act sort of like interfaces; however, there are some subtle differences.

One important difference is that type aliases cannot be extended or implemented from (nor can they extend/implement).
Because you should usually try to leave your types open to extension, you should always use an interface over a type alias if possible.

On the other hand, if you can't express some shape with an interface and you need to use a union or tuple type, type aliases are usually the way to go.