---
title: Error Handling in ASP.NET Core
author: ardalis
description: Explains how to handle errors in ASP.NET Core applications
keywords: ASP.NET Core,error handling,exception handling
ms.author: tdykstra
manager: wpickett
ms.date: 11/30/2016
ms.topic: article
ms.assetid: 4db51023-c8a6-4119-bbbe-3917e272c260
ms.technology: aspnet
ms.prod: asp.net-core
uid: fundamentals/error-handling
ms.custom: H1Hack27Feb2017
---

# Introdução à Manipulação de Erro no ASP.NET Core

Por [Steve Smith](https://ardalis.com/) e [Tom Dykstra](https://github.com/tdykstra/)

Este artigo cobre as abordagens comuns para manipulação de erros em aplicações ASP.NET Core.

[Visualizar ou baixar código demonstrativo](https://github.com/aspnet/Docs/tree/master/aspnetcore/fundamentals/error-handling/sample)

## A página de exceções do desenvolvedor

Para configurar uma aplicação para exibir a página que mostra informações detalhadas sobre exceções, instale o pacote NuGet `Microsoft.AspNetCore.Diagnostics` e adicione uma linha para o [Método de configuração na classe Startup](startup.md):

[!code-csharp[Main](error-handling/sample/Startup.cs?name=snippet_DevExceptionPage&highlight=7)]

Coloque `UseDeveloperExceptionPage` antes de qualquer middleware que você queira armazenar exceções, como o `app.UseMvc`.


>[!AVISO]
> Habilite a página de exceções do desenvolvedor **somente quando a aplicação estiver executando no ambiente de Desenvolvimento**. Você não quer compartilhar informações de exceções detalhadas publicamente quando o app rodar em produção. [Aprenda mais sobre configuração de ambientes](environments.md).

Para ver a página de exceção do desenvolvedor, execute a aplicação demonstrativa com o ambiente configurado para `Development`, e adicione `?throw=true` para a URL base da aplicação. A page inclui diversas abas com informações sobre exceção e requisição. A primeira aba inclui um rastreameto da pilha.

![Rastreamento de pilha](error-handling/_static/developer-exception-page.png)

A próxima aba mostra os parâmetros da query string, se existirem.

![Parâmetros da Query String](error-handling/_static/developer-exception-page-query.png)

Esta requisição não possui nenhum cookie, mas se possuisse, eles apareceriam na aba **Cookies**. Você pode ver o cabeçalho que foi passado na última aba.

![Cabeçalhos](error-handling/_static/developer-exception-page-headers.png)

## Configurando um página de controle de exceção customizada

É uma boa ideia configurar uma página de controle de exceção customizada quando a aplicação não estiver sendo executada no ambiente de `Development`.

[!code-csharp[Main](error-handling/sample/Startup.cs?name=snippet_DevExceptionPage&highlight=11)]

Em uma aplicação MVC, não decore explicitamente o método de ação do controlador de erro com o atributo de método HTTP, como um `HttpGet`. Use explicitamente verbos para previnir alguns requisições de chegarem ao método.

```csharp
[Route("/Error")]
public IActionResult Index()
{
    // Handle error here
}
```

## Configurando as página de código de status

Por padrão, sua aplicação não fornecerá uma rica página de código de status para os códigos de status HTTP como 500 (Erro Interno do Servidor) ou 404 (Não Encontrado). Você pode configurar o `StatusCodePagesMiddleware` ao adicionar um linha no método `Configure`:

```csharp
app.UseStatusCodePages();
```

Por padrão, este middleware adiciona controladores do tipo somente texto para os código de status comuns, tipo 404:

![Página 404](error-handling/_static/default-404-status-code.png)

O middleware supporta muitos métodos de extensão diferentes. Alguns recebem uma expressão lambda, outros um tipo de conteúdo e formato string.

[!code-csharp[Main](error-handling/sample/Startup.cs?name=snippet_StatusCodePages)]

```csharp
app.UseStatusCodePages("text/plain", "Status code page, status code: {0}");
```

Existem também os métodos de extensão de redirecionamento. Uns enviam um código de status 302 para o cliente, outros retornam o código de status original para o cliente, mas também executam o controlador para a URL redirecionada.

[!code-csharp[Main](error-handling/sample/Startup.cs?name=snippet_StatusCodePagesWithRedirect)]

```csharp
app.UseStatusCodePagesWithReExecute("/error/{0}");
```

Se você precisar desabilitar a página de código de satus para certas requisições, você pode fazer assim:

```csharp
var statusCodePagesFeature = context.Features.Get<IStatusCodePagesFeature>();
if (statusCodePagesFeature != null)
{
  statusCodePagesFeature.Enabled = false;
}
```

## Exception-handling code

Code in exception handling pages can throw exceptions. It's often a good idea for production error pages to consist of purely static content.

Also, be aware that once the headers for a response have been sent, you can't change the response's status code, nor can any exception pages or handlers run. The response must be completed or the connection aborted.

## Server exception handling

In addition to the exception handling logic in your app, the [server](servers/index.md) hosting your app performs some exception handling. If the server catches an exception before the headers are sent, the server sends a 500 Internal Server Error response with no body. If the server catches an exception after the headers have been sent, the server closes the connection. Requests that aren't handled by your app are handled by the server. Any exception that occurs is handled by the server's exception handling. Any configured custom error pages or exception handling middleware or filters don't affect this behavior.

## Startup exception handling

Only the hosting layer can handle exceptions that take place during app startup. You can [configure how the host behaves in response to errors during startup](hosting.md#detailed-errors) using `captureStartupErrors` and the `detailedErrors` key.

Hosting can only show an error page for a captured startup error if the error occurs after host address/port binding. If any binding fails for any reason, the hosting layer logs a critical exception, the dotnet process crashes, and no error page is displayed.

## ASP.NET MVC error handling

[MVC](../mvc/index.md) apps have some additional options for handling errors, such as configuring exception filters and performing model validation.

### Exception Filters

Exception filters can be configured globally or on a per-controller or per-action basis in an MVC app. These filters handle any unhandled exception that occurs during the execution of a controller action or another filter, and are not called otherwise. Learn more about exception filters in [Filters](../mvc/controllers/filters.md).

>[!TIP]
> Exception filters are good for trapping exceptions that occur within MVC actions, but they're not as flexible as error handling middleware. Prefer middleware for the general case, and use filters only where you need to do error handling *differently* based on which MVC action was chosen.

### Handling Model State Errors

[Model validation](../mvc/models/validation.md) occurs prior to each controller action being invoked, and it is the action method’s responsibility to inspect `ModelState.IsValid` and react appropriately.

Some apps will choose to follow a standard convention for dealing with model validation errors, in which case a [filter](../mvc/controllers/filters.md) may be an appropriate place to implement such a policy. You should test how your actions behave with invalid model states. Learn more in [Testing controller logic](../mvc/controllers/testing.md).



