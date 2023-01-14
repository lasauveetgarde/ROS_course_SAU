# Заголовок

- [DWA Local Planner](#dwa-local-planner)
- [](#)
- [](#)
- [](#)
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


## Чему научились?

TODOS

## Задание

> :muscle: Текст задачки
> 
> =)

## Вопросики

TODO

## Ресурсы