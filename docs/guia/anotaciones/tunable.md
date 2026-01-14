# @Tunable (Input)

La anotación **`@Tunable`** se utiliza para **inyectar valores** desde el Dashboard hacia el Robot en tiempo real.

Se coloca sobre **campos (variables)** públicos de tu clase `IOSubsystem`. El sistema mantendrá esa variable sincronizada con NetworkTables automáticamente, permitiéndote modificar el comportamiento del robot en vivo.

!!! info "Sincronización Bidireccional"
    Si cambias el valor en el código, se actualiza en el Dashboard. Si cambias el valor en el Dashboard, se actualiza inmediatamente en la variable del código.

---

## **Sintaxis**

```java
@Tunable(key = "NombreEnDashboard")
public double miVariable = 0.0;
```

## **Parámetros**

| Parámetro | Tipo | Default | Descripción |
| :--- | :--- | :--- | :--- |
| **`key`** | `String` | `""` | El nombre con el que aparecerá el dato en NetworkTables. Si se deja vacío (`""`), la librería usará el nombre de la variable automáticamente. |

## **Ejemplos de Uso**

### **1. Sintonización de PID**
Este es el uso más común. Permite ajustar las constantes de un controlador PID mientras el robot está habilitado para encontrar la configuración perfecta rápidamente.

```java
// Cambia kP, kI, kD desde el Dashboard y el robot reaccionará al instante
@Tunable(key = "kP")
public double kP = 0.1;

@Tunable(key = "kI")
public double kI = 0.0;

@Tunable(key = "kD")
public double kD = 0.01;
```

### **2. Configuración de Puntos de Ajuste (Setpoints)**
Ajusta la velocidad deseada o la posición objetivo de un mecanismo para probar diferentes escenarios.

```java
@Tunable(key = "Target RPM")
public double targetRpm = 3000.0;

@Override
public void periodicLogic() {
    // El motor usará el nuevo valor de targetRpm inmediatamente
    controller.setReference(targetRpm);
}
```

### **3. Interruptores de Depuración**
Activa o desactiva logs o funciones específicas sin tener que cambiar el código.

```java
@Tunable(key = "Debug Mode")
public boolean debugEnabled = false;

@Override
public void periodicLogic() {
    if (debugEnabled) {
        System.out.println("Sensor Value: " + sensor.get());
    }
}
```

## **Tipos Soportados**

Para garantizar la seguridad en la inyección de datos,**`@Tunable`** soporta los siguientes tipos:

**Primitivos:**

- `double` (y `Double`)

- `boolean` (y `Boolean`)

- `String`

!!! warning "Inicialización Segura"
    Siempre inicializa tus variables @Tunable con un valor seguro (ej. public double kP = 0.0;).
    Si el Dashboard no está conectado al encender el robot, la variable mantendrá este valor inicial hasta que reciba una actualización de la red.