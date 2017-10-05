---
title: Overview of ASP.NET Core MVC 
author: ardalis
description: Learn how ASP.NET Core MVC is a rich framework for building web apps and APIs using the Model-View-Controller design pattern.
keywords: ASP.NET Core,
ms.author: riande
manager: wpickett
ms.date: 10/14/2016
ms.topic: article
ms.assetid: 89af38d1-52e0-4db7-b791-dbce909b0714
ms.technology: aspnet
ms.prod: asp.net-core
uid: mvc/overview
---
# Overview of ASP.NET Core MVC

Por [Steve Smith](https://ardalis.com/)

O ASP.NET Core MVC é um rico framework para construção de aplicações web usando o padrão de projeto *Model-View-Controller*.

## O que é o padrão MVC?

O padrão de arquitetura *Model-View-Controller* (MVC) separa um aplicação em três grandes grupos principais de componentes: Modelos, Visões, e Controladores. Este padrão ajuda a atingir a [separação de conceitos](http://deviq.com/separation-of-concerns/). Usando este padrão, as requisições de usuário são encaminhadas para o *Controller*, o qual é responsável por trabalhar junto com o *Model* para executar ações de usuários e/ou recuperar resultados de consultas. O *Controller* escolhe a *View* a ser exibida ao usuário, e fornece esta *View* junto com qualquer informações de Modelo que for necessário.

O diagrama seguinte mostra os três componentes principais e como se receferem aos outros:

![MVC Pattern](overview/_static/mvc.png)

Esta delineação de resposabilidades lhe ajuda a esccalar a aplicação em termos de complexidade, porque é fácil códificar, depurar, e testar alguma coisa (*model*,*view*,*controller*) que tenha apenas um trabalho (seguindo o [Princípio da Resposabilidade Única](http://deviq.com/single-responsibility-principle/)). É mais difícil atualizar, testar e depurar um código que dependências espalhadas por duas ou mais destas três áreas. Por exemplo, a lógica da interface de usuário tende a mudar mais frequentemente que a lógica de negócios. Se o código da apresentação e a regra de negócio são combinados em um único objeto, você tem que modificar um objeto contendo regras de negócio, toda vez que você alterar a interface. Isto é suscetível de introduzir erros e requer um novo teste das regras de negócio a cada mínima mudança na interface de usuário.

> [!NOTA]
> Tanto *View* como o *Controller* dependem do modelo. Contudo, o *model* não depende nem da *view* nem do *controller*. Isto é um dos benefícios chave da separação de conceitos. Esta separação permite que o modelo seja construído e testado de forma independente da apresentação visual.

### Responsabilidades do *Model*

O *Model* na aplicação MVC representa o estado da aplicação e de qualquer regra de negócio ou operações que precisam ser executadas por este. Regras de negócio precisam ser encapsuladas no modelo, junto com qualquer implementação lógica para persistência do estado da aplicação. *Views* fortemente tipadas usurão geralmente tipos *ViewModel* especialmente desenvolvidos para conterem as informações de exibição naquela *View*; o *controller* vai criar e popular estas instâncias de *ViewModel* do *model*.

> [!NOTA]
> Existem muitas maneiras de organizar o *model* em uma aplicação que usa o padrão de arquiterura MVC. Aprenda mais em [diferentes tipos de *model types*](http://deviq.com/kinds-of-models/).

### Resposabilidades da *View*

*Views* são responsáveis para apresentar o conteúdo através da interface de usuário. Elas usam o [mecânismo de visualização Razor](#razor-view-engine) para embutir código .NET em marcações HTML. Deve haver um mínimo de lógica dentro das *views*, e qualquer lógica nelas deveria relacionar-se com o conteúdo apresentado. Se você encontrar o necessário para realizar uma grande lógica nos arquivos da *view* para exibir dados de um *model* complexo, considere o uso do [Componente *View*](views/view-components.md), ViewModel, ou modelo de *view* para simplificar a *view*.

### Resposabilidades do *Controller*

*Controllers* são componentes que manipulam a interação do usuário, trabalham com o modelo, e, finalmente, selecionam a *view* para renderização. Em uma aplicação MVC, a *view* somente exibe informação; o *controller* manipula e responde as informações de entrada e interações do usuário. No padrão de projeto MVC, o *controller* é o ponto de entrada inicial, e é resposável por selecionar quais *model types* devem ser usados e qual *view* renderiza (daí o seu nome - ele controla como o aplicativo responde a um determinado pedido).  

>[!NOTA]
>*Controllers* não devem ser demasiadamente complicados carregando muitas responsabilidades. Para manter a lógica do *controller* longe de tornar-se complexa, use o [Princípio da Responsabilidade Única](http://deviq.com/single-responsibility-principle/) para enviar regras de negócio para fora do *controller* e para dentro do modelo de domínio.

>[!DICA]
> Se você achar que suas *actions* do *controller* geralmente executam os mesmos tipos de ações, você pode seguir o[Princípio do não se Repita](http://deviq.com/don-t-repeat-yourself/) ao mover estas *actions* comuns para [filtros](#filters).

## O que é ASP.NET Core MVC

O framework ASP.NET Core MVC é um framework de apresentação, leve, de código aberto, altamente testável e otimizado para uso com o ASP.NET Core.

O ASP.NET Core MVC fornece uma maneira baseada em padrões de projeto para construir website dinâmicos que permitem uma clara separação de conceitos. Isto lhe dá um controle total sobre o *markup*, suportando implementação por TDD-amigável e uso os últimos padrões web.

## Funcionalidades

O ASP.NET Core MVC incluí o seguinte:

* [Roteamento](#routing)
* [Model binding](#model-binding)
* [Validação de *Model*](#model-validation)
* [Injeção de Dependência](../fundamentals/dependency-injection.md)
* [Filtros](#filters)
* [Áreas](#areas)
* [APIs Web](#web-apis)
* [Testabilidade](#testability)
* [Mecânica de *view* Razor](#razor-view-engine)
* [*Views* fortemente tipadas](#strongly-typed-views)
* [Tag Helpers](#tag-helpers)
* [Componentes de *View*](#view-components)

### Roteamento

O ASP.NET Core MVC é feito sob o topo do [roteamento do ASP.NET Core](../fundamentals/routing.md), um poderoso compoenente mapeador de URL que lhe permite construir aplicações que tenham URLs compreensíveis e encontráveis. Isto permite que você defina seus padrões de nomes de URL da aplicação para funcionarem de acordo com sistemas de otimização de busca (SEO) e para geração de atalhos, sem preocupar-se em como seus arquivos no servidor web estão organizados. Você pode definir suas rotas usando uma sintaxe conveniente de modelo de rota que suporta valores limitadores *constraints*, padrões e de valores opcionais.

O *roteamento baseado em convenção* permite que você defina globalmente o formato da URL que sua aplicação aceita e como cada um desses formatos mapeiam um método *action* específico em um determinado *controller*. Quando uma requisição de entrada for recebida, a mecânica de roteamento analisa a URL e coincide-a com um daqueles formado de URL definidos, e então chama o método *action* do *controller* associado.

```csharp
routes.MapRoute(name: "Default", template: "{controller=Home}/{action=Index}/{id?}");
```

O *roteamento de atributo* permite que você especifique informações de roteamento ao decorar seus *controllers* e *actions* com atributos que definem suas rotas de aplicação. Isto significa que suas definições de rota são colocadas próximas do *controller* e *action* com as quais são associadas.


```csharp
[Route("api/[controller]")]
public class ProductsController : Controller
{
  [HttpGet("{id}")]
  public IActionResult GetProduct(int id)
  {
    ...
  }
}
```

### Model binding

ASP.NET Core MVC [model binding](models/model-binding.md) converts client request data  (form values, route data, query string parameters, HTTP headers) into objects that the controller can handle. As a result, your controller logic doesn't have to do the work of figuring out the incoming request data; it simply has the data as parameters to its action methods.

```csharp
public async Task<IActionResult> Login(LoginViewModel model, string returnUrl = null) { ... }
   ```

### Model validation

ASP.NET Core MVC supports [validation](models/validation.md) by decorating your model object with data annotation validation attributes. The validation attributes are checked on the client side before values are posted to the server, as well as on the server before the controller action is called.

```csharp
using System.ComponentModel.DataAnnotations;
public class LoginViewModel
{
    [Required]
    [EmailAddress]
    public string Email { get; set; }

    [Required]
    [DataType(DataType.Password)]
    public string Password { get; set; }

    [Display(Name = "Remember me?")]
    public bool RememberMe { get; set; }
}
```

A controller action:

```csharp
public async Task<IActionResult> Login(LoginViewModel model, string returnUrl = null)
{
    if (ModelState.IsValid)
    {
      // work with the model
    }
    // If we got this far, something failed, redisplay form
    return View(model);
}
```

The framework will handle validating request data both on the client and on the server. Validation logic specified on model types is added to the rendered views as unobtrusive annotations and is enforced in the browser with [jQuery Validation](https://jqueryvalidation.org/).

### Dependency injection

ASP.NET Core has built-in support for [dependency injection (DI)](../fundamentals/dependency-injection.md). In ASP.NET Core MVC, [controllers](controllers/dependency-injection.md) can request needed services through their constructors, allowing them to follow the [Explicit Dependencies Principle](http://deviq.com/explicit-dependencies-principle/).

Your app can also use [dependency injection in view files](views/dependency-injection.md), using the `@inject` directive:

```cshtml
@inject SomeService ServiceName
<!DOCTYPE html>
<html lang="en">
<head>
    <title>@ServiceName.GetTitle</title>
</head>
<body>
    <h1>@ServiceName.GetTitle</h1>
</body>
</html>
```

### Filters

[Filters](controllers/filters.md) help developers encapsulate cross-cutting concerns, like exception handling or authorization. Filters enable running custom pre- and post-processing logic for action methods, and can be configured to run at certain points within the execution pipeline for a given request. Filters can be applied to controllers or actions as attributes (or can be run globally). Several filters (such as `Authorize`) are included in the framework.


```csharp
[Authorize]
   public class AccountController : Controller
   {
```

### Areas

[Areas](controllers/areas.md) provide a way to partition a large ASP.NET Core MVC Web app into smaller functional groupings. An area is effectively an MVC structure inside an application. In an MVC project, logical components like Model, Controller, and View are kept in different folders, and MVC uses naming conventions to create the relationship between these components. For a large app, it may be advantageous to partition the app into separate high level areas of functionality. For instance, an e-commerce app with multiple business units, such as checkout, billing, and search etc. Each of these units have their own logical component views, controllers, and models.

### Web APIs

In addition to being a great platform for building web sites, ASP.NET Core MVC has great support for building Web APIs. You can build services that can reach a broad range of clients including browsers and mobile devices.

The framework includes support for HTTP content-negotiation with built-in support for [formatting data](models/formatting.md) as JSON or XML. Write [custom formatters](advanced/custom-formatters.md) to add support for your own formats.

Use link generation to enable support for hypermedia. Easily enable support for [cross-origin resource sharing (CORS)](http://www.w3.org/TR/cors/) so that your Web APIs can be shared across multiple Web applications.

### Testability

The framework's use of interfaces and dependency injection make it well-suited to unit testing, and the framework includes features (like a TestHost and InMemory provider for Entity Framework) that make [integration testing](../testing/integration-testing.md) quick and easy as well. Learn more about [testing controller logic](controllers/testing.md).

### Razor view engine

[ASP.NET Core MVC views](views/overview.md) use the [Razor view engine](views/razor.md) to render views. Razor is a compact, expressive and fluid template markup language for defining views using embedded C# code. Razor is used to dynamically generate web content on the server. You can cleanly mix server code with client side content and code.

```text
<ul>
  @for (int i = 0; i < 5; i++) {
    <li>List item @i</li>
  }
</ul>
```

Using the Razor view engine you can define [layouts](views/layout.md), [partial views](views/partial.md) and replaceable sections.

### Strongly typed views

Razor views in MVC can be strongly typed based on your model. Controllers can pass a strongly typed model to views enabling your views to have type checking and IntelliSense support.

For example, the following view defines a model of type `IEnumerable<Product>`:

```cshtml
@model IEnumerable<Product>
<ul>
    @foreach (Product p in Model)
    {
        <li>@p.Name</li>
    }
</ul>
```

### Tag Helpers

[Tag Helpers](views/tag-helpers/intro.md) enable server side code to participate in creating and rendering HTML elements in Razor files. You can use tag helpers to define custom tags (for example, `<environment>`) or to modify the behavior of existing tags (for example, `<label>`). Tag Helpers bind to specific elements based on the element name and its attributes. They provide the benefits of server-side rendering while still preserving an HTML editing experience.

There are many built-in Tag Helpers for common tasks - such as creating forms, links, loading assets and more - and even more available in public GitHub repositories and as NuGet packages. Tag Helpers are authored in C#, and they target HTML elements based on element name, attribute name, or parent tag. For example, the built-in LinkTagHelper can be used to create a link to the `Login` action of the `AccountsController`:

```cshtml
<p>
    Thank you for confirming your email.
    Please <a asp-controller="Account" asp-action="Login">Click here to Log in</a>.
</p>
```

The `EnvironmentTagHelper` can be used to include different scripts in your views (for example, raw or minified) based on the runtime environment, such as Development, Staging, or Production:

```cshtml
<environment names="Development">
    <script src="~/lib/jquery/dist/jquery.js"></script>
</environment>
<environment names="Staging,Production">
    <script src="https://ajax.aspnetcdn.com/ajax/jquery/jquery-2.1.4.min.js"
            asp-fallback-src="~/lib/jquery/dist/jquery.min.js"
            asp-fallback-test="window.jQuery">
    </script>
</environment>
```

Tag Helpers provide an HTML-friendly development experience and a rich IntelliSense environment for creating HTML and Razor markup. Most of the built-in Tag Helpers target existing HTML elements and provide server-side attributes for the element.

### View Components

[View Components](views/view-components.md) allow you to package rendering logic and reuse it throughout the application. They're similar to [partial views](views/partial.md), but with associated logic.
