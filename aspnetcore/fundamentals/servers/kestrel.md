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

```csharp
[RequestSizeLimit(100000000)]
public IActionResult MyActionMethod()
```

Veja um exemplo de como configurar um limite para toda aplicação, para cara requisição:

[!code-csharp[](kestrel/sample2/Program.cs?name=snippet_Limits&highlight=5)]

Você pode sobrescrever um requisição específica em um determinado middleware:

[!code-csharp[](kestrel/sample2/Startup.cs?name=snippet_Limits&highlight=3-4)]

Uma exceção acontece se você tentar configurar o limite de uma requisição depois que a aplicação tenha iniado a leitura da requisição. Existe uma propriedade `IsReadOnly` que lhe informa caso a propriedade `MaxRequestBodySize` esteja em um estado de somente leitura, dando a ideia de que é muito tarde para configurar o limite.

**Taxa de dados mínima para o corpo da requisição´** 

O Kestrel verifica a cada segundo se a informação está chegando in um taxa específica de bytes por segundo. Caso a taxa caia para abaixo do mínimo, a conexão expira-se. Período de carência é o tempo total que o Kestrel disponibiliza ao cliente para que ele aumente sua taxa de envio para acima do mínimo; a taxa não é verificada durante neste momento. O período de carência ajuda a evitar a derrubada de conexões que estão inicialmente enviando dados a uma taxa lenta durante o inicio-vagaroso do TCP.

O taxa padrão mínima é 240 bytes por segundo, com 5 segundos de período de carência.

A taxa mínima também é aplicada a uma resposta. O código para configurar o limite de requisição e respota é o mesmo, exceto por haver uma diferença no nome das propriedades e interfaces, `RequestBody` ou `Response`.

Veja um exemplo que mostra como configurar as taxas de dados mínimas no *Program.cs*:

[!code-csharp[](kestrel/sample2/Program.cs?name=snippet_Limits&highlight=6-9)]

Você pode configurar as taxas para requisição no middleware:

[!code-csharp[](kestrel/sample2/Startup.cs?name=snippet_Limits&highlight=5-8)]

Para informações sobre outras opções do Kestrel, veja as classes seguintes:

* [KestrelServerOptions](https://github.com/aspnet/KestrelHttpServer/blob/rel/2.0.0/src/Microsoft.AspNetCore.Server.Kestrel.Core/KestrelServerOptions.cs)
* [KestrelServerLimits](https://github.com/aspnet/KestrelHttpServer/blob/rel/2.0.0/src/Microsoft.AspNetCore.Server.Kestrel.Core/KestrelServerLimits.cs)
* [ListenOptions](https://github.com/aspnet/KestrelHttpServer/blob/rel/2.0.0/src/Microsoft.AspNetCore.Server.Kestrel.Core/ListenOptions.cs)

# [ASP.NET Core 1.x](#tab/aspnetcore1x)

Para informações sobre opções do Kestrel, veja [Classe KestrelServerOptions]
(https://docs.microsoft.com/aspnet/core/api/microsoft.aspnetcore.server.kestrel.kestrelserveroptions).

---

### Configuração do Endpoint

# [ASP.NET Core 2.x](#tab/aspnetcore2x)

Por padrão o ASP.NET Core é vinculado ao endereço `http://localhost:5000`. Você pode configurar prefixos de URL e portas para o Kestrel atender ou escutar, para isso é necessário chamar os métodos `Listen`  ou `ListenUnixSocket` em `KestrelServerOptions`. (`UseUrls`, o argumento de linha de comando `urls`, além da variável de ambiente ASPNETCORE_URLS também funcionam, mas têm algumas limitações [mais adiante neste artigo](#useurls-limitations).)

**Vincular a um socket TCP**

O método `Listen` vincula-se a um socket TCP, e algumas opções lambda permitem a você configurar um certificado SSL:

[!code-csharp[](kestrel/sample2/Program.cs?name=snippet_DefaultBuilder&highlight=9-16)]

Perceba como este exemplo configura o SSL para um endpoint particular ao usar [ListenOptions](https://github.com/aspnet/KestrelHttpServer/blob/rel/2.0.0/src/Microsoft.AspNetCore.Server.Kestrel.Core/ListenOptions.cs). Você pode usar a mesma API para configurar outras configurações do Kestrel para endpoint particulares.

[!INCLUDE[How to make an SSL cert](../../includes/make-ssl-cert.md)]

**Vincular a um socket Unix**

Você pode ouvir/atender a um socket Unix para aprimorar a performance com Nginx, como demonstrado neste exemplo:

[!code-csharp[](kestrel/sample2/Program.cs?name=snippet_UnixSocket)]

**Porta 0**

Se você especificar uma porta número 0, o Kestrel dinamicamente vincula-se a uma porta disponível. O exemplo seguinte mostra como determinar a qual porta atual o Kestrel está vinculado no tempo de execução: 

[!code-csharp[](kestrel/sample2/Startup.cs?name=snippet_Configure&highlight=3,13,16-17)]

<a id="useurls-limitations"></a>

**UseUrls limitations**

Você pode configurar os endpoints ao chamar o método `UseUrls`, usar o argumento de linha de comando `urls` ou através da variável de ambiente ASPNETCORE_URLS. Estes métodos são úteis se você quiser que seu código trabalhe com servidor diferentes do Kestrel. Contudo, esteja ciente destas limitações:

* Você não pode usar SSL com estes métodos.
* Se você usar os dois métodos, `Listen` e `UseUrls`, as informações de endpoint passadas para o método `Listen` sobrescrevem as passadas para o `UseUrls`.

**Configuração de EndPoint para IIS**

Se você usar IIS, o vínculo de URL do IIS sobrescreve qualquer regra de vinculo que você tenha criado através dos métodos `Listen` ou `UseUrls`. Para mais informações, veja [Introdução ao Módulo ASP.NET Core](aspnet-core-module.md).

# [ASP.NET Core 1.x](#tab/aspnetcore1x)

Por padrão o ASP.NET Core é vinculado ao endereço `http://localhost:5000`. Você pode configurar prefixos de URL e portas para o Kestrel atender ou escutar, para isso é necessário chamar o método de extensão `UseUrls`, o argumento para linha de comando `urls` ou o sistema de configuração do ASP.NET Core. Para mais informações sobre estes métodos, veja [Hospedagem](../../fundamentals/hosting.md). Para informações sobre como funciona o vínculo de URL quando você usa o IIS como servidor de proxy reverso, veja [Módulo ASP.NET Core](aspnet-core-module.md). 

---

### Prefixos de URL

Se você chamar `UseUrls` ou usar o argumento de linha de comando `urls` ou a variável de ambiente ASPNETCORE_URLS, os prefixos de URL podem ser em qualquer um destes formatos:

# [ASP.NET Core 2.x](#tab/aspnetcore2x)

Somente prefixos HTTP URL são validos; o Kestrel não suporta SSL quando você configura os vinculos de URL através do método `UseUrls`.

* Endereço IPv4 e número de porta

  ```
  http://65.55.39.10:80/
  ```

  0.0.0.0 é um caso especial que vincula todos os endereços IPv4.

* Endereço IPv6 e número de porta

  ```
  http://[0:0:0:0:0:ffff:4137:270a]:80/ 
  ```

  [::] é o IPv6 que equivale ao IPv4 0.0.0.0.


* Nome de host e número de porta

  ```
  http://contoso.com:80/
  http://*:80/
  ```

  Nome de hosts, *, e +, não são especiais. Qualquer coisa que não seja um endereço de IP reconhecido ou um "localhost" será vinculado a todos IPv4 e IPv6. Se você precisa vincular diferentes nome de hosts à diferentes aplicações ASP.NET Core na mesma porta, use o [HTTP.sys](httpsys.md) ou um proxy reverso como o IIS, Nginx ou Apache.
  
* Nome de "Localhost" e número da porta ou loopback IP e número de porta

  ```
  http://localhost:5000/
  http://127.0.0.1:5000/
  http://[::1]:5000/
  ```

  Quando `localhost` é especificado, o Kestrel tenta vincular-se as interfaces tanto de IPv4 quanto de IPv6. Se a porta requisitada estiver em uso por outro serviço ou mesmo por uma interface de loopback, o Kestrel apresenta um falha na inicialização. Se a interface de loopback não estiver disponível por qualquer outro motivo (mais comumente porque IPv6 não é suportado), o Kestrel armazena um alerta como log.

# [ASP.NET Core 1.x](#tab/aspnetcore1x)

* Endereço IPv4 e número de porta

  ```
  http://65.55.39.10:80/
  https://65.55.39.10:443/
  ```

  0.0.0.0 é um caso especial que vincula todos os endereços IPv4.

* Endereço IPv6 e número de porta

  ```
  http://[0:0:0:0:0:ffff:4137:270a]:80/ 
  https://[0:0:0:0:0:ffff:4137:270a]:443/ 
  ```

  [::] é o IPv6 que equivale ao IPv4 0.0.0.0.


* Nome de host e número de porta

  ```
  http://contoso.com:80/
  http://*:80/
  https://contoso.com:443/
  https://*:443/
  ```

  Nome de hosts, *, e +, não são especiais. Qualquer coisa que não seja um endereço de IP reconhecido ou um "localhost" será vinculado a todos IPv4 e IPv6. Se você precisa vincular diferentes nome de hosts à diferentes aplicações ASP.NET Core na mesma porta, use o [WebListener](weblistener.md) ou um proxy reverso como o IIS, Nginx ou Apache.

* Nome de "Localhost" e número da porta ou loopback IP e número de porta

  ```
  http://localhost:5000/
  http://127.0.0.1:5000/
  http://[::1]:5000/
  ```

  Quando `localhost` é especificado, o Kestrel tenta vincular-se as interfaces tanto de IPv4 quanto de IPv6. Se a porta requisitada estiver em uso por outro serviço ou mesmo por uma interface de loopback, o Kestrel apresenta um falha na inicialização. Se a interface de loopback não estiver disponível por qualquer outro motivo (mais comumente porque IPv6 não é suportado), o Kestrel armazena um alerta como log.  

* Socket Unix

  ```
  http://unix:/run/dan-live.sock  
  ```

**Port 0**

Se você especificar uma porta número 0, o Kestrel dinamicamente vincula-se a uma porta disponível. Vinculo à porta  é permitido para qualquer nome de host ou IP, exceto para o nome `localhost`.

O Exemplo seguinte mostra como determinar qual porta o Kestrel vincula-se em tempo de execução:

[!code-csharp[](kestrel/sample1/Startup.cs?name=snippet_Configure)]

**Prefixos URL para SSL**

Tenha certeza de ter incluído os prefixos URL com `https:` se você chamar o método de extensão `UseHttps`, como demostrado abaixo:

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
> HTTPS e HTTP não podem ser hospedados na mesma porta.

[!INCLUDE[Como fazer um certificado SSL](../../includes/make-ssl-cert.md)]

---
## Próximos passos

Para mais informações, veja os recursos seguintes:

# [ASP.NET Core 2.x](#tab/aspnetcore2x)

* [Aplicação demonstrativa para 2.x](https://github.com/aspnet/Docs/tree/master/aspnetcore/fundamentals/servers/kestrel/sample2)
* [Código fonte Kestrel](https://github.com/aspnet/KestrelHttpServer)

# [ASP.NET Core 1.x](#tab/aspnetcore1x)

* [Aplicação demonstrativa para 2.x](https://github.com/aspnet/Docs/tree/master/aspnetcore/fundamentals/servers/kestrel/sample1)
* [Código fonte Kestrel](https://github.com/aspnet/KestrelHttpServer)

---
