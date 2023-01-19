# Заголовок

- [DWA Local Planner](#dwa-local-planner)
- [Costmap Common Params](#costmap-common-params)
- [Local and Global CostMAP](#local-and-global-costmap)
- [Move Base Params](#move-base-params)
- [Чему научились?](#чему-научились)
- [Задание](#задание)
- [Вопросики](#вопросики)
- [Ресурсы](#ресурсы)


## DWA Local Planner

В прошлом топике мы уже начали знакомиться с локальным планировщиков [DWA local planner](http://wiki.ros.org/dwa_local_planner), однако все параметры в нем были выставлены по default. Давайте посмотрим, что в нем вообще имеется, откроем наш `dwa_local_planner_params_waffle.yaml`. Из названия уже понятно, что там должны быть какие-то параметры :speak_no_evil:

```xml
DWAPlannerROS:

# Robot Configuration Parameters
  max_vel_x: 0.26
  min_vel_x: -0.26

  max_vel_y: 0.0
  min_vel_y: 0.0

# The velocity when robot is moving in a straight line
  max_vel_trans:  0.26
  min_vel_trans:  0.13

  max_vel_theta: 1.82
  min_vel_theta: 0.9

  acc_lim_x: 2.5
  acc_lim_y: 0.0
  acc_lim_theta: 3.2 

# Goal Tolerance Parameters
  xy_goal_tolerance: 0.05
  yaw_goal_tolerance: 0.17
  latch_xy_goal_tolerance: false

# Forward Simulation Parameters
  sim_time: 2.0
  vx_samples: 20
  vy_samples: 0
  vth_samples: 40
  controller_frequency: 10.0

# Trajectory Scoring Parameters
  path_distance_bias: 32.0
  goal_distance_bias: 20.0
  occdist_scale: 0.02
  forward_point_distance: 0.325
  stop_time_buffer: 0.2
  scaling_speed: 0.25
  max_scaling_factor: 0.2

# Oscillation Prevention Parameters
  oscillation_reset_dist: 0.05

# Debugging
  publish_traj_pc : true
  publish_cost_grid_pc: true

```

Так, ну мы вас не обмунули и вы, действительно, можете увидеть тут кучу каких-то параметров, которые пока что особо не понятно, за что отвечают. Настало время разобраться с этим. Все параметры сгруппирированы в несколько категорий, а именно:

- `Robot Configuration Parameters` - параметры конфигурации робота 
- `The velocity when robot is moving in a straight line` - параметры при движении по прямой линии
- `Goal Tolerance Parameters` - параметры допустимого отклонения от цели
- `Forward Simulation Parameters` - параметры симуляции вперед
- `Trajectory Scoring Parameters` - параметры оценки траектории
- `Oscillation Prevention Parameters` - параметры предотвращения колебаний
- `Debugging`- отладка

Запускаем наш созданный в прошлом топике launch `turtlebot3_gazebo_slam.launch`

Для того, чтобы проще было настраивать параметры конфигурации в запущенной системе (и не перезапускать по 20 раз) воспользуемся `rqt_reconfigure`

```bash
rosrun rqt_reconfigure rqt_reconfigure
```

<p align="center">
<img src=../assets/01_11_rqt_reconfigure.png width=900/>
</p>

Напимер, давайте посмотрим на параметр `max_vel_theta`. Для `waffle` этот параметр задан, как `max_vel_theta: 1.82`

<p align="center">
<img src=../assets/01_12_dwa_max_theta1_82.gif width=900/>
</p>

Окей, а что будет, если этот параметр понизить до 0.5? А потом до 0? Пробуем

<p align="center">
<img src=../assets/01_13_dwa_max_theta0_5.gif width=500/> <img src=../assets/01_14_dwa_max_theta0.gif width=500/>
</p>

> :muscle: Время сделать вывод... каким образом изменение параметра `max_vel_theta` сказывается на планировании маршрута робота?

> :muscle: Теперь попробуйте самостоятельно поменять параметры `sim_time` и `occdist_scale`. Сделайте вывод о том, как уменьшение/увеличение этих значений влияет на планирование маршрута робота.

Получилось? Здорово! Тогда двигаемся дальше


## Costmap Common Params

В ROS, как правило, в стеке навигации существуют две карты, хранящие информацию о препятствиях в окружающей среде. Это local and global costmap. Из названия понятно, что одна из них глобальная - та, которая строит протяженный маршрут до конечной цели, а локальная - способна перестраивать маршруты непосредственно в те моменты, когда на свободном пространстве по какой-то причине появилось препятствие, которое следует объехать.

В `costmap_common_params` устанавливается, как правило, что будет касаться, как и локального, так и глобального планировщика.
Тут применяется карта стомоимости, предоставляющая сведения о препятсвиях в мире. Для того, чтобы обеспечить коррректную связь между нужно связать карты затрат с теми устройствами, которые будут предоставлять им информацию об окружающей обстановке. В общем виде конфигурация параметро выглядит так :point_down:

```bash
obstacle_range: 2.5
raytrace_range: 3.0
footprint: [[x0, y0], [x1, y1], ... [xn, yn]]
#robot_radius: ir_of_robot
inflation_radius: 0.55

observation_sources: laser_scan_sensor point_cloud_sensor

laser_scan_sensor: {sensor_frame: frame_name, data_type: LaserScan, topic: topic_name, marking: true, clearing: true}

point_cloud_sensor: {sensor_frame: frame_name, data_type: PointCloud, topic: topic_name, marking: true, clearing: true}
```
Давайте тут уже остановимся чуть-чуть поподробнее. Итак:

- `obstacle_range` - максимальная дальность, при которой робот будет учитывать препятствия и наносить их на свою карту. В нашем случае - 2.5 метра.
- `raytrace_range` - максимальность дальность, при которой будет происходить чистка карты от динамически возникающих объектов. 
- `footprint` - размеры робота, предполагая, что его центр находится в точке (0;0),(0;0)
- `robot_radius` - размеры робота, если он круглый
- `inflation_radius` - увеличение ширины стен и препятствий. Например, если этот параметр задан, как 0.55, то тогда робот будет прокладывать маршрут таким образом, чтобы не притираться к увеличенным стенам/препятствиям на 0.55 метра. Посмотрите рисунок ниже, чтобы лучше понять это.

<p align="center">
<img src=../assets/01_15_tuning_inflation_radius.png width=700/>
</p>

- `observation_sources` - список из датчиков, которые будут публиковать данные в карту затрат
- `laser_scan_sensor` или `point_cloud_sensor`, где в качестве `sensor_frame` устанавливается система координат датчика, `data_type` - тип датчика, `topic` - имя топика, `marking` и `clearing` - определение разрешения на добавление и очистку соответсвенно препятствий с карты затрат.

Надеемся, что теперь стало еще более понятно. Поэтому давайте откроим наш `costmap_common_params_[model_name].yaml`


```bash
obstacle_range: 3.0
raytrace_range: 3.5

footprint: [[-0.205, -0.155], [-0.205, 0.155], [0.077, 0.155], [0.077, -0.155]]
#robot_radius: 0.17

inflation_radius: 1.0
cost_scaling_factor: 3.0

map_type: costmap
observation_sources: scan
scan: {sensor_frame: base_scan, data_type: LaserScan, topic: scan, marking: true, clearing: true}

```

### Local and Global Costmap


- `global_frame` - параметр, определяющий в какой системе координат будет карта стоимости.
- `robot_base_frame` - система координат, 
- `update_frequency` - частота обновления карты 
- `publish_frequency` - частота публикации карта
- `transform_tolerance`
- `static_map` - принимает значения либо `True` либо `False` и определяет должен ли робот локализировать себя на уже существующей, подгруженной карте


`global_costmap_params.yaml`

```bash
global_costmap:
  global_frame: map
  robot_base_frame: base_footprint

  update_frequency: 10.0
  publish_frequency: 10.0
  transform_tolerance: 0.5

  static_map: true
```

В `local_costmap_params.yaml` должно быть все относительно понятно из предыдущего разбора, за исключением параметра `rolling_window`, принимающий либо `True` либо `False`. Если параметр будет принимать значение `True`, то в таком случае карта стоимости будет оставаться центрированной вокруг робота, когда робот перемещается по миру. Параметры `width`, `height` и `resolution` задают ширину, высоту и разрешение в метрах карты затрат.

```bash
local_costmap:
  global_frame: odom
  robot_base_frame: base_footprint

  update_frequency: 10.0
  publish_frequency: 10.0
  transform_tolerance: 0.5  

  static_map: false  
  rolling_window: true
  width: 3
  height: 3
  resolution: 0.05
```


## Move Base Params

```bash
shutdown_costmaps: false
controller_frequency: 10.0
planner_patience: 5.0
controller_patience: 15.0
conservative_reset_dist: 3.0
planner_frequency: 5.0
oscillation_timeout: 10.0
oscillation_distance: 0.2
```

## Чему научились?

TODOS

## Задание

> :muscle: Текст задачки
> 
> =)

## Вопросики

TODO

## Ресурсы