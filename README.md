# capytown_G3 · RC-1: Control PID, del simulador al robot real

Robótica (08079) · Universidad ESAN · 2026-2 · Prof. Marks Calderón Niquin ·
Sección **S-003** · Equipo **3** · Tag de entrega: `v1.0-rc1`

| Integrante | Rol |
| --- | --- |
| César Sebastián Patazca Rios | Integración + Planificación |
| Roger Nahim Georgy Patiño Rodulfo | Control |
| Herman Nicolas Figuera Ferrer | Percepción |

**Informe:** [`informe.pdf`](informe.pdf) (1 página)

## Montaje

- **Simulador:** `yahboomcar_sim` en WSL2 (Ubuntu 22.04, ROS 2 Humble), `noise:=low` (Parte A con `noise:=high`)
- **Robot:** Yahboom MicroROS-Pi5, carro n.º 8 (`roscar8`, 10.42.0.1), `ROS_DOMAIN_ID` 8
- **Odometría del robot:** `/odom` de `odom_corrector` (solo encoders, sin EKF), ≈ 11 Hz
- **Código:** `cuadrado_controller.py` + `core_cuadrado.py` del curso, **sin modificar**; todo se ajustó con parámetros `-p`

## Ganancias finales

| | Recta (kp, ki, kd) | Giro (kp_yaw, ki_yaw, kd_yaw) | Velocidades |
| --- | --- | --- | --- |
| Simulador | 3.0, 0.5, 4.0 | 5.0, 0.0, 0.4 | 0.20 m/s, max_angular 2.0 |
| Robot real | 2.0, 0.0, 4.0 | 5.0, 0.0, 0.4 | 0.10 m/s, max_angular 0.5 |

Re-sintonías en el robot: 1) `max_angular` 1.0 → 0.5 (derrape al girar); 2) `ki` 0.5 → 0 (wind-up);
3) `kp` 3 → 2 (curva inicial más suave). Calibración de `wheel_separation` (b_eff) 0.2259 → 0.2132 m
en [`config/wheel_params_rc1.yaml`](config/wheel_params_rc1.yaml).

## Resultados

| Parte | Prueba | Resultado | Meta |
| --- | --- | --- | --- |
| A | Simulador: `/odom` vs pose real (`noise:=high`) | `/odom` 18.3 cm vs real 106.0 cm | — |
| B | Recta 1A → 1D (bang-bang, P, PD, PID) | 1D: error final 0.21 cm, asentamiento 5.2 s | ≤ 2 cm |
| C | Giro 90° 2a–2f | 2e (kp_yaw 5, kd_yaw 0.4): sobreimpulso 0 %, 1.59 s | — |
| D | Cuadrado 3A / 3B / 3C ×3 | cierre real promedio 45.1 / 18.7 / 42.4 cm | — |
| E5 | Robot: giro 90° ×3 | 0°, 0°, 0° | ≤ 3° |
| E6 | Robot: recta 1D (kp 2) | −1.5, +13.5, +5, +12, +3, +7.5 cm (3 de 6) | ≤ 5 cm |
| E7 | Robot: cuadrado 2×2 m | 3A 15 cm · 3B 75, 44, 54.5 cm (`/odom`: 2.2, 3.2, 1.4 cm) | 3B ≤ 25 cm y < 3A |

En el robot, la diferencia entre `/odom` y la cinta viene del derrape de las 4 ruedas al girar,
que los encoders no ven (ver `informe.pdf`).

## Contenido

| Ruta | Qué es |
| --- | --- |
| `informe.pdf` | Informe de una página |
| `hoja_resultados.csv` | Hoja de resultados: todas las pruebas del simulador y del robot |
| `tabla_sim_robot.csv` | Tabla simulador → robot |
| `config/rc1_pid.yaml` | Ganancias finales del simulador y del robot |
| `config/wheel_params_rc1.yaml` | Parámetros de odometría del robot (b_eff 0.2132) |
| `codigo/` | `cuadrado_controller.py`, `core_cuadrado.py` (nodo y lógica, del curso), `09_plot_csv.py` (gráficas), `cierre_real.py` (cierre real en el simulador) |
| `simulador/parte_A` | `sim_log.txt`, `rqt_graph.png`, CSV y PNG de la Parte A |
| `simulador/parte_B` | Recta 1A–1D: CSV, PNG y `resultados_B.txt` |
| `simulador/parte_C` | Giro 2a–2f: CSV, PNG y `resultados_C.txt` |
| `simulador/parte_D` | Cuadrado 3A–3C: CSV, PNG, cierre real (`*_cierre`) y `resultados_D.txt` |
| `robot/csv`, `robot/plots` | Parte E: un CSV y un PNG por prueba del robot |
| `robot/lab_log.txt` | Verificación del bringup y anotaciones del laboratorio |
| `bags/` | Bags del robot (`/odom`, `/cmd_vel`, `/tf`) |

## Cómo reproducir

Bringup en el robot (dentro del contenedor, tras `./ros2_humble.sh`):

```bash
ros2 launch capytown_esan bringup.launch.py wheel_params:=/root/yahboomcar_ws/capytown_G3/rc1/wheel_params_rc1.yaml
```

Cuadrado 3B con las ganancias finales del robot:

```bash
python3 codigo/cuadrado_controller.py --ros-args -p experimento:=cuadrado -p kp:=2.0 -p ki:=0.0 -p kd:=4.0 -p kp_yaw:=5.0 -p ki_yaw:=0.0 -p kd_yaw:=0.4 -p linear_speed:=0.10 -p max_angular:=0.5 -p log_csv:=exp3b_real_run1.csv
```

Gráfica de un CSV:

```bash
python3 codigo/09_plot_csv.py robot/csv/exp3b_real_run1.csv --experimento cuadrado --lado 2.0
```
