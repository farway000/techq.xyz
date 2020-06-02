---
title:  .TDD学习笔记（一）单元测试
date: 2020-4-30 19:07
tags: 技术
author: 邹溪源
categories:
  - 技术
---
# 使用自定义DelegatingHandler编写更整洁的Typed HttpClient

## **简介**﻿

我写了很多[HttpClient](https://docs.microsoft.com/en-us/dotnet/api/system.net.http.httpclient?view=netcore-3.0)，包括类型化的客户端。自从我发现[Refit](https://github.com/reactiveui/refit)以来，我只使用了那一个，所以我只编写了很少的代码！但是我想到了你！你们中的某些人不一定会使用[Refit，](https://github.com/reactiveui/refit)因此，我将为您提供一些技巧，以使用[HttpClient消息处理程序](https://docs.microsoft.com/en-us/aspnet/web-api/overview/advanced/httpclient-message-handlers)（尤其是[DelegatingHandlers）](https://docs.microsoft.com/en-us/dotnet/api/system.net.http.delegatinghandler?view=netframework-4.8)编写具有最大可重用性的[类型化HttpClient](https://docs.microsoft.com/en-us/dotnet/architecture/microservices/implement-resilient-applications/use-httpclientfactory-to-implement-resilient-http-requests)。

## **编写类型化的HttpClient来转发JWT并记录错误**

这是要清理的[键入的HttpClient](https://docs.microsoft.com/en-us/dotnet/architecture/microservices/implement-resilient-applications/use-httpclientfactory-to-implement-resilient-http-requests)：

```
using DemoRefit.Models;
using DemoRefit.Repositories;
using Microsoft.AspNetCore.Authentication;
using Microsoft.AspNetCore.Http;
using Microsoft.Extensions.Logging;
using System;
using System.Collections.Generic;
using System.Net.Http;
using System.Net.Http.Headers;
using System.Threading.Tasks;

namespace DemoRefit.HttpClients
{
    public class CountryRepositoryClient : ICountryRepositoryClient
    {
        private readonly HttpClient _client;
        private readonly IHttpContextAccessor _httpContextAccessor;
        private readonly ILogger<CountryRepositoryClient> _logger;

        public CountryRepositoryClient(HttpClient client, ILogger<CountryRepositoryClient> logger, IHttpContextAccessor httpContextAccessor)
        {
            _client = client;
            _logger = logger;
            _httpContextAccessor = httpContextAccessor;
        }

        public async Task<IEnumerable<Country>> GetAsync()
        {
            try
            {
                string accessToken = await _httpContextAccessor.HttpContext.GetTokenAsync("access_token");
                if (string.IsNullOrEmpty(accessToken))
                {
                    throw new Exception("Access token is missing");
                }
                _client.DefaultRequestHeaders.Authorization = new AuthenticationHeaderValue("bearer", accessToken);

                var headers = _httpContextAccessor.HttpContext.Request.Headers;
                if (headers.ContainsKey("X-Correlation-ID") && !string.IsNullOrEmpty(headers["X-Correlation-ID"]))
                {
                    _client.DefaultRequestHeaders.Add("X-Correlation-ID", headers["X-Correlation-ID"].ToString());
                }
                
                using (HttpResponseMessage response = await _client.GetAsync("/api/democrud"))
                {
                    response.EnsureSuccessStatusCode();
                    return await response.Content.ReadAsAsync<IEnumerable<Country>>();
                }
            }
            catch (Exception e)
            {
                _logger.LogError(e, "Failed to run http query");
                return null;
            }
        }
    }
}
```

这里有许多事情需要清理，因为它们在您将在同一应用程序中编写的每个客户端中可能都是多余的：

- 从**HttpContext**读取访问令牌
- 令牌为空时，管理访问令牌
- 将访问令牌附加到[HttpClient](https://docs.microsoft.com/en-us/dotnet/api/system.net.http.httpclient?view=netcore-3.0)进行委派
- 从**HttpContext**读取CorrelationId
- 将CorrelationId附加到[HttpClient](https://docs.microsoft.com/en-us/dotnet/api/system.net.http.httpclient?view=netcore-3.0)进行委托
- 使用*EnsureSuccessStatusCode（）*验证Http查询是否成功

## **编写自定义的DelegatingHandler来处理冗余代码**

这是[DelegatingHandler](https://docs.microsoft.com/en-us/dotnet/api/system.net.http.delegatinghandler?view=netframework-4.8)： 	 	

```
using Microsoft.AspNetCore.Authentication;
using Microsoft.AspNetCore.Http;
using Microsoft.Extensions.Logging;
using System;
using System.Net.Http;
using System.Net.Http.Headers;
using System.Threading;
using System.Threading.Tasks;

namespace DemoRefit.Handlers
{
    public class MyDelegatingHandler : DelegatingHandler
    {
        private readonly IHttpContextAccessor _httpContextAccessor;
        private readonly ILogger<MyDelegatingHandler> _logger;

        public MyDelegatingHandler(IHttpContextAccessor httpContextAccessor, ILogger<MyDelegatingHandler> logger)
        {
            _httpContextAccessor = httpContextAccessor;
            _logger = logger;
        }

        protected override async Task<HttpResponseMessage> SendAsync(HttpRequestMessage request, CancellationToken cancellationToken)
        {
            HttpResponseMessage httpResponseMessage;
            try
            {
                string accessToken = await _httpContextAccessor.HttpContext.GetTokenAsync("access_token");
                if (string.IsNullOrEmpty(accessToken))
                {
                    throw new Exception($"Access token is missing for the request {request.RequestUri}");
                }
                request.Headers.Authorization = new AuthenticationHeaderValue("bearer", accessToken);

                var headers = _httpContextAccessor.HttpContext.Request.Headers;
                if (headers.ContainsKey("X-Correlation-ID") && !string.IsNullOrEmpty(headers["X-Correlation-ID"]))
                {
                    request.Headers.Add("X-Correlation-ID", headers["X-Correlation-ID"].ToString());
                }

                httpResponseMessage = await base.SendAsync(request, cancellationToken);
                httpResponseMessage.EnsureSuccessStatusCode();
            }
            catch (Exception ex)
            {
                _logger.LogError(ex, "Failed to run http query {RequestUri}", request.RequestUri);
                throw;
            }
            return httpResponseMessage;
        }
    }
}
```

如您所见，现在它封装了用于同一应用程序中每个[HttpClient](https://docs.microsoft.com/en-us/dotnet/api/system.net.http.httpclient?view=netcore-3.0)的冗余逻辑 。

现在，清理后的[HttpClient](https://docs.microsoft.com/en-us/dotnet/api/system.net.http.httpclient?view=netcore-3.0)如下所示：

```
using DemoRefit.Models;
using DemoRefit.Repositories;
using Microsoft.Extensions.Logging;
using System;
using System.Collections.Generic;
using System.Net.Http;
using System.Threading.Tasks;

namespace DemoRefit.HttpClients
{
    public class CountryRepositoryClientV2 : ICountryRepositoryClient
    {
        private readonly HttpClient _client;
        private readonly ILogger<CountryRepositoryClient> _logger;

        public CountryRepositoryClientV2(HttpClient client, ILogger<CountryRepositoryClient> logger)
        {
            _client = client;
            _logger = logger;
        }

        public async Task<IEnumerable<Country>> GetAsync()
        {
            using (HttpResponseMessage response = await _client.GetAsync("/api/democrud"))
            {
                try
                {
                    return await response.Content.ReadAsAsync<IEnumerable<Country>>();
                }
                catch (Exception e)
                {
                    _logger.LogError(e, "Failed to read content");
                    return null;
                }
            }
        }
    }
}
```

好多了不是吗？🙂

最后，让我们将[DelegatingHandler](https://docs.microsoft.com/en-us/dotnet/api/system.net.http.delegatinghandler?view=netframework-4.8)附加到Startup.cs中的[HttpClient](https://docs.microsoft.com/en-us/dotnet/api/system.net.http.httpclient?view=netcore-3.0)：

```
using DemoRefit.Handlers;
using DemoRefit.HttpClients;
using DemoRefit.Repositories;
using Microsoft.AspNetCore.Builder;
using Microsoft.AspNetCore.Hosting;
using Microsoft.Extensions.Configuration;
using Microsoft.Extensions.DependencyInjection;
using Microsoft.Extensions.Hosting;
using Refit;
using System;

namespace DemoRefit
{
    public class Startup
    {
        public Startup(IConfiguration configuration)
        {
            Configuration = configuration;
        }

        public IConfiguration Configuration { get; }

        // This method gets called by the runtime. Use this method to add services to the container.
        public void ConfigureServices(IServiceCollection services)
        {
            services.AddHttpContextAccessor();

            services.AddControllers();

            services.AddHttpClient<ICountryRepositoryClient, CountryRepositoryClientV2>()
                    .ConfigureHttpClient(c => c.BaseAddress = new Uri(Configuration.GetSection("Apis:CountryApi:Url").Value))
                    .AddHttpMessageHandler<MyDelegatingHandler>();
        }

        // This method gets called by the runtime. Use this method to configure the HTTP request pipeline.
        public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
        {
            if (env.IsDevelopment())
            {
                app.UseDeveloperExceptionPage();
            }

            app.UseHttpsRedirection();

            app.UseRouting();

            app.UseAuthorization();

            app.UseEndpoints(endpoints =>
            {
                endpoints.MapControllers();
            });
        }
    }
}
```



## **使用Refit**

如果您正在使用[Refit](https://github.com/reactiveui/refit)，则绝对可以重用该[DelegatingHandler](https://docs.microsoft.com/en-us/dotnet/api/system.net.http.delegatinghandler?view=netframework-4.8)！

例：

```
using DemoRefit.Handlers;
using DemoRefit.HttpClients;
using DemoRefit.Repositories;
using Microsoft.AspNetCore.Builder;
using Microsoft.AspNetCore.Hosting;
using Microsoft.Extensions.Configuration;
using Microsoft.Extensions.DependencyInjection;
using Microsoft.Extensions.Hosting;
using Refit;
using System;

namespace DemoRefit
{
    public class Startup
    {
        public Startup(IConfiguration configuration)
        {
            Configuration = configuration;
        }

        public IConfiguration Configuration { get; }

        // This method gets called by the runtime. Use this method to add services to the container.
        public void ConfigureServices(IServiceCollection services)
        {
            services.AddHttpContextAccessor();

            services.AddControllers();

            services.AddRefitClient<ICountryRepositoryClient>()
                    .ConfigureHttpClient(c => c.BaseAddress = new Uri(Configuration.GetSection("Apis:CountryApi:Url").Value));
                    .AddHttpMessageHandler<MyDelegatingHandler>();
        }

        // This method gets called by the runtime. Use this method to configure the HTTP request pipeline.
        public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
        {
            if (env.IsDevelopment())
            {
                app.UseDeveloperExceptionPage();
            }

            app.UseHttpsRedirection();

            app.UseRouting();

            app.UseAuthorization();

            app.UseEndpoints(endpoints =>
            {
                endpoints.MapControllers();
            });
        }
    }
}
```

##### 轮子介绍：

Refit是一个深受Square的 Retrofit 库启发的库,目前在github上共有star 4000枚，通过这个框架，可以把你的REST API变成了一个活的接口:

```
public interface IGitHubApi
{
    [Get("/users/{user}")]
    Task<User> GetUser(string user);
}
```

RestService类生成一个IGitHubApi的实现，它使用HttpClient进行调用:

```
var gitHubApi = RestService.For<IGitHubApi>("https://api.github.com");

var octocat = await gitHubApi.GetUser("octocat");
```

查看更多： https://reactiveui.github.io/refit/ 