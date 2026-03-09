# Robotics-Control-Challenge-4.1
# 🤖 xArm Lite 6 - Task-Space Control & Dynamic Analysis (PID vs CTC)

## 👥 Equipo
* Bruno Rubio García
* Unai Para Manchado
* Mikel Para Manchado
* Iñigo González Olarte

## 📝 Descripción del Proyecto
Este repositorio contiene la implementación y el análisis de controladores de posición en el espacio de la tarea (Task-Space) para un manipulador xArm Lite 6. El proyecto compara el rendimiento de un controlador libre de modelo (PID) implementado en hardware real contra un controlador basado en el modelo dinámico (Computed Torque Control - CTC) proyectado mediante simulación basada en datos (*Data-Driven*).

La evaluación mide la robustez de ambas estrategias ante perturbaciones externas siguiendo una trayectoria de Lemniscata de Gerono plana (Z = 0.332 m) generada mediante Cinemática Inversa (IK) por tasa resuelta con prioridad en el eje vertical.

## 📂 Estructura del Repositorio
* `src/`: Scripts de Python correspondientes al generador de trayectoria, Cinemática Inversa y lazos de control (cumpliendo con la guía de estilo PEP8).
* `data/`: Archivos `.csv` con los registros de posición real vs deseada bajo las distintas pruebas (con y sin perturbaciones).
* `Informe_Final.pdf`: Documento con el análisis riguroso de resultados, energía de control y gráficas del Root Mean Square Error (RMSE).

## ⚙️ Requisitos
* Ubuntu 22.04 / ROS 2 Humble
* Paquetes de UFACTORY xArm (`xarm_ros2`)
* MoveIt Servo (`servo_server`)
* Python 3.10+ (NumPy, Pandas, rclpy, tf2_ros)

## 🚀 Instrucciones de Ejecución

**1. Compilar el espacio de trabajo:**
bash
cd ~/dev_ws
colcon build
source install/setup.bash

**2. Lanzar el entorno base y MoveIt Servo:
(Ajustar la IP del robot según la red local)
ros2 launch xarm_planner xarm6_planner_realmove.launch.py robot_ip:=192.168.1.135

**3. Ejecutar el Controlador PID (Estado Nominal):
ros2 run xarm_perturbations position_controller --ros-args \
-p output_topic:=/servo_server/delta_twist_cmds \
-p kp:="[1.5, 1.5, 1.5]" -p kd:="[0.2, 0.2, 0.2]" -p ki:="[0.0, 0.0, 0.0]" \
-p max_speed:=0.08 -p deadband:=0.003

**4. Ejecución con Inyección de Perturbación (Ruido Gaussiano):
(Requiere redirigir la salida del controlador PID hacia /perturbation_input)
ros2 run xarm_perturbations perturbation_injector --ros-args \
-p input_topic:=/perturbation_input \
-p output_topic:=/servo_server/delta_twist_cmds \
-p mode:=gaussian -p noise_mean:=0.0 -p noise_stddev:=0.02 -p noise_axis:=all

📊 Resultados Resumidos (Métrica RMSE)
Controlador,Condición,RMSE (m),Observación Principal
PID (Real),Sin perturbación,0.029,Error nominal debido al desfase temporal reactivo.
PID (Real),Con perturbación,0.397,Pérdida de referencia y saturación (windup).
CTC (Simulado),Sin perturbación,0.0001,Cancelación analítica idealizada de dinámicas.
CTC (Simulado),Con perturbación,0.0009,Alta absorción de fuerzas externas.
