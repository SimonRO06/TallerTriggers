# Taller TRIGGERS

## Caso 1: Control de Stock de Productos

```sql
CREATE TABLE productos (
    id INT PRIMARY KEY AUTO_INCREMENT,
    nombre VARCHAR(50),
    stock INT
);

CREATE TABLE ventas (
    id INT PRIMARY KEY AUTO_INCREMENT,
    id_producto INT,
    cantidad INT,
    FOREIGN KEY (id_producto) REFERENCES productos(id)
);

DELIMITER //
CREATE TRIGGER revisar_stock
BEFORE INSERT ON ventas
FOR EACH ROW
BEGIN
	DECLARE stock_actual INT;
	SELECT stock INTO stock_actual
	FROM productos
	WHERE id = NEW.id_producto;
	IF NEW.cantidad > stock_actual THEN
		SIGNAL SQLSTATE '45000'
		SET MESSAGE_TEXT = 'El stock disponible es inferior a la cantidad';
	END IF;
END //
DELIMITER ;

INSERT INTO productos (nombre, stock) VALUES ('Teclado', 10);
INSERT INTO ventas (id_producto, cantidad) VALUES (1, 15);
```

## Caso 2: Registro Automático de Cambios en Salarios

```sql
CREATE TABLE empleados (
    id INT PRIMARY KEY AUTO_INCREMENT,
    nombre VARCHAR(50),
    salario DECIMAL(10,2)
);

CREATE TABLE historial_salarios (
    id INT PRIMARY KEY AUTO_INCREMENT,
    id_empleado INT,
    salario_anterior DECIMAL(10,2),
    salario_nuevo DECIMAL(10,2),
    fecha TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (id_empleado) REFERENCES empleados(id)
);

DELIMITER //
CREATE TRIGGER before_salario_update
BEFORE UPDATE ON empleados
FOR EACH ROW
BEGIN
    INSERT INTO historial_salarios (id_empleado, salario_anterior, salario_nuevo)
    VALUES (OLD.id, OLD.salario, NEW.salario);
END //
DELIMITER ;

INSERT INTO empleados (nombre, salario) VALUES ('Carlos López', 2500.00);
UPDATE empleados SET salario = 3000.00 WHERE id = 1;
SELECT * FROM historial_salarios;
```

## Caso 3: Registro de Eliminaciones en Auditoría

```sql
CREATE TABLE clientes (
    id INT PRIMARY KEY AUTO_INCREMENT,
    nombre VARCHAR(50),
    email VARCHAR(50)
);

CREATE TABLE clientes_auditoria (
    id INT PRIMARY KEY AUTO_INCREMENT,
    id_cliente INT,
    nombre VARCHAR(50),
    email VARCHAR(50),
    fecha_eliminacion TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

DELIMITER //
CREATE TRIGGER registrar_eliminacion
AFTER DELETE ON clientes
FOR EACH ROW
BEGIN
	INSERT INTO clientes_auditoria (id_cliente, nombre, email)
	VALUES (OLD.id, OLD.nombre, OLD.email);
END //
DELIMITER ;

INSERT INTO clientes (nombre, email) VALUES ('Simon Rubiano', 'simon@gmail.com');
DELETE FROM clientes WHERE id = 1;
SELECT * FROM clientes_auditoria;
```

## Caso 4: Restricción de Eliminación de Pedidos Pendientes

```sql
CREATE TABLE pedidos (
    id INT PRIMARY KEY AUTO_INCREMENT,
    cliente VARCHAR(100),
    estado ENUM('pendiente', 'completado')
);

DELIMITER //
CREATE TRIGGER verificar_eliminacion_pedido
BEFORE DELETE ON pedidos
FOR EACH ROW
BEGIN
	IF OLD.estado = 'pendiente' THEN
		SIGNAL SQLSTATE '45000'
		SET MESSAGE_TEXT = 'No se puede eliminar un pedido pendiente';
	END IF;
END //
DELIMITER ;

INSERT INTO pedidos (cliente, estado) VALUES ('Simon Rubiano', 'pendiente');
DELETE FROM pedidos WHERE id = 1;
```

