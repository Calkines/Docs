---
título: Trabalhando com arquivos estáticos no ASP.NET Core
autor: rick-anderson
tradutor: calkines
descrição: Trabalhando com Arquivos Estáticos
palavras-chave: ASP.NET Core,static files,static assets,HTML,CSS,JavaScript
ms.author: riande
manager: wpickett
ms.date: 4/07/2017
ms.topic: article
ms.assetid: e32245c7-4eee-4831-bd2e-915dbf9f5f70
ms.technology: aspnet
ms.prod: asp.net-core
uid: fundamentals/static-files
ms.custom: H1Hack27Feb2017
---

# Introdução para trabalhar com arquivos estáticos no ASP.NET Core

<a name=fundamentals-static-files></a>

Por [Rick Anderson](https://twitter.com/RickAndMSFT)

Arquivos estáticos, como HTML, CSS, imagens e JavaScript são ativos que uma aplicação ASP.NET Core pode distribuir diretamente aos seus clientes.

[Visualizar ou baixar código demonstrativo](https://github.com/aspnet/Docs/tree/master/aspnetcore/fundamentals/static-files/sample)

## Distribuindo arquivos estáticos

Arquivos estáticos são geralmente encontrados na pasta `web root`(*\<content-root>/wwwroot*). Veja [Pasta raiz de conteúdo](xref:fundamentals/index#content-root) e [Pasta raiz web](xref:fundamentals/index#web-root) para mais informações. Você geralmente configura a pasta raiz de conteúdo para ser o diretório atual, assim sua pasta de projeto `web root` será encontrada durante o desenvolvimento.

[!code-csharp[Main](../common/samples/WebApplication1/Program.cs?highlight=5&start=12&end=22)]

Arquivos estáticos podem ser armazenados em qualquer pasta sob o `web root` e acessados através de um caminho relativo para essa pasta.
Por exemplo, quando você cria um projeto de aplicação Web padrão usando o Visual Studio, existem diversas pastas criadas dentro da pasta *wwwroot* - *css*, *imagens* e *js*. A URI para acessar uma imagem, na subpasta *imagens*:

* `http://<app>/imagens/<imageFileName>`
* `http://localhost:9189/imagens/banner3.svg`

Para que os arquivos estáticos sejam distribuídos, você precisa cofnigurar o [Middleware](middleware.md) para adicionar arquivos estáticos ao pipeline. O middleware de arquivo estático pode ser configurado ao adicionar um dependência para o pacote *Microsoft.AspNetCore.StaticFiles* em seu projeto e então chamar o método de extensão `UseStaticFiles` da `Startup.Configure`:

[!code-csharp[Main](../fundamentals/static-files/sample/StartupStaticFiles.cs?highlight=3&name=snippet1)]

`app.UseStaticFiles();` faz os arquivos na `web root` (*wwwroot* por padrão) distribuíveis. Depois eu mostrarei como fazer outros diretórios ganharem a mesma característica com o `UseStaticFiles`.

Você precisa incluir o pacote NuGet "Microsoft.AspNetCore.StaticFiles".

> [!NOTE]
> `web root` por padrão usa a pasta *wwwroot*, mas você pode configurar o direitório `web root` através do `UseWebRoot`.

Suponha que você tenha uma hierarquia de projeto, na qual os arquivos estáticos que você queira distribuir estejam fora do diretório `web root`. Por exemplo:

* wwwroot
  * css
  * imagens
  * ...
* MeusArquivosEstaticos
  * teste.png

Para que a resuisição ganhe acesso a *teste.png*, configure o middleware de arquivo estático da seguinte maneira:

[!code-csharp[Main](../fundamentals/static-files/sample/StartupTwoStaticFiles.cs?highlight=5,6,7,8,9,10&name=snippet1)]

Um requisição para `http://<app>/MeusArquivosEstaticos/teste.png` vai distribuir o arquivo *teste.png*.

`StaticFileOptions()` pode configurar os cabeçalhos de resposta. Por exemplo, o código abaixo configura a distribuição de arquivos estáticos da pasta *wwwroot*, além do cabeçalho `Cache-Control` para tornar os arquivos públicamente armazenados no cache por 10 minutos (600 segundos).

[!code-csharp[Main](../fundamentals/static-files/sample/StartupAddHeader.cs?name=snippet1)]

![Cabeçalhos de resposta mostrando o que item Cache-Control foi adicionado](static-files/_static/add-header.png)

## Autorização de arquivo estático

O módulo de arquivo estático **não** fornece verificação de autorização. Quaisquer arquivos adicionados por ele, incluindo outros sob a pasta *wwwroot* são disponibilizados de forma pública. Para distribuir arquivos baseado em autorização:

* Armazená-los fora de *wwwroot* e qualquer direitório acessível pelo middleware de arquivos estáticos **e**
* Distrubui-los através de ação de controle, retornando um `FileResult`, no qual a autorização foi aplicada

## Habilitar pesquisa de diretório

Pesquisa de diretório permite que o usuário de sua aplicação web veja um lista de diretórios e arquivos dentro de um diretório específico. Esta pesquisa é desabilitada por padrão, por motivo de segurança (veja [Considerações](#considerations)). Para habilitar a pesquisa de diretórios, chame o método de extensão `UseDirectoryBrowser` no `Startup.Configure`:

[!code-csharp[Main](static-files/sample/StartupBrowse.cs?name=snippet1)]

E adicione os serviços requeridos, através de chamada ao método de extensão `AddDirectoryBrowser` em `Startup.ConfigureServices`:

[!code-csharp[Main](static-files/sample/StartupBrowse.cs?name=snippet2)]

O código abaixo habilita a pesquisa de diretório na pasta *wwwroot/images* usando a URL http://\<app>/myImages, juntamente com atalhos para cada arquivo e pasta:

![esquisa de direitório](static-files/_static/dir-browse.png)

Veja [Considerações](#considerations) sobre riscos de segurança quando habilita-se a pesquisa de direitório.

Perceba as duas chamadas ao `app.UseStaticFiles`. A primeira é necessário para distribuir CSS, imagens e JavaScript presentes na pasta *wwwroot*, enquanto a segunda chamada serve para a pesquisa de diretório na pasta *wwwroot/images*, usando a URL http://\<app>/MyImages:

[!code-csharp[Main](static-files/sample/StartupBrowse.cs?highlight=3,5&name=snippet1)]


## Distribuindo um documento padrão

Configurar uma página web padão oferece aos visitantes um lugar para inicar quando navegar pelo seu site. Assim para sua aplicação web distribuir uma página padrão sem que o usuário tenha que informar a URI totalmente qualificada, chame o método de extensão `UseDefaultFiles` no `Startup.Configure` como a seguir:

[!code-csharp[Main](../fundamentals/static-files/sample/StartupEmpty.cs?highlight=3&name=snippet1)]

> [!NOTE]
> `UseDefaultFiles` precisa ser chamado antes de `UseStaticFiles` para servir como arquivo padrão. `UseDefaultFiles` é uma sobrescrita de URL que não necessariamente distribui o arquivo. Você precisa habilitar o middleware de arquivos estáticos (`UseStaticFiles`) para distribui-lo como arquivo.

Quando se usa `UseDefaultFiles`, requisições para um pasta vão buscar por:

* default.htm
* default.html
* index.htm
* index.html

O primeiro arquivo, da lista acima, encontrado será distribuido como se fosse requisitado na URI totalmente qualificada (portanto a URL continuará a mostrar a URI requisitada).

O código seguinte mostra como mudar o nome do arquivo padrão para *mydefault.html*.

[!code-csharp[Main](static-files/sample/StartupDefault.cs?name=snippet1)]

## UseFileServer

`UseFileServer` combina a funcionalidade do `UseStaticFiles`,`UseDefaultFiles` e `UseDirectoryBrowser`.

O código seguinte habilita os arquivos estáticos e o arquivo padrão para serem distribuidos, mas não permite pesquisa de diretório:

```csharp
app.UseFileServer();
```

O código seguinte habilita os arquivos estáticos, arquivos padrões e pesquisa de diretório:

```csharp
app.UseFileServer(enableDirectoryBrowsing: true);
```

Veja [considerações](#considerações) sobre riscos de segurança quando permitir pesquisa de diretório. Da mesma maneira que ocorre em `UseStaticFiles`, `UseDefaultFiles` e `UseDirectoryBrowser`, se você quiser distribuir arquivos que existem fora fora do `web root`, você instância e configura um objeto `FileServerOptions` que você passa como parâmetro para o `UseFileServer`. Por exemplo, dado a seguinte hierarquia de diretório em sua aplicação web:

* wwwroot

  * css

  * imagens

  * ...

* MyStaticFiles

  * teste.png

  * default.html

Usando a hierarquia de exemplo acima, você pode querer habilitar arquivos estáticos, arquivos padrões e pesquisa de diretório para `MyStaticFiles`. No trecho de código seguinte, isso é realizado com uma única chamada ao método `FileServerOptions`.

[!code-csharp[Main](static-files/sample/StartupUseFileServer.cs?highlight=5,6,7,8,9,10,11&name=snippet1)]

Se a propriedade `enableDirectoryBrowsing` for configurada para `true` você precisará chamar o método de extensão `AddDirectoryBrowser` em `Startup.ConfigureServices`:

[!code-csharp[Main](static-files/sample/StartupUseFileServer.cs?name=snippet2)]

Usando a hierarquia de arquivos e o código acima:


| URI            |                             Resposta  |
| ------- | ------|
| `http://<app>/StaticFiles/test.png`    |      MyStaticFiles/test.png |
| `http://<app>/StaticFiles`              |     MyStaticFiles/default.html |

Se o arquivo de nome padrão não estiver no diretório *MyStaticFiles*, http://\<app>StaticFiles retorna a listagem de diretórios com atalhos clicáveis.

![Static files list](static-files/_static/db2.PNG)

> [!NOTE]
> `UseDefaultFiles` e `UseDirectoryBrowser` vai pegar a url http://\<app>/StaticFiles sem a barra diagonal e causar um redirecionamento pelo cliente para http://\<app>/StaticFiles/ (adicionando a barra invertida). Sem a barra diagonal URLs relativas dentro dos documento podem estar incorretas.

## FileExtensionContentTypeProvider

A classe `FileExtensionContextTypeProvider` contem uma coleção que mapeia extensão de arquivo para tipos de conteúdo MIME. No exemplo seguinte, diversas extensões de arquivos são registradas para tipos MIME conhecidos, o ".rtf" é substituído, enquanto o ".mp4" é removido.

[!code-csharp[Main](../fundamentals/static-files/sample/StartupFileExtensionContentTypeProvider.cs?highlight=3,4,5,6,7,8,9,10,11,12,19&name=snippet1)]

Veja [tipos de conteúdo MIME](http://www.iana.org/assignments/media-types/media-types.xhtml).

## Tipos de conteúdo não padrões

O middleware de arquivos estáticos ASP.NET conhece quase 400 tipos de coteúdo de arquivos. Se um usuário requisitar um arquivo de tipo desconhecido, o middleware de arquivo estático retorna uma respota de HTTP 404 (Não encontrado). Se a pesquisa de diretório estiver habilitada, um atalho para o arquivo será exibido, mas a URI vai retonar um erro HTTP 404.

O código seguinte habilita distribuíção de tipos desconhecidos e vai renderizar o arquivo desconhecido como um imagem.

[!code-csharp[Main](static-files/sample/StartupServeUnknownFileTypes.cs?name=snippet1)]

Com o código acima, uma requisição para um arquivo com tipo de conteúdo deconhecido vai retornar como uma imagem.

>[!WARNING]
> Habilitar `ServeUnknowFileTypes` é um risco de segurança e seu uso é desencorajado. `FileExtensionContentTypeProvider` (explicado acima) fornece uma alternativa segura para distribuir arquivos de extensões desconhecidas.

### Considerações

>[!WARNING]
> `UseDirectoryBrowser` e `UseStaticFiles` podem expor segredos. Nós recomendamos que você **não** habilite a pesquisa de diretório em produção. Tome cuidado com quais diretórios você habilitará `UseStaticFiles` ou `UseDirectoryBroswer`, como também fique atento se deve expor todo diretório, inclusive suas subpastas, pois estas tornar-se-ão acessíveis. Nós recomendamos manter o conteúdo público em seu próprio diretório, por exemplo *<content root>/wwwroot*, longe das visualizações da aplicação, arquivos e configuração e outros.

* As URLs para conteúdo expostos com `UseDirectoryBrowser` e `UseStaticFiles` estão sujeitas a sensibilidade entre caracteres maiúsculos e minúsculos, além de restrição de caracteres oriundas do sistema de arquivos. Por exemplo, o Windows faz distinção entre caracteres maiúsculos e minúsculos, mas o Mac e o Linux não.

* Aplicações ASP.NET Core hospedadas no IIS usam o módulo ASP.NET Core para encaminhar todas requisições para a aplicação, incluindo requisições para arquivos estáticos. O manipulador de arquivos estáticos do IIS não é usado porque ele deixa as requisições serem manipuladas pelo módulo do ASP.NET Core.

* Para remover o manipulador de arquivos estáticos do IIS (no nível de servidor ou website):

     * Vá até a funcionalidade **Modules**
     
     * Selecione **StaticFileModule** na lista
     
     * Aperte **Remove** na barra lateral **Actions**

>[!WARNING]
> Caso o manipulador de arquivo estático do IIS esteja habilitado **e** o módulo ASP.NET Core (ANCM) não esteja corretamente configura (por exemplo se o *web.config* não foi implantado), arquivos estáticos serão distribuídos.

* Arquivos de código (incluindo C# e Razor) precisam ser colocados fora da pasta de projeto `web root`(*wwwroot* por padrão). Isto cria um boa separação entre seu conteúdo de client-side da aplicação e o código fonte do server-side, além de previnir vazamento de código existente no lado servidor.

## Recursos Adicionais

* [Middleware](middleware.md)

* [Introdução ao ASP.NET Core](../index.md)
