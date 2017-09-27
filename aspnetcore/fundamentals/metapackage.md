---
título: Microsoft.AspNetCore.All meta-pacote para ASP.NET Core 2.x ou mais recente
autor: Rick-Anderson
tradutor: calkines
descrição: O Microsoft.AspNetCore.All meta-pacote inclui todos os pacotes suportados para ASP.NET Core e Entity Framework Core, juntamente com suas dependências.
palavras-chave: ASP.NET Core,NuGet,package,Microsoft.AspNetCore.All,metapackage
ms.author: riande
manager: wpickett
ms.date: 09/20/2017
ms.topic: article
ms.technology: aspnet
ms.prod: asp.net-core
uid: fundamentals/metapackage
---

#Microsoft.AspNetCore.All meta-pacotes para ASP.NET Core 2.x

Este recurso exige ASP.NET Core 2.x.

O meta-pacote [Microsoft.AspNetCore.All)](https://www.nuget.org/packages/Microsoft.AspNetCore.All) para ASP.NET Core inclui:

* Todos pacotes suportados pelo time do ASP.NET Core.
* Todos pacotes suportados pelo Entity Framework Core.
* Dependências internas e de terceiros usadas pelo ASP.NET Core e EntityFramework Core.

Todos os recursos do ASP.NET Core 2.x e Entity Framework Core 2.x estão inclusos no pacote `Microsoft.AspNetCore.All`. O modelo padrão de projeto utiliza este pacote.

O número de versão do meta-pacote `Microsoft.AspNetCore.All` representa as versões dos produtos: ASP.NET Core e Entity Framework Core (de acordo com a versão do .NET Core).

Aplicações que usam o meta-pacote `Microsoft.AspNetCore.All` recebem automaticamente a vantagem do [.NET Core Runtime Store](https://docs.microsoft.com/dotnet/core/deploying/runtime-store). O Runtime Store contém todos conjuntos de tempo de execução necessários para executar aplicações ASP.NET Core. Quando você usa o meta-pacote `Microsoft.AspNetCore.All`, **nenhum** ativo daqueles pacotes referenciados no ASP.NET Core NuGet são implantados com a aplicação &madash; o .NET Core Runtimes mantem estes ativos. Os ativos no Runtime Store são precompilados para aprimorar o tempo de inicialização da aplicação.

Você pode usar o processo de seleção de pacotes para remover aqueles que você não utiliza. Os pacotes selecionados são excluidos da saída de publicação do aplicativo.

O arquivo seguinte *.csproj* referencia o meta-pacote `Microsoft.AspNetCore.All` para o ASP.NET Core:

[!code-xml[Main](..\mvc\views\view-compilation\sample\MvcRazorCompileOnPublish2.csproj?highlight=9)]
