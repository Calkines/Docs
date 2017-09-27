---
título: Middleware ASP.NET Core 
autor: rick-anderson
tradutor: calkines
desrição: Explica middleware e o pipeline de requisições
palavras-chave: ASP.NET Core,Middleware,pipeline,delegate
ms.author: riande
manager: wpickett
ms.date: 08/14/2017
ms.topic: article
ms.assetid: db9a86ab-46c2-40e0-baed-86e38c16af1f
ms.technology: aspnet
ms.prod: asp.net-core
uid: fundamentals/middleware
---

# Fundamentos sobre Middleware ASP.NET Core

<a name=fundamentals-middleware></a>

Por [Rick Anderson](https://twitter.com/RickAndMSFT) e [Steve Smith](https://ardalis.com/)

[Visualizar ou baixar código demonstrativo](https://github.com/aspnet/Docs/tree/master/aspnetcore/fundamentals/middleware/sample)


## O que é middleware

Middleware é um software que é agregado em um pipeline de aplicação para controlar as requisições e respostas. Cada componente:

* Escolhe quando passar a requisição para o próximo componente no pipeline.
* Pode executar trabalho antes e depois do próximo componente no pipeline em questão.

Delegados de requisição são usados para construir o pipeline de requisição. Os delegados de requisição manipulam cada requisição HTTP.

Delegados de requisição são configurados usando os métodos de extensão [Run]https://docs.microsoft.com/aspnet/core/api/microsoft.aspnetcore.builder.runextensions), [Map](https://docs.microsoft.com/aspnet/core/api/microsoft.aspnetcore.builder.mapextensions), e [Use](https://docs.microsoft.com/aspnet/core/api/microsoft.aspnetcore.builder.useextensions). Um delegado de requisição individual pode ser especificado "in-line" como um método anônimo (chamado de "in-line" middleware), ou este pode ser definido em um classe reutilizável. Estas classes reutilizáveis e métodos anônimos "in-line" são *middleware*, ou *componentes middleware*. Cada componente middleware no pipeline de requisição é responsável por invocar o próximo componente do pipeline, ou curto-circuitar a cadeia, caso seja apropriado.

[Migrar Módulos HTTP para Middleware](../migration/http-modules.md) explica a dierença entre os pipelines de requisição no ASP.NET Core e as versões anteriores, além de fornecer mais exemplos de middleware.

## Criando um pipeline middleware com IApplicationBuilder

O pipeline de requisição do ASP.NET Core consiste em uma sequencia de delegados de requisição, chamados um após o outro, como mostrado neste diagrama (a execução do thread segue as setas pretas):

![Processamento do padrão de requisição, exibindo uma chegada de requisição, que está sendo processada através de três middlewares, e a resposta está saindo da aplicação. Cada middleware executa sua lógica e libera a requisição para o próximo middleware através da declaração next(). Depois o processamento da requisição pelo terceiro middleware, a requisição é devolvida, passando de volta pelos outros dois middleware para processamento adicional, respeitando cada declaração next() e deixando a aplicação como resposta ao cliente.(middleware/_static/request-delegate-pipeline.png)]

A aplicação mais simples possível de ASP.NET Core configura apenas um delegado de requisição que controla todas requisições. Este caso não inclui um pipeline de requisições atual. Em vez disso, apenas uma função anônima é chamada para responder a cada requisição HTTP.

[!code-csharp[Main](middleware/sample/Middleware/Startup.cs)]

O primeiro delegado [app.Run](https://docs.microsoft.com/aspnet/core/api/microsoft.aspnetcore.builder.runextensions) termina o pipeline.

Você pode encadear diversos delegados de requisição com o método de extensão [app.Use](https://docs.microsoft.com/aspnet/core/api/microsoft.aspnetcore.builder.useextensions). O parâmetro `next` representa o próximo delegado no pipeline. (Lembre-se que você pode curto-circuitar o pipeline ao *não* chamar o parâmetro *next*.) Você pode geralmente realizar ações tanto antes como depois do próximo delegado, como demonstrado neste exemplo:

[!code-csharp[Main](middleware/sample/Chain/Startup.cs?name=snippet1)]

>[!WARNING]
> Não chame `next.Invoke` depois de um resposta ter sido enviado ao cliente. Mudanças no `HttpResponse` depois da resposta ter sido enviada vai causar um falha. Por exemplo, mudanças como configurações de cabeçalho, código de status, etc, vão causar a exceção. Escrever para o corpo da resposta, depois chamar o `next`:
> - Pode causar uma violação de protocolo. Por exemplo, escrever mais do que o indicado no `context-length`.
> - Pode corromper o formato do corpo da resposta. Por exemplo, escrever um rodapé HTML em um arquivo CSS.
>
> [HttpResponse.HasStarted](https://docs.microsoft.com/aspnet/core/api/microsoft.aspnetcore.http.features.httpresponsefeature#Microsoft_AspNetCore_Http_Features_HttpResponseFeature_HasStarted) é uma boa sugestão para perceber se o cabeçalho já foi enviado e/ou se o corpo da resposta foi modificado.

## Ordenação

A ordem nas quais os componentes middleware são adicinados ao método `Configure` define a order as quais eles serão invocados nas requisições, para resposta a ordem é inversa. Este ordenamento é crítico para a segurança, performance e funcionamento.

Para o método `Configure` (mostrado abaixo), adicione os componentes middleware seguintes:

1. Manipulação de erro/exceção
2. Servidor de arquivo estático
3. Autenticação
4. MVC

```csharp
public void Configure(IApplicationBuilder app)
{
    app.UseExceptionHandler("/Home/Error"); // Call first to catch exceptions
                                            // thrown in the following middleware.

    app.UseStaticFiles();                   // Return static files and end pipeline.

    app.UseIdentity();                     // Authenticate before you access
                                           // secure resources.

    app.UseMvcWithDefaultRoute();          // Add MVC to the request pipeline.
}
```

No código acima, `UseExceptionHandler` é o primeiro componente middleware adicionado ao futuro pipeline, ele captura quaisquer exceções que vierem a ocorrer nas próximas chamadas.

O middleware de arquivo estático é chamado cedo no pipeline, assim ele pode controlar requisições e circu-circuitar sem passar pelos próximos componentes. O middleware de arquivo estático **não** fornece verificações de autorização. Quaisquer arquivos servidos por ele, incluindo aqueles da pasta *wwwroot*, estão disponíveis publicamente. Veja [Trabalhando com arquivos estáticos](xref:fundamentals/static-files) para uma visão de como proteger arquivos estáticos.

Se a requisição não for manipulada pelo middleware de arquivo estático, ela será passada para o middleware de identidade (`app.UseIdentity`), o qual cuida da autenticação. Este middleware, de identidade, não curto-circuitará requisições não autenticadas. Contudo o middleware de identidade autentica requisições, mas autorização (ou rejeição) ocorre apenas depois do MVC selecionar um *controller* específico e sua respectiva ação.

O exemplo seguinte demonstra um ordenamento de middleware, no qual as requisições para arquivos estáticos são manipuladas pelo middleware de arquivos estáticos antes do middleware de compressão de resposta. Arquivos estáticos não são comprimidos nesta ordem apresentada. As repostas MVC do método [UseMvcWithDefaultRoute](https://docs.microsoft.com/aspnet/core/api/microsoft.aspnetcore.builder.mvcapplicationbuilderextensions#Microsoft_AspNetCore_Builder_MvcApplicationBuilderExtensions_UseMvcWithDefaultRoute_Microsoft_AspNetCore_Builder_IApplicationBuilder_) podem ser comprimidas. 


```csharp
public void Configure(IApplicationBuilder app)
{
    app.UseStaticFiles();         // Static files not compressed
                                  // by middleware.
    app.UseResponseCompression();
    app.UseMvcWithDefaultRoute();
}
```

<a name=middleware-run-map-use></a>

### Usar, Executar e Mapear *(Use, Run e Map)*

Você configura o pipeline HTTP usando os métodos `Use`,`Run` e `Map`. O método `Use` pode curto-circuitar o pipeline (isso é, se ele não chamar um próximo delegado de requisição, através do método `next`. `Run` é uma convenção, e alguns componentes middleware podem expor métodos `Run[Middleware]`, que executam no final do pipeline.

Extensões `Map*` são usadas como convenções para ramificar o pipeline. [Mapas](https://docs.microsoft.com/aspnet/core/api/microsoft.aspnetcore.builder.mapextensions) ramificam o pipeline de requisição baseado-se na correspondência de determinados caminhos da requisição. Se o caminho requisitado coindidir com o caminho informado no *Map*, a ramificação será executada. 

[!code-csharp[Main](middleware/sample/Chain/StartupMap.cs?name=snippet1)]

A tabela seguinte mostra as requisições e respostas de `http://localhost:1234` usando o códio anterior:


| Requisição | Resposta |
| --- | --- |
| localhost:1234 | Hello from non-Map delegate.  |
| localhost:1234/map1 | Map Test 1 |
| localhost:1234/map2 | Map Test 2 |
| localhost:1234/map3 | Hello from non-Map delegate.  |

Quando o método `Map` é usado, o segmento coindicente é removido de `HttpRequest.Path` e apensado a `HttpRequest.PathBase` para cada requisição.

When `Map` is used, the matched path segment(s) are removed from `HttpRequest.Path` and appended to `HttpRequest.PathBase` for each request.

O métod [MapWhen](https://docs.microsoft.com/aspnet/core/api/microsoft.aspnetcore.builder.mapwhenextensions) ramifica o pipeline de requisição com base no resultado de determinado predicado. Qualquer predicado do tipo `Func<HttpContext, bool>` pode ser usado para mapear requisições para uma nova ramificação do pipeline. No exemplo seguinte, um predicado é usado para detectar a presença de uma variável de texto de consulta chamada `branch`;

[!code-csharp[Main](middleware/sample/Chain/StartupMapWhen.cs?name=snippet1)]

The following table shows the requests and responses from `http://localhost:1234` using the previous code:

| Request | Response |
| --- | --- |
| localhost:1234 | Hello from non-Map delegate.  |
| localhost:1234/?branch=master | Branch used = master|

`Map` supports nesting, for example:

```csharp
app.Map("/level1", level1App => {
       level1App.Map("/level2a", level2AApp => {
           // "/level1/level2a"
           //...
       });
       level1App.Map("/level2b", level2BApp => {
           // "/level1/level2b"
           //...
       });
   });
   ```

`Map` can also match multiple segments at once, for example:

 ```csharp
app.Map("/level1/level2", HandleMultiSeg);
```

## Built-in middleware

ASP.NET Core ships with the following middleware components:

| Middleware | Description |
| ----- | ------- |
| [Authentication](xref:security/authentication/identity) | Provides authentication support. |
| [CORS](xref:security/cors) | Configures Cross-Origin Resource Sharing. |
| [Response Caching](xref:performance/caching/middleware) | Provides support for caching responses. |
| [Response Compression](xref:performance/response-compression) | Provides support for compressing responses. |
| [Routing](xref:fundamentals/routing) | Defines and constrains request routes. |
| [Session](xref:fundamentals/app-state) | Provides support for managing user sessions. |
| [Static Files](xref:fundamentals/static-files) | Provides support for serving static files and directory browsing. |
| [URL Rewriting Middleware](xref:fundamentals/url-rewriting) | Provides support for rewriting URLs and redirecting requests. |

<a name=middleware-writing-middleware></a>

## Writing middleware

Middleware is generally encapsulated in a class and exposed with an extension method. Consider the following middleware, which sets the culture for the current request from the query string:

[!code-csharp[Main](middleware/sample/Culture/StartupCulture.cs?name=snippet1)]

Note: The sample code above is used to demonstrate creating a middleware component. See [
Globalization and localization](xref:fundamentals/localization) for ASP.NET Core's built-in localization support.

You can test the middleware by passing in the culture, for example `http://localhost:7997/?culture=no`.

The following code moves the middleware delegate to a class:

[!code-csharp[Main](middleware/sample/Culture/RequestCultureMiddleware.cs)]

The following extension method exposes the middleware through [IApplicationBuilder](https://docs.microsoft.com/aspnet/core/api/microsoft.aspnetcore.builder.iapplicationbuilder):

[!code-csharp[Main](middleware/sample/Culture/RequestCultureMiddlewareExtensions.cs)]

The following code calls the middleware from `Configure`:

[!code-csharp[Main](middleware/sample/Culture/Startup.cs?name=snippet1&highlight=5)]

Middleware should follow the [Explicit Dependencies Principle](http://deviq.com/explicit-dependencies-principle/) by exposing its dependencies in its constructor. Middleware is constructed once per *application lifetime*. See *Per-request dependencies* below if you need to share services with middleware within a request.

Middleware components can resolve their dependencies from dependency injection through constructor parameters. [`UseMiddleware<T>`](https://docs.microsoft.com/aspnet/core/api/microsoft.aspnetcore.builder.usemiddlewareextensions#methods_summary) can also accept additional parameters directly.

### Per-request dependencies

Because middleware is constructed at app startup, not per-request, *scoped* lifetime services used by middleware constructors are not  shared with other dependency-injected types during each request. If you must share a *scoped* service between your middleware and other types, add these services to the `Invoke` method's signature. The `Invoke` method can accept additional parameters that are populated by dependency injection. For example:

```c#
public class MyMiddleware
{
    private readonly RequestDelegate _next;

    public MyMiddleware(RequestDelegate next)
    {
        _next = next;
    }

    public async Task Invoke(HttpContext httpContext, IMyScopedService svc)
    {
        svc.MyProperty = 1000;
        await _next(httpContext);
    }
}
```

## Resources

* [Sample code used in this doc](https://github.com/aspnet/Docs/tree/master/aspnetcore/fundamentals/middleware/sample)
* [Migrating HTTP Modules to Middleware](../migration/http-modules.md)
* [Application Startup](startup.md)
* [Request Features](request-features.md)
