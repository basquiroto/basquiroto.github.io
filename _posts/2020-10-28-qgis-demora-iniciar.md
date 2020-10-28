---
layout: post
title: Como verificar qual plugin esta deixando o QGIS lento?
---

Ultimamente, meu QGIS tem demorado muito para iniciar. As vezes ele fica um tempão na tela inicial (splashscreen). 
Busquei algumas soluções e encontrei esse pedaço de código que permite que você verifique qual plugin demora mais
para carregar. É um cógido simples, que é rodado dentro do terminal Python do QGIS (Ctrl+Alt+P ou Complementos > Python).

``python
import pprint
pprint.pprint(qgis.utils.plugin_times)
``

O resultado será algo similar ao seguinte:

`
{'GarminCustomMap': '2.375000s',
 'db_manager': '0.031250s',
 'mapbiomascollection': '0.000000s',
 'processing': '4.359375s',
 'quick_map_services': '3.828125s',
 'splitmultipart': '0.031250s'}
 `
 
 Veja que a partir desses dados, posso desligar o plugin GarminCustomMap para tentar acelerar as coisas. Você pode desligar os plugins
 em Complementos > Gerenciar e Instalar Complementos.
 
 Você pode estar se perguntando, por que não desligar o plugin quick_map_services e o processing? Esses dois plugins eu utilizo com 
 frequência, então não compensa para mim ter que ficar ligando eles cada vez que eu utilizá-los.
 
 Referência consultada: https://gis.stackexchange.com/questions/209129/how-to-tell-which-qgis-plugins-are-slow-to-load
