CancellationToken
====

CancellationToken은 실행중인 작업(혹은 실행 대기중인)을 취소시키는 인터페이스를 제공합니다.<br>
<br>
예를 들어<br>
아래 코드는 취소 토큰을 이용하여 1초의 타임아웃을 가지는 `HTTP` 요청을 수행합니다.
```cs
var cts = new CancellationTokenSource(1000);
var http = new System.Net.Http.HttpClient();

http.GetAsync("http://www.naver.com", cts.Token)
    .ContinueWith(result => {
        Console.WriteLine(
            result.Result.Content.ReadAsStringAsync().Result);
    });
```

CancellationToken vs CancellationTokenSource
----
__CancellationToken__
* Consumer
* 취소 되었는지, 취소되었을 때 실행할 작업 등록 가능
<br>
__CancellationTokenSource__
* Producer
* 취소 이벤트 발생 / 또는 시점 지정 가능 

이러한 구조는 마치 ID에 권한을 부여하듯, 각각 코드가 자기 역할만을 수행하여 오작동을 방지하도록 해줍니다.<br>
<br>
실제로, __CancellationToken__은 한개의 Task가 독점하여 사용하는 리소스가 아닙니다.<br>
여러개의 Task가 한개의 __CancellationToken__를 바라보고 작업하는 상황은 얼마든지 발생할 수 있으며, 만약 Task 쪽에서도 토큰의 취소작업이 가능하게 되면 해당 토큰을 바라보는 모든 Task에 대해 의도하지 않은 취소가 줄줄히 일어날 수 있습니다.   


작업을 취소하기 (CancellationTokenSource)
----
__지정 시간 후에 취소되도록 하기__
```cs
// 1 초 뒤 자동 취소상태가 됩니다.
var cts = new CancellationTokenSource(1000);
```

__수동으로 취소하기__
```cs
var cts = new CancellationTokenSource();

// 호출 시점에 취소 상태가 됩니다.
cts.Cancel();
```
<br>
주의할 점은, 취소 상태가 설정된다고 해서 실행중인 작업이 즉시 취소되지는 않습니다.<br>
실질적 취소는 실행중인 작업 내부에서 다음번 취소 상태를 검사할 때 일어나게 되며, 만약 작업에서 취소 상태를 검사하는 코드가 없을 경우 취소 여부에 관계없이 작업이 계속 실행될 수 있습니다.<br>
<br>
아래의 예제에서는 취소가 가능한 작업을 만드는 방법에 대해 설명합니다.

작업이 취소되었는지 알아보기 (CancellationToken)
----
```cs
var cts = new CancellationTokenSource(1000);

Task.Run(() => {
    // 대충 20*100ms 동안 - 출력
    for(int i=0;i<20;i++) {
        Console.Write("-");
        Thread.Sleep(100);
    }
}, cts.Token);
```

```cs
var cts = new CancellationTokenSource(1000);

Task.Run(() => {
    // 대충 20*100ms 동안 - 출력
    for(int i=0;i<20;i++) {
        Console.Write("-");
        Thread.Sleep(100);

        cts.Token.ThrowIfCancellationRequested();
    }
}, cts.Token);
```

만약 메소드로 만들어 취소 토큰이 외부로부터 와야 하는 상황이라면 아래와 같이 해주면 됩니다.
```cs
Task DoSomethingAsync(CancellationToken ct) {
    return Task.Run(() => {
        /* .... */

        ct.ThrowIfCancellationRequested();
    }, ct);
}
```


IsCancellationRequested와 ThrowIfCancellationRequested
----
실행중인 작업을 중도 취소시키기 위해서 위 예제에서는 `ThrowIfCancellationRequested`를 사용했습니다.<br>
이는 작업을 취소시키는 가장 일반적인 방법이며, 별도의 루프 종료 처리도 필요하지 않습니다.<br>
<br>
작업 취소에 익셉션을 사용한다는 점이 조금 의아할 수도 있습니다만, 익셉션은 단순히 에러만을 나타내기 위한 도구가 아닙니다.<br>
Caller에게 __성공__ 이외의 실행 결과를 알려주기 위해서 사용할 수 있습니다. (보통의 경우 __에러__라는 실행 결과를 나타냅니다. 이곳에서는 __중도 취소됨__이라는 실행 결과로써 익셉션을 사용합니다.)
또한 `throw`는 애초부터 작업의 중도취소를 위해서 만들어진 키워드이므로 현재 상황에도 매우 적합합니다.<br>
<br>

하지만 경우에 따라서는 익셉션이 아닌 `true/false`를 반환하는 방법, 혹은 그냥 취소 하던 말던 아무 상관이 없는 작업을 만들고 싶은 경우도 존재합니다.<br>
아래의 예제는 `ThrowIfCancellationRequested`대신 `IsCancellationRequested`를 사용하는 방법을 보여줍니다.

```cs
// 취소 요청되면 리턴
if (ct.IsCancellationRequested)
    return false;

/* ... */

return true;
```