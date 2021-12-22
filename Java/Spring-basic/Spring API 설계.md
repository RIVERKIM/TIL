# Spring API 및 결과 데이터 구조 설계

### API 설계

1. 리소스의 사용목적에 따라 Http Method를 구분해서 사용한다.
    1. GET: 서버에 주어진 리소스의 정보를 요청(읽기)
    2. POST: 서버에 리소스를 제출 (쓰기)
    3. PUT: 서버에 리소스를 제출한다. POST와 달리 리소스 갱신 시 사용(수정)
    4. DELETE: 서버에 주어진 리소스를 삭제 요청(삭제)

1. 리소스에 Mapping된 주소 체계를 정형화 한다.
    1. GET /v1/user/{userId}: 회원 userId에 해당하는 정보 조회
    2. GET /v2/users : 회원 리스트 조회
    3. POST /v1/user: 신규 회원정보를 입력
    4. PUT /v1/user: 기존 회원의 정보를 수정
    5. DELETE: /v1/user/{userId}: userId로 기존 회원의 정보 삭제
    
2. 결과 데이터의 구조를 표준화하여 정의 한다.
    1. 결과 데이터는 결과 데이터 + api 요청 결과 데이터로 구성.

```java
// 기존 USER 정보
{
    "msrl": 1,
    "uid": "",
    "name": ""
}
// 표준화한 USER 정보
{
  "data": {
    "msrl": 1,
    "uid": ",
    "name": ""
  },
  "success": true
  "code": 0,
  "message": "성공하였습니다."
}
```

### 구현1. 결과 모델의 정의

- API의 실행 결과를 담는 공통 모델
    - API의 처리 상태 및 메시지를 내려주는 데이터로 구성
        - success: api 성공 실패 여부
        - code: 응답 코드
        - msg: 응답 메시지

```java
@Getter
@Setter
public class CommonResult {
	@ApiModelPropery(value = "응답 성공 여부")
	private boolean success;

	@ApiModelProperty(value = "응답 코드 번호 : >= 0 정상, < 0 비정상")
	private int code;
	
	@ApiModelProperty(value = "응답 메시지")
	private String msg;
}
```

- 결과가 단일건인 api를 담는 모델
    - Generic Interface에 <T>를 지정하여 어떤 형태의 값도 넣을 수 있도록 구현

```java
@Getter
@Setter
public class SingleResult<T> extends CommonResult {
    private T data;
}
```

- 결과가 여러 건인 api를 담는 모델
    - 결과 필드를 List 형태로 선언하고 Generic interface에 <T>를 지정하여 어떤 형태의 List값도 넣을 수 있도록 구현

```java
@Getter
@Setter
public class ListResult<T> extends CommonResult {
    private List<T> list;
}
```

### 구현2. 결과 모델을 처리할 Service 정의

- 결과 모델에 데이터를 넣어주는 역할을 하는 Service 정의

```java
@Service // 해당 Class가 Service임을 명시합니다.
public class ResponseService {

    // enum으로 api 요청 결과에 대한 code, message를 정의합니다.
    public enum CommonResponse {
        SUCCESS(0, "성공하였습니디."),
        FAIL(-1, "실패하였습니다.");

        int code;
        String msg;

        CommonResponse(int code, String msg) {
            this.code = code;
            this.msg = msg;
        }

        public int getCode() {
            return code;
        }

        public String getMsg() {
            return msg;
        }
    }
    // 단일건 결과를 처리하는 메소드
    public <T> SingleResult<T> getSingleResult(T data) {
        SingleResult<T> result = new SingleResult<>();
        result.setData(data);
        setSuccessResult(result);
        return result;
    }
    // 다중건 결과를 처리하는 메소드
    public <T> ListResult<T> getListResult(List<T> list) {
        ListResult<T> result = new ListResult<>();
        result.setList(list);
        setSuccessResult(result);
        return result;
    }
    // 성공 결과만 처리하는 메소드
    public CommonResult getSuccessResult() {
        CommonResult result = new CommonResult();
        setSuccessResult(result);
        return result;
    }
    // 실패 결과만 처리하는 메소드
    public CommonResult getFailResult() {
        CommonResult result = new CommonResult();
        result.setSuccess(false);
        result.setCode(CommonResponse.FAIL.getCode());
        result.setMsg(CommonResponse.FAIL.getMsg());
        return result;
    }
    // 결과 모델에 api 요청 성공 데이터를 세팅해주는 메소드
    private void setSuccessResult(CommonResult result) {
        result.setSuccess(true);
        result.setCode(CommonResponse.SUCCESS.getCode());
        result.setMsg(CommonResponse.SUCCESS.getMsg());
    }
}
```

- 구현3. HttpMethod와 정형화된 주소 체계로 Controller 수정
    - 리소스의 사용 목적에 따라 GetMapping, PostMapping, PutMapping, DeleteMapping 사용.

```java
@Api(tags = {"1. User"})
@RequiredArgsConstructor
@RestController
@RequestMapping(value = "/v1")
public class UserController {

    private final UserJpaRepo userJpaRepo;
    private final ResponseService responseService; // 결과를 처리할 Service

    @ApiOperation(value = "회원 리스트 조회", notes = "모든 회원을 조회한다")
    @GetMapping(value = "/users")
    public ListResult<User> findAllUser() {
        // 결과데이터가 여러건인경우 getListResult를 이용해서 결과를 출력한다.
        return responseService.getListResult(userJpaRepo.findAll());
    }

    @ApiOperation(value = "회원 단건 조회", notes = "userId로 회원을 조회한다")
    @GetMapping(value = "/user/{msrl}")
    public SingleResult<User> findUserById(@ApiParam(value = "회원ID", required = true) @PathVariable long msrl) {
        // 결과데이터가 단일건인경우 getBasicResult를 이용해서 결과를 출력한다.
        return responseService.getSingleResult(userJpaRepo.findById(msrl).orElse(null));
    }

    @ApiOperation(value = "회원 입력", notes = "회원을 입력한다")
    @PostMapping(value = "/user")
    public SingleResult<User> save(@ApiParam(value = "회원아이디", required = true) @RequestParam String uid,
                                   @ApiParam(value = "회원이름", required = true) @RequestParam String name) {
        User user = User.builder()
                .uid(uid)
                .name(name)
                .build();
        return responseService.getSingleResult(userJpaRepo.save(user));
    }

    @ApiOperation(value = "회원 수정", notes = "회원정보를 수정한다")
    @PutMapping(value = "/user")
    public SingleResult<User> modify(
            @ApiParam(value = "회원번호", required = true) @RequestParam long msrl,
            @ApiParam(value = "회원아이디", required = true) @RequestParam String uid,
            @ApiParam(value = "회원이름", required = true) @RequestParam String name) {
        User user = User.builder()
                .msrl(msrl)
                .uid(uid)
                .name(name)
                .build();
        return responseService.getSingleResult(userJpaRepo.save(user));
    }

    @ApiOperation(value = "회원 삭제", notes = "userId로 회원정보를 삭제한다")
    @DeleteMapping(value = "/user/{msrl}")
    public CommonResult delete(
            @ApiParam(value = "회원번호", required = true) @PathVariable long msrl) {
        userJpaRepo.deleteById(msrl);
        // 성공 결과 정보만 필요한경우 getSuccessResult()를 이용하여 결과를 출력한다.
        return responseService.getSuccessResult();
    }
}
```