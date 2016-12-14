
# Notas da aula de ElasticSearch

![ElasticSearch](https://avatars0.githubusercontent.com/u/6764390?v=3&s=400)

## Subindo o ambiente

![Docker](https://d3nmt5vlzunoa1.cloudfront.net/phpstorm/files/2015/10/large_v-trans.png)

* Tenha o Docker instalado
* Tenha o Docker Compose instalado
* Tenha um espaço em ram considerável, o ElasticSearch é tipo um Google Chrome Sênior no que diz respeito ao consumo de memória.
* Rode o comando na raiz do projeto:

```
  docker-compose up
```

* Todo o conteúdo do ElasticSearch será persistido na pasta ./data

## RESTful client

Para utilizar o treinamento é necessário utilizar um client HTTP. O browser por si só é um client legal, porém ele é limitado nativamente por requisições POST e GET, e utilizaremos vários outros métodos REST. Para isso indico os seguintes clients

* [Postman (Chrome Plugin)](https://chrome.google.com/webstore/detail/postman/fhbjgbiflinjbdggehcddcbncdddomop)
* [REST Client (Atom)](https://atom.io/packages/rest-client)



## CRUD com ElasticSearch

O ElasticSearch é gerenciado a partir de uma API REST, e para fazer um Crud Simples utilizando o banco e um client REST é simples.

### Criando um documento - POST

Para criar um documento do zero, e deixar que o ElasticSearch gerencie o indice e o numero identificador, utilizaremos o método POST como convencional. Faremos um request JSON para a URL do nosso endpoint

* REQUEST:

```
POST /produtos/geladeiras

{
    "produto" : "Geladeira Consul Frost Free 500xx",
    "cores" : ["branca", "cinza", "preta"],
    "marca" : "Consul",
    "valor" : 930.00,
    "estado" : "nova"
}
```

* RESPONSE:

```
{
    "_index": "produtos",
    "_type": "geladeiras",
    "_id": "AVj7M-ZgBQRwiJ3RYe1N",
    "_version": 1,
    "_shards": {
        "total": 2,
        "successful": 1,
        "failed": 0
    },
    "created": true
}

```

### Substituindo um documento com PUT
Podemos utilizar o método PUT para sobrescrever um documento com novas propriedades, para isso somos obrigados a fornecer o /indice/tipo/identificador na URL e um JSON com os novos valores na BODY do request.

* REQUEST

```

```

### Ajustando o numero de replicas do index para 0

* REQUEST
```
PUT /produtos/_settings
{
    "index" : {
        "number_of_replicas" : 0
    }
}
```

* RESPONSE
```
{
    "acknowledged": true
}
```

### Atualizando somente alguns campos do documento

Podemos atualizar somente alguns campos do documento utilizando o método POST.
Precisamos fornecer no request um parâmetro "doc" obrigatoriamente

* REQUEST

```
POST /produtos/geladeiras/AVj7M-ZgBQRwiJ3RYe1N/_update

{
  "doc": {
    "valor": 666.66
  }
}
```

* RESPONSE

```
{
  "_index": "produtos",
  "_type": "geladeiras",
  "_id": "AVj7M-ZgBQRwiJ3RYe1N",
  "_version": 2,
  "_shards": {
      "total": 1,
      "successful": 1,
      "failed": 0
  }
}
```

### Pegando a quantidade total de itens do indice por tipo

Podemos retornar um valor total do numero de itens pertencentes ao nosso indice utilizando o parâmetro _count via GET


* REQUEST
```
GET /produtos/geladeiras/_count
{}
```

* RESPONSE
```
{
    "count": 2,
    "_shards": {
        "total": 5,
        "successful": 5,
        "failed": 0
    }
}
```

### Pegando um documento pelo numero identificador

Podemos fornecer um ID especifico para retornar seus registros.

* Request
```
GET /produtos/geladeiras/AVj7M-ZgBQRwiJ3RYe1N
{}
```

* RESPONSEs

```
{
    "_index": "produtos",
    "_type": "geladeiras",
    "_id": "AVj7M-ZgBQRwiJ3RYe1N",
    "_version": 1,
    "found": true,
    "_source": {
        "produto": "Geladeira Consul Frost Free 500xx",
        "cores": [
            "branca",
            "cinza",
            "preta"
        ],
        "marca": "Consul",
        "valor": 930,
        "estado": "nova"
    }
}
```


### Retornando todos os registros do indice sem filtro com o parametro _search

* REQUEST
```
GET /produtos/geladeiras/_search
{}
```

* RESPONSE
```
{
    "took": 42,
    "timed_out": false,
    "_shards": {
        "total": 5,
        "successful": 5,
        "failed": 0
    },
    "hits": {
        "total": 2,
        "max_score": 1,
        "hits": [
            {
                "_index": "produtos",
                "_type": "geladeiras",
                "_id": "AVj7My6qBQRwiJ3RYe1M",
                "_score": 1,
                "_source": {
                    "user": "kimchy",
                    "post_date": "2009-11-15T14:12:12",
                    "message": "trying out Elasticsearch"
                }
            },
            {
                "_index": "produtos",
                "_type": "geladeiras",
                "_id": "AVj7M-ZgBQRwiJ3RYe1N",
                "_score": 1,
                "_source": {
                    "produto": "Geladeira Consul Frost Free 500xx",
                    "cores": [
                        "branca",
                        "cinza",
                        "preta"
                    ],
                    "marca": "Consul",
                    "valor": 930,
                    "estado": "nova"
                }
            }
        ]
    }
}
```

### Parâmetros de limite e paginação no _search

Para definir o numero máximo de registros retornados da busca, utilizamos o parâmetro 'size'. Esse parametro é importante, porque por default, o Elasticsearch retorna somente 10 documentos na consulta.

```
GET /produtos/geladeiras/_search?size=50
{}
```

Para inserir uma paginação, ou seja, a partir de um numero de itens, para criar uma paginação e etc, utilizamos o parâmetro from

```
GET /produtos/geladeiras/_search?size=50&from=10
{}
```

### Retornando todos os registros do indice com filtro

Quando não oferecemos parametros de campo para o _search,ele automaticamente concatena todos os itens dos indices em um campo chamado _all, logo quando montamos uma busca com
_search?q=consul é o mesmo que fazer _search?q=_all:consul

* REQUEST

```
GET /produtos/geladeiras/_search?q=ponto
{}
```

* RESPONSE

```
{
    "took": 94,
    "timed_out": false,
    "_shards": {
        "total": 5,
        "successful": 5,
        "failed": 0
    },
    "hits": {
        "total": 1,
        "max_score": 0.11506981,
        "hits": [
            {
                "_index": "produtos",
                "_type": "geladeiras",
                "_id": "AVj7OgJVBQRwiJ3RYe1Y",
                "_score": 0.11506981,
                "_source": {
                    "produto": "Geladeira Ponto Frio",
                    "cores": [
                        "branca"
                    ],
                    "marca": "Consul",
                    "valor": 1000,
                    "estado": "nova"
                }
            }
        ]
    }
}
```

### Adicionando parametros na busca

Para fornecer um campo específico para a busca, devemos utilizar o parametro _search também, porém _search?q=marca:consul.
Para adicionar mais parâmetros, seguimos o padrão de query http _search?q=marca:consul&cores:branca

* REQUEST

```
GET /produtos/geladeiras/_search?q=marca:consul
```

* RESPONSE

```
{
    "took": 29,
    "timed_out": false,
    "_shards": {
        "total": 5,
        "successful": 5,
        "failed": 0
    },
    "hits": {
        "total": 2,
        "max_score": 0.30685282,
        "hits": [
            {
                "_index": "produtos",
                "_type": "geladeiras",
                "_id": "AVj7M-ZgBQRwiJ3RYe1N",
                "_score": 0.30685282,
                "_source": {
                    "produto": "Geladeira Consul Frost Free 500xx",
                    "cores": [
                        "branca",
                        "cinza",
                        "preta"
                    ],
                    "marca": "Consul",
                    "valor": 666.66,
                    "estado": "nova"
                }
            },
            {
                "_index": "produtos",
                "_type": "geladeiras",
                "_id": "AVj7OgJVBQRwiJ3RYe1Y",
                "_score": 0.30685282,
                "_source": {
                    "produto": "Geladeira Ponto Frio",
                    "cores": [
                        "branca"
                    ],
                    "marca": "Consul",
                    "valor": 1000,
                    "estado": "nova"
                }
            }
        ]
    }
}
```
