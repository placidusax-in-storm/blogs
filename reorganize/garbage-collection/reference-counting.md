# Reference counting

## Mechanism

1. when reference is assign to a variable/property, reference count + 1.
2. when variable/property reassign to other value, original reference count - 1.
3. when a reference 's reference count is 0, and it 's property also no have any variable referencing to it, destroy happen.

## Limitation

Circle reference:

1. `A` reference `B`, `B` reference `A`
2. Reference count at least 1, so can not be free

## MDN example is wrong

```js
function f() {
  var x = {};
  var y = {};
  x.a = y; // x references y
  y.a = x; // y references x

  return "azerty";
}

f();
```

`x` and `y` are stored in function stack.

After the function is executed, the stack is destroyed, so there is no memory leak.
