Long Running Task
====

__좀 많이 오래 도는 태스크 만들기__<br>


걸러야 할 것
----
__CPU__ 코어는 무한이 아닙니다. 스레드 많이 만들어서 돌려봤자 어차피 처리할 수 있는 양에는 한계가 있습니다.<br>
예를들어 아래 `HeavyJinwoo` 작업은 굉장히 오래 걸리겠지만, 어쩔 수 없습니다. 

```cs
Task HeavyJinwoo() {
    return Task.Run(() => {

        for (int i=0;i<int.Max;i++)
            HeavyCalcWork(); 
    });
}
```


```cs
Task SlowJinwoo() {
    return Task.Run(() => {

        
    });
}
```

스레드 풀 옵션 조정하기
----

일반적으로, 생성된 `Task`는 `ThreadPool` 클래스 내부에서 실행됩니다.<br>
`.Net`의 스레드풀은 아래와같은 두가지 옵션을 설정할 수 있습니다. 
```cs
ThreadPool.SetMinThreads(30, 30);

ThreadPool.SetMaxThreads(30, 30);
```

API만 보면 __Max__를 조정하면 될 것 같이 보이지만, __Min__을


태스크 자체에 옵션 조정하기
----

```cs
return Task.Factory.StartNew(() => {
    /* CODE */
}, TaskCreationOptions.LongRunning);
````

<br>
__Task.Run__과 __Task.Factory.StartNew__ 차이점<br>
