# Notas da aula de ElasticSearch

## Subindo o ambiente
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

* POSTman (Chrome Plugin)
* REST Client (Atom)



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

*Response

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

### Retornando todos os registros do indice com filtro

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
