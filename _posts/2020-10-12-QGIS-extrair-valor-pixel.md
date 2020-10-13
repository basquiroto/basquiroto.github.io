---
layout: post
title: Como extrair o valor de um pixel de um raster no QGIS?
---

Em algumas situações, temos um conjunto de pontos e gostaríamos de saber quais são os valores de elevação naquelas coordenadas específicas. 
Esse procedimento é possível usando o QGIS, basta ter em mãos o MDE de interesse e o shapefile com os pontos de amostragem.

Neste tutorial, iremos usar os modelos digitais de elevação (MDE) SRTM, que estão disponíveis no site da [EMBRAPA – Brasil em Relevo](https://www.cnpm.embrapa.br/projetos/relevobr/download/index.htm). 
Tais MDE apresentam resolução espacial de 90 metros (3 segundos de arco) e estão no datum WGS84. 

Utilizaremos o MDE da carta [SH-22-X-B](https://www.cnpm.embrapa.br/projetos/relevobr/download/sc/sh-22-x-b.htm) 
(a qual representa parcialmente o sul de Santa Catarina) e o conjunto de coordenadas geográficas apresentadas na Tabela 1. 
Após adicionar tanto o MDE, quando os pontos, você terá algo similar à Figura 1.

_Tabela 1_:

|   ID   |   X   |   Y   |
|   --   |   -   |   -   |
|   1   |   -49,4354   |   -28,0554   |
|   2   |   -49,1240   |   -28,0554   |
|   3   |   -48,7802   |   -28,0486   |
|   4   |   -49,4150   |   -28,4485   |
|   5   |   -49,1376   |   -28,4519   |
|   6   |   -49,4218   |   -28,8961   |

Caso você não saiba como adicionar essas coordenadas no QGIS, [confira este vídeo](https://www.youtube.com/watch?v=FMJ6KO3Ytco).

_Figura 1_:

![Imagem mostrando QGIS com arquivos adicionados]({{ site.baseurl }}/images/ext_pixel_value_comeco.png "Tela principal do QGIS contendo pontos e MDE")

Com isso, vá no menu Processar e clique em Caixa de Ferramenta (ou simplemente aperte Ctrl + Alt + T). 
Busque pela ferramenta ‘Amostrar valores do raster’ (‘Sample raster values’) e em seguida, dê dois clique para abrir a ferramenta.

Na janela aberta (Figura 2), você deverá informar o shapefile de pontos, que serão usados para extrair os valores; a camada raster que você quer ter os valores nos pontos; 
o nome da coluna onde serão salvos os valores; e onde você quer salvar o shapefile criado (que é uma cópia do shapefile de entrada, porém com os valores do raster 
salvo na tabela de atributos).

_Figura 2_:

![Janela da Ferramenta Amostrar valores do Raster]({{ site.baseurl }}/images/ext_pixel_value_tool.png "Janela da Ferramenta Amostrar valores do Raster")

Após preencher os valores, clique em Executar.
Um novo shapefile será adicionado ao QGIS e você terá uma coluna na tabela de atributos mostrando qual é o valor do pixel logo abaixo do ponto, conforme Figura 3.

_Figura 3_:

![Tabela de atributos contendo os valores do MDE]({{ site.baseurl }}/images/ext_pixel_value_saida.png "Tabela de atributos contendo os valores do MDE")

Caso você utilize o ArcGIS, pode conferir este procedimento [neste tutorial](http://2engenheiros.com/2016/10/23/extraindo-dados-com-pontos-no-arcgis/).
