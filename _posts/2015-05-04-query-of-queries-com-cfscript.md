---
layout: post
title: "Query of Queries com CFScript"
description: "Utilizando Query of Queries com CFScript"
category: ColdFusion/CFML
tags: [ColdFusion,CFML]
comments: true
---
Boas pessoal!
<br>
O Coldfusion tem um recurso muito útil conhecido como “Query of Queries” ( QoQ ) que permite você fazer uma query em um resultado de outra query sem a necessidade de acessar o banco de dados ( ou outra fonte ) novamente.
<br>
Os 3 principais servidores Coldfusion suportam QoQ e para maiores detalhes veja a documentação do [ACF - Adobe ColdFusion](http://help.adobe.com/en_US/ColdFusion/10.0/Developing/WSc3ff6d0ea77859461172e0811cbec0e4fd-7ff9.html), do [Railo](https://github.com/getrailo/railo/wiki/Query-of-Queries) ou do [Lucee](http://www.luceedocs.org/tag/cfquery).

A ideia neste post é registrar um exemplo de como utilizar QoQ totalmente em CFScript, já que é amplamente recomendado usar CFScript em componentes CFC no lugar de tags.

Dada a classe _MyClass.cfc_ abaixo, temos uma lista de linguagens de programação (que poderia ter vindo de qualquer fonte de dados, como um arquivo Excel por exemplo) e queremos que nosso método _getLanguages()_ retorne esta lista em ordem alfabética.

{% highlight coldfusion%}
component output="false" displayname="MyClass"  {

  public function init(){
    return this;
  }

  public function getLanguages() {
    local.result = "";
    local.qLang = queryNew("id,name", "Integer,Varchar",
                        [
                          {id=1,name="CFML"},
                          {id=2,name="Ruby"},
                          {id=3,name="Python"},
                          {id=4,name="Perl"},
                          {id=5,name="Java"}
                        ]
      );

    local.qoq = new Query();

    local.qoq.setAttributes(  originalResult = local.qLang,
                              DBType="query",
                              Name="exampleQuery"
                            );

    local.result = local.qoq.execute( sql = "SELECT * FROM originalResult ORDER BY name").getResult();

    return local.result;
  }
}
{% endhighlight %}

Vamos destrinchá-lo!
<br>

<pre><code>
    local.qoq = new Query();
</code></pre>
Criamos um novo objeto Query chamado _qoq_, que futuramente executará a consulta na query de origem que chamamos de _qLang_.

<pre>
<code>
    local.qoq.setAttributes(  originalResult = local.qLang,
                              DBType="query",
                              Name="exampleQuery"
                            );
</code>
</pre>

Em seguida, atribuímos algumas propriedades para esta nova query.

O grande facilitador aqui é a propriedade _originalResult_, que poderia se chamar qualquer outra coisa. Nela atribuímos o resultSet da query original ( _qLang_ ) que será utilizada no **FROM** da nossa query final.

A propriedade **DBType** indica que esta query irá consultar uma outra query ( e não o banco de dados ).

A propriedade **Name** já é mais conhecida. É o nome da query e ajuda muito quando estamos debugando.

<pre>
<code>
    local.result = local.qoq.execute( sql = "SELECT * FROM originalResult ORDER BY name").getResult();
</code>
</pre>

Acima executamos a query com o **sql** que precisamos e atribuímos o seu resultado para a variável _result_.

Repare que no **FROM** desta query utilizamos _originalResult_, que nada mais é que a propriedade que criamos com o **setAttributes** e populamos com o resultSet da query original ( _qLang_ ).

Por fim, basta retornar o valor da variável _result_.

Neste exemplo, utilizamos um simples "order by, mas, o céu é o limite!

O código fonte completo deste exemplo está no meu [Github](https://github.com/edmilton/cfmlExamples/tree/master/queryOfQueries).

E você, como utiliza os poderes da Query of Query? Gostaria de acrescentar, corrigir ou sugerir algo? Comente a vontade!

**PS:** Este código foi testado no ACF 10.

Gde. abraço!

Ed

#### Todos os links deste artigo

* [ACF - Adobe ColdFusion](http://help.adobe.com/en_US/ColdFusion/10.0/Developing/WSc3ff6d0ea77859461172e0811cbec0e4fd-7ff9.html)
* [Railo](https://github.com/getrailo/railo/wiki/Query-of-Queries)
* [Lucee](http://www.luceedocs.org/tag/cfquery)
* [Github](https://github.com/edmilton/cfmlExamples/tree/master/queryOfQueries)

{% include disqus.html %}
{% include analytics.html %}