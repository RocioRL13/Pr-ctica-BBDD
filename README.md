# Pr-ctica-BBDD
DDL – Definición de datos

CREATE TABLE empleados (
  id INT PRIMARY KEY,
  nombre VARCHAR(50),
  edad INT,
  salario DECIMAL(10,2)
);
ALTER TABLE empleados
ADD COLUMN direccion VARCHAR(100);
CREATE TABLE departamentos (
  id_dep INT PRIMARY KEY,
  nombre VARCHAR(50),
  id_empleado INT,
  CONSTRAINT fk_departamentos_empleado
    FOREIGN KEY (id_empleado) REFERENCES empleados(id));
    ALTER TABLE empleados
ADD COLUMN id_dep INT;

ALTER TABLE empleados
ADD CONSTRAINT fk_empleados_departamento
FOREIGN KEY (id_dep) REFERENCES departamentos(id_dep);

INSERT INTO empleados (id, nombre, edad, salario, direccion)
VALUES
(1, 'Rocío Rivera Lopez', 32, 1600.00, 'Calle A 123'),
(2, 'Raúl Martin Guerrero', 28, 1800.00, 'Calle B 45'),
(3, 'Claudia Grego Rodríguez', 35, 1600.00, 'Calle C 9');
INSERT INTO departamentos (id_dep, nombre, id_empleado)
VALUES
(10, 'IT', 2),
(20, 'Marketing', 1);
UPDATE empleados SET id_dep = 10 WHERE id = 2;
UPDATE empleados SET id_dep = 20 WHERE id IN (1,3);

UPDATE empleados
SET salario = 1900.00
WHERE id = 2;

DELETE FROM empleados
WHERE salario < 1500.00;

SELECT * FROM empleados;

SELECT nombre, salario
FROM empleados
WHERE edad > 30
ORDER BY salario DESC;

SELECT edad, COUNT(*) AS num_empleados
FROM empleados
GROUP BY edad;

SELECT edad, AVG(salario) AS salario_medio
FROM empleados
GROUP BY edad
HAVING AVG(salario) > 1800.00;

SELECT e.nombre AS empleado, d.nombre AS departamento
FROM empleados e
JOIN departamentos d ON e.id_dep = d.id_dep;

CREATE USER 'usuario1'@'localhost' IDENTIFIED BY 'Usuario1_Clave!';

GRANT SELECT, INSERT ON empresa_technova.empleados TO 'usuario1'@'localhost';

REVOKE INSERT ON empresa_technova.empleados FROM 'usuario1'@'localhost';

START TRANSACTION;

INSERT INTO empleados (id, nombre, edad, salario, direccion, id_dep)
VALUES (4, 'Nuevo Empleado', 25, 1700.00, 'Calle Nueva 1', 10);

SAVEPOINT sp1;

UPDATE empleados
SET salario = 2200.00
WHERE id = 4;

ROLLBACK TO sp1;

COMMIT;

CREATE OR REPLACE VIEW vista_empleados_activos AS
SELECT
  e.id,
  e.nombre,
  e.edad,
  e.salario,
  d.nombre AS departamento
FROM empleados e
JOIN departamentos d ON e.id_dep = d.id_dep
WHERE e.salario >= 1500.00;

CREATE OR REPLACE VIEW vista_resumen_salarios AS
SELECT
  edad,
  COUNT(*) AS total_empleados,
  AVG(salario) AS salario_medio
FROM empleados
GROUP BY edad;

SELECT *
FROM vista_empleados_activos
ORDER BY salario DESC;

SELECT *
FROM vista_resumen_salarios
WHERE salario_medio > 2000.00;

CREATE ROLE 'rol_consulta';
CREATE ROLE 'rol_editor_empleados';

GRANT SELECT ON empresa_technova.vista_empleados_activos TO 'rol_consulta';
GRANT SELECT ON empresa_technova.vista_resumen_salarios TO 'rol_consulta';
GRANT SELECT ON empresa_technova.empleados TO 'rol_consulta';
GRANT SELECT ON empresa_technova.departamentos TO 'rol_consulta';

GRANT SELECT, INSERT, UPDATE ON empresa_technova.empleados TO 'rol_editor_empleados';

GRANT 'rol_consulta' TO 'usuario1'@'localhost';
SET DEFAULT ROLE 'rol_consulta' TO 'usuario1'@'localhost';

CREATE USER 'usuario2'@'localhost' IDENTIFIED BY 'Usuario2_Clave!';

GRANT 'rol_editor_empleados' TO 'usuario2'@'localhost';

SET DEFAULT ROLE 'rol_editor_empleados' TO 'usuario2'@'localhost';

SELECT * FROM empresa_technova.vista_empleados_activos;

INSERT INTO empresa_technova.empleados (id, nombre, edad, salario)
VALUES (99, 'Prueba', 22, 1600.00);

INSERT INTO empresa_technova.empleados (id, nombre, edad, salario, direccion, id_dep)
VALUES (100, 'Empleado Prueba', 30, 1750.00, 'Calle Test 10', 20);

UPDATE empresa_technova.empleados
SET salario = 1850.00
WHERE id = 100;

SELECT * FROM empresa_technova.vista_empleados_activos;

REVOKE 'rol_editor_empleados' FROM 'usuario2'@'localhost';
REVOKE 'rol_consulta' FROM 'usuario1'@'localhost';

REVOKE SELECT ON empresa_technova.vista_empleados_activos FROM 'rol_consulta';
REVOKE SELECT ON empresa_technova.vista_resumen_salarios FROM 'rol_consulta';
REVOKE SELECT ON empresa_technova.empleados FROM 'rol_consulta';
REVOKE SELECT ON empresa_technova.departamentos FROM 'rol_consulta';

REVOKE SELECT, INSERT, UPDATE ON empresa_technova.empleados FROM 'rol_editor_empleados';

DROP ROLE 'rol_consulta';
DROP ROLE 'rol_editor_empleados';

CREATE TABLE empleados_salario_log (
  id_log INT PRIMARY KEY AUTO_INCREMENT,
  id_empleado INT,
  salario_anterior DECIMAL(10,2),
  salario_nuevo DECIMAL(10,2),
  fecha_cambio TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  usuario_bd VARCHAR(50)
);

CREATE TRIGGER trg_auditoria_salario
AFTER UPDATE ON empleados
FOR EACH ROW
BEGIN
  IF OLD.salario <> NEW.salario THEN
    INSERT INTO empleados_salario_log
      (id_empleado, salario_anterior, salario_nuevo, fecha_cambio, usuario_bd)
    VALUES
      (NEW.id, OLD.salario, NEW.salario, NOW(), CURRENT_USER());
  END IF;
END$$

DELIMITER ;
DELIMITER $$

DROP TRIGGER IF EXISTS trg_auditoria_salario $$

CREATE TRIGGER trg_auditoria_salario
AFTER UPDATE ON empleados
FOR EACH ROW
BEGIN
  IF OLD.salario <> NEW.salario THEN
    INSERT INTO empleados_salario_log
      (id_empleado, salario_anterior, salario_nuevo, fecha_cambio, usuario_bd)
    VALUES
      (NEW.id, OLD.salario, NEW.salario, NOW(), CURRENT_USER());
  END IF;
END $$

DELIMITER ;

UPDATE empleados
SET salario = salario + 50
WHERE id = 1;

SELECT * FROM empleados_salario_log ORDER BY id_log DESC;

DELIMITER $$

DROP TRIGGER IF EXISTS trg_salario_minimo $$

CREATE TRIGGER trg_salario_minimo
BEFORE INSERT ON empleados
FOR EACH ROW
BEGIN
  IF NEW.salario < 1000 THEN
    SET NEW.salario = 1000;
  END IF;
END $$

DELIMITER ;

INSERT INTO empleados (id, nombre, edad, salario, direccion, id_dep)
VALUES (200, 'Empleado Salario Bajo', 24, 800, 'Calle Prueba', 20);

SELECT id, nombre, salario
FROM empleados
WHERE id = 200;

CREATE TABLE empleados_borrados (
  id INT,
  nombre VARCHAR(50),
  edad INT,
  salario DECIMAL(10,2),
  fecha_borrado TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

DELIMITER $$

DROP TRIGGER IF EXISTS trg_empleados_before_delete $$

CREATE TRIGGER trg_empleados_before_delete
BEFORE DELETE ON empleados
FOR EACH ROW
BEGIN
  INSERT INTO empleados_borrados
    (id, nombre, edad, salario, fecha_borrado)
  VALUES
    (OLD.id, OLD.nombre, OLD.edad, OLD.salario, NOW());
END $$

DELIMITER ;

DELETE FROM empleados
WHERE id = 200;

SELECT *
FROM empleados_borrados
ORDER BY fecha_borrado DESC;

SELECT edad, COUNT(*) AS total
FROM empleados
GROUP BY edad;

CREATE INDEX idx_empleados_edad
ON empleados (edad);

SELECT edad, COUNT(*) AS total
FROM empleados
GROUP BY edad;

CREATE INDEX idx_empleados_salario
ON empleados (salario);

SELECT id, nombre, salario
FROM empleados
WHERE salario BETWEEN 1500 AND 2500;

INSERT INTO departamentos (id_dep, nombre, id_empleado)
VALUES (999, 'Marketing', 1);

CREATE INDEX idx_empleados_edad_salario
ON empleados (edad, salario);

SELECT id, nombre, edad, salario
FROM empleados
WHERE edad = 30
ORDER BY salario;
