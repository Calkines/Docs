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

Para configurar uma aplicação a exibir uma página que mostra informações detalhadas sobre exceções, instale o pacote NuGet `Microsoft.AspNetCore.Diagnostics` e adicione uma linha para o [Método de configuração na classe Startup](startup.md):

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
## Código de manipulação de exceção

O Código em uma página de manipulação de exceções pode estourar uma exceção. É geralmente uma boa ideia para páginas de erro em produção consistir consistir de contéudo puramente estático.

Também, esteja ciente que uma vez que os cabeçalhos para uma resposta tenham sido enviados, você não pode alterar seu código de status, nem executar qualquer páginas ou controladores. A resposta precisa ser completada ou a conexão abortada.

## Tratamento de exeção no Servidor

Além da lógica de manipulaçã de exceção em sua aplicação, o [servidor](servers/index.md) que hospeda sua aplicação executa alguns tratamentos de exceções. Se o servidor encontrar uma exceção antes que o cabeçalho seja enviado, o server envia uma resposta 500 (Erro Interno do Servidor) sem corpo. Se o servidor encontrar uma exceção após o envio dos cabeçalhos, o server fecha a conexão. Requisições que não são tratadas pela sua aplicação são tratadas pelo servidor. Qualquer exceção que ocorra é tratada pelo tratamento de exceções do servidor. Quaisquer página de erro customizada configuradas ou middleware de tratamento ou filtros não afetam este comportamento.

## Tratamento de exceções de inicialização

Apenas a camada de hospedagem pode manipular exceções que foram colocadas na inicialização da aplicação. Você pode [configurar como o host comporta-se em resposta a errros durante inicialização](hosting.md#detailed-errors) usando `captureStartupErrors` e a chave `detailedErros`.

A hospedagem pode apenas mostrar uma página de erro para erros capturados na inicialização se o erro ocorrer após o vínculo entre host e endereço/porta. Se qualquer vinculo falhar por alguma razão, a camada de hospedagem armazena o log de exceções críticas, o processo dotnet falha, e nenhuma página de erro é exibida.

## Manipulação de erro com ASP.NET MVC

Aplicações [MVC](../mvc/index.md) tem algumas opções adicionais para manipulação de erros, como a configuração de filtros de exceção e execução de validação de modelo.

### Filtros de exceção

Filtros de exceção podem ser configurados globalmente ou em um controle ou ação básicos na aplicação MVC. Estes fitlros manipulam qualquer exceção não tratada que ocorrer durante a execução de um ação de controle ou outro filtro, e não são chamado de outra forma. Aprenda mais sobre filtros de exceção em [Filtros](../mvc/controllers/filters.md).

>[!DICA]
> Filtros de exceção são bons para encurralar exceções que ocorrem dentro de ações MVC, mas eles não são tão flexíveis como o middleware de tratamento de exceções. Prefira o middleware para casos geral, e use os filtros apenas nos locais que você precisar fazer um tratamento de erro *diferenciado* baseado nas quais ações MVC você escolheu.

### Manipulação do Modelo de Erros por Estado

[Validação de Modelo](../mvc/models/validation.md) ocorre antes de cada ação de controle começar a ser invocada, e ela é a ação do método respnsável por inspecionar `ModelState.IsValid` e reagir adequadamente.

Algumas aplicação vão escolher seguir uma convenção padrão para lidar com o modelo de validação de erros, neste caso um [filtro](../mvc/controllers/filters.md) pode ser um lugar apropriado para implementar tal politica. Você precisa testar como suas ações comportam-se com modelos de estado inválidos. Aprenda mais em [Testando a lógica de um controlador](../mvc/controllers/testing.md).

