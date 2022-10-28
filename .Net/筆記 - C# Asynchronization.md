# C# Asynchronization 非同步程式處理

## async/await

* 修飾詞置於Function (目前不支援Property)，且回傳型別須為`Task` or `Task<T>`
* `await`的意義為等待該行跑完再往下跑，因此置入`async`後，至少要有一個`await`，否則就沒有等待回傳的意義
* 由於型別改為Task，會傳染相關reference functions，建議撰寫程式時優先撰寫async方法 (因為非同步方法可允許呼叫同步方法、但反之則需要使用Task.Run來達成)
* 同步程式等同於.Wait()或.Result()，不過建議改用.GetAwaitor().GetResult()，當然最好將程式改寫為async方法以備之後需呼叫async方法

## Task
* `void` → `Task`
* `T` → `Task<T>`
* `Action` → `Func<Task>`
* `Func<TOut>` → `Func<Task<TOut>>`

## WhenAll/WaitAll

* 當需要使用非同步平行運算時，需要使用此類方法等待特定Task跑完再繼續下去
* Task.WaitAll不會回傳Task
* Task.WhenAll會回傳一個統合等待的Task，使用上較為彈性，注意需使用await或.Wait()

```C#
public async Task MyFunctionAsync()
{
    // 要一起跑的Task注意不用加await
    Task task1 = ...;
    Task task2 = ...;
    Task task3 = ...;

    // 這邊相當於用一個await等待所有Task跑完
    Task whenAll = Task.WhenAll(task1, task2, task3);
    await whenAll;
}

public void MyFunction()
{
    // 同步方法須調用Task.Run
    Task task1 = Task.Run(() => ...);
    Task task2 = Task.Run(() => ...);
    Task task3 = Task.Run(() => ...);

    // 務必Wait()等待所有Task完成
    Task whenAll = Task.WhenAll(task1, task2, task3);
    whenAll.Wait();
}
```

* Exception處理:</br> 當非同步執行時，Exception建議調用每個Task的Exception一次性Throw，對於Log Error來說會較為完整
  
```C#
public static async Task WaitAllTasksAndCatchExceptions(Task whenAll)
{
    try
    {
        // 等待WhenAll的所有Task跑完
        await whenAll;
    }
    catch { /* 待會再一次性蒐集Exceptions */ }

    if (whenAll.Exception != null)
    {
        string allExceptions = @"Multiple errors from tasks has occured.";
        foreach (var ex in whenAll.Exception.InnerExceptions.Select((value, i) => new { value, i }))
        {
            allExceptions += $@"
[TASK {ex.i}]
{ex.value.GetType().Name}: {ex.value.ToString()}";
        }
        throw new Exception(allExceptions);
    }
}
```


## 參考資源

* [微軟官方文件(經典的煎早餐範例)](https://learn.microsoft.com/zh-tw/dotnet/csharp/programming-guide/concepts/async/)
* [黑大-GetAwaiter 到底能不能用？](https://blog.darkthread.net/blog/getawaiter-or-not/)
* [C# Task 使用 WhenAll 和 WaitAll 需要注意的坑](https://www.cnblogs.com/stulzq/p/16067610.html)
* [[.NET] Task 等待多個任務- Task.WaitAll 與Task.WhenAll](https://marcus116.blogspot.com/2019/02/c-task-waitall-whenall.html)