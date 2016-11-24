return Task vs return await Task
====

다른 라이브러리의 API를 자기 코드에 맞게 쓰기 위해서 래핑하는 경우가 있습니다.<br>
만약 `Async`일 경우, 그 반환값은 `Task<>`형이 되게 되는데, 이 `Task<>`가 어차피 래핑 함수의 반환값과 동일한 경우 그대로 바이패스 해버리면 되지 않을까 싶기도 합니다.<br>
<br>
예를들어 아래와 같은 라이브러리 API가 있는 상황에서<br>
(StackExchange.Redis의 API를 적당히 가져왔습니다.)
```cs
// StackExchange.Redis
//    반환값은 저장 성공 여부
Task<bool> StringSetAsync(string key, string value);
```

해당 인터페이스가 마음에 들지 않거나, 로깅 등 추가적인 작업이 필요해 `MyRedis` 클래스를 제작하려고 합니다.<br>
`MyRedis`의 String 저장 함수는 아래와 같이 될 수 있습니다.

```cs
static Task<bool> MySet(string key, string value);
```

이를 실제로 구현하고자 할 때, 아래와 같은 두가지 방법을 사용할 수 있는데<br>
<br>
첫번째는 `StringSetAsync`의 `Task<>`를 바로 리턴하는것이고,<br>
두번째는 __await__를 통해서 `MySet` 메소드에서 대기 후, 리턴하는 방법입니다.
```cs
// StringSetAsync Task를 바로 리턴
static Task<bool> MySet(string key, string value) {
    return StringSetAsync(key, value);
}

// await로 대기 후 리턴
static async Task<int> MySet(string key, string value) {
    return await StringSetAsync(key, value);
}
```

위의 두 방식은 결과적으로는 같아 보입니다.<br>
하지만 `StringSetAsync`에서 익셉션이 발생한 경우, 그때는 두가지 API의 동작이 다르게 됩니다.<br>
어떻게 다른지는 아래에<br>
<br>
__await 하지 않은 경우__<br>
* 익셉션 콜스택에서 `MySet`는 찍히지도 않습니다.
* `StringSetAsync`는 감시되지 않는 Task 상태가 됩니다. 
  * 만약 `MySet`의 호출자가 await을 사용했다면 다행이지만
  * `MySet` 호출 후, 대기하지 않았을 경우 발생한 익셉션은 붕 뜬 상태가 됩니다. (익셉션 콜스택 안찍힘)
```
처리되지 않은 예외: System.Exception: 'System.Exception' 형식의 예외가 Throw되었습니다.
  위치: async_await_playground.Program.<>c.<StringSetAsync>b__0_0()
  /* ..... */
  위치: async_await_playground.Program.<Foo>d__2.MoveNext()
  /* ..... */
```  

__await 한 경우__<br>
* 콜스택도 정상이고 모든게 기대대로입니다.
```
처리되지 않은 예외: System.Exception: 'System.Exception' 형식의 예외가 Throw되었습니다.
  위치: async_await_playground.Program.<>c.<StringSetAsync>b__0_0()
  /* ..... */
  위치: async_await_playground.Program.<MySet>d__1.MoveNext()
  /* ..... */
  위치: async_await_playground.Program.<Foo>d__2.MoveNext()
```
<br>
네
