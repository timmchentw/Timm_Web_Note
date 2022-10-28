# Web Api

## Client side

### Filter統一處理ModelState問題

https://blog.cashwu.com/blog/asp-net-core-action-filter-model-state/

### HttpClient擴充實作

添加方法，讓HttpClient能享有"同一站台API共用Client"、"Parse Json Response Content"、"Output強型別"、"設置Base Url"、"Handle Exception"等等功能 (持續擴充當中)

1. 建立擴充Request, Response 相關 class & Interface

    ```C#
    public class HttpRequestDto
    {
        public object RequestContent { get; set; }
        public string Route { get; set; } = "";
        public AccessTokenDto TokenDto { get; set; } = null;
        public List<KeyValuePair<string, string>> AdditionalHeaders { get; set; } = new List<KeyValuePair<string, string>>();

        public HttpRequestDto(object requestContent)
        {
            RequestContent = requestContent;
        }
    }

    public class HttpResponseBody<TOut>
    {
        public HttpStatusCode StatusCode { get; set; }
        public TOut Body { get; set; }
        public string RequestUrl { get; set; }
        public TimeSpan RequestTime { get; set; }
    }

    public class AccessTokenDto
    {
        public string TokenType { get; set; }
        public string Token { get; set; }
    }
    ```

    ```C#
    internal interface IHttpClientExt : IDisposable
    {
        // Get方法待補充Additional Headers
        HttpResponseBody<TOut> HttpGet<TOut>(Object requestContent = null, string route = "", AccessTokenDto tokenDto = null);
        HttpResponseBody<TOut> HttpGet<TOut>(List<KeyValuePair<string, string>> requestContent, string route = "", AccessTokenDto tokenDto = null);
        Task<HttpResponseBody<TOut>> HttpGetAsync<TOut>(Object requestContent = null, string route = "", AccessTokenDto tokenDto = null, CancellationToken cancellationToken = default(CancellationToken));
        Task<HttpResponseBody<TOut>> HttpGetAsync<TOut>(List<KeyValuePair<string, string>> requestContent, string route = "", AccessTokenDto tokenDto = null, CancellationToken cancellationToken = default(CancellationToken));

        HttpResponseBody<TOut> HttpSendContentJson<TOut>(HttpMethod httpMethod, HttpRequestDto requestDto);
        Task<HttpResponseBody<TOut>> HttpSendContentJsonAsync<TOut>(HttpMethod httpMethod, HttpRequestDto requestDto, CancellationToken cancellationToken = default(CancellationToken));
        HttpResponseBody<TOut> HttpSendContentUrlEncoded<TOut>(HttpMethod httpMethod, HttpRequestDto requestDto);
        Task<HttpResponseBody<TOut>> HttpSendContentUrlEncodedAsync<TOut>(HttpMethod httpMethod, HttpRequestDto requestDto, CancellationToken cancellationToken = default(CancellationToken));
        Task<HttpResponseBody<TOut>> HttpSendContent<TOut>(HttpMethod httpMethod, HttpContent requestContent, HttpRequestDto requestDto, CancellationToken cancellationToken = default(CancellationToken));
    }
    ```

2. 實作HttpClient擴充功能、處理型別轉換 & Exception Handling

   ```C#
   public class HttpClientExt : IHttpClientExt
    {
        private readonly IList<KeyValuePair<string, string>> _headers = new List<KeyValuePair<string, string>>();
        public Uri RequestBaseUri { get; }
        public HttpClient Client { get; }

        private readonly Stopwatch TimerStopwatch = new Stopwatch();

        public HttpClientExt(string requestUrl, TimeSpan? timeout = null, IList<KeyValuePair<string, string>> headers = null)
        {
            RequestBaseUri = new Uri(requestUrl);
            Client = new HttpClient() { BaseAddress = RequestBaseUri };
            Client.DefaultRequestHeaders.Add("Keep-Alive", "true");
            // 統一處理Timeout (建議)
            if (timeout != null)
            {
                Client.Timeout = timeout ?? new TimeSpan();
            }
            // 統一處理Headers
            if (headers != null)
            {
                foreach (var header in headers)
                {
                    _headers.Add(header);
                }
            }
        }

        public void Dispose()
        {
            try
            {
                Client.Dispose();
            }
            catch { /* 能關閉就關閉，關不起來則防止Crush */ }
        }

        // 呼叫實僅需傳入Url後段不同的Route即可 (注意Url最後須包含"/"符號避免Part遺失)
        public HttpResponseBody<TOut> HttpGet<TOut>(Object requestContent = null, string route = "", AccessTokenDto tokenDto = null)
        {
            return this.HttpGetAsync<TOut>(requestContent, route, tokenDto).GetAwaiter().GetResult();
        }

        public HttpResponseBody<TOut> HttpGet<TOut>(List<KeyValuePair<string, string>> requestContent, string route = "", AccessTokenDto tokenDto = null)
        {
            return this.HttpGetAsync<TOut>(requestContent, route, tokenDto).GetAwaiter().GetResult();
        }

        public async Task<HttpResponseBody<TOut>> HttpGetAsync<TOut>(Object requestContent = null, string route = "", AccessTokenDto tokenDto = null, CancellationToken cancellationToken = default(CancellationToken))
        {
            var queryParms = Util.ConvertClassToKeyValuePairs(requestContent);
            return await this.HttpGetAsync<TOut>(queryParms, route, tokenDto, cancellationToken);
        }

        public async Task<HttpResponseBody<TOut>> HttpGetAsync<TOut>(List<KeyValuePair<string, string>> requestContent, string route = "", AccessTokenDto tokenDto = null, CancellationToken cancellationToken = default(CancellationToken))
        {
            string queryString = Util.ConvertKeyValuePairsToUrlQueryString(requestContent);

            HttpRequestMessage requestMsg = new HttpRequestMessage(HttpMethod.Get, $"{route}?{queryString}");
            requestMsg.Headers.Authorization = tokenDto != null ? new AuthenticationHeaderValue(tokenDto.TokenType, tokenDto.Token) : null;
            if (_headers != null)
            {
                foreach (KeyValuePair<string, IEnumerable<string>> kvp in _headers)
                {
                    requestMsg.Headers.Add(kvp.Key, kvp.Value);
                }
            }

            // 計時 & 送出Request
            TimerStopwatch.Restart();
            HttpResponseMessage response = await Client.SendAsync(requestMsg).ConfigureAwait(false);
            TimerStopwatch.Stop();

            // 處理Response & Exception
            HttpResponseBody<TOut> output = await ProcessResponseAsync<TOut>(response);
            return output;
        }

        public HttpResponseBody<TOut> HttpSendContentJson<TOut>(HttpMethod httpMethod, HttpRequestDto requestDto)
        {
            return HttpSendContentJsonAsync<TOut>(httpMethod, requestDto).GetAwaiter().GetResult();
        }

        // 擴充Json Input方法
        public async Task<HttpResponseBody<TOut>> HttpSendContentJsonAsync<TOut>(HttpMethod httpMethod, HttpRequestDto requestDto, CancellationToken cancellationToken = default(CancellationToken))
        {
            string requestBodyJson = JsonConvert.SerializeObject(requestDto.RequestContent);
            var jsonContent = new StringContent(requestBodyJson, Encoding.UTF8, "application/json");

            return await HttpSendContent<TOut>(httpMethod, jsonContent, requestDto);
        }

        public HttpResponseBody<TOut> HttpSendContentUrlEncoded<TOut>(HttpMethod httpMethod, HttpRequestDto requestDto)
        {
            return HttpSendContentUrlEncodedAsync<TOut>(httpMethod, requestDto).GetAwaiter().GetResult();
        }

        // 擴充Form Url Encoded方法
        public async Task<HttpResponseBody<TOut>> HttpSendContentUrlEncodedAsync<TOut>(HttpMethod httpMethod, HttpRequestDto requestDto, CancellationToken cancellationToken = default(CancellationToken))
        {
            var contentKeyValuePairs = Util.ConvertClassToKeyValuePairs(requestDto.RequestContent);
            var urlEncodedContent = new FormUrlEncodedContent(contentKeyValuePairs);

            return await HttpSendContent<TOut>(httpMethod, urlEncodedContent, requestDto);
        }

        public async Task<HttpResponseBody<TOut>> HttpSendContent<TOut>(HttpMethod httpMethod, HttpContent requestContent, HttpRequestDto requestDto, CancellationToken cancellationToken = default(CancellationToken))
        {
            HttpRequestMessage requestMsg = new HttpRequestMessage(httpMethod, requestDto.Route);
            // 添加Headers
            if (_headers != null)
            {
                foreach (KeyValuePair<string, IEnumerable<string>> header in _headers)
                {
                    requestMsg.Headers.Add(header.Key, header.Value);
                }
            }
            if (requestDto.AdditionalHeaders != null)
            {
                foreach (var header in requestDto.AdditionalHeaders)
                {
                    requestMsg.Headers.Add(header.Key, header.Value);
                }
            }
            // 添加Token
            if (requestDto.TokenDto != null)
            {
                requestMsg.Headers.Authorization = new AuthenticationHeaderValue(requestDto.TokenDto.TokenType, requestDto.TokenDto.Token);
            }
            requestMsg.Content = requestContent;

            // 計時 & 送出Request
            TimerStopwatch.Restart();
            HttpResponseMessage response = await Client.SendAsync(requestMsg).ConfigureAwait(false);
            TimerStopwatch.Stop();

            // 處理Response & Exception
            HttpResponseBody<TOut> output = await ProcessResponseAsync<TOut>(response);
            return output;
        }

        public HttpResponseBody<TOut> ProcessResponse<TOut>(HttpResponseMessage response)
        {
            return ProcessResponseAsync<TOut>(response).GetAwaiter().GetResult();
        }

        // 處理Response，包含強型別轉換 & Exception擴增
        public async Task<HttpResponseBody<TOut>> ProcessResponseAsync<TOut>(HttpResponseMessage response, CancellationToken cancellationToken = default(CancellationToken))
        {
            string responseContent = await response.Content.ReadAsStringAsync();

            // 方便Exception handling時取得更完整資訊 (待增加Request content)
            string errorText = $@"Failed API url: 
    {response.RequestMessage.RequestUri.AbsoluteUri} 
    Error response: 
    {response.ToString()} 
    Error response content: 
    {responseContent}";

            if (response.IsSuccessStatusCode ||
                (response.StatusCode == HttpStatusCode.NotFound && typeof(TOut) != typeof(string)))
            {
                try
                {
                    // 404有可能是找不到資料而已，Response body可能還是正常
                    var returnObj = new HttpResponseBody<TOut>()
                    {
                        StatusCode = response.StatusCode,
                        Body = (typeof(TOut) == typeof(string)) ? (TOut)Convert.ChangeType(responseContent, typeof(TOut))
                                                                : JsonConvert.DeserializeObject<TOut>(responseContent),
                        RequestUrl = response.RequestMessage.RequestUri.AbsoluteUri,
                        RequestTime = TimerStopwatch.Elapsed,
                    };
                    return returnObj;
                }
                catch (Exception ex)
                {
                    // Parse失敗代表可能真的有問題，並附上擴充的API資訊
                    throw new HttpRequestException(errorText, ex);
                }
            }
            else
            {
                // 附上擴充的API資訊
                throw new HttpRequestException(errorText);
            }
        }
    }

    public class Util
    {
        // 此方法不適用Nested class
        public static List<KeyValuePair<string, string>> ConvertClassToKeyValuePairs(object obj)
        {
            var outputList = new List<KeyValuePair<string, string>>();

            if (obj != null)
            {
                var propertyList = obj.GetType().GetProperties();
                foreach (PropertyInfo property in propertyList)
                {
                    var output = new KeyValuePair<string, string>(property.Name, property.GetValue(obj).ToString());
                    outputList.Add(output);
                }
            }

            return outputList;
        }

        public static string ConvertKeyValuePairsToUrlQueryString(List<KeyValuePair<string, string>> parms)
        {
            string queryString = "";

            List<string> queryParameters = new List<string>();
            foreach (var parameter in parms)
            {
                queryParameters.Add($"{Uri.EscapeDataString(parameter.Key)}={Uri.EscapeDataString(parameter.Value)}");
            }

            queryString += string.Join("&", queryParameters);
            return queryString;
        }
    }
   ```

3. API引入Interface作為底層方法呼叫

   ```C#
    public class TeamsWebHookApi : IWebApi
    {
        private readonly IHttpClientExt _httpClient;
        public TeamsWebHookApi(string requestUrl, TimeSpan? timeout = null)
        {
            // 這邊可選擇性直接new instance，或是交由呼叫端input instance (DI)
            _httpClient = new HttpClientExt(requestUrl, timeout);
        }

        public void Dispose()
        {
            _httpClient.Dispose();
        }

        public bool SendTeamsMessage(TeamsWebHookMessage message)
        {
            var response = _httpClient.HttpSendContentJson<string>(HttpMethod.Post, new HttpRequestDto(message));
            return response.Body == "1" ? true : throw new HttpRequestException("Invalid response: " + response.Body);
        }
    }
   ```

## Server side