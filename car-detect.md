# Детекция и трекиг машины

Недавно Яндекс [показал](https://www.youtube.com/watch?v=7hUut7Hsgys) зимние испытания своего беспилотника.
В [архиве](TODO) находятся данные с сенсоров, установленных на одной из машин.
По этим данным вам предлагается написать детектор автомобилей в трехмерном пространстве. 
Такой детектор нужен автомобилю для разъездов с другими участниками движения, поэтому он должен определять не только расстояние до объекта, 
но и размер объекта, а так же его скорость.

## Данные

Данне предствлены в формате [rosbag](http://wiki.ros.org/rosbag), это формат логирования операционной системы робота, [ROS](http://www.ros.org/).
В таком логе записаны все показания сенсоров и вспомогательная информация. 
На машине установлены сенсоры [LiDAR](https://ru.wikipedia.org/wiki/Лидар) и несколько камер.

### LiDAR

Лидар осуществляет сканирование трехмерного пространства вокруг машины. Он формирует разреженную трехмерную карту пространства, называемую облаком точек.
В этом проезде было задействовано три лидара, один на крыше и два по бокам, над дверьми.

В логах доступно четыре облака точек, по одному на каждый из лидаров и одно общее, собранное с трех датчиков.
Все точки приведены к системе координат связаной с машиной. Подробнее о системах координат ниже.

### Камеры

На машине установлены четыре камеры: фронтальная, две боковых и задняя. 
Картинка с камер уже предобработана, в частности для нее убраны искажения привнесенные оптикой.
Для каждой из камер известна матрица соответствущей [pinhole камеры](https://en.wikipedia.org/wiki/Camera_matrix) 
и точка ее подвеса относительно системы координат связанной с машиной.

### Положение машины

Положение машины в глобальной системе координат (т.е. связанной с некоторой референсной точкой),
 скорость и ориентация машины пишется в логфайл несколько десятков раз в секунду. В рамках этого проезда можно считать эти данные абсолютно точными.

### Системы координат

На равне с показаниями сенсоров в лог пишутся данные о системах координат и правилах перехода между ними.
Наибольший интерес представяет система координат маины и мира.

#### Система координат машины

Система координат машины жестко связана с машиной и направлена так, что ось `z` смотрит вверх, ось `y` - налево, а ось `x` - вперед.
Начало системы координат находится на серидине проекции задней оси автомобиля на землю.

#### Система координат мира

Начало системы координат находится в некоторой референсной точке, скажем в кремле, ее оси `x` и `y` смотрят по сторонам света, а ось `z` смотрит наверх.
Референсная точка находится достаточно близко к месту проезда и искажениями связанными с кривизной земли можно пренебречь.

Преобразования между системами координат записаны в лог и доступны для использования, подробнее о системах координат в `ROS` можно ознакомиться в [статье](http://wiki.ros.org/tf).

## Что нужно сделать

В логах сохранен момент встречного разъезда двух машин. Нужно определить положение, размеры и скорость второй машины. Это нужно делать в реальном или
в почти реальном времени (по совести, даже оффлайн решение мы примем, но скорее всего, не оценим высоко). Для построения детекций можно использовать данные с лидара и картинку с камеры.
Для обработки картинки с камеры можно расчитывать на наличие графического ускорителя, во время провекрки вашего задания у нас такой будет.

Подходы могут быть любыми.
Например, можно реализовать лидарный трекер по примеру [работы](http://ieeexplore.ieee.org/document/7535461/figures).
Или использовать сверточные сети для детекций машины на картинке, а лидарные данные для оценки ее положения.

## Как сдавать

Проект расчитан на одного или двух человек.

Реализуйте любой алгоритм (или набор алгоритмов, например, для детекции и трекинга отдельно) который вам понравился и который будет работать хорошо. Запишите rosbag с детекциями для каждого момента времени в котором ваш алгоритм заметил машину. Подумайте сами о соотношении точности и полноты уместном в этой задаче. Публикуйте в rosbag те детекции которые считаете важными, помните, что заметить "фантомную" машину бывает не менее опасно чем не заметить настоящей.

При записи rosbag для отчета используйте приложенный формат [сообщения](TODO).

## Что будет проверяться 

1) Код должен быть размещен на [github](http://github.com)
2) Проект должен содержать в себе описание подхода в виде `md` или `pdf` файлов
3) Реализация проекта должна состоять из одного [пространства](http://wiki.ros.org/catkin/workspaces) проекта ROS. Пространство может содержать один или нескольких [catkin](http://wiki.ros.org/catkin/CMakeLists.txt) модулей. Для написания модулей используйте `C++` или `python`, но помните, о том, что реалтайм решение лучше оффлайн решения.
4) Пространство должно собираться командой [ctakin_make](http://wiki.ros.org/catkin/commands/catkin_make) без дополнительных параметров
5) Проект должен запускаться одним [launch](http://wiki.ros.org/roslaunch) файлом, в описании надо указать каким. Этот лонч файл должен запускать проигрывание логов с показаниями сенсоров и ваш модуль
6) Отдельно нужно отправить rosbag с детекциями
7) Научитесь визуализировать детекции в [rviz](http://wiki.ros.org/rviz), найдите подходящий модуль или [напишите](http://docs.ros.org/lunar/api/rviz_plugin_tutorials/html/display_plugin_tutorial.html) свой.

Мы посмотрим на качество детекций, почитаем код, попробуем собрать и запустить ваш проект. По результату поставим оценку, не стоит расчитывать на зачет если в марте у вас нет никакого варианта детекций (пусть даже и "пустого"). Хорошая работа должна компилироваться и запускаться без лишних вопросов. В отличной работе хочется видеть интересные мысли.

## Несколько советов

Это не самый простой с точки зрения окружения проект. Если вы решили взяться за него, будьте готовы, что на настройку окружения и на знакомство с ним может уйти несколько недель или даже месяц. Я не рекомендую откладывать знакомство с `ROS` далее февраля.

По личному опыту, рекомендую работать на "настоящей" (не виртуальной) [Ubuntu](https://www.ubuntu.com/). Отвечать на вопросы "почему у меня не работает" для других конфигураций я не смогу. Для `ROS` заявлена поддержка `MacOS`, но там есть тонкости, в первую очередь, с визуализацией. Аналогично, трудно заставить работать `ROS` в `docker` под хостовой системой отличной от `linux`.
Если вы не готовы ставить к себе настоящую Ubuntu, а попытать свои силы с роботами все же хочется, попробуйте работать в виртуальном окружении, например в [Virtual Box](https://www.virtualbox.org/). Этот вариант проверено работает, хотя, некоторые вещи выполняются существено медленнее чем на хосте.


