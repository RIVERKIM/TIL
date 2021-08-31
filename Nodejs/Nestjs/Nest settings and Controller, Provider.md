# Nest settings and Controller, Provider

생성일: 2021년 8월 31일 오후 5:40

## Set up

```jsx
npm i -g @nestjs/cli
nest new project-name
```

- **Project structure**

```jsx
src
	app.controller.spec.ts // unit test for the controller
	app.controller.ts //controller for single route
	app.module.ts // root module of the application
	app.service.ts // basic service with a single method
	main.ts // entry file of the application which 
					//uses the core function NestFactory to create a Next application instance
```

- **main.ts**

```jsx
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  await app.listen(3000);
}
bootstrap();
```

Nest Application을 만들기 위해서는 NestFactory class를 이용해야 한다.

NestFactory class는 몇가지 정적 메소드를 가지며 create는 application object를 리턴한다.

### Controller

controller는 클라이언트로부터 오는 요청을 처리하고 응답을 반환하는 역할을 한다.

routing mechanism은 Controller가 어떤 요청을 받는지를 결정한다.

각 Controller는 한개 이상의 라우트를 가지며 서로 다른 라우트는 다른 역할을 수행한다.

![Untitled](Nest%20settings%20and%20Controller,%20Provider%20a41e4a06ae284be1b542155bef1a1954/Untitled.png)

- **Routing**

```jsx
import { Controller, Get } from '@nestjs/common';

@Controller('cats') // route path
export class CatsController {
  @Get('/profile') // => cats/profile
  findAll(): string {
    return 'This action returns all cats';
  }
}
```

route path는 Controller에 선언된 path에 method 데코레이터에 명시된 path가 합쳐져서 생성된다.

cf) 각 path는 정규표현식으로 표현 할 수 있다.

- **Nestjs response 처리 방식**
1. Standard: 각 request handler는 자바스크립트 object나 array를 반환한다. 이 반환된 값은 자동으로 JSON 형태로 직렬화 된다. 하지만 만약 자바스크립트의 기본 타입을 리턴할 경우 반환값을 직렬화 하지 않고 값 그대로 반환한다. 응닶의 status code는 항상 200 이 기본값이며, (POST는 201) 이 값은 @HttpCode() 데코레이터를 추가함으로써 변경할 수 있다.
2. Library-specific: 각 라이브러리의 응답 객체를 사용할 수 있다. 이 값은 @Res() 데코레이터를 추가함으로써 사용할 수 있다.(findAll(@Res() response)) 이렇게 함으로써 그 라이브러리에서 사용하는 방식대로 객체를 조작할 수 있다.

- **요청 객체**

각 handler는 클라이언트 요청 객체에 @Req() 데코레이터를 사용하여 접근할 수 있다.

```jsx
import { Controller, Get, Req } from '@nestjs/common';
import { Request } from 'express';

@Controller('cats')
export class CatsController {
  @Get()
  findAll(@Req() request: Request): string {
    return 'This action returns all cats';
  }
}
```

- **다양한 decorator 예시**

```jsx
import { Controller, Get, Post } from '@nestjs/common';

@Controller('cats')
export class CatsController {
  @Post()
	@HttpCode(204)//status code
	@Headers('Cache-Control', 'none') //headers
  create(): string {
    return 'This action adds a new cat';
  }

  @Get()
  findAll(): string {
    return 'This action returns all cats';
  }
}
```

- Redirection

```jsx
@Get()
@Redirect('https://nestjs.com', 301)
//Redirect는 두개의 값을 받는다.
{
	"url": string,
	"statusCode": number
}

@Get('docs')
@Redirect('https://docs.nestjs.com', 302)
getDocs(@Query('version') version) {
	if(version && version === '5') {
		return {url:'https://docs.nestjs.com/v5/' }
	}
}
```

- Route parameters

라우트 파라미터를 정의하기 위해서는 라우트 파라미터 토큰을 라우트 path에 정의해야 한다. 

```jsx
@Get(:id)
findOne(@Param() params): string {
	console.log(params.id)
}
```

- Asynchronicity

모든 async function은 Promise를 반환한다. 

```jsx
@Get()
async findAll(): Promise<any[]> {
	return []
}
```

또한 Nest route handler는 RxJS observable streams을 반환할 수 있다. nest는 자동으로 그 source를 구독하며 가장 최신 값을 취한다.

```jsx
@Get()
findAll(): Observable<any[]> {
	return of([])
}
```

- Request payloads

DTO(Data Transfer Object)정의한 후 @Body() 를 이용해서 값 받기

```jsx
export class CreateCatDto { // interface 는 transfiling과정에서 사라짐.
// 따라서 이 값을 계속 참조하기 위해서는 class 형태로 정의해야 한다.
  name: string;
  age: number;
  breed: string;
}
@Post()
async create(@Body() createCatDto: CreateCatDto) {
  return 'This action adds a new cat';
}
```

전체 코드

```jsx
import { Controller, Get, Query, Post, Body, Put, Param, Delete } from '@nestjs/common';
import { CreateCatDto, UpdateCatDto, ListAllEntities } from './dto';

@Controller('cats')
export class CatsController {
  @Post()
  create(@Body() createCatDto: CreateCatDto) {
    return 'This action adds a new cat';
  }

  @Get()
  findAll(@Query() query: ListAllEntities) {
    return `This action returns all cats (limit: ${query.limit} items)`;
  }

  @Get(':id')
  findOne(@Param('id') id: string) {
    return `This action returns a #${id} cat`;
  }

  @Put(':id')
  update(@Param('id') id: string, @Body() updateCatDto: UpdateCatDto) {
    return `This action updates a #${id} cat`;
  }

  @Delete(':id')
  remove(@Param('id') id: string) {
    return `This action removes a #${id} cat`;
  }
}
```

```jsx
import {Module} from '@nestjs/common'
import {CatsController} from './cats/cats.controller'

@Module({
controller: [CatsController]
})

export class AppModule {}
```

### Providers

기본적인 Nest 클래스는 Providers로 간주 된다.  service, repositories, factories 등.

Providers는 DI가 가능하다. 

Controller는 HTTP 요청만을 처리하고 더 복잡한 일은 Providers에게 위임해야한다.

- Services

```jsx
nest g service cats
```

```jsx
import { Injectable } from '@nestjs/common';
import { Cat } from './interfaces/cat.interface';

@Injectable()
export class CatsService {
  private readonly cats: Cat[] = [];

  create(cat: Cat) {
    this.cats.push(cat);
  }

  findAll(): Cat[] {
    return this.cats;
  }
}
```

@Injectable() 데코레이터는 CatsService가 Nest IoC container에 의해 관리 된다는 것을 의미한다. 

위의 서비스를 이용하는 Controller는 아래와 같다.

```jsx
import { Controller, Get, Post, Body } from '@nestjs/common';
import { CreateCatDto } from './dto/create-cat.dto';
import { CatsService } from './cats.service';
import { Cat } from './interfaces/cat.interface';

@Controller('cats')
export class CatsController {
  constructor(private catsService: CatsService) {}

  @Post()
  async create(@Body() createCatDto: CreateCatDto) {
    this.catsService.create(createCatDto);
  }

  @Get()
  async findAll(): Promise<Cat[]> {
    return this.catsService.findAll();
  }
}
```

CatsService 는 constructor class를 통해 CatsController에 주입 되었다.

- Scope

Providers는 기본적으로 application lifecycle과 같은 lifecycle을 갖는다. application이 bootstrap 될 때 모든 의존성은 결합되며 그에 따라 모든 providers가 동작한다. 

- Optional providers

반드시 주입 될 필요없는 의존성을 갖을 때 Optional로 할 수 있다

```jsx
import { Injectable, Optional, Inject } from '@nestjs/common';

@Injectable()
export class HttpService<T> {
	constructor(@Optional() @Inject('HTTP_OPTIONS') private httpClient: T) {}
}
```

- Property-based Injection

클래스가 여러개의 Providers에 종속될 때 사용할 수 있다.

```jsx
import { Injectable, Inject } from '@nestjs/common';

@Injectable()
export class HttpService<T> {
  @Inject('HTTP_OPTIONS')
  private readonly httpClient: T;
}
```

- Provider registration

```jsx
import { Module } from '@nestjs/common';
import { CatsController } from './cats/cats.controller';
import { CatsService } from './cats/cats.service';

@Module({
  controllers: [CatsController],
  providers: [CatsService],
})
export class AppModule {}
```