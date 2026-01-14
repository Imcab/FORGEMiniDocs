# Optimizer

**Optimizer** es el gestor de memoria del sistema de **ForgeMini**.

El roboRIO tiene recursos limitados (CPU, RAM y Disco). Esta clase actúa como un "conserje" automático que trabaja en segundo plano para asegurar que el robot no sufra de *lag spikes* por falta de memoria o fallos por disco lleno.

---

## **Implementación**

Para activar el Optimizer, solo necesitas agregar dos líneas en tu archivo `Robot.java`.

=== "Robot.java"

    ```java
    package frc.robot;

    import edu.wpi.first.wpilibj.TimedRobot;
    import com.stzteam.forgemini.Optimizer; // Importar

    public class Robot extends TimedRobot {

        @Override
        public void robotInit() {
            // 1. Inicializa limpieza de disco y optimización de red
            Optimizer.init();
        }

        @Override
        public void robotPeriodic() {
            // 2. Monitorea la RAM y el estado de la batería
            Optimizer.update();
            
            // ... resto de tu código
        }
    }
    ```

---

## **Funciones Automáticas**

### **1. Gestión de Memoria RAM**
Java utiliza un "Garbage Collector" (GC) para liberar RAM. Si el GC se ejecuta durante un partido, puede congelar el robot por unos milisegundos (Lag Spike).

* **¿Qué hace Optimizer?** Monitorea la RAM disponible en cada ciclo.
* **¿Cuándo actúa?** Si la memoria libre cae por debajo del **20%** y el robot está **DESHABILITADO**.
* **Resultado:** Fuerza una limpieza de memoria segura mientras el robot está quieto, reduciendo la probabilidad de que ocurra durante el match.

!!! warning "Seguridad"
    Optimizer nunca forzará el Garbage Collector mientras el robot esté habilitado (Teleop o Autónomo) para evitar interrupciones en el control.

### **2. Limpieza de Logs**
Los archivos de log (`.wpilog`) pueden llenar el disco del roboRIO rápidamente, causando que el código deje de desplegarse o falle al iniciar.

* **Límite:** Mantiene solo los últimos **10 archivos** de log.
* **Acción:** Al iniciar el robot (`init()`), elimina los archivos más viejos automáticamente.
* **Rendimiento:** Esta tarea corre en un **hilo separado** (Thread) para no ralentizar el encendido del robot.

### **3. Optimización de Red**
WPILib habilita por defecto el `LiveWindow`, un sistema de telemetría que envía datos de *todos* los sensores y motores a la red.

* **Problema:** Consume mucho ancho de banda y CPU, y raramente se usa en competencia oficial.
* **Solución:** `Optimizer.init()` desactiva `LiveWindow` completamente (`disableAllTelemetry`), ahorrando recursos valiosos para tu código de visión o control.

---

## **Métodos Útiles**

Además de las tareas automáticas, Optimizer ofrece utilidades estáticas accesibles desde cualquier parte del código.

| Método | Descripción |
| :--- | :--- |
| `Optimizer.getVoltage()` | Retorna el voltaje actual de la batería (shorthand para `RobotController`). |

```java
// Ejemplo de uso en un comando para proteger subsistemas
if (Optimizer.getVoltage() < 10.0) {
    System.out.println("¡Batería baja! Reduciendo velocidad del intake.");
    intake.setSpeed(0.5);
}
```