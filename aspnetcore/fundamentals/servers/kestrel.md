---
título: Implementação do servidor web Kestrel no ASP.NET Core
autor: tdykstra
tradutor: calkines
descrição: Apresentação Kestrel, o servidor web multi-plataforma para ASP.NET Core baseado no libuv.
palavra-chave: ASP.NET Core,Kestrel,libuv,prefixos url
ms.author: tdykstra
manager: wpickett
ms.date: 08/02/2017
ms.topic: article
ms.assetid: 560bd66f-7dd0-4e68-b5fb-f31477e4b785
ms.technology: aspnet
ms.prod: asp.net-core
uid: fundamentals/servers/kestrel
ms.custom: H1Hack27Feb2017
---

# Introdução à implementação do servidor web Kestrel no ASP.NET Core

Por [Tom Dykstra](https://github.com/tdykstra), [Chris Ross](https://github.com/Tratcher), and [Stephen Halter](https://twitter.com/halter73)

O Kestrel é um [Servidor web para ASP.NET Core](index.md) multi-plataforma baseado no [libuv](https://github.com/libuv/libuv), uma biblioteca I/O multi-plataforma assíncrona. Kestrel é o servidor web que é incluído por padrão nos modelos de projeto do ASP.NET Core.

O Kestrel suporta os seguintes recursos:

* HTTPS
* Atualização opaca usada para permitir [WebSockets](https://github.com/aspnet/websockets)
* Sockets Unix para alta performance por trás de Nginx.

O Kestrel é suportado em todas plataformas e versões que o .NET Core suporta.

# [ASP.NET Core 2.x](#tab/aspnetcore2x)

[Visualizar ou baixar o código demonstrativo para 2.x](https://github.com/aspnet/Docs/tree/master/aspnetcore/fundamentals/servers/kestrel/sample2)

# [ASP.NET Core 1.x](#tab/aspnetcore1x)

[Visualizar ou baixar o código demonstrativo 1.x](https://github.com/aspnet/Docs/tree/master/aspnetcore/fundamentals/servers/kestrel/sample1)

---

## Quando usar o Kestrel com um proxy reverso

# [ASP.NET Core 2.x](#tab/aspnetcore2x)

Você pode usar somente o Kestrel ou combiná-lo com um *servidor de proxy reverso*, como o IIS, Nginx ou Apache. O servidor de proxy reverso recebe requisições HTTP da internet e as encaminha para o Kestrel após alguns considerações preliminares.

![Kestrel comunica-se diretamente com a Internet sem um servidor de proxy reverso](kestrel/_static/kestrel-to-internet2.png)

![Kestrel comunica-se indiretamente com a Internet através de um servidor de proxy reverso, como o IIS, Nginx ou Apache](kestrel/_static/kestrel-to-internet.png)

Ambas configurações *mdassh; com o sem um servidor de proxy reverso &mdash; podem ser usadas se o Kestrel for exposto apenas para uma rede interna.

# [ASP.NET Core 1.x](#tab/aspnetcore1x)

Se sua aplicação aceitar requisições apenas de uma rede interna, você pode usar somente o próprio Kestrel.

![Kestrel comunica-se diretamente com sua rede interna](kestrel/_static/kestrel-to-internal.png)

Se você expuser sua aplicação para a Internet, você precisa usar o IIS, Nginx ou o Apache como *servidor de proxy reverso*. Um servidor de proxy reverso recebe solicitações HTTP da Internet e as encaminha para o Kestrel, após algumas considerações preliminares.

![Kestrel comunica-se indiretamente com a Internet através de um servidor de proxy reverso, como o IIS, Nginx ou Apache](kestrel/_static/kestrel-to-internet.png)

Um proxy reverso é requerido para implantações de borda (expostas ao tráfego da Internet) por razões de segurança. A versão 1.x do Kestrel não possui um complemento total de defesas contra ataques. Isso incluí, mas não com limite apropriado, expiração por tempo, limites por tamanho, e limitaçõe de conexões atuais.

---

Um cenário que requer um proxy reverso é quando você tem diversas aplicações que compartilham o mesmo IP e porta, sendo executadas em um único servidor. Isso não funciona diretamente com o Kestrel porque este não suporta compartilhamento do mesmo IP e porta entre processos múltiplos. Quando você configura o Kestrel para atender a uma porta, ele manipula todo tráfego para aquela porta, independente do cabeçalho do host. Um proxy reverso que pode compartilhar portas precisa encaminhar os cabeçalhos divididos para o Kestrel em um único IP e porta.

Mesmo que um servidor de proxy reverso não seja requerido, usar um pode ser uma boa escohe por outras rasões:

* Limitar a area de exposição de sua aplicação.
* Contar com uma camada adicional de configuração e defesa.
* Possibilidade de melhor integração com uma infraestrutura existente.
* Simplifica o processo de carregamento balanceado e cofigurações de SSL. Seu servidor de proxy reverso requer apenas um certificado SSL, e ele poderá comunicar-se com seus servidores de aplicação na rede interna usando simples HTTP.

## Como usar o Kestrel nas aplicações ASP.NET Core

# [ASP.NET Core 2.x](#tab/aspnetcore2x)

O pacote [Microsoft.AspNetCore.Server.Kestrel](https://www.nuget.org/packages/Microsoft.AspNetCore.Server.Kestrel/) é incluído no [Microsoft.AspNetCore.All meta-pacote](xref:fundamentals/metapackage).

O modelo de projeto do ASP.NET Core usa o Kestrel por padrão. No *Program.cs*, o código modelo chama `CreateDefaultBuilder`, que chama [UseKestrel](https://docs.microsoft.com/aspnet/core/api/microsoft.aspnetcore.hosting.webhostbuilderkestrelextensions#Microsoft_AspNetCore_Hosting_WebHostBuilderKestrelExtensions_UseKestrel_Microsoft_AspNetCore_Hosting_IWebHostBuilder_) por trás das cenas, nos bastidores.

[!code-csharp[](kestrel/sample2/Program.cs?name=snippet_DefaultBuilder&highlight=7)]

Se você precisar configurar as opções do Kestrel, chame `UseKestrel` no *Program.cs* como exibido no exemplo seguinte:

[!code-csharp[](kestrel/sample2/Program.cs?name=snippet_DefaultBuilder&highlight=9-16)]

# [ASP.NET Core 1.x](#tab/aspnetcore1x)

Install the [Microsoft.AspNetCore.Server.Kestrel](https://www.nuget.org/packages/Microsoft.AspNetCore.Server.Kestrel/) NuGet package.

Call the [UseKestrel](https://docs.microsoft.com/aspnet/core/api/microsoft.aspnetcore.hosting.webhostbuilderkestrelextensions#Microsoft_AspNetCore_Hosting_WebHostBuilderKestrelExtensions_UseKestrel_Microsoft_AspNetCore_Hosting_IWebHostBuilder_) extension method on `WebHostBuilder` in your `Main` method, specifying any [Kestrel options](https://docs.microsoft.com/aspnet/core/api/microsoft.aspnetcore.server.kestrel.kestrelserveroptions) that you need, as shown in the next section.

[!code-csharp[](kestrel/sample1/Program.cs?name=snippet_Main&highlight=13-19)]

---

### Opções do Kestrel

# [ASP.NET Core 2.x](#tab/aspnetcore2x)

O servidor web Kestrel possui opções limitação para configuração que são especialmente úteis em implantações que lidam com a Internet. Veja alguns limites que você pode configurar:
- Número máximo de conexões de clientes
- Tamanho máximo para o corpo da requisição
- Taxa de dados mínima para o corpo da requisição

Você configura essas limitações, além de outras, na propriedade `Limits` da classe [KestrelServerOptions](https://github.com/aspnet/KestrelHttpServer/blob/rel/2.0.0/src/Microsoft.AspNetCore.Server.Kestrel.Core/KestrelServerOptions.cs). Esta propriedade mantem uma instância da classe [KestrelServerLimits](https://github.com/aspnet/KestrelHttpServer/blob/rel/2.0.0/src/Microsoft.AspNetCore.Server.Kestrel.Core/KestrelServerLimits.cs).

**Número máximo de conexões de clientes**

O número máximo de conexões TCP abertas no momento atual pode ser configurado para toda aplicação com o código seguinte:

[!code-csharp[](kestrel/sample2/Program.cs?name=snippet_Limits&highlight=3-4)]

Existe um limite separado para conexões que precisam ser atualizadas de HTTP ou HTTPS para outro protocolo (com por exemplo, em requisições WebSockets). Após uma conexão ser atualizada, ela não é mais considerada pelo limite de `MaxConcurrentConnections`.

O número máximo de conexões é, por padrão, ilimitado (vazio).

**Tamanho máximo para o corpo da requisição **

O valor padrão para o tamanho máximo do corpo da requisição é 30,000,000 bytes, que é aproximadamente 28.6MB.

A maneira recomendada de sobrescrever o limite deste item em uma aplicação ASP.NET Core MVC é usar o atributo [RequestSizeLimite](https://github.com/aspnet/Mvc/blob/rel/2.0.0/src/Microsoft.AspNetCore.Mvc.Core/RequestSizeLimitAttribute.cs) em um método de ação:

The recommended way to override the limit in an ASP.NET Core MVC app is to use the [RequestSizeLimit](https://github.com/aspnet/Mvc/blob/rel/2.0.0/src/Microsoft.AspNetCore.Mvc.Core/RequestSizeLimitAttribute.cs) attribute on an action method:

```csharp
[RequestSizeLimit(100000000)]
public IActionResult MyActionMethod()
```

Here's an example that shows how to configure the constraint for the entire application, every request:

[!code-csharp[](kestrel/sample2/Program.cs?name=snippet_Limits&highlight=5)]

You can override the setting on a specific request in middleware:

[!code-csharp[](kestrel/sample2/Startup.cs?name=snippet_Limits&highlight=3-4)]
 
An exception is thrown if you try to configure the limit on a request after the application has started reading the request. There's an `IsReadOnly` property that tells you if the `MaxRequestBodySize` property is in read-only state, meaning it's too late to configure the limit.

**Minimum request body data rate**

Kestrel checks every second if data is coming in at the specified rate in bytes/second. If the rate drops below the minimum, the connection is timed out. The grace period is the amount of time that Kestrel gives the client to increase its send rate up to the minimum; the rate is not checked during that time. The grace period helps avoid dropping connections that are initially sending data at a slow rate due to TCP slow-start.

The default minimum rate is 240 bytes/second, with a 5 second grace period.

A minimum rate also applies to the response. The code to set the request limit and the response limit is the same except for having `RequestBody` or `Response` in the property and interface names. 

Here's an example that shows how to configure the minimum data rates in *Program.cs*:

[!code-csharp[](kestrel/sample2/Program.cs?name=snippet_Limits&highlight=6-9)]

You can configure the rates per request in middleware:

[!code-csharp[](kestrel/sample2/Startup.cs?name=snippet_Limits&highlight=5-8)]

For information about other Kestrel options, see the following classes:

* [KestrelServerOptions](https://github.com/aspnet/KestrelHttpServer/blob/rel/2.0.0/src/Microsoft.AspNetCore.Server.Kestrel.Core/KestrelServerOptions.cs)
* [KestrelServerLimits](https://github.com/aspnet/KestrelHttpServer/blob/rel/2.0.0/src/Microsoft.AspNetCore.Server.Kestrel.Core/KestrelServerLimits.cs)
* [ListenOptions](https://github.com/aspnet/KestrelHttpServer/blob/rel/2.0.0/src/Microsoft.AspNetCore.Server.Kestrel.Core/ListenOptions.cs)

# [ASP.NET Core 1.x](#tab/aspnetcore1x)

For information about Kestrel options, see [KestrelServerOptions class](https://docs.microsoft.com/aspnet/core/api/microsoft.aspnetcore.server.kestrel.kestrelserveroptions).

---

### Endpoint configuration

# [ASP.NET Core 2.x](#tab/aspnetcore2x)

By default ASP.NET Core binds to `http://localhost:5000`. You configure URL prefixes and ports for Kestrel to listen on by calling `Listen` or `ListenUnixSocket` methods on `KestrelServerOptions`. (`UseUrls`, the `urls` command-line argument, and the ASPNETCORE_URLS environment variable also work but have the limitations noted [later in this article](#useurls-limitations).)

**Bind to a TCP socket**

The `Listen` method binds to a TCP socket, and an options lambda lets you configure an SSL certificate:

[!code-csharp[](kestrel/sample2/Program.cs?name=snippet_DefaultBuilder&highlight=9-16)]

Notice how this example configures SSL for a particular endpoint by using [ListenOptions](https://github.com/aspnet/KestrelHttpServer/blob/rel/2.0.0/src/Microsoft.AspNetCore.Server.Kestrel.Core/ListenOptions.cs). You can use the same API to configure other Kestrel settings for particular endpoints.

[!INCLUDE[How to make an SSL cert](../../includes/make-ssl-cert.md)]

**Bind to a Unix socket**

You can listen on a Unix socket for improved performance with Nginx, as shown in this example:

[!code-csharp[](kestrel/sample2/Program.cs?name=snippet_UnixSocket)]

**Port 0**

If you specify port number 0, Kestrel dynamically binds to an available port. The following example shows how to determine which port Kestrel actually bound to at runtime:

[!code-csharp[](kestrel/sample2/Startup.cs?name=snippet_Configure&highlight=3,13,16-17)]

<a id="useurls-limitations"></a>

**UseUrls limitations**

You can configure endpoints by calling the `UseUrls` method or using the `urls` command-line argument or the ASPNETCORE_URLS environment variable. These methods are useful if you want your code to work with servers other than Kestrel. However, be aware of these limitations:

* You can't use SSL with these methods.
* If you use both the `Listen` method and `UseUrls`, the `Listen` endpoints override the `UseUrls` endpoints.

**Endpoint configuration for IIS**

If you use IIS, the URL bindings for IIS override any bindings that you set by calling either `Listen` or `UseUrls`. For more information, see [Introduction to ASP.NET Core Module](aspnet-core-module.md).

# [ASP.NET Core 1.x](#tab/aspnetcore1x)

By default ASP.NET Core binds to `http://localhost:5000`. You can configure URL prefixes and ports for Kestrel to listen on by using the `UseUrls` extension method, the `urls` command-line argument, or the ASP.NET Core configuration system. For more information about these methods, see [Hosting](../../fundamentals/hosting.md). For information about how URL binding works when you use IIS as a reverse proxy, see [ASP.NET Core Module](aspnet-core-module.md). 

---

### URL prefixes

If you call `UseUrls` or use the `urls` command-line argument or ASPNETCORE_URLS environment variable, the URL prefixes can be in any of the following formats. 

# [ASP.NET Core 2.x](#tab/aspnetcore2x)

Only HTTP URL prefixes are valid; Kestrel does not support SSL when you configure URL bindings by using `UseUrls`.

* IPv4 address with port number

  ```
  http://65.55.39.10:80/
  ```

  0.0.0.0 is a special case that binds to all IPv4 addresses.


* IPv6 address with port number

  ```
  http://[0:0:0:0:0:ffff:4137:270a]:80/ 
  ```

  [::] is the IPv6 equivalent of IPv4 0.0.0.0.


* Host name with port number

  ```
  http://contoso.com:80/
  http://*:80/
  ```

  Host names, *, and +, are not special. Anything that is not a recognized IP address or "localhost" will bind to all IPv4 and IPv6 IPs. If you need to bind different host names to different ASP.NET Core applications on the same port, use [HTTP.sys](httpsys.md) or a reverse proxy server such as IIS, Nginx, or Apache.

* "Localhost" name with port number or loopback IP with port number

  ```
  http://localhost:5000/
  http://127.0.0.1:5000/
  http://[::1]:5000/
  ```

  When `localhost` is specified, Kestrel tries to bind to both IPv4 and IPv6 loopback interfaces. If the requested port is in use by another service on either loopback interface, Kestrel fails to start. If either loopback interface is unavailable for any other reason (most commonly because IPv6 is not supported), Kestrel logs a warning. 

# [ASP.NET Core 1.x](#tab/aspnetcore1x)

* IPv4 address with port number

  ```
  http://65.55.39.10:80/
  https://65.55.39.10:443/
  ```

  0.0.0.0 is a special case that binds to all IPv4 addresses.


* IPv6 address with port number

  ```
  http://[0:0:0:0:0:ffff:4137:270a]:80/ 
  https://[0:0:0:0:0:ffff:4137:270a]:443/ 
  ```

  [::] is the IPv6 equivalent of IPv4 0.0.0.0.


* Host name with port number

  ```
  http://contoso.com:80/
  http://*:80/
  https://contoso.com:443/
  https://*:443/
  ```

  Host names, \*, and + aren't special. Anything that isn't a recognized IP address or "localhost" binds to all IPv4 and IPv6 IPs. If you need to bind different host names to different ASP.NET Core applications on the same port, use [WebListener](weblistener.md) or a reverse proxy server such as IIS, Nginx, or Apache.

* "Localhost" name with port number or loopback IP with port number

  ```
  http://localhost:5000/
  http://127.0.0.1:5000/
  http://[::1]:5000/
  ```

  When `localhost` is specified, Kestrel tries to bind to both IPv4 and IPv6 loopback interfaces. If the requested port is in use by another service on either loopback interface, Kestrel fails to start. If either loopback interface is unavailable for any other reason (most commonly because IPv6 is not supported), Kestrel logs a warning. 

* Unix socket

  ```
  http://unix:/run/dan-live.sock  
  ```

**Port 0**

If you specify port number 0, Kestrel dynamically binds to an available port. Binding to port 0 is allowed for any host name or IP except for `localhost` name.

The following example shows how to determine which port Kestrel actually bound to at runtime:

[!code-csharp[](kestrel/sample1/Startup.cs?name=snippet_Configure)]

**URL prefixes for SSL**

Be sure to include URL prefixes with `https:` if you call the `UseHttps` extension method, as shown below.

```csharp
var host = new WebHostBuilder() 
    .UseKestrel(options => 
    { 
        options.UseHttps("testCert.pfx", "testPassword"); 
    }) 
   .UseUrls("http://localhost:5000", "https://localhost:5001") 
   .UseContentRoot(Directory.GetCurrentDirectory()) 
   .UseStartup<Startup>() 
   .Build(); 
```

> [!NOTE]
> HTTPS and HTTP cannot be hosted on the same port.

[!INCLUDE[How to make an SSL cert](../../includes/make-ssl-cert.md)]

---
## Next steps

For more information, see the following resources:

# [ASP.NET Core 2.x](#tab/aspnetcore2x)

* [Sample app for 2.x](https://github.com/aspnet/Docs/tree/master/aspnetcore/fundamentals/servers/kestrel/sample2)
* [Kestrel source code](https://github.com/aspnet/KestrelHttpServer)

# [ASP.NET Core 1.x](#tab/aspnetcore1x)

* [Sample app for 1.x](https://github.com/aspnet/Docs/tree/master/aspnetcore/fundamentals/servers/kestrel/sample1)
* [Kestrel source code](https://github.com/aspnet/KestrelHttpServer)

---
