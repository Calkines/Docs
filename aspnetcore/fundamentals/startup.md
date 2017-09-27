---
título: Inicialização da aplicação no ASP.NET Core
autor: ardalis
tradutor: calkines
descrição: Aborda a classe Startup no ASP.NET Core.
keywords: ASP.NET Core,Startup,método Configure,método ConfigureServices 
ms.author: tdykstra
manager: wpickett
ms.date: 02/29/2017
ms.topic: artigo
ms.technology: aspnet
ms.prod: asp.net-core
uid: fundamentals/startup
---
# Inicialização da aplicação no ASP.NET Core

Por [Steve Smith](https://ardalis.com/) and [Tom Dykstra](https://github.com/tdykstra/)

A classe `Startup` configura serviços e o pipeline de requisições da aplicação.

## A classe de Inicialização

Aplicações ASP.NET Core requerem uma classe `Startup`. Por convenção, a classe `Startup` é recebe o nome de "Startup". Você especifica o nome da classe de inicialização através do método [`UseStartup<TStartup>`](https://docs.microsoft.com/aspnet/core/api/microsoft.aspnetcore.hosting.webhostbuilderextensions#Microsoft_AspNetCore_Hosting_WebHostBuilderExtensions_UseStartup__1_Microsoft_AspNetCore_Hosting_IWebHostBuilder_) dentro de seu *program.cs*, ver também [WebHostBuilderExtensions](https://docs.microsoft.com/aspnet/core/api/microsoft.aspnetcore.hosting.webhostbuilderextensions). Veja [hospedagem](xref:fundamentals/hosting) para aprender mais sobre o `WebHostBuilder`, que é executado antes do `Startup`.

Você pode definir classes `Startup` separadas para diferentes ambientes, e a qual for aproprieada será selecionada em tempo de execução. Se você especificar `startupAssembly` nas [Configurações do WebHost](https://docs.microsoft.com/aspnet/core/fundamentals/hosting?tabs=aspnetcore2x#configuring-a-host) ou nas opções, o processo de hospedagem vai carregar aquele assembler de inicialização e procurar por um tipo `Startup` ou `Startup[Environment]`. A classe cujo o sufixo do nome combinar com o nome do ambiente será priorizada, então se a aplicação  estiver sendo executada no ambiente de *Development*, e incluir tanto uma classe `Startup` quanto outra chamada `StartupDevelopment`, esta última será usada. Veja [FindStartupType](https://github.com/aspnet/Hosting/blob/rel/1.1.0/src/Microsoft.AspNetCore.Hosting/Internal/StartupLoader.cs) no `StartupLoader` e [Trabalhando com ambientes múltiplos](environments.md#startup-conventions).

De forma alternativa, você pode definir uma classe `Startup` fixa que será usada independente do ambiente, isso é feito chamando o `UseStartup<TStartup>`. Esta é a abordagem recomendada.



O construtor da classe `Startup` pode aceitar dependências que são fornecidas através [dependency injection](xref:fundamentals/dependency-injection). Uma abordagem comum é utilizar o `IHostingEnvironment` para indicar as [Configurações](xref:fundamentals/configuration) iniciais.

A classe `Startup` precisa incluir o método `Configure` e pode opcionalmente incluir um método `ConfigureService`, ambos são chamados quando a aplicação inicia. A classe também pode incluir [Versões especificas de ambientes para estes métodos](xref:fundamentals/environments#startup-conventions). Se o método `ConfigureServices` estiver presente ele será chamado antes o método `Configure`.

Aprenda sobre [manipular exceções durante inicialização de aplicações](xref:fundamentals/error-handling#startup-exception-handling).

## O método ConfigureServices

O método [ConfigureServices](https://docs.microsoft.com/aspnet/core/api/microsoft.aspnetcore.hosting.startupbase#Microsoft_AspNetCore_Hosting_StartupBase_ConfigureServices_Microsoft_Extensions_DependencyInjection_IServiceCollection_) é opcinal; mas caso seja utilizado, ele é chamado antes do método `Configure` pelo host web. O host web pode configurar alguns serviços antes do método ``Startup`` ser chamado (veja [hospedagem](xref:fundamentals/hosting)). Por padrão, [Opções de configuração](xref:fundamentals/configuration) são feitas neste método.

Para recursos que requerem um preparo substâncial existem métodos de estensão `Add[Service]` no [IServiceColletion](https://docs.microsoft.com/aspnet/core/api/microsoft.extensions.dependencyinjection.iservicecollection). Este exemplo vindo do modelo padrão de página web configura a aplicação para usar serviços para Entity Framework, Identity e MVC:

[!code-csharp[Main](../common/samples/WebApplication1/Startup.cs?highlight=4,7,11&start=40&end=55)]

Adicionar serviços ao recipiente de serviços faz que eles fiquem disponíveis para sua aplicação via [injeção de dependência](xref:fundamentals/dependency-injection).


## Serviços Disponíveis na Inicialização (Startup)

A injeção de dependência do ASP.NET Core fornece serviços durante a inicialização da aplicação. Você pode requisitar estes serviços ao incluir a interface apropriada como um parâmetro no construtor de sua classe `Startup` ou no método `Configure` dela. O método `ConfigureServices` somente possui um parâmetro `IServiceCollection` (mas qualquer serviço resgistrado pode ser matido nesta coleção, então parâmetros adicionais não são necessários).

Abaixo estão alguns serviços normalmente requisitados por métodos `Startup`:

* No construtor: `IhostingEnvironment`, `ILogger<Startup>`
* No `ConfigureServices`: `IServiceCollection`
* No `Configure` : `IApplicationBuilder`, `IHostingEnvironment`, `ILoggerFactory`

Quaisquer serviços adicionados através dos métodos ``WebHostBuilder`` ``ConfigureServices`` podem ser requeridos ao construtor da classe ``Startup``ou seu método ``Configure``. Use `WebHostBuilder` para fornecer quaisquer serviços que você precisar durante a fase de inicialização (`Startup`).

## O método Configure

O método `Configure` é usado para especificar como a aplicação ASP.NET responderá as requisições HTTP. O pipeline de requisições é configurado ao adicionarmos componentes [middleware](middleware.md) a uma instância `IApplicationBuilder` que é fornecida por injeção de dependência.

No exemplo seguinte, que vem do modelo padrão de páginas web do ASP.NET Core, diversos métodos de extensão são usados para confgiurar o pipeline com suporte para [BrowserLink](http://vswebessentials.com/features/browserlink), páginas de erro, arquivos estáticos, ASP.NET MVC, e Identidade.

[!code-csharp[Main](../common/samples/WebApplication1/Startup.cs?highlight=8,9,10,14,17,19,21&start=58&end=84)]

Cada método de extensão `Use` adiciona um componente [middleware](xref:fundamentals/middleware) ao pipeline de requisições. Por exemplo, o método de extensão `UseMvc`adiciona o middleware [rotas](routing.md) ao pipeline de requisições e configura o [MVC](xref:mvc/overview) como o manipulador padrão.

Para mais informações sobre como usar o `IApplicationBuilder`, veja [Middleware](xref:fundamentals/middleware).

Serviços adicionais, como `IhostingEnvironment` e `ILoggerFactory` podem ser especificados na assinatura do método, caso qual estes serviços serão [injetados](dependency-injection.md) se estiverem disponíveis.

## Recursos Adicionais

* [Trabalhando com Ambientes Múltiplos](xref:fundamentals/environments)
* [Middleware](xref:fundamentals/middleware)
* [Logging](xref:fundamentals/logging)
* [Configurações](xref:fundamentals/configuration)
