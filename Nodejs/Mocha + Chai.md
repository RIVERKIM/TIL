# Mocha + Chai

생성일: 2021년 8월 30일 오후 9:52

### Mocha

Mocha는 테스트 코드를 실행시켜주는 테스트 러너이다.

assertion이 지원이 안되기 때문에 assertion library를 써야한다. (chai)

before(), after(), beforeEach(), afterEach() 와 같은 훅을 제공.

Promise와 async/await 사용.

### Mocha 문법

- describe: 테스트들을 구분 짓고, 테스트에 대해 설명하는 함수로 주로 Object명이나 function 명을 작성. describe는 중첩가능.
- it: 어떤 결과가 나와야하는지 명시. 보통 어떤 함수의 결과로 기대되는 값을 작성. it의 콜백인수로 done을 사용하면 자동으로 비동기 테스트로 인식되고, 비동기 로직이 완료된 후 done()을 호출하게 되면 테스트가 완료.

```jsx
const assert = require('assert');

describe('Array', () => {
  describe('#indexOf()', () => {
    it('should return -1 when the value is not present', () => {
      assert.equal([1, 2, 3].indexOf(4), -1);
    });
    it('should return 1 when the value is 2', () => {
      assert.equal([1, 2, 3].indexOf(2), 1);
    })    
  });
});
```

- Mocha Hook

```jsx
describe('hooks', function() {
  before(() => {
    //블록 안에 첫번 째 테스트가 실행되기 전에 단 한번 실행console.log("####before");
  });

  after(() => {
    //블록 안에 테스트가 모두 끝난 후 단 한번 실행console.log("####after");
  });

  beforeEach(() => {
    //블록 안에 테스트들이 실행되기 전마다 실행console.log("####beforeEach");
  });

  afterEach(() => {
    //블록 안에 테스트들이 끝날 때마다 실행console.log("####afterEach");
  });

  it("Test 1", () => {
    console.log("####Test1");
  });
  
  it("Test 2", () => {
    console.log("####Test2");
  });
});
```

- Only: 실행되어야하는 단일 테스트들을 선택.
- skip: 해당 단일 테스트 건너뜀.

```jsx
describe('hooks', function() {
  it("Test 1", () => {
    //실행안됨
    console.log("####Test1");
  });
  
  it.only("Test 2", () => {
    //실행됨
    console.log("####Test2");
  });

  it("Test 3", () => {
    //실행안됨
    console.log("####Test3");
  });
});

describe('hooks', function() {
  it("Test 1", () => {
    //실행됨
    console.log("####Test1");
  });
  
  it.skip("Test 2", () => {
    //건너뜀
    console.log("####Test2");
  });

  it("Test 3", () => {
    //실행됨
    console.log("####Test3");
  });
});
```

### 코드 수정될 때마다 Test실행

```jsx
mocha -w <이름>
```

### Chai

assertion library for nodejs

체이닝을 이용하여 하나의 영어 문장으로 단일 테스트를 작성할 수 있어서 좀 더 직관적인 느낌을 줌.

chain은 실질적으로 테스트 결과에 영향을 미치지는 않고 하나의 문장을 만들기 위한 도구.

> to, be, been, is, that, which, and, has, have, with, at, of, same, but, does, still

테스트 함수 종류

- Equal: 값 비교, type까지 비교하기에 ===와 같다. not.equal() 같지 않음. object와 비교시 deep.equal()

```jsx
expect(sum(4, 5)).to.equal(9);
expect(sum(4, 5)).to.not.equal(7);
expect({ a : 1 }).to.deep.euqal({a : 1});
```

- Above, Least, Below, Most, Within, lengthOf : 어떤 값보다 크고 작음 혹은 어떤 값이 해당 범위에 있는지 검증.

```jsx
expect(Math.max(...arr)).to.above(3);
expect(Math.max(...arr)).to.least(4);
expect(Math.min(...arr)).to.below(2);
expect(Math.min(...arr)).to.most(1);
expect(arr.length).to.within(3, 5);
expect(arr).to.have.lengthOf(4);
```

- Keys: object의 key들을 비교하는 함수.

```jsx
//모두
expect(obj).to.have.all.keys('a', 'b');
//어느 하나라도
expect(obj).to.have.any.keys('a', 'c');
```

- Include: array에 어떤 값이 포함되는지

```jsx
expect([1, 2, 3]).to.include(1);
expect([{ a : 1 }]).to.deep.include({ a : 1 });
```

- A, An: type을 검증해주는 함수.

```jsx
expect('foo').to.be.a('string');
expect({a: 1}).to.be.an('object');
expect(null).to.be.a('null');
expect(undefined).to.be.an('undefined');
expect(new Error).to.be.an('error');
```

Throw: 에러가 발생했는지 검증

```jsx
const badFn = function () { throw new TypeError('Illegal salmon!'); };
//에러 검증
expect(badFn).to.throw(TypeError);
//에러 메시지 확인
expect(badFn).to.throw('Illegal salmon!');

//Error type과 메시지 같이 확인
expect(badFn).to.throw(TypeError, 'Illegal salmon!');
```

### Chai Assertion Styles

```jsx
//should
chai.should();

foo.should.be.a('string');
foo.should.equal('bar')

//expect 
var expect = chai.expect;

expect(foo).to.be.a('string');
expect(tea).to.have.property('flavors')
.with.lengthOf(3);

//Assert
var assert = chai.assert;

assert.typeOf(foo, 'string')
assert.equal(foo, 'bar')

```

### Testing 종류

- Assertion: comparison which throws an exception upon failure

```jsx
const assert = value => {
	if(!value) {
		throw new Error("assertion Failure");
	}
}
```

- Unit Test: assert a Unit behaves as intended. A unit is typically a single function.

```jsx
<name>.spec.js
```

- Integration Test: assert an aggregate of units or module behave as intended

### Unit test (1/4)

mocha doesn't have any functionality in it to make assertions

so we have to use node.js assert module or assertion library

```jsx
const assert = require("assert")
const requestTime = require("../lib/request-time")
```

### Unit test(2/4)

```jsx
const assert = require("assert")
const requestTime = require("../lib/request-time")
//Create a test with it(title, callback)

describe("requestTime middleware",function () {
	it('should add a `requestTime` property to the req `parameter`', function () {
		//call function
		//make assertion
	})
})
```

### Unit test(3/4)

```jsx
describe('requestTime middleware', function() {
	it('should add a timestamp `requestTime` prop to `req`', function() {
		const req = {};
		requestTime(req);
		assert.ok(req.requestTime > 0);
	})
})
```

### Unit test(4/4)

```jsx
// mocha 실행
"scripts": {
  "test": "mocha"
}
npm test
```

### Integration test

Supertest for integration tests against express servers

```jsx
const assert = require('assert');
const app = require('../app');
const request = require('supertest')

describe('GET /unix-timestamp', function() {
	it('should respond with JSON object containing timestamp', function(done) {
		return request(app).get('/unix-timestamp')
		.expect(200).then((res) => {
			assert.ok(res.body.timestamp < 1e10);
			done();
		}).catch(err => done(err))
	})
})
```