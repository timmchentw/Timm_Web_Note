## Command and Query Responsibility Segregation (CQRS)

* 將存取功能分離成Command與Query，進行讀寫分離
* 優點: 職責分離、測試與多工開發方便
* 範例: 將有讀寫功能的Repository，拆分成CommandHandler & QueryHandler

## Mediator

* 服務中介層，可讓高層級端不須注入特定型別的Service，僅需呼叫這個中介層，達到完全解耦 (比如兩個不能互相相依的projects)
* 為中介者模式實作，一般會搭配CQRS模式
* 套件: MediatR
* 範例

### 相關連結

* [MediatR介紹與CQRS簡介](https://hackmd.io/@spyua/rJZuyK3L_)
* [MSDN](https://learn.microsoft.com/en-us/azure/architecture/patterns/cqrs)

## Event Driven Architecture (TDA)
* 優點: 低耦合關係，使得不同Component能夠跨域溝通
* Subscribe
  * 用於訂閱事件，比如Get/Set特定Cache時，綁定Reset Delegate
* Publish
  * 用於對事件進行變動(更新)，比如觸發Reset特定Cache的Delegate
* 使用`ConcurrentDictionary`跨執行緒紀錄事件
* 範例

```C#
public class EventBroker : IEventBroker
{
    private readonly ConcurrentDictionary<EventType, ConcurrentDictionary<string, Delegate>> _eventSubscribers;

    public EventBroker()
    {
        _eventSubscribers = new ConcurrentDictionary<EventType, ConcurrentDictionary<string, Delegate>>();
    }

    // 呼叫Event的觸發點(包含輸入參數)
    public async Task<bool> PublishAsync<T>(EventType eventType, T message)
    {
        // 沒有訂閱(註冊)的Type不適用
        if (!_eventSubscribers.ContainsKey(eventType))
        {
            return true;
        }

        // 取出訂閱的Actions
        ConcurrentDictionary<string, Delegate> actions = _eventSubscribers[eventType];

        if (actions?.IsEmpty != false)
        {
            return true;
        }

        try
        {
            List<Task> tasks = new();

            // 執行所選Type的Actions & 轉入參數
            foreach (KeyValuePair<string, Delegate> action in actions)
            {
                tasks.Add(Task.Run(() => action.Value.Method.Invoke(action.Value.Target,
                                                                    action.Value.Method.GetParameters().Length > 0
                                                                    ? new object[] { message }
                                                                    : null)));
            }

            await Task.WhenAll(tasks).ConfigureAwait(false);

            return true;
        }
        catch (AggregateException ex)
        {
            foreach (Exception innerException in ex.InnerExceptions)
            {
                // Exception content handling
            }
            return false;
        }
        catch (Exception ex)
        {
            // Exception content handling
            return false;
        }
    }

    // 綁定Event action所用
    public string Subscribe(EventType eventType, Delegate action)
    {
        ConcurrentDictionary<string, Delegate> actions = _eventSubscribers.GetOrAdd(eventType, new ConcurrentDictionary<string, Delegate>());
        string subscribeToken = Guid.NewGuid().ToString();
        actions[subscribeToken] = action;

        return subscribeToken;
    }

    // 解除綁定Event
    public bool UnSubscribe(EventType eventType, string subscribeToken)
    {
        return _eventSubscribers.ContainsKey(eventType)
               && _eventSubscribers[eventType].TryRemove(subscribeToken, out _);
    }
}

public enum EventType
{
    ProductAdd,
    ProductRemove,
    SetProductCache,
    DeleteProductCache
}

// 綁定範例
public class ProductCache : IProductCache
{
    private readonly IEventBroker _eventBroker;
    private readonly IProductRepository _repository;

    public ProductDataCache(IEventBroker eventBroker, IProductRepository repository)
    {
        _cache = new MemoryCache(options);
        _eventBroker = eventBroker;
        _repository = repository;

        SubscribeEvents();
    }

    public Task<IProductCacheModel> GetAsync(int productId)
    {
        if (productId <= 0)
        {
            return Task.FromResult<IProductCacheModel>(new IProductCacheModel());
        }

        return _cache.GetOrCreateAsync(productId, async entry =>
        {
            entry.SetSlidingExpiration(new TimeSpan(1, 0, 0));
            entry.SetSize(1);
            return await _repository.GetProductAsync(productId) ?? new IProductCacheModel();
        });
    }

    // 註冊的部分
    private void SubscribeEvents()
    {
        _eventBroker.Subscribe(EventType.DeleteProductCache, RemoveFromCacheAction());
    }

    private Action<int> RemoveFromCacheAction()
    {
        return RemoveFromCache;
    }

    private void RemoveFromCache(int productId)
    {
        // 將操作Cache的部分委派給Event
        _cache.Remove(productId);
    }
}

// 取用範例
public class MyApp
{
    private readonly IEventBroker _eventBroker;

    public MyApp(IEventBroker eventBroker)
    {
        _eventBroker = eventBroker;
    }

    public async Task<bool> DeleteProduct(int productId)
    {
        // ...
            _ = _eventBroker.PublishAsync(EventType.DeleteProductCache, productId);
        // ...
        return true;
    }
}
```

  ### 相關連結
  * [Event Driven Architecture (TDA)](https://www.linkedin.com/pulse/event-driven-architectureeda-pattern-mukesh-kumar--snudf)


## Queue

### System.Threading.Channels

* 類似於RabbitMQ，是Thread當中的Queue，如下範例提供寫入與取出的用法

```C#
using System;
using System.Threading.Channels;
using System.Threading.Tasks;

class Program
{
    static async Task Main(string[] args)
    {
        // 創建一個無界通道
        var channel = Channel.CreateUnbounded<int>();

        // 啟動生產者任務
        var producer = Task.Run(async () =>
        {
            for (int i = 0; i < 10; i++)
            {
                await channel.Writer.WriteAsync(i);
                Console.WriteLine($"Produced: {i}");
                await Task.Delay(100); // 模擬生產延遲
            }
            channel.Writer.Complete();
        });

        // 啟動消費者任務
        var consumer = Task.Run(async () =>
        {
            await foreach (var item in channel.Reader.ReadAllAsync())
            {
                Console.WriteLine($"Consumed: {item}");
                await Task.Delay(150); // 模擬消費延遲
            }
        });

        // 等待生產者和消費者完成
        await Task.WhenAll(producer, consumer);
    }
}

```
