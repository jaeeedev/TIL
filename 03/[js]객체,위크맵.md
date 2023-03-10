파일명 필히 바꿔야 한다 !!!

## 깃

## 객체(hasOwnProperty, hasOwn, in)

```js
const person = {
  name: "jaeee",
};

person.hasOwnProperty("name"); // true
```

`hasOwnProperty`은 객체에 특정 프로퍼티가 있는지를 확인할 수 있는 메서드

하지만 `hasOwnProperty`이라는 명칭 자체를 보호하고있지 않아서

```js
var foo = {
  hasOwnProperty: function () {
    return false;
  },
  bar: "Here be dragons",
};

foo.hasOwnProperty("bar"); // always returns false
```

이런 경우에 문제가 발생  
`hasOwnProperty`라는 프로퍼티를 객체 내부에 또 선언했기 때문에 hasOwnProperty를 호출하게 되면 우선적으로 `foo`객체 내부의 `hasOwnProperty`가 먼저 실행되어 `bar` 프로퍼티가 있다고 해도 `false`만을 반환하게 된다.

### 해결 방법

```js
Object.prototype.hasOwnProperty.call(foo, "bar"); // true
```

Object의 프로토타입에서 직접 `hasOwnProperty`를 호출해서 사용하는 방법

```js
function hasOwnProp(targetObj, targetProp) {
  return Object.prototype.hasOwnProperty.call(targetObj, targetProp);
}
```

이런 식으로 유틸 함수를 만들어서 활용하기
`call`은 잘 안써서 계속 까먹음 `Fn.call(thisObj, args)` 이런 느낌이다  
thisObj에서 Fn을 쓰겠다고!!!! args는 Fn에 필요한 인수들이고

### 다른 프로퍼티 유무 확인하는 방법

```js
Object.hasOwn(foo, "bar"); // true
```

`hasOwn`은 어떤 객체에 특정 프로퍼티가 있는지 확인할 수 있는 `Object` 객체의 메서드

```js
"bar" in foo;
```

`in`도 객체에 프로퍼티가 있는지 확인할 수 있는 메서드  
배열에서는 인덱스로 확인 가능

클래스에서 **private**으로 선언해놓은 경우에는 저 방법들 모두 확인 불가능하다

```js
class Person {
  #name = "jaeee";
}

const person = new Person();

person.hasOwnProperty("#name"); // false
```

## 위크맵

```js
let jaeee = { name: "jaeee" };

let map = new Map();
map.set(jaeee, "123");

jaeee = null;
```

`Map`의 경우 jaeee를 null로 덮어씌워도 map에 그대로 남아있어 가비지 컬렉팅 대상이 되지 않는다.

반면 `WeakMap`은 Map과 다르게 가비지 컬렉션이 가능하다.

### WeakMap의 특징

> 키가 반드시 객체여야 한다(원시값을 키로 쓸 수 없다.)

```js
let weakMap = new WeakMap();
let obj = {};
weakMap.set(obj, "ok");
weakMap.set("test", "오류 발생"); // Invalid value used as weak map key
```

위크맵의 키를 아무도 참조하지 않는다면 해당 객체는 메모리와 위크맵에서 삭제된다.

```js
let john = { name: "John" };

let weakMap = new WeakMap();
weakMap.set(john, "..."); // WeakMap {{…} => '...'}

john = null;

// weakMap을 확인해보면 WeakMap {} 으로 비워져 있음
```

> 위크맵은 반복 작업, keys(), values(), entries() 메서드를 지원하지 않는다.

```js
//위크맵이 지원하는 메서드

weakMap.get(key);
weakMap.set(key, value);
weakMap.delete(key);
weakMap.has(key);
```

### 사용 예시

```js
let cache = new WeakMap();

function process(obj) {
  if (!cache.has(obj)) {
    let result = obj;

    cache.set(obj, result);
  }

  return cache.get(obj);
}

// 📁 main.js
let obj = {
  /* ... */
};

let result1 = process(obj);
let result2 = process(obj);

// 객체가 쓸모없어지면 덮어쓰기
obj = null;
```

obj를 null로 비우면 캐싱된 결과도 메모리에서 자동으로 삭제됨
