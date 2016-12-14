# Notas sobre Analyzers

### Por que Analyzers
Analyzers são filtros de busca utilizados para full text search, onde não temos um tipo avaliação binária, como 'esse termo é igual ao buscado' ou 'esse termino não é igual ao buscado'. No caso, imagine que tenhamos que buscar o termo 'musica', porém temos registros que possuem acentos como 'música' ou outros que tenham letras maiúsculas como 'Musica'. Esses termos não vão ser encontrados porque binariamente não são iguais ao buscado. O ElasticSearch serve para provisionar buscas flexíveis, portão o uso dos Analyzers são muito recorrentes, pois trabalham de uma forma diferente, gerando pontuação de comparação entre os termos buscados, por exemplo, música é 0.96 igual a musica.

## Tipos de Analyzers mais conhecidos

### Standard

O Standart é o analyzer default do elasticsearch que funciona bem de uma maneira geral com qualquer linguagem. Ele quebra palavra por palavra, removendo todos os caracteres especiais e pontos, converte tudo para caixa baixa para ser analisado com o termo buscado. Em seguida compara todos os tokens gerados com o termo buscado e gerar o score de comparação.



### Simple

Esse analyzer quebra o texto da mesma forma que o Standard, porém retira todos os termnos que não são letras. Se fornecer uma frase como "Meu nome é Matheus e eu tenho 21 anos", os tokens gerados serão parecido com "meu", "nome","é" ,"matheus","e","eu","tenho","anos".



### whitespace

Esse analyzer é o mais simples, ele mantém os números, mantem a caixa padrão e caracteres especiais.
No caso "Meu nome é Matheus e eu tenho 21 anos" ficaria "Meu", "nome", "é", "Matheus", "e", "eu", "tenho", "21", "anos".


# Criando um Indice com Analyzers

Para criar um indice e já atribuir os Analyzers utilizados em cada campo, temos que fazer um request PUT para o indice

```
PUT /pessoas

{
  "settings": {
    "index": {
      "number_of_shards": 3,
      "number_of_replicas": 0
    },
    "analysis": {
      "filter": {
         "portuguese_stop": {
          "type":       "stop",
          "stopwords":  "_portuguese_"
        },
        "portuguese_stemmer": {
          "type": "stemmer",
          "language": "light_portuguese"
        }
      },
      "analyzer": {
        "sinonimos": {
          "tokenizer": "standard",
          "filter": [
            "lowercase",
            "portuguese_stop",
            "portuguese_stemmer"
          ]
        }
      }
    }
  },
  "mappings": {
    "registros": {
      "_all": {
        "type": "string",
        "index": "analyzed",
        "analyzer": "portuguese"
      },
      "properties": {
        "cidade": {
          "type": "string",
          "fields": {
            "original": {
              "type": "string",
              "index": "not_analyzed"
            }
          },
          "index": "analyzed",
          "analyzer": "portuguese"
        },
        "estado": {
          "type": "string",
          "index": "not_analyzed"
        },
        "formação": {
          "type": "string",
          "fields": {
            "original": {
              "type": "string",
              "index": "not_analyzed"
            }
          },
          "index": "analyzed",
          "analyzer": "portuguese"
        },
        "interesses": {
          "type": "string",
          "index": "analyzed",
          "analyzer": "portuguese",
          "search_analyzer": "sinonimos"
        },
        "nome": {
          "type": "string",
          "fields": {
            "original": {
              "type": "string",
              "index": "not_analyzed"
            }
          },
          "index": "analyzed",
          "analyzer": "portuguese"
        },
        "país": {
          "type": "string",
          "fields": {
            "original": {
              "type": "string",
              "index": "not_analyzed"
            }
          },
          "index": "analyzed",
          "analyzer": "portuguese"
        }
      }
    }
  }
}
```

* RESPONSE

```
{
    "acknowledged": true
}
```
