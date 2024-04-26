[TOC]

# 1 机器人实例化

## 1.1 SDK初始化

- 函数接口

>  bool init();

- 返回参数

| 返回参数 | 参数类型 | 参数说明                |
| :------- | :------- | :---------------------- |
| True     | bool     | Trus：成功  False：失败 |

## 1.2 SDK释放

- 函数接口

>  bool uninit();

- 返回参数

| 返回参数 | 参数类型 | 参数说明                |
| :------- | :------- | :---------------------- |
| True     | bool     | Trus：成功  False：失败 |

## 1.3 注册构建机器人

- 函数接口

>  bool addRobot(string robot_id, string path);

- 请求参数

| 请求参数 | 参数类型 | 参数说明           |
| :------- | :------- | :----------------- |
| robot_id | string   | Builder内机器人Id  |
| path     | string   | 机器人配置文件路径 |

- 返回参数

| 返回参数 | 参数类型 | 参数说明                        |
| :------- | :------- | :------------------------------ |
| True     | bool     | Trus：注册成功  False：注册失败 |

## 1.4 删除机器人

- 函数接口

>  bool deleteRobot(string robot_id);

- 请求参数

| 请求参数 | 参数类型 | 参数说明          |
| :------- | :------- | :---------------- |
| robot_id | string   | Builder内机器人Id |

# 2 构型参数

## 2.1 计算构型参数

- 函数接口

>  void calculateRobotConfig(string robot_id, ref List<int> S, ref List<int> T, List<double> joint_list);

- 请求参数

| 请求参数   | 参数类型     | 参数说明                      |
| :--------- | :----------- | :---------------------------- |
| robot_id   | string       | Builder内机器人Id             |
| S          | List<int>    | 构型参数S，列表合并后为二进制 |
| T          | List<int>    | 构型参数T，列表合并后为二进制 |
| joint_list | List<double> | 六轴关节坐标                  |

## 2.2 获取构型参数

- 函数接口

>  void getRobotConfiguration(string robot_id, ref List<int> S, ref List<int> T);

- 请求参数

| 请求参数 | 参数类型  | 参数说明                      |
| :------- | :-------- | :---------------------------- |
| robot_id | string    | Builder内机器人Id             |
| S        | List<int> | 构型参数S，列表合并后为二进制 |
| T        | List<int> | 构型参数T，列表合并后为二进制 |

# 3 示教走点

## 3.1 点动

- 函数接口

>  bool motion(string robot_id, List<double> joint, int vel, int acc, int smoot);

- 请求参数

| 请求参数 | 参数类型     | 参数说明          |
| :------- | :----------- | :---------------- |
| robot_id | string       | Builder内机器人Id |
| joint    | List<double> | 关节坐标          |
| vel      | int          | 速度              |
| acc      | int          | 加速度            |
| smoot    | int          | 平滑度            |

- 返回参数

| 返回参数 | 参数类型 | 参数说明                |
| :------- | :------- | :---------------------- |
| True     | bool     | Trus：成功  False：失败 |

## 3.2 连续走点（旧）

- 函数接口

>  bool motionarray(string robot_id, List<List<double>> joint, int vel, int acc, int smoot, int flag);

- 请求参数

| 请求参数 | 参数类型           | 参数说明          |
| :------- | :----------------- | :---------------- |
| robot_id | string             | Builder内机器人Id |
| joint    | List<List<double>> | 关节坐标          |
| vel      | int                | 速度              |
| acc      | int                | 加速度            |
| smoot    | int                | 平滑度            |
| flag     | int                | flag              |

- 返回参数

| 返回参数 | 参数类型 | 参数说明                |
| :------- | :------- | :---------------------- |
| True     | bool     | Trus：成功  False：失败 |

## 3.3 连续走点（新）

- 函数接口

>  bool motion_array(string robot_id, List<List<double>> joint, List<List<double>> tcp_frame, List<int> vel, List<int> acc, List<int> smoot, List<int> move_type, List<int> pose_type, List<bool> connect_flag);

- 请求参数

| 请求参数     | 参数类型           | 参数说明                             |
| :----------- | :----------------- | :----------------------------------- |
| robot_id     | string             | Builder内机器人Id                    |
| joint        | List<List<double>> | 关节坐标                             |
| tcp_frame    | List<List<double>> | 每个点位的tcp(坐标+旋转)             |
| vel          | List<int>          | 点位速度                             |
| acc          | List<int>          | 点位加速度                           |
| smoot        | List<int>          | 点位平滑度                           |
| move_type    | List<int>          | 点位运动方式，0:关节运动，1:线性运动 |
| pose_type    | List<int>          | 点位flag                             |
| connect_flag | List<int>          | 点位协同状态                         |


- 返回参数

| 返回参数 | 参数类型 | 参数说明                |
| :------- | :------- | :---------------------- |
| True     | bool     | Trus：成功  False：失败 |

## 3.4 开始/继续

- 函数接口

>  bool motionStart(string robot_id);

- 请求参数

| 请求参数 | 参数类型 | 参数说明          |
| :------- | :------- | :---------------- |
| robot_id | string   | Builder内机器人Id |

- 返回参数

| 返回参数 | 参数类型 | 参数说明                |
| :------- | :------- | :---------------------- |
| True     | bool     | Trus：成功  False：失败 |

## 3.5 暂停

- 函数接口

>  bool motionPause(string robot_id);

- 请求参数

| 请求参数 | 参数类型 | 参数说明          |
| :------- | :------- | :---------------- |
| robot_id | string   | Builder内机器人Id |

- 返回参数

| 返回参数 | 参数类型 | 参数说明                |
| :------- | :------- | :---------------------- |
| True     | bool     | Trus：成功  False：失败 |

## 3.6 查询是否在运动

- 函数接口

>  bool isMotion(string robot_id);

- 请求参数

| 请求参数 | 参数类型 | 参数说明          |
| :------- | :------- | :---------------- |
| robot_id | string   | Builder内机器人Id |

- 返回参数

| 返回参数 | 参数类型 | 参数说明                        |
| :------- | :------- | :------------------------------ |
| True     | bool     | Trus：正在运动  False：停止运动 |

## 3.7 获取运动状态

- 函数接口

>  bool getStatus(string robot_id, ref List<double> joint, int count, ref List<double> position, ref List<double> rotate, ref double index);

- 请求参数

| 请求参数 | 参数类型     | 参数说明                  |
| :------- | :----------- | :------------------------ |
| robot_id | string       | Builder内机器人Id         |
| joint    | List<double> | 机器人当前关节坐标+附加轴 |
| count    | int          | joint列表数量             |
| position | List<double> | 机器人位置                |
| rotate   | List<double> | 机器人旋转姿态            |
| index    | double       | 当前运动点位下标          |

- 返回参数

| 返回参数 | 参数类型 | 参数说明                |
| :------- | :------- | :---------------------- |
| True     | bool     | Trus：成功  False：失败 |

## 3.8 设置当前关节姿态

- 函数接口

>  bool setCurrentJoint(string robot_id, List<double> joint);

- 请求参数

| 请求参数 | 参数类型     | 参数说明           |
| :------- | :----------- | :----------------- |
| robot_id | string       | Builder内机器人Id  |
| joint    | List<double> | 机器人当前关节坐标 |

- 返回参数

| 返回参数 | 参数类型 | 参数说明                |
| :------- | :------- | :---------------------- |
| True     | bool     | Trus：成功  False：失败 |

## 3.9 设置坐标系

- 函数接口

>  void setFrames(string robot_id, List<List<double>> rail_transformInfo_list, List<List<double>> bwj_transformInfo_list, List<double> rail_transformInfo_type_list, List<double> bwj_transformInfo_type_list);

- 请求参数

| 请求参数                     | 参数类型           | 参数说明                                       |
| :--------------------------- | :----------------- | :--------------------------------------------- |
| robot_id                     | string             | Builder内机器人Id                              |
| rail_transformInfo_list      | List<List<double>> | 地轨、悬臂、机器人(节点相对坐标+旋转)          |
| bwj_transformInfo_list       | List<List<double>> | 变位机(节点相对坐标+旋转)                      |
| rail_transformInfo_type_list | List<double>       | 运动类型（0-5:xyz正负移动，6-11：xyz正负旋转） |
| bwj_transformInfo_type_list  | List<double>       | 运动类型（0-5:xyz正负移动，6-11：xyz正负旋转） |