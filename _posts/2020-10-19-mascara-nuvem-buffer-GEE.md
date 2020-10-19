---
layout: post
title: Como aplicar uma máscara de nuvens e sombra em LANDSAT 8 no Earth Engine com Python?
---

A aplicação de uma máscara de nuvens e sombra para as imagens LANDSAT 8 pode ser realizado usando a banda 'pixel_qa' e bitmasks. 
Além disso, iremos usar o código disponibilizado por Cesar Aybar ([GitHub](https://github.com/csaybar)) para visualizar os resultados, 
sendo a função criada por ele apresentada abaixo.

```python
def Mapdisplay(center, dicc, Tiles="OpensTreetMap",zoom_start=10):
    '''
    :param center: Center of the map (Latitude and Longitude).
    :param dicc: Earth Engine Geometries or Tiles dictionary
    :param Tiles: Mapbox Bright,Mapbox Control Room,Stamen Terrain,Stamen Toner,stamenwatercolor,cartodbpositron.
    :zoom_start: Initial zoom level for the map.
    :return: A folium.Map object.
    '''
    mapViz = folium.Map(location=center,tiles=Tiles, zoom_start=zoom_start)
    for k,v in dicc.items():
      if ee.image.Image in [type(x) for x in v.values()]:
        folium.TileLayer(
            tiles = v["tile_fetcher"].url_format,
            attr  = 'Google Earth Engine',
            overlay =True,
            name  = k
          ).add_to(mapViz)
      else:
        folium.GeoJson(
        data = v,
        name = k
          ).add_to(mapViz)
    mapViz.add_child(folium.LayerControl())
    return mapViz
```

Agora, vamos criar algumas funções para remover as nuvens e sombras em apenas uma imagem ou numa coleção de imagens (neste ultimo caso, usando a função .map()).

```python
def maskL8SR(imagem):
    '''Filtro para remover nuvens e sombras
    das imagens LANDSAT 8 - Para usar no .map()'''
    
    cloudShadowBitmask = int(2**3)
    cloudBitmask = int(2**5)
    qa = imagem.select('pixel_qa')
    mask = qa.bitwiseAnd(cloudShadowBitmask).eq(0).And(\
           qa.bitwiseAnd(cloudBitmask).eq(0))
        
    return imagem.updateMask(mask)

def maskL8SR_100(imagem):
    '''Filtro para remover nuvens e sombras
    das imagens LANDSAT 8 num buffer de 100
    - Para usar no .map()'''
    
    cloudShadowBitmask = int(2**3)
    cloudBitmask = int(2**5)
    qa = imagem.select('pixel_qa')
    mask = qa.bitwiseAnd(cloudShadowBitmask).eq(0).And(\
           qa.bitwiseAnd(cloudBitmask).eq(0))\
        .focal_min(radius = 100, units = 'meters')
          
    return imagem.updateMask(mask)

def mask_bufferL8SR(imagem, buffer):
    """Aplica máscara de nuvens e sombra e
    aplica buffer em metros no entorno de uma imagem."""
    
    cloudShadowBitmask = int(2**3)
    cloudBitmask = int(2**5)
    qa = imagem.select('pixel_qa')
    mask = qa.bitwiseAnd(cloudShadowBitmask).eq(0).And(\
           qa.bitwiseAnd(cloudBitmask).eq(0))\
        .focal_min(radius = buffer, units = 'meters')
          
    return imagem.updateMask(mask)
```

Com tudo isso encaminhado, vamos carregar nossa imagem e aplicar essas funções. 
No código abaixo, deixei como comentário as linhas que aplicavam as funções numa coleção de imagens, bastando você descomentar elas e ajustar as variáveis.

```python
img_1 = ee.Image('LANDSAT/LC08/C01/T1_SR/LC08_220080_20200420')
img_noCloud_1 = mask_bufferL8SR(img_1, 0)
img_noCloudBuf_1 = mask_bufferL8SR(img_1, 250)

# imageC = ee.ImageCollection('LANDSAT/LC08/C01/T1_SR')\
#     .filterDate('2020-04-20', '2020-04-21')\
#     .filter(ee.Filter.eq('WRS_PATH', 220))\
#     .filter(ee.Filter.eq('WRS_ROW', 80))

#img_1C = imageC.first()
    
# img_noCloudC = imageC.map(maskL8SR)
# img_noCloud_1 = img_noCloudC.first()
# img_noCloudBufC = imageC.map(maskL8SR_100)
# img_noCloudBuf_1 = img_noCloudBufC.first()
```

Agora vamos obter a geometria da imagem e pegar a coordenada central dela para visualizar os resultados.
Também vamos definir os valores máximos e mínimos e gama para a visualização e vamos salvar o mapa em um HTML.

```python
dict_center = img_1.geometry().centroid().getInfo()
coord_center = list(dict_center.values())[1]

coord_center.reverse()
minimo = 0
maximo = 6000
gamma = 1.4
img_ID = img_1.getMapId({'bands': ['B4','B3','B2'], 'min': minimo, 'max': maximo, 'gamma': gamma})
img_NC_ID = img_noCloud_1.getMapId({'bands': ['B4','B3','B2'], 'min': minimo, 'max': maximo, 'gamma': gamma})
img_NC_Buf_ID = img_noCloudBuf_1.getMapId({'bands': ['B4','B3','B2'], 'min': minimo, 'max': maximo, 'gamma': gamma})

mapa = Mapdisplay(coord_center, {'Normal Image': img_ID, 
                           'No Cloud Image': img_NC_ID, 
                           'No Cloud and Buffer Image': img_NC_Buf_ID})
mapa.save("mask_buffer.html")
```

Você pode conferir o código completo [clicando aqui](https://github.com/basquiroto/EEwP/blob/master/GEE_mask_buffer.py). 
Uma amostra dos resultados é apresentado abaixo.

_Figura 1_ - Imagem com nuvem:

![Imagem com nuvem]({{ site.baseurl }}/images/imagem_001_Normal.png "Imagem LANDSAT 8 com nuvem")

_Figura 2_ - Imagem sem nuvem:

![Imagem sem nuvem]({{ site.baseurl }}/images/imagem_003_Buffer.png "Imagem LANDSAT 8 sem nuvem")

_Figura 3_ - Imagem sem nuvem e com buffer:

![Imagem sem nuvem e com buffer]({{ site.baseurl }}/images/imagem_002_SemNuvem.png "Imagem LANDSAT 8 sem nuvem e com buffer")


Algumas fontes consultadas (ou que serão consultadas) para desenvolver e melhorar o código.
* Sam Murphy. Cloud masking with Sentinel 2. Disponível em https://github.com/samsammurphy/cloud-masking-sentinel2/blob/master/cloud-masking-sentinel2.ipynb - Acesso em 19 out. 2020.

* Open Geo Blog. Working with Bitmasks. Disponível em https://mygeoblog.com/2019/07/25/working-with-bitmasks/ - Acesso em 19 out. 2020.

* Gonzalo Garcia. Cloud masking of multitemporal remote sensing images. Disponível em https://github.com/IPL-UV/ee_ipl_uv/blob/master/examples/multitemporal_cloud_masking_sample.ipynb - Acesso em 19 out. 2020.

