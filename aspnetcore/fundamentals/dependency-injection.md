---
title: Dependency Injection in ASP.NET Core
author: ardalis
description: Learn how ASP.NET Core implements dependency injection and how to use it.
keywords: ASP.NET Core,dependency injection,di
ms.author: riande
manager: wpickett
ms.date: 10/14/2016
ms.topic: article
ms.assetid: fccd69be-7ad1-47fb-b203-b3633b6b9a9b
ms.technology: aspnet
ms.prod: asp.net-core
uid: fundamentals/dependency-injection
ms.custom: H1Hack27Feb2017
---
# Introduction to Dependency Injection in ASP.NET Core

<a name=fundamentals-dependency-injection></a>

Por [Steve Smith](https://ardalis.com/) e [Scott Addie](https://scottaddie.com)

O ASP.NET Core foi desenvolvido desde a base até o topo para suportar e proporcionar a injeção de dependência. As aplicações ASP.NET Core podem usufluir de serviços de framework prontos por os terem injetado em seus métodos na classe Startup, e serviços de aplicação também podem ser configurados para injeção. O recipiente padrão de serviços fornecido pelo ASP.NET Core proporciona um conjunto mínimo de funcinalidades e não tem a intenção de substituir outros recipientes.

[Visualizar ou baixar código demonstrativo](https://github.com/aspnet/Docs/tree/master/aspnetcore/fundamentals/dependency-injection/sample)

## O que é Injeção de Dependência?

Injeção de Dependência, abreviado em inglês como DI, é uma técnica para alcançar baixo acoplamento entre objetos e seus colaboradores, ou dependências. Em vez de instanciar diretamente colaboradores, ou usar referências estáticas, os objetos que a classe precisa para executar suas ações são fornecidos para a ela da mesma forma. Mais frequentemente, as classes vã declarar suas dependências através de seus construtores, permitindo a eles seguir o [Princípio das Dependências Explícitas](http://deviq.com/explicit-dependencies-principle/). Esta abordagem é conhecida como "Injeção de Construtor".

Quando classes são desenvolvidas com o DI em mente, elas são fracamente acopladas porque elas não possuem diretamente dependências hard-coded em seus colaboradores. Isto segue o [Princípio da Inversão de Dependência](http://deviq.com/dependency-inversion-principle/), que afirma que *"módulos de alto nível não devem depender de módulos de baixo nível; ambos devem depender de abstrações."* Em vez de referênciar implementações específicas, classes requerem abstrações (geralmente `interfaces`) que são fornecidas no momento da construção das classes. Extração de dependências em interfaces e fornecimento de implementações destas interfaces como parâmetros é também um exemplo do [Padrão de Design Estratégia](http://deviq.com/strategy-design-pattern/).

Quando o sistema é desenvolvido usando DI, com muitas classes requisitando suas dependências através de seus construtores (ou propriedades), é útil ter um classe dedicada para criar estas classes com suas dependências associadas. Estas classes são referenciadas como *containers* (recipientes), ou mais especificamente, recipientes de [Inversão de Controle, sigla em inglês (IoC)](http://deviq.com/inversion-of-control/) ou de Injeção de Dependência (DI). Um recipiente é essencialmente uma fábrica que é responsável por fornecer instâncias de tipos que são requisitadas. Se um determinado tipo for declarado possuindo dependências, e o recipiente tiver sido configurado para fornecer estes tipos de dependências, o recipiente criará as dependências como parte da criação da instância de requisição. Deste modo, linha complexas de dependências podem ser fornecidas para classes sem a necessidade de qualquer construção "hard-coded". Além da criação de objetos com suas dependências, recipientes geralmente gerenciam o tempo de vida dos objetos na aplicação.

O ASP.NET Core incluí um simples recipiente pré-montado (representado pela interface `IServiceProvider`) que suporta a injeção por construtor por padrão, o ASP.NET disponibiliza  serviços através de DI. Recipientes ASP.NET referem-se aos tipos que ele gerencia como *serviços*. Por todo resto deste artigo, *serviços* vão referir-se a tipos que são gerenciados pelo recipiente IoC do ASP.NET Core. Você configura o serviço do recipiente pré-fabricado no método `ConfigureServices` de sua classe `Startup` em sua aplicação. 

> [!NOTA]
> Martin Fowler escrereveu um artigo extenso em [Recipientes de Inversão de Controle e Padrão de Injeção de Dependência](https://www.martinfowler.com/articles/injection.html). Padrões Microsoft e Práticas também possuem uma vasta descrição da [Injeção de Dependência](https://msdn.microsoft.com/library/hh323705.aspx).

> [!NOTA]
> Este artigo aborda a Injeção de Dependência como aplicada a todas aplicações ASP.NET. A Injeção de Dependência dentro dos controles MVC é abordada em [Injeção de Dependência e Controles](../mvc/controllers/dependency-injection.md).

### Comportamento da Injeção por Construtor

Injeção por Construtor requer que o contrutor em questão seja *público*. Caso contrário, sua aplicação lancará uma exceção `InvalidOperationException`:

> Um construtor adequado para o tipo 'SeuTipo' não pode ser localizado. Garanta que o tipo é concreto e os serviços estão registrados para todos os parâmetros do construtor público.

A injeção por contrutor requer que apenas um construtor aplicável exista. Sobrecarga de construtores são suportadas, mas apenas apenas uma sobrecarga pode existir, cujo argumentos podem ser todos realizados por injeção de dependência. Se mais de um existir, sua aplicação vai levantar uma `InvalidOperationExeption`:

> Construtores múltiplos aceitando todos os tipos de argumentos foram encontrados no tipo 'seuTipo'. Deveria haver apenas um construtor aplicável.

Construtores podem aceitar argumentos que não são fornecidos por injeção de dependência, mas estes precisam suportar valores padrões. Por exemplo:

```csharp
// throws InvalidOperationException: Unable to resolve service for type 'System.String'...
public CharactersController(ICharacterRepository characterRepository, string title)
{
    _characterRepository = characterRepository;
    _title = title;
}

// runs without error
public CharactersController(ICharacterRepository characterRepository, string title = "Characters")
{
    _characterRepository = characterRepository;
    _title = title;
}
```

## Usando os Serviços de Fornecimento do Framework

O método `ConfigureServices` na classe `Startup` é responsável pela definição dos serviços que a aplicação usará, incluindo funcionalidades de plataforma como o Entity Framework Core e o ASP.NET Core MVC. Inicialmente, a `IServiceCollection` fornecida para o `ConfigureServices` possui os seguintes serviços definidos (dependendo de [como o host foi configurado](xref:fundamentals/hosting));

| Tipo do Serviço | Tempo de Vida |
| ----- | ------- |
| [Microsoft.AspNetCore.Hosting.IHostingEnvironment](https://docs.microsoft.com/aspnet/core/api/microsoft.aspnetcore.hosting.ihostingenvironment) | Singleton |
| [Microsoft.Extensions.Logging.ILoggerFactory](https://docs.microsoft.com/aspnet/core/api/microsoft.extensions.logging.iloggerfactory) | Singleton |
| [Microsoft.Extensions.Logging.ILogger&lt;T&gt;](https://docs.microsoft.com/aspnet/core/api/microsoft.extensions.logging.ilogger) | Singleton |
| [Microsoft.AspNetCore.Hosting.Builder.IApplicationBuilderFactory](https://docs.microsoft.com/aspnet/core/api/microsoft.aspnetcore.hosting.builder.iapplicationbuilderfactory) | Transient |
| [Microsoft.AspNetCore.Http.IHttpContextFactory](https://docs.microsoft.com/aspnet/core/api/microsoft.aspnetcore.http.ihttpcontextfactory) | Transient |
| [Microsoft.Extensions.Options.IOptions&lt;T&gt;](https://docs.microsoft.com/aspnet/core/api/microsoft.extensions.options.ioptions-1) | Singleton |
| [System.Diagnostics.DiagnosticSource](https://docs.microsoft.com/dotnet/core/api/system.diagnostics.diagnosticsource) | Singleton |
| [System.Diagnostics.DiagnosticListener](https://docs.microsoft.com/dotnet/core/api/system.diagnostics.diagnosticlistener) | Singleton |
| [Microsoft.AspNetCore.Hosting.IStartupFilter](https://docs.microsoft.com/aspnet/core/api/microsoft.aspnetcore.hosting.istartupfilter) | Transient |
| [Microsoft.Extensions.ObjectPool.ObjectPoolProvider](https://docs.microsoft.com/aspnet/core/api/microsoft.extensions.objectpool.objectpoolprovider) | Singleton |
| [Microsoft.Extensions.Options.IConfigureOptions&lt;T&gt;](https://docs.microsoft.com/aspnet/core/api/microsoft.extensions.options.iconfigureoptions-1) | Transient |
| [Microsoft.AspNetCore.Hosting.Server.IServer](https://docs.microsoft.com/aspnet/core/api/microsoft.aspnetcore.hosting.server.iserver) | Singleton |
| [Microsoft.AspNetCore.Hosting.IStartup](https://docs.microsoft.com/aspnet/core/api/microsoft.aspnetcore.hosting.istartup) | Singleton |
| [Microsoft.AspNetCore.Hosting.IApplicationLifetime](https://docs.microsoft.com/aspnet/core/api/microsoft.aspnetcore.hosting.iapplicationlifetime) | Singleton |

Abaixo está um exemplo de como adicionar serviços adicionais aos recipientes usando um número de métodos de extensão como `AddDbContext`, `AddIdentity`, e `AddMvc`.

[!code-csharp[Main](../common/samples/WebApplication1/Startup.cs?highlight=5-6,8-10,12&range=39-56)]

Os recursos e middleware fornecidos pelo ASP.NET, como o MVC, seguem uma convenção de usar um único método de extensão Add*NomeDoServico* para registrar todos os serviços requeridos por aquela funcionalidade.

>[!DICA]
> Você pode requisitar certo serviço fornecido pelo framework dentro dos métodos `Startup` através de suas listas de parâmetros - veja [Inicialização de Aplicação](startup.md) para mais detalhes.

## Registrando seus Próprios Serviços

Você pode registrar seu próprio serviço de aplicação da seguinte maneira.O primeiro tipo genérico representa o tipo (geralmente uma interface) que será requisitada do recipiente. O segundo tipo genérico representa o tipo concreto que será instanciado pelo recipiente e usado para realizar as requisições. 

[!code-csharp[Main](../common/samples/WebApplication1/Startup.cs?range=53-54)]

> [!NOTA]
> Cada método de extensão `services.Add<ServiceName>` adiciona (e potencialmente configura) serviços. Por exemplo, `services.AddMvc()` adiciona os serviços que o MVC requer. É recomendado que você siga esta convenção, colocando métodos de entensão no namespace `Microsoft.Extensions.DependencyInjection`, para encapsular grupos de registros de serviço.

O método `AddTransient` é usado ara mapear tipos abstratos para serviços conretos que são instanciados separadamente para cada objeto que o requer. Isto é conhecido como o *tempo de vida* do serviço, e opções adicionais de tempo de vida de serviço são descritas abaixo. É importante escolher um tempo de vida apropriado para cada serviço que você registrar. Uma nova instância de cada serviço deveria ser fornecida para cada classe que a requisitasse? Uma instância deveria ser usada durante um determinada requisição web? Ou uma única instânacia deveria ser usada para o tempo de vida da aplicação?

Para fins de exemplo deste artigo, existe um *controller* simples que exibe nome de personagens, chamado `CharactersController`. É o método `Index` que exibe a lista atual de personagens que foram armazenados na aplicação, e inicializa a coleção com um punhado de personagens, se nenhum existir. Perceba que  apesar desta aplicação usar o Entity Framework Core e a classe `ApplicationDbContext` para esta persistência, nada disso é aparente no *controller*. Pelo contrário, o mecânismo específico de acesso a dados foi abstraído em uma interface, `ICharacterRepository`, a qual segue o [padrão de repositório](http://deviq.com/repository-pattern/). Uma instância de `ICharacterRepository` é requisitada através de construtor e atribuída a um campo privado, o qual é usado para acessar o personagem.

[!code-csharp[Main](../fundamentals/dependency-injection/sample/DependencyInjectionSample/Controllers/CharactersController.cs?highlight=3,5,6,7,8,14,21-27&range=8-36)]

A interface `ICharacterRepository` define dois métodos que o *controller* precisa para trabalhar com instâncias de `Character`.

[!code-csharp[Main](../fundamentals/dependency-injection/sample/DependencyInjectionSample/Interfaces/ICharacterRepository.cs?highlight=8,9)]

Esta interface é implementada, em momento oportuno, por um tipo concreto, `CharacterRepository`, que é usado em tempo de execução.

> [!NOTA]
> A maneira como a DI é usada na classe `CharacterRepository` é uma modelo geral que você pode seguir para em todos serviços de sua aplicação, não apenas em repositórios ou classes de acesso a dados.

[!code-csharp[Main](../fundamentals/dependency-injection/sample/DependencyInjectionSample/Models/CharacterRepository.cs?highlight=9,11,12,13,14)]

Perceba que `CharacterRepository` requer um `AplicationDbContext` em seu construtor. Isto não é incomum para injeção de dependência ser usada de maneira encadeada como essa, na qual cada dependência requisitada, no tempo correto, requisite suas próprias dependências. O recipiente é responsável por resolver todas dependências na cadeia e retornar o serviço totalmente resolvido.

> [!NOTA]
> A criação do objeto requisitado, e todos seus objetos requeridos, é algumas vezes chamado referenciada como *object graph*. Da mesma forma, a coleção coletiva das dependências que precisam ser resolvidas é geralmente chamada de *dependency tree*, em português árvore de dependência, ou *dependency graph*, cadeia de dependência.

Neste caso, ambos `ICharacterRepository` e, ao seu tempo, `ApplicationDbContext` precisam ser registrados no repositório de serviços `ConfigureServices` no `Startup`. `ApplicationDbContext` é configurado com a chamada ao método de extensão `AddDbContext<T>`. O código seguinte mostra o processo de registro do tipo `CharacterRepository`.

[!code-csharp[Main](dependency-injection/sample/DependencyInjectionSample/Startup.cs?highlight=3-5,11&range=16-32)]

O contexto do Entity Framework precisa ser adicionado ao recipiente de serviços usando o tempo de vida `Scoped`. Isto é feito automaticamente se você usar métodos de ajuda como mostrado abaixo. Repositórios que farão uso do Entity Framework precisam usar o mesmo tempo de vida.

>[!ALERTA]
> O principal perigo, o qual precisamos nos acaltelar, é resolver o serviço `Scoped` de um *singleton*. Isso é desejado nesse caso que o serviço vai ter um estado incorreto, quando estiver processando requisições subsequentes.

Serviços que possuem dependências precisam registrá-las no recipiente. Se um construtor do serviço requerer um tipo primitivo, como um `string`, isto pode ser injetado usando os [padrões de opção e configuração](configuration.md).

## Tempos de Vida dos Serviços e Opções de Registro

Os serviços ASP.NET podem ser configurados com os seguintes tempos de vida:

**Transiente (que não permanece, transitório) -Transient-**

Serviços com tempo de vida transiente são criados cada vez que são requisitados. Este tempo de vida funciona melhorr para serviços leves, sem estados.

**Com Escopo -Scoped- **

Serviços com tempo de vida por escopo são criado uma vez por requisição.

**Único -Singleton-**

Serviços com tempo de vida únicos são criados a primeira vez que são requisitados (ou quando `ConfigureServices` é executado se você especificar uma instância ali) e então cada chamada subsequente vai usar a mesma instância. Se sua aplicação requerer o comportamento único *singleton* permitindo o recipiente de serviços gerenciar o tempo de vida do serviço, é recomendado em vez disso implementar o padrão de projeto *singleton* e gerenciar, você mesmo, o tempo de vida de seu objeto na classe.

Serviços podem ser registrados nos recipientes de diferentes formas. Nós já vimos como registrar uma implementação de serviço com um determinado tipo ao especificar o tipo concreto de uso. Além disso, uma fábrica pode ser especificada, a qual será então usada para criar uma instância sob demanda. A terceira abordagem é especificar diretamente a instância do tipo a ser usado, que, neste caso, o recipiente nunca tentará criar uma instância (nem tentará descartá-la). 

Para demonstrar a diferente entre esses tempos de vida e opções de registro, considera uma interface simples que representa uma ou mais tarefas como uma *operação* com um identificador único, `OperationId`. Dependendo de como nós configurarmos o tempo de vida deste serviço, o recipiente proverá tanto o mesma ou diferente instâncias para o serviço à classe requisitante. Para deixar claro qual tempo de vida está sendo requisitado, nós criaremos um tipo de tempo de vida por opção:

[!code-csharp[Main](../fundamentals/dependency-injection/sample/DependencyInjectionSample/Interfaces/IOperation.cs?highlight=5-8)]

Nós implementamos estas interfaces usando uma única classe, `Operation`, que aceita um `Guid` em seu construtor, ou usa um novo `Guid`, caso nenhum tenha sido informado.

Na sequência, no `ConfigureServices`, cada tipo é adicionado ao recipiente de acodo com seu tempo de vida:

[!code-csharp[Main](dependency-injection/sample/DependencyInjectionSample/Startup.cs?range=26-32)]

Perceba que o serviço `IOperationSingletonInstance` é usado para especificar a instância com um ID conhecido de `Guid.Empty`, isso deixará claro quando este tipo estiver em uso (este Guid será zerado). Nós também registramos um `OperationService` que depende de cada um dos tipos `Operation`, então ficará claro dentro da requisição se este serviço está pegando a mesma instância que o *controller*, ou uma nova, para cada tipo de operação. Todos estes serviços expõem suas dependências como propriedades, então eles podem ser exibidos em uma *view*.

[!code-csharp[Main](dependency-injection/sample/DependencyInjectionSample/Services/OperationService.cs)]

Para demonstrar o tempo de vida do objeto dentro e entre as requisições individuais separadas à aplicação, a demonstração incluí um `OperationsController` que requisita cada tipo do modelo `IOperation` como também um `OperationService`. A ação `Index` entã exibe todos os valores de `OperationId` para *controller* e o serviço.

[!code-csharp[Main](dependency-injection/sample/DependencyInjectionSample/Controllers/OperationsController.cs)]

Agora duas requisições separadas são feitas a esta *action* do *controller*:

![A *view Operations* do exemplo da aplicação web de Injeção de Dependência executando no Microsoft Edge, exibindo os valores ID das operações (GUID's) para as operações de serviço Transient, Scoped, Singleton e Instance *Controller* na primeira requisição.](dependency-injection/_static/lifetimes_request1.png)

![A *view Operations* exibindo os valores ID de operações para a segunda requisição](dependency-injection/_static/lifetimes_request2.png)

Observe que os valores de `OperationId` variam dentro da requisição, e entre as requisições.

* Objetos *Transient* são sempre diferentes; uma nova instância é sempre fornecida para cada *controller* e cada *service*.

* Objetos *Scoped* são o mesmo dentro da requisição, mas diferentes entre requisições diferentes.

* Objetos *Singleton* são o mesmo para cada objeto e cada requisição (independente de se a instância é fornecida no `ConfigureServices`)
 
## Serviços de Requisição

Os serviços disponíveis dentro de uma requisição ASP.NET do `HttpContext` são expostos através da coleção `RequestServices`.



![HttpContext Request Services Intellisense contextual dialog stating that Request Services gets or sets the IServiceProvider that provides access to the request's service container.](dependency-injection/_static/request-services.png)

Request Services represent the services you configure and request as part of your application. When your objects specify dependencies, these are satisfied by the types found in `RequestServices`, not `ApplicationServices`.

Generally, you shouldn't use these properties directly, preferring instead to request the types your classes you require via your class's constructor, and letting the framework inject these dependencies. This yields classes that are easier to test (see [Testing](../testing/index.md)) and are more loosely coupled.

> [!NOTE]
> Prefer requesting dependencies as constructor parameters to accessing the `RequestServices` collection.

## Designing Your Services For Dependency Injection

You should design your services to use dependency injection to get their collaborators. This means avoiding the use of stateful static method calls (which result in a code smell known as [static cling](http://deviq.com/static-cling/)) and the direct instantiation of dependent classes within your services. It may help to remember the phrase, [New is Glue](https://ardalis.com/new-is-glue), when choosing whether to instantiate a type or to request it via dependency injection. By following the [SOLID Principles of Object Oriented Design](http://deviq.com/solid/), your classes will naturally tend to be small, well-factored, and easily tested.

What if you find that your classes tend to have way too many dependencies being injected? This is generally a sign that your class is trying to do too much, and is probably violating SRP - the [Single Responsibility Principle](http://deviq.com/single-responsibility-principle/). See if you can refactor the class by moving some of its responsibilities into a new class. Keep in mind that your `Controller` classes should be focused on UI concerns, so business rules and data access implementation details should be kept in classes appropriate to these [separate concerns](http://deviq.com/separation-of-concerns/).

With regards to data access specifically, you can inject the `DbContext` into your controllers (assuming you've added EF to the services container in `ConfigureServices`). Some developers prefer to use a repository interface to the database rather than injecting the `DbContext` directly. Using an interface to encapsulate the data access logic in one place can minimize how many places you will have to change when your database changes.

### Disposing of services

The container will call `Dispose` for `IDisposable` types it creates. However, if you add an instance to the container yourself, it will not be disposed.

Example:

```csharp
// Services implement IDisposable:
public class Service1 : IDisposable {}
public class Service2 : IDisposable {}
public class Service3 : IDisposable {}

public void ConfigureServices(IServiceCollection services)
{
    // container will create the instance(s) of these types and will dispose them
    services.AddScoped<Service1>();
    services.AddSingleton<Service2>();

    // container did not create instance so it will NOT dispose it
    services.AddSingleton<Service3>(new Service3());
    services.AddSingleton(new Service3());
}
```

> [!NOTE]
> In version 1.0, the container called dispose on *all* `IDisposable` objects, including those it did not create.

## Replacing the default services container

The built-in services container is meant to serve the basic needs of the framework and most consumer applications built on it. However, developers can replace the built-in container with their preferred container. The `ConfigureServices` method typically returns `void`, but if its signature is changed to return `IServiceProvider`, a different container can be configured and returned. There are many IOC containers available for .NET. In this example, the [Autofac](https://autofac.org/) package is used.

First, install the appropriate container package(s):

* `Autofac`
* `Autofac.Extensions.DependencyInjection`

Next, configure the container in `ConfigureServices` and return an `IServiceProvider`:

```csharp
public IServiceProvider ConfigureServices(IServiceCollection services)
{
    services.AddMvc();
    // Add other framework services

    // Add Autofac
    var containerBuilder = new ContainerBuilder();
    containerBuilder.RegisterModule<DefaultModule>();
    containerBuilder.Populate(services);
    var container = containerBuilder.Build();
    return new AutofacServiceProvider(container);
}
```

> [!NOTE]
> When using a third-party DI container, you must change `ConfigureServices` so that it returns `IServiceProvider` instead of `void`.

Finally, configure Autofac as normal in `DefaultModule`:

```csharp
public class DefaultModule : Module
{
    protected override void Load(ContainerBuilder builder)
    {
        builder.RegisterType<CharacterRepository>().As<ICharacterRepository>();
    }
}
```

At runtime, Autofac will be used to resolve types and inject dependencies. [Learn more about using Autofac and ASP.NET Core](http://docs.autofac.org/en/latest/integration/aspnetcore.html).

### Thread safety

Singleton services need to be thread safe. If a singleton service has a dependency on a transient service, the transient service may also need to be thread safe depending how it’s used by the singleton.

## Recommendations

When working with dependency injection, keep the following recommendations in mind:

* DI is for objects that have complex dependencies. Controllers, services, adapters, and repositories are all examples of objects that might be added to DI.

* Avoid storing data and configuration directly in DI. For example, a user's shopping cart shouldn't typically be added to the services container. Configuration should use the [Options Model](configuration.md#options-config-objects). Similarly, avoid "data holder" objects that only exist to allow access to some other object. It's better to request the actual item needed via DI, if possible.

* Avoid static access to services.

* Avoid service location in your application code.

* Avoid static access to `HttpContext`.

> [!NOTE]
> Like all sets of recommendations, you may encounter situations where ignoring one is required. We have found exceptions to be rare -- mostly very special cases within the framework itself.

Remember, dependency injection is an *alternative* to static/global object access patterns. You will not be able to realize the benefits of DI if you mix it with static object access.

## Additional Resources

* [Application Startup](startup.md)

* [Testing](../testing/index.md)

* [Writing Clean Code in ASP.NET Core with Dependency Injection (MSDN)](https://msdn.microsoft.com/magazine/mt703433.aspx)

* [Container-Managed Application Design, Prelude: Where does the Container Belong?](https://blogs.msdn.microsoft.com/nblumhardt/2008/12/26/container-managed-application-design-prelude-where-does-the-container-belong/)

* [Explicit Dependencies Principle](http://deviq.com/explicit-dependencies-principle/)

* [Inversion of Control Containers and the Dependency Injection Pattern](https://www.martinfowler.com/articles/injection.html) (Fowler)
