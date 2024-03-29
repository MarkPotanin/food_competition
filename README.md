# Food Competition
Информация и код для соревнования.
 
Задача: "Детекция еды на фото."

1) Необходимо определить продукт, изображенный на фотографии (или продукты, если их много). Например, яблоко, киви, пицца. 

2) Одновременно с этим необходимо определить местоположение и масштаб каждого из продуктов на фотографии.
Соответственно перед участниками ставится задача Object Detection. И предсказанием являются координаты bbox и метка класса.
 
 
В распоряжении участников имеется:
* 11493 изображения, лежат в папке train/images
* соответствующие этим изображениям аннотации. то есть информация о разметке данных. лежат в файле train/annotations.json
* бейзлайн food_detection_baseline.ipynb, в котором находится пример решения текущей задачи
* скрипт, который используется для локальной оценки качества (см. конец ноутбука с бейзлайном)
* скрипт evaluate.py для инференса (entry_point) 
 
Предлагается провести соревнование в докер контейнерах. То есть пайплайн отправки решений следующий: 
* участник загружает свой скрипт с обученной моделью и Dockerfile
* запускается докер контейнер
* внутрь подкладываются тестовые изображения (public)
* делаются предсказания
* оценивается качество, для размещения результата на лидерборде
 
Ссылка на данные - https://drive.google.com/file/d/1oEFlMYD8eWPahCx5OwIorwbFtZz28Y3E/view?usp=sharing . Внутри есть две папки: to_admin и to_participants. Соответственно участникам соревнования доступны только данные из to_participants. Данные из to_admin предназначаются для администраторов соревнования и настройки системы.

Ссылка на новые данные (15.04.21) - https://drive.google.com/file/d/1u0WS2kFYolCQpmX671v_udV7Z98EOXEw/view?usp=sharing. Расширены public/private.

Ссылка на чекпойнт модели - https://drive.google.com/file/d/1HErx5YDHLjDu8JcCuLGSGYJuYN1Jhv1z/view?usp=sharing

Данные должны быть в папке data/

Чекпойнт в папке output/

# Описание данных

## Входные данные

* изображения лежат в папке ```train/images```
* аннотации в файле ```train/annotations.json```

```annotations.json``` - словарь. Он имеет три ключа - ```['images', 'annotations', 'categories']```.

```annotations['images']``` - это список словарей вида ```{'id': 0, 'file_name': '0.jpg', 'width': 464, 'height': 464}```. То есть в ```annotations['images']``` лежит список словарей с информацией об изображениях. В частности там указан ```id``` изображения, путь до него - ```file_name```, а так же его ширина и высота - ```width``` и ```height```.

```annotations['categories']``` - это список словарей вида ```{'id': 0, 'name': 'картофель фри'}```. Здесь просто лежит список всех категорий в датасете с указанием численного  ```id``` и названия ```name``` .

```annotations['annotations']``` - это список словарей вида ```{'id': 0, 'image_id': 0, 'category_id': 10, 'area': 44320.0, 'bbox': [94, 62, 239, 323], 'iscrowd': 0, 'bbox_mode': 1}```. Тут лежит информация о выделенных объектах на изображениях, то есть информация о каждом bounding box. ```id``` - айдишник аннотации; ```image_id``` - номер изображения к которому относится данная аннотация;  ```category_id``` - номер категории (то есть лэйбл) того, что изображено в аннотации (хлеб, сыр, кофе);  ```area``` - не используется;  ```bbox``` - координаты bounding box в формате XYWH (XY - коордианты левого верхнего угла, WH - длина и ширина bounding box); ```iscrowd``` - не используется; ```bbox_mode``` - формат bounding box, используется XYWH.

## Выходные данные

Пример предсказания представлен в файле ```predict.json```. Он должен у вас получиться после предсказания на тестовых изображениях, которые система положит в ваш контейнер.

```predict.json``` - это список словарей вида ```{'image_id': 0, 'category_id': 10, 'bbox': [105.05099487304688, 75.59319305419922, 223.532958984375, 314.0073547363281], 'score': 0.9641055464744568}```. Каждый элемент (словарь в списке) является предсказанием на конкретном изображении (одно изображение может иметь несколько предсказаний).  ```image_id``` - айдишник изображения для предсказания; ```category_id``` категория предсказанного bounding box (например хлеб, сыр, кофе и т.д.); ```bbox``` - предсказанные координаты bounding box; ```score``` - скор (вероятность) предсказания. В предоставленном бейзлайне ```food_detection_baseline.ipynb``` показан пример генерации выходного файла ```predict.json```. А так же оценка качества на валидационном датасете.
