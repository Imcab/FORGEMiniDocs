# SmartChooser

**SmartChooser** es un "wrapper" inteligente para el `SendableChooser` de WPILib.

Esta clase soluciona las molestias comunes del selector estándar, agregando una **API Fluida** (encadenamiento de métodos), validación de nulos y la capacidad de recuperar el **nombre** de la opción seleccionada, no solo el objeto.

---

## **Ejemplo de Implementación**

El uso más común es para el **Selector de Autónomo** en `RobotContainer`.

=== "RobotContainer.java"

    ```java
    package frc.robot;

    import com.stzteam.forgemini.io.SmartChooser;
    import edu.wpi.first.wpilibj2.command.Command;
    import edu.wpi.first.wpilibj2.command.WaitCommand;

    public class RobotContainer {

        // 1. Definir el Chooser con el tipo de objeto (Command)
        private final SmartChooser<Command> autoChooser;

        public RobotContainer() {
            // 2. Configurar opciones usando encadenamiento
            autoChooser = new SmartChooser<>("Auto Mode")
                .setDefault("Do Nothing", new WaitCommand(0))
                .add("Taxi Simple", new TaxiCommand())
                .add("Score & Balance", new BalanceCommand());
            
            // 3. ¡Importante! Publicar en el Dashboard
            autoChooser.publish();
        }

        public Command getAutonomousCommand() {
            // 4. Obtener la opción seleccionada
            // También puedes loguear el NOMBRE de la selección
            System.out.println("Auto seleccionado: " + autoChooser.getSelectedName());
            
            return autoChooser.get();
        }
    }
    ```

!!! tip "Encadenamiento de Métodos"
    Gracias a la API Fluida, no necesitas repetir `chooser.addOption(...)` diez veces. Simplemente encadena `.add()` después de `.setDefault()`.

---

## **Métodos Principales**

| Método | Descripción |
| :--- | :--- |
| `setDefault(String name, T value)` | Establece la opción por defecto (la que aparece seleccionada al inicio). **Debe llamarse antes que los `add`**. |
| `add(String name, T value)` | Agrega una opción extra al menú desplegable. |
| `publish()` | Envía el widget a **SmartDashboard**. Sin esto, no aparecerá en el Dashboard. |
| `get()` | Retorna el objeto (`T`) seleccionado actualmente. |
| `getSelectedName()` | Retorna el **texto** (String) de la opción seleccionada. Útil para imprimir en consola qué estrategia se eligió. |

!!! warning "Orden de Operaciones"
    Se recomienda llamar a `.setDefault()` **antes** de agregar otras opciones con `.add()`. Si no estableces un default, el Dashboard podría mostrar un comportamiento errático o vacío al inicio.