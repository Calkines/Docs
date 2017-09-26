---
título: Módulo ASP.NET Core
autor: tdykstra
tradutor: calkines
descrição: Apresenta o ASP.NET Core Módulo (ANCM*), um módulo IIS, que permite o servidor web Kestrel usar o IIS ou IIS Express como um servidor de proxy reservo.
palavra-chave: ASP.NET Core,IIS,IIS Express,ASP.NET Core Module,UseIISIntegration
ms.author: tdykstra
gerente: wpickett
ms.date: 08/03/2017
ms.topic: article
ms.assetid: 4661af33-34c5-4d71-93a0-8c7632f43580
ms.technology: aspnet
ms.prod: asp.net-core
uid: fundamentals/servers/aspnet-core-module
ms.custom: H1Hack27Feb2017
---
# Introdução ao ASP.NET Core Module

Por [Tom Dykstra](https://github.com/tdykstra), [Rick Strahl](https://github.com/RickStrahl), e [Chris Ross](https://github.com/Tratcher) 

O Módulo ASP.NET Core (ANCM) permite que você execute aplicações ASP.NET Core por trás do IIS, usando o IIS para aquilo o que ele faz bem (segurança, gerenciameto, e diversas outras) e usando o [Kestrel](kestrel.md) para aquilo que ele faz bem (iniciar realmente rápido), e, portanto, utilizando os benefícios das duas tecnologias ao mesmo tempo. **ANCM funciona apenas com Kestrel; esta tecnologia não é compatível com WebListener (in ASP.NET Core 1.x) ou HTTP.sys (in 2.x).**

Versões do Windows suportadas:

* Windows 7 e Windows Server 2008 R2 e recentes

[Visualizar ou baixar códigos de demonstração](https://github.com/aspnet/Docs/tree/master/aspnetcore/fundamentals/servers/aspnet-core-module/sample)

## O que o Módulo ASP.NET Core faz?

O ANCM é um nativo módulo IIS, que se encaixa no pipeline IIS e redireciona o trágefo para a aplicação backend ASP.NET Core. A maioria dos outros módulos, como a de autenticação via windows, ainda podem ser executados. O ANCM somente assume o controle quando um manipulador é selecionado para requisições, e o mapeamento do manipulador for definido no arquivo *web.config* da aplicação.

Porque a aplicação ASP.NET Core funciona em processo separado daquele do IIS, o ANCM também realiza um gerenciamento de processos. O ANCM inicia o processo para a aplicação ASP.NET Core no momento em que a primeira requisição chega, e o reinicia quando este sofrer uma parada inesperada. Este é o essencialmente o mesmo comportamento de uma aplicação clássica em ASP.NET, que executa in-process no IIS e é gerenciada por WAS (Serviço de Ativação do Windows).

Aqui está um diagrama que ilustra o relacionamento entre IIS, ANCM e as aplicações ASP.NET Core.

![Módulo ASP.NET Core](aspnet-core-module/_static/ancm.png)

Requisições vêm da Web e encontram o driver do kernel Http.Sys, que os redireciona para o IIS, na porta primária (80) ou na porta SSL (443). ANCM encaminha as requisições para a aplicação ASP.NET Core, na porta HTTP configurada para a aplicação, a qual não é a 80/443.

O Kestrel atende o tráfego que vem do ANCM. O ANCM, em sua inicialização, especifica a porta destino via variável de ambiente, e o método [UseIISIntegration](#call-useiisintegration) configura o servidor para atender o `http://localhost:{port}`. Existem verificações adicionais para rejeitar requisições que não sejam do ANCM. (O ANCM não oferece suporte a encaminhamento de HTTPS, então requisições são encaminhadas por HTTP mesmo se forem recebidas via IIS HTTPS.)

O Kestrel recolhe as requisições do ANCM e as envia para o middleware pipeline do ASP.NET Core, o qual as trata e passa via instãncia do `HttpContext` para a lógica da aplicação. As respostas da aplicação são então passadas de volta ao IIS, que as envia de volta ao cliente HTTP, que iniciou as requisições.

O ANCM possui algumas outras funções:

* Configurar variáveis de ambientes.
* Anotar as saídas de `stdout` em um arquivo de armazenamento.
* Encaminhar tokens de autenticação Windows.

## Como usar o ANCM nos aplicativos ASP.NET Core

Esta seção fornece uma visão geral de como o processo de configuração de um servidor IIS e uma aplicação ASP.NET Core. Para instruções detalhadas, veja [Publicando para o IIS](.../../publishing/iis.md).

### Instalar o ANCM

O módulo ASP.NET Core precisa ser instalado no IIS em seus servidores e no IIS Express nas máquinas usadas pra desenvolvimento. Para servidores, o ANCM é incluído no [.NET Core Windows Server bundle](https://aka.ms/dotnetcore.2.0.0-windowshosting). Para máquina de desenvolvedores, o Visual Studio instala automaticamente o ANCM no IIS, e no IIS se este estiver presente na máquina.

### Instalar o pacote IISIntegration via NuGET 

# [ASP.NET Core 2.x](#tab/aspnetcore2x)

O pacote [Microsoft.AspNetCore.Server.IISIntegration](https://www.nuget.org/packages/Microsoft.AspNetCore.Server.IISIntegration/) é incluso nos meta-pacotes do ASP.NET Core ([Microsoft.AspNetCore](https://www.nuget.org/packages/Microsoft.AspNetCore/) e [Microsoft.AspNetCore.All](xref:fundamentals/metapackage)). Se você não usar um meta-pacote, instale o `Microsoft.AspNetCore.Server.IISIntegration` separadamente. O pacote `IISIntegration` é um pacote interoperável, e realiza a leitura de variáveis de ambiente via broadcast para cofigurar sua aplicação. As variáveis de ambiente fornecem informações de configuração, como a porta atendida naquele momento.

# [ASP.NET Core 1.x](#tab/aspnetcore1x)

Em sua aplicação, instale Microsoft.AspNetCore.Server.IISIntegration](https://www.nuget.org/packages/Microsoft.AspNetCore.Server.IISIntegration/). O pacote `IISIntegration` é inteoperável, e realiza a leitura de variáveis de ambiente via broadcast para cofigurar sua aplicação. As variáveis de ambiente fornecem informações de configuração, como a porta atendida naquele momento.

---

### Chamar o UseIISIntegration

# [ASP.NET Core 2.x](#tab/aspnetcore2x)

O método de extensão `UseIISIntegration` no [`WebHostBuilder`](https://docs.microsoft.com/aspnet/core/api/microsoft.aspnetcore.hosting.webhostbuilder) é chamado automaticamente quando você executa com o IIS.

Se você não estiver usando um meta-pacote ASP.NET Core e não instalou o pacote `Microsoft.AspNetCore.Server.IISIntegration`, você receberá um erro em tempo de execução. Se você chamar explicitamente `UseIISIntegration`, você receberá um erro em tempo de compilção, caso o pacote não esteja instado.

# [ASP.NET Core 1.x](#tab/aspnetcore1x)

No método `Main` de sua aplicação, chame o método de extensão `UseIISIntegration` no [`WebHostBuilder`](https://docs.microsoft.com/aspnet/core/api/microsoft.aspnetcore.hosting.webhostbuilder). 

[!code-csharp[](aspnet-core-module/sample/Program.cs?name=snippet_Main&highlight=12)]

---

O método `UseIISIntegration` procura por variáveis de ambientes que o ANCM configura, e caso não encontre nada é feito. Este comportamento facilita cenários como os de desenvolvimento e testes no macOS e Linux, e a implantação em servidores que executam IIS. Enquanto executando no macOS ou Linux, o Kestrel age como um servidor web; mas quando a aplicação é implementada em um ambiente IIS, ela automaticamente usa ANCM e IIS.

### O vínculo de portas do ANCM sobrescreve outros vínculos

# [ASP.NET Core 2.x](#tab/aspnetcore2x)

O ANCM gera uma porta dinâmica para atribuí-la ao processo de back-end. O método `UseIISIntegration` recupera esta porta dinâmica e configura o Kestrel para atender o `http://localhost:{ dynamicPort}/`. Isso sobrescreve outras configurações de URL, como as chamadas feitas ao método `UseUrls` ou [Kestrel's Listen API](xref:fundamentals/servers/kestrel?tabs=aspnetcore2x#endpoint-configuration). Portanto, você não precisa chamar nenhum destes itens, `UseUrls` ou a API do Kestrel, quando estiver usando o ANCM. Se você chamar `UseUrls` ou `Listen`, o Kestrel vai atender somente a porta que você especificou quando iniciou a aplicação sem o IIS.

# [ASP.NET Core 1.x](#tab/aspnetcore1x)

O ANCM gera uma porta dinâmica para atribuí-la ao processo de back-end. O método `UseIISIntegration` recupera esta porta dinâmica e configura o Kestrel para atender o `http://localhost:{dynamicPort}/`. Isso sobrescreve outras configurações de URL, como as chamadas ao método `UseUrls` quando você usa o ANCM. Se você chamar `UseUrls`, o Kestrel atende a porta especificada no momento que você executou a aplicação sem o IIS.

No ASP.NET Core 1.0, se você chamar `UseUrls`, faça isso **antes** de chamar `UseIISIntegration`, assim a porta configurado pelo ANCM não será sobrescrita. Essa order de chamada não é necessária no ASP.NET Core 1.1, porque a configuração do ANCM sobrescreve o `UseUrls`.

---

### Configurar opções do ANCM no web.config

A configuração para o ASP.NET Core Module é armazenada no arquivo *Web.config*, que está localizado a pasta raiz da aplicação. Configurações feitas neste arquivo apontam para o comando de inicialização e para os argumentos que iniciaram sua aplicação ASP.NET Core. Para demonstrações de código web.config e um guia sobre opções de configuração, veja [Referência de Configuração do Módulo ASP.NET Core](../../hosting/aspnet-core-module.md).

### Executar com o IIS Express em modo desenvolvimento

O IIS Express pode ser iniciado pelo Visual Studio usando o perfil padrão definido por em um modelo ASP.NET Core.

## Próximos passos

Para mais informações, veja os recursos seguintes:

* [Exemplos de aplicações deste artigo](https://github.com/aspnet/Docs/tree/master/aspnetcore/fundamentals/servers/aspnet-core-module/sample)
* [Código fonte do Módulo ASP.NET](https://github.com/aspnet/AspNetCoreModule)
* [Referência de Configuração para Módulo ASP.NET Core](../../hosting/aspnet-core-module.md)
* [Publicando para o IIS](../../publishing/iis.md)
