---
layout: post
title: Qual é o impacto do tamanho da área amostrada na média?
---

Métricas como 'diferença das médias' (Mean Difference) são usados no sensoriamento remoto para comparar simulações e modelos. Mas qual
será o impacto do tamanho da área de estudo na média?

O código abaixo é um teste rápido para calcular a média do retângulo que vai se espandindo em cada iteração. Os dados usados são
de temperatura de superfície do MODIS e os cálculos são realizados com o Earth Engine em Python.


O código mostrado aqui também esta disponível [aqui](https://github.com/basquiroto/EEwP/blob/master/area_vs_mean.py).

```python
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
import ee
ee.Initialize()

ponto = [-50.8472518, -26.9704873]
ee_ponto = ee.Geometry.Point(ponto)

media = []
buffer_list = np.linspace(0.01, 5.00, 30)

for i in range(len(buffer_list)):
    Rect = [(ponto[0]+buffer_list[i]),(ponto[1]-buffer_list[i]),
            (ponto[0]-buffer_list[i]),(ponto[1]+buffer_list[i])]
    lim_Rect = ee.Geometry.Rectangle(Rect)
    
    # 2014_10_27 e 2014_09_25
    img = ee.Image('MODIS/006/MOD11A1/2010_01_01').select('LST_Day_1km')\
        .clip(lim_Rect).multiply(ee.Image(0.02))
    
    media_temp_raw = img.reduceRegion(**{'reducer': ee.Reducer.mean(),
                                     'geometry': lim_Rect,
                                     'scale': 1000})
    media_temp = list(media_temp_raw.getInfo().values())[0]
    media.append((round(buffer_list[i], 2), media_temp))
    
mean_df = pd.DataFrame(media, columns = ['Buffer', 'Média'])

grafico = mean_df.plot(x = 'Buffer', y = 'Média')
plt.show()
```

Ao final do código, é plotado o gráfico com a variação da média ao longo do tamanho do buffer dado a partir do ponto estabelecido.
A imagem abaixo mostra o gráfico gerado.

_Figura_ - Variação da média em função da área:

![Imagem com nuvem]({{ site.baseurl }}/images/impact_mean_area.png "Variação média com a área")
