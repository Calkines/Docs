---
título: Rotas no ASP.NET Core
autor: ardalis
tradutor: Calkines
descricao: 
palavras-chave: ASP.NET Core,
ms.author: riande
manager: wpickett
ms.date: 10/14/2016
ms.topic: article
ms.assetid: bbbcf9e4-3c4c-4f50-b91e-175fe9cae4e2
ms.technology: aspnet
ms.prod: asp.net-core
uid: fundamentals/routing
---

# Roteamento no ASP.NET Core

Por [Ryan Nowak](https://github.com/rynowak), [Steve Smith](https://ardalis.com/), e [Rick Anderson](https://twitter.com/RickAndMSFT)

A funcionalidade de roteamento é responsável por mapear um requisição de entrada e por um controlador sua rota. Rotas são definidas na aplicação ASP.NET e configuradas quando acontece a inicialização. Uma rota pode opcionalmente extrair valores de uma URL contida em uma requisição, e estes valores podem ser usados para processamento de requisição. Usando informações de rota da aplicação ASP.NET, a funcionalidade de roteamento também é capaz de gerar URLs que mapeiam os controladores de rota. Portanto, o roteamento pode encontrar um controlardor de rota com base na URL, ou uma URL correspondente a um determinado manipulador de rotas, leavando em consideração as informações do manipulador de rotas.

>[!IMPORTANTE]
> Este documento cobre cobre o baixo nível de roteamento do ASP.NET Core. Para roteamento ASP.NET Core MVC, veja [Roteamento para Ações de Controle](../mvc/controllers/routing.md)

[Visualizar ou baixar códigos demonstrativos](https://github.com/aspnet/Docs/tree/master/aspnetcore/fundamentals/routing/sample)

## Roteamento básico

O roteamento usa *routes* (implementações do [IRouter](https://docs.microsoft.com/aspnet/core/api/microsoft.aspnetcore.routing.irouter)) 
para:

* mapear requisições de entrada para *controladores de rota*

* gerar URLs usada nas respostas

Geralmente, uma aplicação possui uma única coleção de rotas. Quando uma requisição chega, a coleção de rotas é processada. A requisição de entrada procura por um rota que coincida com a URL requisitada ao chamar o método `RouteAsync` em cada rota disponível na coleção de rotas. Por outro lado, a respota pode usar o roteamento para gerar URLs (por exemplo, para redirecimento ou atalhos) baseadas nas informações de rota, e, assim, deixar as URL fixas de lado, o que ajuda na manutenibilidade.

O roteamento é **conectado ao pipeline** de [middleware](middleware.md) pela classe `RouterMiddleware`. [ASP.NET MVC](../mvc/overview.md) adiciona o roteamento ao pipeline de middleware como parte de sua configuração. Para aprender sobre como usar o roteamento como um componente isolado, veja [usando-roteamento-middleware](#using-routing-middleware).

<a name=url-matching-ref></a>

### Coincidência de URL

Coincidência de URL é o processo pelo qual o roteamento despacha uma requisição de entrada para um *controlador* (handler). Este processo é geralmente baseado em dados de entrada que foram informados no caminho URL, mas pode ser estendido para considerar qualquer informação da requisição. A habilidade de despachar requisições para diversos controladores é a chave escalonar o tamanho e a complexidade de uma aplicação.

Requisições de entrada entram no `RouterMiddleware`, o qual chama o método `RouteAsync` para cada rota encontrada na sequência. A instância de `IRouter` escolhe se *controla* a requisição ao configurar o `RouteContext` `Handler`para um `RequestDelegate` não nulo. Se uma rota definir um controlador para a requisição, o processo de rota para e o controlador será invocado para processar a requisição. Se todas as rotas forem testadas e nenhum controlador for encontrado para a requisição, o middleware chama *next* e o próximo middleware no pipeline de requisições será invocado.

A entrada primária para `RouteAsync` é o `RouteContext` `HttpContext` associados à requisição atual. O `RouteContext.Handler` e `RouteContext` `RouteData` são saída que vão ser configuradas após a coincidência de rota.  

Um coindicência durante `RouteAsync` também vai configurar as propriedades do `RouteContext.RouteData` para valores apropriados com base no término do processo de requisição. Se uma rota coindice com uma requisição, o `RouteContext.RouteData` vai conter importante informações de estado sobre o *resultado*.

`RouteData` `Values` é um direcionário de *valores de rotas* produzido por uma rota. Estes valores são usualmente determinados ao transformar a URL em token, e podem ser usados para aceitar dados de entrada do usuário, ou fazer encaminhamento de decisões futuras dentro da aplicação.

`RouteData` `DataTokens` é um sacola de propriedades de dados relacionados a coincidência de rota. Os "DataTokens" são fornecidos para suportar a associação de dados de estado com cada rota para que o aplicativo possa tomar decisões posteriormente com base na rota correspondente. Estes valores são definidos pelo desenvolvedor e **não** afetam o compotamento do roteamento de nenhum modo. Além disso, valores escondidos em tokens de informação podem ser de qualquer tipo, diferentemente de valores de rotas, que precisam ser facilmente convertidos para texto e vice-versa.

`RouteData` `Routers` é uma lista de rotas que pega a parte correta do processo de coindicência da requisição. Rotas podem ser aninhadas umas dentros das outras, e a propriedade `Routers` reflete o caminho pelo qual a árvore lógica resultou em uma coindidência. Geralmente o primeiro item no `Routers` é uma coleção de rotas, e precisa ser usado para geração de URL. O último item no `Routers` é o controlador de rota que coindidiu.

### Geração de URL

Geração de URL é o processo pelo qual o roteamento pode criar um caminho URL com base no conjunto de valores de rota. Isto permite uma separação lógica entre seus controladores e as URLs que os acessam.

A geração de URL segue um processo interativo similar, mas inicia com o usuário ou o código framework fazendo uma chamada ao método `GetVirtualPath` da coleção de rota. Cada *route* então terá seu método `GetVirtualPath` chamado em sequência até que um `VirtualPathData` não nulo seja retornado.

Os dados de entrada primários para `GetVirtualPath` são:

* `VirtualPathContext` `HttpContext`

* `VirtualPathContext` `Values`

* `VirtualPathContext` `AmbientValues`

Inicialmente rotas usam os valores de rota fornecidos pelo `Values` e `AmbientValues` para decidir onde é possível gerar uma URL e quais valores incluir. `AmbientValues` são o conjunto de valores de rota que foram produzidos pela coincidência da requisição atual com o sistema de rotas. Por outro lado, `Values` são os valores de rotas que especificam como gerar a URL desejada para a operação atual. O `HttpContext`é fornecido no caso das rotas precisarem de acesso a serviços ou dados associados adicionais com o contexto atual.

Dica: Pense nos `Values` como um conjunto de sobrescritas para o `AmbientValues`. A geração de URL tenta reutilizar os valores de rota da requisição atual para tornar isso facil para gerar URLs para atalhos usando a mesma rota ou valores de rotas.

A saída de `GetVirtualPath` é um `VirtualPathData`. `VirtualPathData` é um paralelo de `RouteData`; isso contem o `VirtualPath` para a saída de URL como também algumas propriedade adicionais que precisam ser configuradas pela rota.

As propriedades `VirtualPathData` `VirtualPath` contem o *caminho vitual* produzido pela rota. Dependendo de suas necessidades você pode precisar processar o caminho em momento futuro. Por exemplo, se você  quiser renderizar a URL gerada em HTML você precisa inserir o caminho base da aplicação.

O `VirtualPathData` `Router` é uma referência à rota que gerou corretamente uma URL.

As propriedades `VirtualPathData` `DataTokens` são um dicionário de dados relacionados adicionais para a rota que gerou a URL. Este é o paralelo de `RouteData.DataTokens`.


### Criando rotas

O roteamento fornece a classe `Route` como uma implementação padrão de `IRouter`. `Route` usa uma sintaxe de *modelo de rota* para definir matrizes que vão coincidir nos caminhos URL quando `RouteAsync` for chamado. `Route` vai usar o mesmo modelo de rota para gerar um URL quando `GetVirtualPath` for chamado.

A maioria das aplicações vai criar rotas ao chamar `MapRoute` ou um método de extensão similar definido no `IRouteBuilder`. Todos estes métodos vão criar uma instância de `Route` e adicioná-la à coleção de rota.

Nota: `MapRoute` não recebe um parâmetro controlador de rota - ele apenas adiciona rotas que serão controladas por `DefaultHandler`. Já que o *default handler* é um `IRouter`, ele pode decidir não controlar a requisição. Por exemplo, o ASP.NET MVC é geralmete configurado como um controlador padrão que somente controla requisições que coincidam com um *controller* e uma ação disponíveis. Para aprender mais sobre o roteamento para MVC, veja [Roteamento para Ações de Controle](../mvc/controllers/routing.md).

Este é um exemplo de uma chamada `MapRoute` usada pela definição típica de rota do ASP.NET MVC:

```csharp
routes.MapRoute(
    name: "default",
    template: "{controller=Home}/{action=Index}/{id?}");
```

Este modelo vai coincidir o caminho URL como `/Produtos/Detalhes/17` e extrair os valores de rota `{controller = Produtos, action = Detalhes, id = 17}`. Os valores de rota são determinados pela divisão do caminho URL em seguimentos, e a coincidência de cada segmento com o nome do *parâmetro de rota* no modelo de rota apresenta acima. Parâmetros de rota são nomeados. Eles são definidos quando os cercamos com chaves `{ }`;

O modelo acima também pode coincidir o caminho URL `/` and produziria os valores `{ controller = Home, action = Index}`. Isto acontece porque os parâmetros de rota `{controller}` e `{action}` possuem valores padrões, e o parâmetro de rota `id` é opcional. Um sinal de igual `=` seguido por um valor, depois de um parâmetro de rota, define um valor padrão para um parãmetro. Uma marcação de interrogação `?` depois do parâmetro de rota define um parâmetro opcional. Parâmetros de rota com valores padrões *sempre* produzem um valor de rota quando há coincidência da rota - parâmetros opcionais não produzirão um valor de rota se não houver correspondência no segmento de caminho de URL. 

Veja [Referência de modelo de rota](#route-template-reference) para um descrião detalhada das funcionalidades do modelo de rota e de sua sintaxe.

Este exemplo incluí um *limitação de rota*:

```csharp
routes.MapRoute(
    name: "default",
    template: "{controller=Home}/{action=Index}/{id:int}");
```

Este modelo vai coincidir um caminho de URL como `/Produtos/Detalhes/17`, mas não `/Produtos/Detalhes/Apples`. A definição do parâmetro de rota `{id:int}` configura um *limite de rota* para o parâmetro de rota `id`. Limitações de rota implementam `IRouteConstraint` e inspecionam os valores da rota para verificá-los. Neste exemplo o valor de rota `id` precisa ser convertido para um inteiro. Veja [Referência de limitações de rota](#route-constraint-reference) para uma explicação mais detalhada sobre limitações de rota que são fornecidas pelo framework.

Sobrecargas adicionais do `MapRoute` aceitam valores para `constraints`, `dataTokens`, e `defaults`. Estes parâmetros adicionais do `MapRoute` são definidos como tipo `object`. O uso comum destes parâmetros é passar um objeto de tipo anônimo, no qual os nomes de propriedade do tipo anônimo coincidam com o nome dos parâmetros da rota.

Os dois exemplos seguintes criam rotas equivalentes:


```csharp
routes.MapRoute(
    name: "default_route",
    template: "{controller}/{action}/{id?}",
    defaults: new { controller = "Home", action = "Index" });

routes.MapRoute(
    name: "default_route",
    template: "{controller=Home}/{action=Index}/{id?}");
```

Dica: A sintáxe de linha para definição de limites e padrões pode ser mais conveniente para rotas simples. Contudo, existem recursos como os tokens de dados que não são suportados por essa sintaxe.

Este exemplo demonstra mais alguns recursos:

```csharp
routes.MapRoute(
  name: "blog",
  template: "Blog/{*article}",
  defaults: new { controller = "Blog", action = "ReadArticle" });
```

Este modelo vai coincidir um caminho de URL `/Blog/All-About-Routing/Introduction` e extraíra os valores `{controller = Blog, action = ReadArticle, article = All-About-Routing/Introduction}`. Os valores padrões de rota para `controller` e `action` são produzidos pela rota mesmo quando não exista parâmetros de rota correspondente no modelo. Valores padrões podem ser especificados em um modelo de rota. O parâmetro de rota `article` é definido como um *pegar-tudo* pela presença do asterisco antes do nome do parâmetro de rota. Parêtros do tipo *catch-all* (pega-tudo) capturam o restante do caminho URL e também podem coincidir uma string vazia.

Este exemplo adiciona uma limitante de rota e tokens de dados:

```csharp
routes.MapRoute(
    name: "us_english_products",
    template: "en-US/Products/{id}",
    defaults: new { controller = "Products", action = "Details" },
    constraints: new { id = new IntRouteConstraint() },
    dataTokens: new { locale = "en-US" });
```

Este modelo vai coincidir o caminho de URL `/Products/5` e vai extrair os valores `{controller = Products, action = Details, id = 5}`, além dos tokens de dados `{locale = en-US}`.

![Tokens locais do Windows](routing/_static/tokens.png)

<a name=id1></a>

### Geração de URL

A classe `Route` também pode realizar a geração de URL ao combinar um conjunto de valores de rota com seus modelos de rota. Isso é logicamente o processo inverso do coincidência de caminho de URL.

Dica: Para entender melhor a geração de URL, imagine a URL que você gostaria de gerar e então pense sobre como o modelo de rota coincidiria aquela URL. Quais valores deverão ser produzidos? Isto é a mesma forma de como a geração de URL funciona em uma classe `Route`.

Este exemplo usa um estilo básico de rotas ASP.NET MVC:

```csharp
routes.MapRoute(
    name: "default",
    template: "{controller=Home}/{action=Index}/{id?}");
```

Com os valores de rota `{ controller = Products, action = List }`, esta rota vai gerar a URL `/Products/List`. Os valores de rota são substituídos pelos correspondentes parâmetros de rota formando o caminho URL. Já que o `id` é um parâmetro de rota opcional, ele não há problema deixá-lo sem um valor. 

Com os valores de rota `{ controller = Home, action = Index }`, esta rota será gerada a URL `/`. Os valores de rota que foram fornecidos coincidem com os valor padrões, então os seguimentos correspondentes àqueles valores podem ser seguramente omitidos. Perceba que ambas URLs geradas vão e voltam nesta definição de rota e produzem os mesmos valores de rota que foram usados para gerar a URL.

Dica: Uma aplicação usando ASP.NET MVC precisa usar `UrlHelper` para gerar URLs em vez de chamar as rotas diretamente.

Para mais detalhes sobre o processo de geração de URL, veja [Referência à geração de URL](#url-generation-reference).

## Usando o Middleware de Roteamento

Adicione o pacote NuGet "Microsoft.AspNetCore.Routing".

Adicione o roteamento ao recipiente de serviço no *Startup.cs*:

[!code-csharp[Main](../fundamentals/routing/sample/RoutingSample/Startup.cs?highlight=3&start=11&end=14)]

As rotas precisam ser configuradas no método `Configure` da classe `Startup`. A demonstração abaixo usa estas APIs:

* `RouteBuilder`
* `Build`
* `MapGet`  Faz o processo de coincidência apenas para requisições HTTP GET
* `UseRouter`

<!-- literal_block {"xml:space": "preserve", "source": "fundamentals/routing/sample/RoutingSample/Startup.cs", "ids": [], "linenos": false, "highlight_args": {"linenostart": 1}} -->

```csharp
public void Configure(IApplicationBuilder app, ILoggerFactory loggerFactory)
{
    var trackPackageRouteHandler = new RouteHandler(context =>
    {
        var routeValues = context.GetRouteData().Values;
        return context.Response.WriteAsync(
            $"Hello! Route values: {string.Join(", ", routeValues)}");
    });

    var routeBuilder = new RouteBuilder(app, trackPackageRouteHandler);

    routeBuilder.MapRoute(
        "Track Package Route",
        "package/{operation:regex(^(track|create|detonate)$)}/{id:int}");

    routeBuilder.MapGet("hello/{name}", context =>
    {
        var name = context.GetRouteValue("name");
        // This is the route handler when HTTP GET "hello/<anything>"  matches
        // To match HTTP GET "hello/<anything>/<anything>,
        // use routeBuilder.MapGet("hello/{*name}"
        return context.Response.WriteAsync($"Hi, {name}!");
    });

    var routes = routeBuilder.Build();
    app.UseRouter(routes);
}
```

A tabela abaixo mostra as resposta com as URIs determinadas:

| URI | Resposta  |
| ------- | -------- |
| /package/create/3  | Hello! Route values: [operation, create], [id, 3] |
| /package/track/-3  | Hello! Route values: [operation, track], [id, -3] |
| /package/track/-3/ | Hello! Route values: [operation, track], [id, -3]  |
| /package/track/ | \<Fall through, no match> |
| GET /hello/Joe | Hi, Joe! |
| POST /hello/Joe | \<Fall through, matches HTTP GET only> |
| GET /hello/Joe/Smith | \<Fall through, no match> |

Se você estiver configurando apenas uma rota, chame o `app.UseRouter` passando uma instãncia de `IRoute`. Você não precisará chamar `RouteBuilder`.

O framework fornece um conjunto de métodos de extensão para criação de rotas, como:

* `MapRoute`
* `MapGet`
* `MapPost`
* `MapPut`
* `MapDelete`
* `MapVerb`

Alguns destes métodos como o `MapGet` requer um `RequestDelegate` para ser fornecido. O `RequestDelegate` será usado como o *controlador de rota* quando uma rota for encontrada. Outros método nesta familia permitem configurar um pipeline middleware, que será usado como o controlador de rota. Se o método *Map* não aceitar um controlador, como o `MapRoute`, então ele usará o `DefaultHandler`.

Os métodos `Map[Verb]` usam *constraints* para limitar a rota para um Verb HTTP no nome do método. Por exemplo, veja [MapGet](https://github.com/aspnet/Routing/blob/1.0.0/src/Microsoft.AspNetCore.Routing/RequestDelegateRouteBuilderExtensions.cs#L85-L88) e [MapVerb](https://github.com/aspnet/Routing/blob/1.0.0/src/Microsoft.AspNetCore.Routing/RequestDelegateRouteBuilderExtensions.cs#L156-L180).

## Referência de modelo de rotas

Tokens dentro de chaves (`{ }`) definem *parâmetros de rota* que serão limitados se a rota coincidir. Você pode definir mais do que um parâmetro de rota em um segmento de rota, mas eles precisam estar separados por um valor literal. Por exemplo `{controller=Home}{action=Index}` não seria uma rota válida, já que não existe valor literal entre `{controller}` e `{action}`. Estes parâmetros de rota precisam ter um nome, e podem ter atributos adicionais especificados.
 
 Texto literal diferente de parâmetros de rota (por exemplo, `{id}`) e os separadores de caminho `/` precisam coincidir com o texto da URL. A coincidência de texto é *case-**in**sensitive* e representação decodificada do caminho URL. Para coincidir o delimitador de parâmetro de rota literal `{` ou `}`, use o caracter de escape repetindo o delimitador (`{{` ou `}}`).

Matrizes de URL, que tentam capturar um nome de arquivo com extensão opcional de arquivo, possuem considerações adicionais. Por exemplo, usar o modelo `files/{filename}.{ext?}` - Quando ambos `filename` e `ext` existirem, os dois valores serão populados. Se apenas `filename` existir na URL, haverá coincidência de rota porque o ponto final `.` é opcional. As URLs seguintes coincidiriam com esta rota:

* `/files/myFile.txt`
* `/files/myFile.`
* `/files/myFile`

Você pode usar o caracter `*`  como prefixo para um parâmetro de rota para o vincular ao resto da URI - isso é chamado de parãmetro *catch-all*. Por exemplo, `blog/{*slug}` coincidiria quaisquer URIs que fossem iniciadas com `/blog` e que tivessem qualquer valor na sequência (o qual seria atribuído ao valor de rota `slug`). Parâmetros *cacth-all* (pega-tudo) podem ser coincidir com texto em branco.

Parâmetros de rota podem ter *valores padrões*, designados ao especificar o padrão após o nome do parâmetro, separado por um sinal de `=`. Por exemplo, `controller=Home` definiria `Home` como o valor padrão para o `controller`. O valor padrão é usado se nenhum valor estiver presente na URL para o parâmetro. Além disso, parâmetros de rota podem ser opcionais (incluíndo um ponto de interrogação `?` ao final do nome do parâmetro, assim `id?`). A diferença entre o opcional e o "padrão" é que um parâmetro de rota com um valor padrão sempre produz um valor; entquanto um parâmetro adiconal somente terá um valor se este for informado.

Parâmetros de rota podem ser limitantes, os quais precisam coincidir com o valor da rota para encontrar a URL. Adicionando dois pontos `:` e o nome do limitante após o nome do parâmetro de rota especificamos um *limitante de linha* em um parâmetro de rota. Se o limitante precisar daqueles argumentos de forma *required*, forneça-os cercados por parênteses depois do nome do limitante. Múltiplos limitantes de linha podem ser especificados ao incluir outro `:` e o nome do limitante. O nome do limitante é passado para o serviço `IInlineConstraintResolver` para criar uma instãncia de `IRouterContraint` para ser usado na URL em processamento. Por exemplo, o modelo de rota `blog/{article:minlength(10)}` especifica o limitante `minlength` com o argumento `10`. Para mais detalhes sobre limitantes de rota, e uma lista de provedores de limitantes do framework, veja [Referência de limitantes de rota](#route-constraint-reference).

A tabela seguinte demonstra alguns modelos de rota e seus comportamentos.

| Modelo de Rota | Exemplo de coincidência de URL | Notas |
| -------- | -------- | ------- |
| hello  | /hello  | Coincide apenas um único caminho `/hello` |
| {Page=Home} | / | Coincide e configura `Page` para `Home` |
| {Page=Home}  | /Contact  | | Coincide e configura `Page` para `Contact` |
| {controller}/{action}/{id?} | /Products/List | Maps to `Products` controller and `List`  action |
| {controller}/{action}/{id?} | /Products/Details/123  |  Maps to `Products` controller and  `Details` action.  `id` set to 123 |
| {controller=Home}/{action=Index}/{id?} | /  |  Maps to `Home` controller and `Index`  method; `id` is ignored. |

Usar um modelo é geralmente a maneira mais simples de criar rotas. Limitantes e padrões também podem ser especificadas fora do modelo de rota.

Dica: Habilite [Logging](logging.md) para ver como implementações de rotas do framework, como `Route`, coincidem requisições.


## Route Constraint Reference

Route constraints execute when a `Route` has matched the syntax of the incoming URL and tokenized the URL path into route values. Route constraints generally inspect the route value associated via the route template and make a simple yes/no decision about whether or not the value is acceptable. Some route constraints use data outside the route value to consider whether the request can be routed. For example, the `HttpMethodRouteConstraint` can accept or reject a request based on its HTTP verb.

>[!WARNING]
> Avoid using constraints for **input validation**, because doing so means that invalid input will result in a 404 (Not Found) instead of a 400 with an appropriate error message. Route constraints should be used to **disambiguate** between similar routes, not to validate the inputs for a particular route.

The following table demonstrates some route constraints and their expected behavior.

| constraint | Example | Example Matches | Notes |
| --------   | ------- | ------------- | ----- |
| `int` | `{id:int}` | `123456789`, `-123456789`  | Matches any integer |
| `bool`  | `{active:bool}` | `true`, `FALSE` | Matches `true` or `false` (case-insensitive) |
| `datetime` | `{dob:datetime}` | `2016-12-31`, `2016-12-31 7:32pm`  | Matches a valid `DateTime` value (in the invariant culture - see warning) |
| `decimal` | `{price:decimal}` | `49.99`, `-1,000.01` | Matches a valid `decimal` value (in the invariant culture - see warning) |
| `double`  | `{weight:double}` | `1.234`, `-1,001.01e8` | Matches a valid `double` value (in the invariant culture - see warning) |
| `float`  | `{weight:float}` | `1.234`, `-1,001.01e8` | Matches a valid `float` value (in the invariant culture - see warning) |
| `guid`  | `{id:guid}` | `CD2C1638-1638-72D5-1638-DEADBEEF1638`, `{CD2C1638-1638-72D5-1638-DEADBEEF1638}` | Matches a valid `Guid` value |
| `long` | `{ticks:long}` | `123456789`, `-123456789` | Matches a valid `long` value |
| `minlength(value)` | `{username:minlength(4)}` | `Rick` | String must be at least 4 characters |
| `maxlength(value)` | `{filename:maxlength(8)}` | `Richard` | String must be no more than 8 characters |
| `length(length)` | `{filename:length(12)}` | `somefile.txt` | String must be exactly 12 characters long |
| `length(min,max)` | `{filename:length(8,16)}` | `somefile.txt` | String must be at least 8 and no more than 16 characters long |
| `min(value)` | `{age:min(18)}` | `19` | Integer value must be at least 18 |
| `max(value)` | `{age:max(120)}` |  `91` | Integer value must be no more than 120 |
| `range(min,max)` | `{age:range(18,120)}` | `91` | Integer value must be at least 18 but no more than 120 |
| `alpha` | `{name:alpha}` | `Rick` | String must consist of one or more alphabetical characters (`a`-`z`, case-insensitive) |
| `regex(expression)` | `{ssn:regex(^\\d{{3}}-\\d{{2}}-\\d{{4}}$)}` | `123-45-6789` | String must match the regular expression (see tips about defining a regular expression) |
| `required`  | `{name:required}` | `Rick` |  Used to enforce that a non-parameter value is present during URL generation |

>[!WARNING]
> Route constraints that verify the URL can be converted to a CLR type (such as `int` or `DateTime`) always use the invariant culture - they assume the URL is non-localizable. The framework-provided route constraints do not modify the values stored in route values. All route values parsed from the URL will be stored as strings. For example, the [Float route constraint](https://github.com/aspnet/Routing/blob/1.0.0/src/Microsoft.AspNetCore.Routing/Constraints/FloatRouteConstraint.cs#L44-L60) will attempt to convert the route value to a float, but the converted value is used only to verify it can be converted to a float.

## Regular expressions 

The ASP.NET Core framework adds `RegexOptions.IgnoreCase | RegexOptions.Compiled | RegexOptions.CultureInvariant` to the regular expression constructor. See [RegexOptions Enumeration](https://docs.microsoft.com/dotnet/api/system.text.regularexpressions.regexoptions) for a description of these members.

Regular expressions use delimiters and tokens similar to those used by Routing and the C# language. Regular expression tokens must be escaped. For example, to use the regular expression `^\d{3}-\d{2}-\d{4}$` in Routing, it needs to have the `\` characters typed in as `\\` in the C# source file to escape the `\` string escape character (unless using [verbatim string literals](https://docs.microsoft.com/dotnet/csharp/language-reference/keywords/string). The `{` , `}` , '[' and ']' characters need to be escaped by doubling them to escape the Routing parameter delimiter characters.  The table below shows a regular expression and the escaped version.

| Expression               | Note |
| ----------------- | ------------ | 
| `^\d{3}-\d{2}-\d{4}$` | Regular expression |
| `^\\d{{3}}-\\d{{2}}-\\d{{4}}$` | Escaped  |
| `^[a-z]{2}$` | Regular expression |
| `^[[a-z]]{{2}}$` | Escaped  |

Regular expressions used in routing will often start with the `^` character (match starting position of the string) and end with the `$` character (match ending position of the string). The `^` and `$` characters ensure that the regular expression match the entire route parameter value. Without the `^` and `$` characters the regular expression will match any sub-string within the string, which is often not what you want. The table below shows some examples and explains why they match or fail to match.

| Expression               | String | Match | Comment |
| ----------------- | ------------ |  ------------ |  ------------ | 
| `[a-z]{2}` | hello | yes | substring matches |
| `[a-z]{2}` | 123abc456 | yes | substring matches |
| `[a-z]{2}` | mz | yes | matches expression |
| `[a-z]{2}` | MZ | yes | not case sensitive |
| `^[a-z]{2}$` |  hello | no | see `^` and `$` above |
| `^[a-z]{2}$` |  123abc456 | no | see `^` and `$` above |

Refer to [.NET Framework Regular Expressions](https://docs.microsoft.com/dotnet/standard/base-types/regular-expression-language-quick-reference) for more information on regular expression syntax.

To constrain a parameter to a known set of possible values, use a regular expression. For example `{action:regex(^(list|get|create)$)}` only matches the `action` route value to `list`, `get`, or `create`. If passed into the constraints dictionary, the string "^(list|get|create)$" would be equivalent. Constraints that are passed in the constraints dictionary (not inline within a template) that don't match one of the known constraints are also treated as regular expressions.

## URL Generation Reference

The example below shows how to generate a link to a route given a dictionary of route values and a `RouteCollection`.

[!code-csharp[Main](../fundamentals/routing/sample/RoutingSample/Startup.cs?range=45-59)]

The `VirtualPath` generated at the end of the sample above is `/package/create/123`.

The second parameter to the `VirtualPathContext` constructor is a collection of *ambient values*. Ambient values provide convenience by limiting the number of values a developer must specify within a certain request context. The current route values of the current request are considered ambient values for link generation. For example, in an ASP.NET MVC app if you are in the `About` action of the `HomeController`, you don't need to specify the controller route value to link to the `Index` action (the ambient value of `Home` will be used).

Ambient values that don't match a parameter are ignored, and ambient values are also ignored when an explicitly-provided value overrides it, going from left to right in the URL.

Values that are explicitly provided but which don't match anything are added to the query string. The following table shows the result when using the route template `{controller}/{action}/{id?}`.

| Ambient Values | Explicit Values | Result |
| -------------   | -------------- | ------ |
| controller="Home" | action="About" | `/Home/About` |
| controller="Home" | controller="Order",action="About" | `/Order/About` |
| controller="Home",color="Red" | action="About" | `/Home/About` |
| controller="Home" | action="About",color="Red" | `/Home/About?color=Red`

If a route has a default value that doesn't correspond to a parameter and that value is explicitly provided, it must match the default value. For example:

```csharp
routes.MapRoute("blog_route", "blog/{*slug}",
  defaults: new { controller = "Blog", action = "ReadPost" });
```

Link generation would only generate a link for this route when the matching values for controller and action are provided.
