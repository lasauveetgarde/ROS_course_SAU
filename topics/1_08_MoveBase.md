# Заголовок

- [Состав move_base](#состав-move_base)
- [Карты стоимостей](#карты-стоимостей)
- [Планнер](#планнер)
- [Переходим к практике](#переходим-к-практике)
- [Чему научились?](#чему-научились)
- [Задание](#задание)
- [Вопросики](#вопросики)
- [Ресурсы](#ресурсы)



## Состав move_base

Стек навигации включает в себя множество пакетов для решения целей навигации: планировщики, работа с картами, локализация. Каждый узел существует незавсимо и может быть написан вручную или найден на просторах GitHub. Для объединения всего этого добра в ROS существует узел move_base. На картинке ниже показана схема работы узла move_base и его взаимодействия с другими компонентами. Синие узлы различаются в зависимости от платформы робота, серые являются необязательными, а белые узлы являются обязательными. На первый взгляд схема может показаться запутанной, но по мере изучения отдельных узлов навигации рекомендуется к ней возвращаться, и рано или поздно всё встанет на свои места =)


<p align="center">
<img src=../assets/01_08_overview_move_base.png width=900/>
</p>

Сам узел move_base находится в черном квадрате посередине и включает две большие задачи: построение карты стоимостей и планирование маршрутов.

> Во многих пакетах ROS встречается разделение на global и local. Local обычно привязывается к системе координат движущегося робота, а global к неподвижной системе координат карты. 

Начнем разбиратся по порядку.

## Карты стоимостей

Для того чтобы начать какое-либо движение, необходимо понимать вид окружающей среды: где можно ездить, а где нельзя. move_base преобразует данные бортовых датчиков (камеры, лидары и т.п) и предзаписанных карт в формат costmap.  Основной формат внутреннеого представление карт в ROS носит название OccupancyGrid, где пространтсво вокруг робота разбивается на клетки заданного размера. У каждой клетки есть значение "occ", которое отражает вероятность занятости клетки в процентах. В простейшем случае есть 3 вида клеток:
- свободная область: occ = 0
- занятая область: occ = 100
- неизвестная область: occ = -1

Обычно используется следующий подход: в global costmap отправляется карта всей местности (план этажа, квартиры, завода), а в local costmap отправляются данные лазерных лидаров и камер, которые помогут учитывать препятсвия возникающие перед роботом.

:octocat: TODO
Написать как у нас сейчас запускается тартлбот, я не читал...

Для того чтобы увидить отличие на примере запустим нашего тартлбота переходим в меню выбора отображения (`Add`->`By topic`), выбираем путь `move_base/global_costmap/costmap` и там выбираем Map. Перед тем, как окончательно выбрать, внизу в Display Name наберите "Global Map". Аналогично добавляем local_costmap. Должно получиться что-то вроде этого, где цветным показанна локальная карта стоимостей, а в оттенках серого - глобальная.

<p align="center">
<img src=../assets/01_09_costmaps.png width=500/>
</p>

> Для смены режима отображения карты, в меню Displays нажимаем на локальную карту и в выпадающем меню в разделе Color Scheme выбираем "`costmap`" 

:octocat: TODO
Написать как у нас сейчас запускается тартлбот, я не читал...

## Планнер

После создание карты, переходим к планированию движения. Если для карт стоимостей подход создания локальной и глобальной карты идентичен, то у планировщиков всё немного сложнее. Глобальный и локальный планнеры выполняют разные задачи и реализуются разными пакетами. Алгоритм следующий: 
1) Global Planner, используя global costmap, строит кратчайший путь к цели.
2) Local Planner следует глобальному пути, но учитывает возникающие препятсвия и габариты робота.

Работу планнеро так же можно увидеть в Rviz, для этого переходим в меню выбора отображения (`Add`->`By topic`), выбираем путь `move_base/NavfnROS/plan` и там выбираем Path (зеленая линия). Перед тем, как окончательно выбрать, внизу в Display Name наберите "Global Path". Аналогично добавьте "Local Path". 

> :muscle: Попробуйте позадавать цели и посмотреть на работу локального и глобального планнера. На сколько далеко строится маршрут? По какому принципу строится локальная и глобальная траектория?

:octocat: TODO
В целом это уже было в уроке по тартлботу, я хз че тут можно добавить.

## Переходим к практике

Сейчас давайте уже подробнее разберемся, как же заставить робота самостоятельно ехать по заданной траектории, а именно посмотрим на пакет [move_base](http://wiki.ros.org/move_base),располагающийся в стеке навигации, который мы с вами рассмотрели чуть раньше.

Для того, чтобы наш робот поехал, а именно заработала автономная навигация - нужно подготовить дополнительные файлы. Давайте скопируем файлы из `turtlebot3_navigation/param` к себе в папочку `config`.

Согласно умной [страничке в гугле](http://wiki.ros.org/turtlebot3_navigation) для навигации нашего робота нужно скопировать к себе следующие файлы:

```bash
base_local_planner_params.yaml              # The parameter of the speed command to the robot
costmap_common_params_burger.yaml           # The parameter of costmap configuration consists
costmap_common_params_waffle.yaml           # The parameter of costmap configuration consists
costmap_common_params_waffle_pi.yaml        # The parameter of costmap configuration consists
dwa_local_planner_params.yaml               # The parameter of the speed command to the robot
global_costmap_params.yaml                  # The parameter of the global area motion planning
local_costmap_params.yaml                   # The parameter of the local area motion planning
move_base_params.yaml                       # parameter setting file of move_base that supervises the motion planning.
```
Сделаем это через `roscp`:
Для нашего случая будет такой ряд команд, если вы ранее в качестве робота выбрали `waffle`:

```bash
# Путь к нашей папке config'ов
COPY_TARGET=`rospack find super_robot_package`/config
# Копируем файлы 
roscp turtlebot3_navigation costmap_common_params_waffle.yaml $COPY_TARGET
roscp turtlebot3_navigation local_costmap_params.yaml $COPY_TARGET
roscp turtlebot3_navigation global_costmap_params.yaml $COPY_TARGET
roscp turtlebot3_navigation move_base_params.yaml $COPY_TARGET
roscp turtlebot3_navigation dwa_local_planner_params_waffle.yaml $COPY_TARGET
```
Осталось совсем немного... скачаем локальный планировщик [dwa_local_planner](http://wiki.ros.org/dwa_local_planner)

```bash
sudo apt install ros-noetic-dwa-local-planner
```

Так, теперь нам осталось создать launch-файл для запуска всего этого добра. Давайте посмотрим, что у нас уже имеется и возьмем к себе на вооружение.
Итак, нам нужно посмотреть `turtlebot3_navigation/move_base.launch`. Если пакет `move_base` еще не установлен, то вы знаете, что делать :smirk:

```bash
roscat turtlebot3_navigation move_base.launch
```
Ну-с, тут вроде бы все достаточно понятно... вначале у нас идут аргументы, потом подгружаются конфиги, которые мы с вами уже скопировали к себе.

```xml
<launch>
  <!-- Arguments -->
  <arg name="model" default="$(env TURTLEBOT3_MODEL)" doc="model type [burger, waffle, waffle_pi]"/>
  <arg name="cmd_vel_topic" default="/cmd_vel" />
  <arg name="odom_topic" default="odom" />
  <arg name="move_forward_only" default="false"/>

  <!-- move_base -->
  <node pkg="move_base" type="move_base" respawn="false" name="move_base" output="screen">
    <param name="base_local_planner" value="dwa_local_planner/DWAPlannerROS" />
    <rosparam file="$(find turtlebot3_navigation)/param/costmap_common_params_$(arg model).yaml" command="load" ns="global_costmap" />
    <rosparam file="$(find turtlebot3_navigation)/param/costmap_common_params_$(arg model).yaml" command="load" ns="local_costmap" />
    <rosparam file="$(find turtlebot3_navigation)/param/local_costmap_params.yaml" command="load" />
    <rosparam file="$(find turtlebot3_navigation)/param/global_costmap_params.yaml" command="load" />
    <rosparam file="$(find turtlebot3_navigation)/param/move_base_params.yaml" command="load" />
    <rosparam file="$(find turtlebot3_navigation)/param/dwa_local_planner_params_$(arg model).yaml" command="load" />
    <remap from="cmd_vel" to="$(arg cmd_vel_topic)"/>
    <remap from="odom" to="$(arg odom_topic)"/>
    <param name="DWAPlannerROS/min_vel_x" value="0.0" if="$(arg move_forward_only)" />
  </node>
</launch>

```
Давайте создадим себе `turtlebot3_move_base.launch` и удалим из `turtlebot3_navigation/move_base.launch` конфиги, которые мы не будем использовать. В принципе, чтобы наш с вами робот поехал автономно осталось уже совсем немного. Осталось написать launch-файл для запуска симулятора, навигации и построения карты. Наверно, я вас сейчас обрадую:). У нас уже с вами все есть.

> :muscle: Попробуй написать launch-файл `turtlebot3_gazebo_slam.launch`, в котором будет запуск `turtlebot3_sim_start.launch`, `turtlebot3_my_gmapping.launch`, `turtlebot3_move_base.launch` и не забудь добавить запуск `rviz`

<p align="center">
<img src=../assets/01_10_slam.gif width=900/>
</p>


## Чему научились?

TODOS

## Задание

> :muscle: Текст задачки
> 
> =)

## Вопросики

TODO

## Ресурсы