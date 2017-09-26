---
título: Recusos de Requisição no ASP.NET Core
autor: ardalis
tradutor: calkines
decrição: 
palavra-chave: ASP.NET Core,
ms.author: riande
manager: wpickett
ms.date: 10/14/2016
ms.topic: article
ms.assetid: d1fbd23c-2ff9-4216-b908-0201ff3afb7c
ms.technology: aspnet
ms.prod: asp.net-core
uid: fundamentals/request-features
---

# Recursos de Requisição no ASP.NET Core

Por [Steve Smith](https://ardalis.com/)

Detalhes de implementações de servidor web relacionadas a requisições HTTP e respostas são definidos em interfaces. Estas interfaces são usadas por implementações de servidor e middleware para criar e modificar o pipeline da aplicação host.

## Funcionalidades de interface

O ASP.NET Core define um número de interfaces funcionais para HTTP no `Microsoft.AspNetCore.Http.Features`, que são usadas pelos servidores para identificar os recursos suportados. As interfaces funcionais seguintes manipulam requisições e retornam respostas:

`IHttpRequestFeature`
   Define a estrutura de uma requisição HTTP, incluindo o protocolo, caminho, texto de consulta, cabeçado e corpo.

`IHttpResponseFeature`
   Define a estrutura de uma respota HTTP, incluindo o código de status, cabeçalho e corpo da respota.

`IHttpAuthenticationFeature`
   Define o suporte para identificação de usuários com base em `ClaimsPrincipal` e especificação de um manipulador de autenticação.
   
`IHttpUpgradeFeature`
   Define o suporte para [Atualizações HTTP](https://tools.ietf.org/html/rfc2616.html#section-14.42), que permitem ao cliente especificar quais protocolos adicionais seriam utilizados caso o servidor precisa-se trocar protocolos.
   
`IHttpBufferingFeature`
   Define métodos para desabilitar o buffering de requisições e/ou repostas.   

`IHttpConnectionFeature`
   Define propriedades e portas para endereços locais e remotos.   

`IHttpRequestLifetimeFeature`
   Define o suporte para abortar conexões, ou detectar se uma requisição foi terminada prematuramente, como no caso de uma desconexão do cliente.   

`IHttpSendFileFeature`
   Define um método para enviar arquivos de forma assíncrona.   

`IHttpWebSocketFeature`
   Define uma API para dar suporte a web sockets.   

`IHttpRequestIdentifierFeature`
   Adicona uma propriedade que pode ser implementada para identificação única de requisições.   

`ISessionFeature`   
   Define as abstrações `ISessionFactory`e `ISession` para dar suporte a seções de usuário.   

`ITlsConnectionFeature`
   Define uma API para receber certificados de clientes.   

`ITlsTokenBindingFeature`
   Define métodos para trabalhar com parâmetros vinculados de token TLS.   

> [!NOTE]
> `ISessionFeature` não é uma funcionalidade de servidor, mas é implementada via `SessionMiddleware` (veja [Gerenciando Estado de Aplicação](app-state.md)).

## Coleções de recursos

A propriedade `Features` do `HttpContext` fornece uma interface para recuperar e configurar recursos HTTP disponíveis para a requisição atual. Uma vez que a coleção de recursos é mutável, mesmo dentro do contexto de uma requisição, o middleware pode ser usado para modificar esta coleção e passar a suporte novos recursos.

## Middleware e recursos de requisição

Enquanto servidores são responsáveis pela criação de coleção de recursos, middleware podem tanto adicionar itens a esta coleção quato consumí-los. Por exemplo, o `StaticFileMiddleware` acessa o recurso `IHttpSendFileFeature`. Se o recurso existir, ele será usado para enviar um o arquivo estatíco requisitado de seu local físico. De outro modo, um método alternativo mais vagaroso é usado para enviar o arquivo. Quando disponível, o `IHttpSendFileFeature` permite ao sistema operacional abrir o arquivo e realizar uma cópia de modo kernel direta para a placa de rede.

Adicionalmente, middleware pode fazer adições a uma coleção de recurso estabelecida pelo servidor. Recursos existentes podem ser substituídos pelo middleware, permitindo aumentar o funcionamento do servidor. Recursos adicionados à coleção estão disponíveis imediatamente a outro middleware ou, depois, à própria aplicação, no pipeline requisitado.

Ao combinar implementações customizadas de servidor e melhoramentos específicos de middleware, o conjunto preciso de recursos necessários a uma aplicação podem ser construídos. Isso permite que recursos inexistêntes sejam adicionados sem requisições de mudança no servidor, e garantem que apenas um mínimo montante de recursos seja exposto, limitando, assim, ataques de superfície e melhorando a performance.

## Sumário

Interfaces de recursos definem características especificas de HTTP, que uma determinada requisição pode suportar. Servidores definem coleções de recursos, e o conjunto inicial de funcionalidades suportadas por aquele servidor, mas middlewares podem ser usados para aumentar estas características.

## Recursos Adicionais

* [Servidores](servers/index.md)

* [Middleware](middleware.md)

* [Interface Web Aberta para .NET (OWIN)](owin.md)
