
# 기본 제공 클래스 확장

Array, Map 및 기타 와 같은 기본 제공 클래스도 확장할 수 있습니다.

아래 예시에, 네이티브 `Array`에서 상속받은 `PowerArray`가 있습니다.

```js run
// 하나 또는 여러개의 메서드를 추가할 수 있습니다
class PowerArray extends Array {
  isEmpty() {
    return this.length === 0;
  }
}

let arr = new PowerArray(1, 2, 5, 10, 50);
alert(arr.isEmpty()); // false

let filteredArr = arr.filter(item => item >= 10);
alert(filteredArr); // 10, 50
alert(filteredArr.isEmpty()); // false
```

Please note a very interesting thing. Built-in methods like `filter`, `map` and others -- return new objects of exactly the inherited type. They rely on the `constructor` property to do so.
여기에 유의해야할 매우 흥미로운 점이 있습니다. `filter`, `map` 등과 같은 기본으로 제공 메서드는 상속된 형식의 새로운 개체를 반환합니다. 이렇게 하려면 `생성자` 속성에 의존합니다.

위의 예시에서는
```js
arr.constructor === PowerArray
```

So when `arr.filter()` is called, it internally creates the new array of results using exactly `new PowerArray`, not basic `Array`. That's actually very cool, because we can keep using `PowerArray` methods further on the result.
따라서 `arr.filter()`라고 불릴 때 기본 `Array`이 아닌 `new PowerArray`을 사용하여 새 결과 배열을 내부적으로 만듭니다. 결과에 `PowerArray`메서드를 계속 사용할 수 있기 때문에, 매우 좋은 방법입니다.

Even more, we can customize that behavior.
더 나아가, 그 행동을 사용자 정의 할 수 있습니다.

We can add a special static getter `Symbol.species` to the class. If exists, it should return the constructor that JavaScript will use internally to create new entities in `map`, `filter` and so on.
특별한 정적 getter인 `Symbol.species`을 클래스에 추가할 수 있습니다. 만약에 이미 존재한다면, 자바스크립트가 내부적으로 사용하는 생성자가  `map`, `filter` 같은 것들에서 새로운 entities를 만듭니다.

If we'd like built-in methods like `map` or `filter` to return regular arrays, we can return `Array` in `Symbol.species`, like here:
`map` 또는 `filter`와 같은 기본 제공 메서드를 사용하여 일반 배열을 반환하려면 다음과 같이 `Symbol.species`의 `Array`를 반환할 수 있습니다.

```js run
class PowerArray extends Array {
  isEmpty() {
    return this.length === 0;
  }

*!*
  // built-in methods will use this as the constructor
  static get [Symbol.species]() {
    return Array;
  }
*/!*
}

let arr = new PowerArray(1, 2, 5, 10, 50);
alert(arr.isEmpty()); // 거짓

// filter는 arr.constructor[Symbol.species]를 생성자로 사용해서 새로운 배열을 생성합니다.
let filteredArr = arr.filter(item => item >= 10);

*!*
// filteredArr is not PowerArray, but Array
// filteredArr 는 PowerArray가 아니지만 Array 입니다.
*/!*
alert(filteredArr.isEmpty()); // 에러: filteredArr.isEmpty는 함수가 아님
```

As you can see, now `.filter` returns `Array`. So the extended functionality is not passed any further.
위의 예시에서 `.filter`는 `Array`를 반환하기 때문에 확장된 기능은 더 이상 전달되지 않습니다.

## 내장된 기능에는 정적 상속 없음

Built-in objects have their own static methods, for instance `Object.keys`, `Array.isArray` etc.
내장 객체들은 각자 정적 메서드를 가지고 있습니다. 예를들면, `Object.keys`, `Array.isArray` 같은 것들이죠.

As we already know, native classes extend each other. For instance, `Array` extends `Object`.
이미 알고 있듯이, 네이티브 클래스들은 서로 확장합니다. 예를들면, `Array`는 `Object`를 상속받은 것이죠.

Normally, when one class extends another, both static and non-static methods are inherited. That was thoroughly explained in the chapter [](info:static-properties-methods#statics-and-inheritance).
보통은, 하나의 클래스가 다른것을 확장할때, 정적인 메서드들과 아닌 메서드들도 모두 상속됩니다. 이것은 [](info:static-properties-methods#statics-and-inheritance)챕터에서 자세히 설명되어 있습니다.

But built-in classes are an exception. They don't inherit statics from each other.
그러나 내장 클래스들은 예외입니다. 그것들은 정적 메서드를 서로 상속하지 않습니다.

For example, both `Array` and `Date` inherit from `Object`, so their instances have methods from `Object.prototype`. But `Array.[[Prototype]]` does not reference `Object`, so there's no `Array.keys()` and `Date.keys()` static methods.
예를들면, `Array`와 `Date` 모두 `Object`로 부터 상속되었습니다. 그래서, 그것들의 인스턴스들은 `Object.prototype`의 메서드들을 가지고 있습니다. 그러나 `Array.[[Prototype]]` 은 `Object` 를 참조하지 않습니다. 그래서 `Array.keys()` 와  `Date.keys()` 라는 정적인 메서드들은 없습니다.

Here's the picture structure for `Date` and `Object`:
아래 그림에 `Date`와 `Object`의 구조도가 있습니다.

![](object-date-inheritance.svg)

As you can see, there's no link between `Date` and `Object`. They are independent, only `Date.prototype` inherits from `Object.prototype`.
위의 그림처럼, `Date`과 `Object`에는 연관성이 없습니다. 그것들은 독립적이고, 오직 `Date.prototype`만 `Object.prototype`으로 부터 상속받습니다.
