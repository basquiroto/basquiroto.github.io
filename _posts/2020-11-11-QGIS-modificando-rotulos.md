---
layout: post
title: Como modificar rótulos / labels de shapefiles no QGIS?
---

Você esta representando o estado que esta inserido a sua área de estudo, então você quer dar destaque à ele e só rotular ele. 
Outra situação é que você quer apresentar somente o nome do rio que esta próximo do seu empreendimento, como você pode fazer isso
no QGIS? Vamos realizar o primeiro exemplo com o shapefile dos [limites dos estados brasileiros](https://geoftp.ibge.gov.br/organizacao_do_territorio/malhas_territoriais/malhas_municipais/municipio_2015/Brasil/BR/).

Para mostrar todos os nomes dos estados, após abrir o shapefile no QGIS, você irá clicar sobre ele, acessar suas propriedades e ir na aba Rótulos (Labels).
Troque o 'Não Rotular' por rótulo símples e selecione em Valor a coluna com o nome dos estados (i.e. NM_ESTADO).

Para alterarmos os rótulos de forma automática, vamos utilizar Expressões. Você pode acessá-las clicando no botão com um E, no canto direito, ao lado do menu do Valor.
Abaixo apresento alguns códigos que uso com frequência aplicado aos estados brasileiros.

![Estados Brasileiros com rótulos na região sul]({{ site.baseurl }}/images/rotulos_sul.jpg "Estados Brasileiros com rótulos na região sul")

#### Colocar primeira letra maiúscula e resto em minúscula

```
title(NM_ESTADO)
```

#### Mostrar apenas o nome de um estado

```
IF (NM_ESTADO = 'SANTA CATARINA', NM_ESTADO, '')
```

#### Mostrar o nome de um número x de estado

```
CASE
WHEN  "NM_ESTADO" IS 'SANTA CATARINA' THEN "NM_ESTADO"
WHEN  "NM_ESTADO" IS 'RIO GRANDE DO SUL' THEN "NM_ESTADO"
WHEN  "NM_ESTADO" IS 'PARANÁ' THEN "NM_ESTADO"
ELSE ''
END
```

#### Juntar área com unidade de medida

```
concat(AREA, ' km2')
```

#### Mostar apenas a cota das curvas principais (5 em 5 metros)

```
CASE
WHEN  ("ELEVATION" / 5) - to_int("ELEVATION" / 5) IS 0 THEN  "ELEVATION" 
END
```

Esses são alguns exemplos que você pode começar a adotar nos seus mapas. Lembrando que as expressões são usadas em vários locais no QGIS.
Além disso, você pode utilizar ao invés do Rótulo Simples, o Rótulo Baseado em Regra, que usa justamente esses tipos de expressões
apresentadas aqui.

Caso você esteja usando o ArcGIS, você pode conferir essa postagem sobre o mesmo tema [Como inserir e modificar rótulos (labels) de shapefiles no ArcGIS?](http://2engenheiros.com/2016/12/18/rotulando-feicoes-no-arcgis/).
