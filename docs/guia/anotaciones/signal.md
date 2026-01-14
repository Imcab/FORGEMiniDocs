# @Signal (Output)

La anotación **`@Signal`** se utiliza para **publicar datos** desde el Robot hacia el Dashboard (NetworkTables).

Se coloca sobre **métodos** (generalmente getters) dentro de una clase que extienda de `IOSubsystem`. El sistema llamará a este método automáticamente en cada ciclo para obtener el valor más reciente y enviarlo a la red.

---

## **Sintaxis**

```java
@Signal(key = "NombreEnDashboard", onChange = false, slowScale = 1)
public tipoDeDato miMetodo() {
    return variable;
}
```

## **Parámetros**

| Parámetro | Tipo | Default | Descripción |
| :--- | :--- | :--- | :--- |
| **`key`** | `String` | `""` | El nombre con el que aparecerá el dato en NetworkTables. Si se deja vacío (`""`), la librería usará el nombre del método automáticamente. |
| **`onChange`** | `boolean` | `false` | Si es `true`, el dato solo se enviará cuando su valor cambie respecto al ciclo anterior. Ideal para ahorrar ancho de banda en estados booleanos. |
| **`slowScale`** | `int` | `1` | Define la frecuencia de actualización en ciclos. <br>• `1` = Actualiza cada ciclo (20ms). <br>• `50` = Actualiza cada 50 ciclos (1 segundo). |

---

## **Ejemplos de Uso**

### **1. Publicación Básica**
Esta es la forma más común. El valor se actualiza en cada ciclo del robot (normalmente cada 20ms).

```java
// Se publicará en la tabla como "Battery Voltage"
@Signal(key = "Battery Voltage")
public double getVoltage() {
    return RobotController.getBatteryVoltage();
}
```

### **2. Filtrado por Cambio (`onChange`)**
Útil para valores que se mantienen constantes por mucho tiempo, como un sensor de límite o el estado de "listo" de un mecanismo.

```java
// Solo envía datos a la red cuando el valor cambia (True -> False o viceversa)
@Signal(key = "Is Shooter Ready", onChange = true)
public boolean isReady() {
    return currentRpm > targetRpm;
}
```

### **3. Actualización Lenta (`slowScale`)**
Ideal para datos de diagnóstico, temperaturas o contadores que no necesitas ver en tiempo real absoluto. Esto reduce drásticamente el uso de CPU.

```java
// Actualiza una vez por segundo (50 ciclos * 20ms = 1000ms)
@Signal(key = "Motor Temp", slowScale = 50)
public double getTemp() {
    return motor.getTemperature();
}
```

---

## **Objetos Complejos (Structs)**

**ForgeMini** soporta nativamente la publicación de objetos complejos que implementen el protocolo Struct de WPILib.

No necesitas configuración extra; simplemente retorna el objeto y la librería usará **Magic Structs** para serializarlo eficientemente.

### **Ejemplo con Pose2d**
Este método publicará la posición del robot de una forma que herramientas como **AdvantageScope** pueden visualizar en 3D automáticamente.

```java
@Signal(key = "Robot Pose")
public Pose2d getPose() {
    // Retorna directamente el objeto Pose2d
    return m_odometry.getPoseMeters();
}
```

### **Ejemplo con ChassisSpeeds**
```java
@Signal(key = "Desired Speeds")
public ChassisSpeeds getSpeeds() {
    return m_kinematics.toChassisSpeeds(m_moduleStates);
}
```

---

## **Tipos Soportados**

La anotación `@Signal` puede colocarse en métodos que retornen cualquiera de los siguientes tipos:

1.  **Primitivos:** `double`, `boolean`, `String`.
2.  **Wrappers:** `Double`, `Boolean`.
3.  **Structs:** `Pose2d`, `Rotation2d`, `Translation2d`, `ChassisSpeeds`, `SwerveModuleState`, `Twist2d`, y cualquier clase custom que tenga un campo `public static final Struct<T> struct`.