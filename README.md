## Prueba Tecnica Arquitectura

### Ejercicio 1 - Diseñe una arquitectura de tablas (tabla o tablas que considere oportunas) para almacenar la información anterior que permita responder a esas preguntas

```bash
 create table Ubicacion(
Id INT AUTO_INCREMENT PRIMARY KEY,
Vehiculo INT,
 Latitud DECIMAL(10, 6) NOT NULL,
    Longitud DECIMAL(10, 6) NOT NULL,
    Altitud DECIMAL(10, 2) NOT NULL,
    Fecha DATETIME NOT null,
    FOREIGN KEY (Vehiculo) REFERENCES Vehiculos(Id)

)

CREATE TABLE Vehiculos (
    Id INT AUTO_INCREMENT PRIMARY KEY,
    Matricula VARCHAR(20) NOT NULL,
    TipoCombustible ENUM('Gasolina', 'Electrico') NOT NULL

);

CREATE TABLE Sensores (
    Id INT AUTO_INCREMENT PRIMARY KEY,
    Vehiculo INT,
    ConsumoCombustible DECIMAL(10, 2),
    NivelCombustible INT,
    VoltajeBateria DECIMAL(6, 2),
    TiempoCarga INT,
    Fecha DATETIME NOT null,
    FOREIGN KEY (Vehiculo) REFERENCES Vehiculos(Id)
);
```

#### Realice las consultas SQL que respondan la información anterior

##### 1) Obtención de la ubicación de un vehículo en un rango de fechas

```
SELECT
    v.Matricula,
    u.Latitud,
    u.Longitud,
    u.Altitud,
    u.Fecha
FROM
    Vehiculos v
JOIN
    Ubicacion u ON v.Id = u.Vehiculo
WHERE
    v.Matricula = 'ABC123'
    AND u.Fecha BETWEEN '2023-12-20 00:00:00' AND '2023-12-20 11:00:00';
```

##### 2) Repostajes realizados por un vehículo en un rango de fechas (entendiendo que un repostaje

se produce cuando hay un incremento de combustible del 15%)

```
WITH Repostajes AS (
    SELECT
        s.*,
        LAG(s.NivelCombustible) OVER (PARTITION BY s.Vehiculo) AS NivelAnterior, (s.NivelCombustible - LAG(s.NivelCombustible) OVER (PARTITION BY s.Vehiculo ORDER BY s.Fecha)) as DiferenciaNivel
    from Sensores s
)


select
	r.Id,
    v.Matricula,
    r.Fecha AS FechaRepostaje,
    r.ConsumoCombustible,
    r.NivelCombustible,
    r.VoltajeBateria,
    r.TiempoCarga,
    r.NivelAnterior,
   r.DiferenciaNivel,((r.DiferenciaNivel *100) / r.NivelAnterior) as Porcentaje
FROM
    Vehiculos v
JOIN
    Repostajes r ON v.Id = r.Vehiculo

WHERE
   ((r.DiferenciaNivel *100) / r.NivelAnterior) > 15
    AND r.Fecha BETWEEN '2023-12-01 00:00:00' AND '2023-12-31 23:59:59'
   and v.Id = 1;
```

#### 3) Estado actual de cada vehículo: corresponde con la última lectura de cada vehículo (la de fecha más reciente). Esta consulta es la más importante de todas, el tiempo de respuesta debe ser inmediato

```
select  max(s.Id),  v.Matricula, s.NivelCombustible, s.NivelCombustible, s.VoltajeBateria, s.VoltajeBateria , s.TiempoCarga, max(s.Fecha) as lastDate
from vehiculos v
join sensores s on s.Vehiculo = v.Id
group by v.Id
order by s.Id;
```

### ¿Qué pasa si queremos cambiar la matrícula de un vehículo? ¿Qué alternativas tenemos y cuál son las ventajas e inconvenientes de cada una de ellas?

```http
  Solucion 1: Agregar una nueva columna llamada "Matricula Nueva" en la tabla Vehiculos
```

|                                   Ventajas                                    | Desventajas                                                            |
| :---------------------------------------------------------------------------: | ---------------------------------------------------------------------- |
|    `Mantiene un historial de cambios al conservar la matrícula anterior.`     | `Puede generar duplicidad de datos si no se maneja correctamente.`     |
| `Evita problemas de consistencia si la matrícula se utiliza en otras tablas.` | `Requiere ajustes en las consultas para considerar la nueva columna. ` |

```http
  Solucion 2: Crear una tabla de historial de matrículas, de manera que se vallan registrando las matriculas y el vehiculo cuando se realize un cambio de matriculo :
```

|                            Ventajas                             | Desventajas                                                         |
| :-------------------------------------------------------------: | ------------------------------------------------------------------- |
| `Mantiene un historial de todos los cambios en las matrículas.` | `Puede requerir cambios en las consultas para acceder al historial` |
|                 `Evita duplicidad de datos.. `                  | `Se necesita más espacio de almacenamiento.`                        |

#### Creeria que la segunda opciones es la mejor, ya que siempre se debe guardar el historial de las matriculas de cada vehiculo, aunque agregue un poco mas de complejidad y almacenamiento de datos no se vera afectado el rendimiento de la base de datos ya que las consultas que se realizaran seran puntuales a un vehiculo.

# Ejercicio 4

Teniendo en cuenta los factores como tiempo de respuesta, tolerancia a fallos y aislamiento entre modulos, la arquitectura ideal para este tipo de entornos es la de MicroServicios.
Esta arquitectura nos permite **desacoplar** los diferentes modulos, por lo tanto se puede crear un microservicio para cada modulo(_Modulo de informenes_, _Localizacion Gps_, _Modulo Tacografo_). Esto tambien nos permitira aplicar diferentes tecnologias en caso de caso de ser necesario para cada microservicio, aprovechando al maximo las ventajas que nos proporciona cada tecnologia BackEnd y Base de datos.

Al estar todos los modulos desacoplados, nos proporcionara **Tolerancia a Fallos**, ya que cada microservicio se puede alojar en contenedores diferentes con sus propieas Base de datos, asi que ni un microservicio da problemas no afectara a los demas.

Al estar cada modulo en un Microservicio tambien sera escalable, cada micro servicio podra crecer en funcionalidad ademas se pueden agregar mas modulos en esta arquitectura, asi que el crecimiento sera tanto vertical como horizonta.

Con respecto al **tiempo de respuesta**, esta se optimizara ya que la arquitectura de microservicios permite dividir la funcionalidad en servicios más pequeños y específicos. Esto significa que cada microservicio se centra en una tarea o conjunto de tareas específicas.
La división funcional facilita la optimización de cada microservicio para que realice su tarea de manera eficiente y responda rápidamente a las solicitudes relacionadas con esa funcionalidad específica. Ademas al desacoplar los servicios, puedes elegir las tecnologías y herramientas más adecuadas para cada microservicio. Esto incluye seleccionar bases de datos especializadas, lenguajes de programación eficientes y frameworks diseñados para tareas específicas.
