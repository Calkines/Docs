---
titulo: Implementação de web server ASP.NET Core
autor: tdykstra
descrição: Apresentação de web servers Kestrel e WebListener para ASP.NET Core. Fornece um guia sobre como escolher um servidor e quando usar um servidor com sistema de proxy reverso.
tradutor: calkines
palavra-chave: ASP.NET Core,IServer,servidor web,Kestrel,WebListener,proxy reverso
ms.author: tdykstra
gerente: wpickett
ms.date: 08/03/2017
ms.topic: article
ms.assetid: dba74f39-58cd-4dee-a061-6d15f7346959
ms.technology: aspnet
ms.prod: asp.net-core
uid: fundamentals/servers/index
---

# Implementação de servidor web em ASP.NET Core

Por [Tom Dykstra](https://github.com/tdykstra), [Steve Smith](https://ardalis.com/), [Stephen Halter](https://twitter.com/halter73), e [Chris Ross](https://github.com/Tratcher)

Uma aplicação ASP.NET Core funciona com uma implementação de servidor HTTP in-process. A implementação do servidor escuta requisições HTTP e as entrega à aplicação como um conjunto de [recursos de requisição](https://docs.microsot.com/asp.net/core/fundamentals/request-features) composto em um `HttpContext`.

O ASP.NET Core fornece duas implementações de servidores:

# [ASP.NET Core 2.x](#tab/aspnetcore2x)

* [Kestrel](kestrel.md), que é um servidor HTTP para multi-plataforma baseado na [libuv](https://github.com/libuv/libuv), uma biblioteca I/O assíncrona multi-plataforma.

* [HTTP.sys](httpsys.md), que é um servidor apenas para Windows beaseado no [Http.Sys kernel driver](https://msdn.microsoft.com/library/windows/desktop/aa364510.aspx).

# [ASP.NET Core 1.x](#tab/aspnetcore1x)

* [Kestrel](kestrel.md), que é um servidor HTTP para multi-plataforma baseado na [libuv](https://github.com/libuv/libuv), uma biblioteca I/O assíncrona multi-plataforma.

* [WebListener](weblistener.md), que é um servidor apenas para Windows beaseado no [Http.Sys kernel driver](https://msdn.microsoft.com/library/windows/desktop/aa364510.aspx).
---

## Kestrel

O Kestrel é o servidor web que é incluso por padrão em modelos de um novo projeto ASP.NET.

# [ASP.NET Core 2.x](#tab/aspnetcore2x)

Você pode usar somente Kestrel limpo ou com um *servidor de proxy reverso*, como um IIS, Nginx ou Apache. Um servidor de proxy reverso recebe uma requisição HTTP da internet e a encaminha ao Kestrel após algumas considerações preliminares.

![Kestrel comunica-se diretamente com a Internet sem um servidor de proxy reverso](kestrel/_static/kestrel-to-internet2.png)

![Kestrel comunica-se indiretamente com a Internet através de um servidor de proxy reverso, como o IIS, Ngix ou Apache](kestrel/_static/kestrel-to-internet.png)

Ambas configurações &mdash; com ou sem servidor de proxy reverso &mdash; podem ser usadas se o Kestrel for exposto apenas para redes internas.

Para informações sobre quando usar o Kestrel com um proxy reverso, veja [Introdução ao Kestrel](kestrel.md).

# [ASP.NET Core 1.x](#tab/aspnetcore1x)

Se sua aplicação aceitar requisições apenas de redes internas, você pode usar somente o Kestrel.

![Kestrel comunica-se diretamente com sua rede interna](kestrel/_static/kestrel-to-internal.png)

Se você expor sua aplicação para a Internet, você precisa usar o IIS, Ngix ou Apache como um *servidor de proxy reverso*. Um servidor de proxy reverso recebe requisições HTTP da Internet e as encaminha ao Kestrel depois de algumas considerações preliminares, como exibe no diagrama seguinte:

![Kestrel comunica-se indiretamente com a Internet através de um servidor de proxy reverso, como IIS, Ngix ou Apache](kestrel/_static/kestrel-to-internet.png)

A razão mais importante para usar um proxy reverso, para implantações de borda (aquelas expostas ao tráfego da internet), é a segurança. As versões 1.x do Kestrel não possuem um complemento total de defesas contra possível agressores. Isto inclui, mas não é tudo: tempo de expiração apropriados, limite de tamanhos, e limite a quantidade de conexões.

Para mais informações sobre quando usar o Kestrel com um proxy reverso, veja [Introdução ao Kestrel](kestrel.md)

---

Você não pode usar o IIS, Nginx ou Apache sem uma [implementação de servidor customizada](#custom-servers) do Kestrel. O ASP.NET Core foi projetado para ser executado em seu próprio processo, podendo, assim, compartar-se de maneira consistênte em diversas plataformas. O IIS, Nginx e o Apache ditam seus próprios processos de inicialização e ambientes; para usá-los diretamente, o ASP.NET Core precisaria adaptar-se as necessidades de cada um. Usar uma implementação de servidor web como a do Kestrel permite ao ASP.NET Core controlar todo processo de inicialização do processo e ambiente. Então, em vez de tentar adaptar o ASP.NET Core ao IIS, Nginx ou Apache, você simplesmente configura aqueles servidores web para requisições web e encaminha ao Kestrel. Isso permite suas classes `Program.Main` e `Startup` serem essencialmente as mesmas, independente de onde você as implementar.

### Kestrel com IIS

Quando você utiliza o IIS ou o IIS Express como um proxy reverso para o ASP.NET Core, esta aplicação executa em um processo separado daqueles. No processo do IIS, um módulo IIS especial é executado para coordenar os relacionamentos de proxy reverso. Este é o *ASP.NET Core Module*. As funções primárias do módulo ASP.NET Core são para iniciar a aplicação ASP.NET Core, você deve reiniciá-lo quando ele parar de responder, após isso, e encaminhar o tráfego HTTP novamente a ele. Para mais detalhes, veja [Módulo ASP.NET Core](aspnet-core-module.md).

### Kestrel com Nginx

Para informações sobre como usar o Nginx no Linux como um servidor de proxy reverso para o Kestrel, veja [Publicando em um ambiente de produção Linux](../../publishing/linuxproduction.md).

### Kestrel com Apache

Para informações sobre como usar o Apache no Linux como um servidor de proxy reverso para o Kestrel, veja [Usando o Servidor Web Apache como um proxy reverso](../../publishing/apache-proxy.md).

## HTTP.sys

# [ASP.NET Core 2.x](#tab/aspnetcore2x)

Se você executar sua aplicação ASP.NET Core no Windows, o HTTP.sys é uma alternativa em relação ao Kestrel. Você pode usar o HTTP.sys para situações nas quais você expõe seu aplicativo para a Internet e você precisa de funcionalidades do HTTP.sys, as quais não são suportadas pelo Kestrel.

![HTTP.sys comunica-se diretamente com a Internet](httpsys/_static/httpsys-to-internet.png)

O HTTP.sys também pode ser usado para aplicações que são expostas apenas a rede interna.

![HTTP.sys comunica-se diretamente com seu rede interna](httpsys/_static/httpsys-to-internal.png)

Para cenários envolvendo redes internas, o Kestrel é geralmente recomendado por melhor performance; mas, em alguns cenários, você pode querer usar alguma funcionalidade oferecida apenas pelo HTTP.sys. Para mais informações sobre funcionalidades do HTTP.sys, veja [HTTP.sys](httpsys.md)

# [ASP.NET Core 1.x](#tab/aspnetcore1x)

 O HTTP.sys é chamado de WebListener no ASP.NET Core 1.x. Se você executar sua aplicação ASP.NET Core no Windows, o WebListener é uma alternativa que você pode usar para cenários onde você queira expor seu aplicativo para a Internet, mas não queira usar o IIS.

![Weblistener comunica-se diretamente com a Internet](weblistener/_static/weblistener-to-internet.png)

O WebListener também pode ser usado no lugar do Kestrel para aplicações que são expostas somente para a rede interna, se você precisar de funcionalidades do Weblistener que o Kestrel não suporte.

![Weblistener comunica-se diretamente com a rede interna](weblistener/_static/weblistener-to-internal.png)

Para cenários de rede interna, o Kestrel é geralmente recomendado pela melhor performance, mas em alguns cenários você pode querer usar uma funcionalidade oferecida apenas pelo WebListener. Para mais informações sobre funcionalidades WebListener, veja [WebListener](weblistener.md).

---

## Notas sobre a infraestrutura do servidor ASP.NET Core

O [`IApplicationBuilder`](/aspnet/core/api/microsoft.aspnetcore.builder.iapplicationbuilder), dispoível na classe `Startup`, mais precisamente no método `Configure`, expõe a propriedade `ServerFeatures` do tipo [`IFeatureCollection`](/aspnet/core/api/microsoft.aspnetcore.http.features.ifeaturecollection). Tanto Kestrel quanto WebListener expõe apenas uma única funcionalidade, [`IServerAddressesFeature`](/aspnet/core/api/microsoft.aspnetcore.hosting.server.features.iserveraddressesfeature), mas implementações de servidores diferentes podem expor funcionalidades adicionais.

A interface `IServerAddressesFeature` pode ser usada para encontrar qual porta do servidor de implementação está vinculada ao tempo de execução.

## Server customizado

Se os servidores disponíveis por padrão não atendem sua necessidade, você pode criar uma implementação de servidor customizada. A [Guia para Interface Web Aberta para .NET [OWIN*](../owin.md) demonstra como escrever um [Nowin](https://github.com/Bobris/Nowin) com bease na implementação [IServer](https://docs.microsoft.com/aspnet/core/api/microsoft.aspnetcore.hosting.server.iserver). Você é livre para implementar apenas as funcionalidades necessárias à sua aplicação, embora, no mínimo, você deve suportar [IHttpRequestFeature](https://docs.microsoft.com/aspnet/core/api/microsoft.aspnetcore.http.features.ihttprequestfeature) e [IHttpResponseFeature](https://docs.microsoft.com/aspnet/core/api/microsoft.aspnetcore.http.features.ihttpresponsefeature).

## Próximos passos

Para mais informações, veja os recursos seguintes:

# [ASP.NET Core 2.x](#tab/aspnetcore2x)

- [Kestrel](kestrel.md)
- [Kestrel com IIS](aspnet-core-module.md)
- [Kestrel com Nginx](../../publishing/linuxproduction.md)
- [Kestrel com Apache](../../publishing/apache-proxy.md)
- [HTTP.sys](httpsys.md)

# [ASP.NET Core 1.x](#tab/aspnetcore1x)

- [Kestrel](kestrel.md)
- [Kestrel com IIS](aspnet-core-module.md)
- [Kestrel com Nginx](../../publishing/linuxproduction.md)
- [Kestrel com Apache](../../publishing/apache-proxy.md)
- [WebListener](weblistener.md)

---
