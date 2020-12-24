---
layout: post
title: Regressão Linear, Coeficiente de Pearson e Downloads de Imagens no GEE
---

Alguns pedaços de código em Python para realizar algumas tarefas no Google Earth Engine (GEE) em Python.
A parte inicial do código é para carregar as imagens para o exemplo.

O arquivo com o código completo encontra-se [aqui](https://github.com/basquiroto/EEwP/blob/master/GEE_Regression_Pearson_Download.py).

```python
import ee
import time
import requests
import shutil
ee.Initialize()

começo = time.time()

lim_Rect = ee.Geometry.Rectangle([-59,-6, -49, -16])
img_01 = ee.Image('MODIS/006/MOD11A1/2018_07_08')\
    .select('LST_Day_1km').clip(lim_Rect).multiply(ee.Image(0.02))
img_02 = ee.Image('MODIS/006/MOD11A1/2018_07_09')\
    .select('LST_Day_1km').clip(lim_Rect).multiply(ee.Image(0.02))

```

## Como fazer uma regressão linear no GEE?

```python
constant = ee.Image(1)
xVar = img_01
yVar = img_02
imgRegress = ee.Image.cat(constant, xVar, yVar);
regress = imgRegress.reduceRegion(
        reducer = ee.Reducer.linearRegression(
            numX = 2,
            numY = 1),
        geometry= lim_Rect, 
        scale= 1000)

intercept = list(regress.getInfo().values())[0][0][0]
slope = list(regress.getInfo().values())[0][1][0]

print('y = ', round(intercept, 4), ' + ', round(slope, 4), 'x')
```

## Como calcular o coeficiente de Pearson no GEE?

```python
Two_Bands = ee.Image.cat([img_01, img_02])
R2_dict = Two_Bands.reduceRegion(
    reducer = ee.Reducer.pearsonsCorrelation(),
    geometry = lim_Rect, 
    scale = 1000)
R2 = list(R2_dict.getInfo().values())[0]

print('R**2: ', round(R2, 4))
```

## Download automático de imagens (Thumbnails)

```python
min_max_01 = img_01.reduceRegion(**{'reducer': ee.Reducer.minMax(),
                                    'geometry': lim_Rect,
                                    'scale': 1000})
max_value01 = list(min_max_01.getInfo().values())[0]
min_value01 = list(min_max_01.getInfo().values())[1]

img_01_URL = img_01.getThumbURL({'min': min_value01, 'max': max_value01})
filePath = 'C:/Users/ferna/Desktop/MOD11A1_2018_07_08.png'

r = requests.get(img_01_URL, stream = True)

if r.status_code == 200:
    r.raw.decode_content = True
    
    with open(filePath,'wb') as f:
        shutil.copyfileobj(r.raw, f)
        
    print('Imagem baixada com sucesso: ', filePath.split('/')[-1])
else:
    print('Falha no download da imagem.')
```

Mais detalhes sobre download automático de imagens, [aqui](https://towardsdatascience.com/how-to-download-an-image-using-python-38a75cfa21c).
