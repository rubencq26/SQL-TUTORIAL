# SQL
## Tablas de bases de datos
Las bases de datos relacionales suelen estar compuestas por una o varias tablas 

![image](https://github.com/user-attachments/assets/39559fba-c7da-49a1-88ed-612280d23303)

**Nota: SQL no distingue entre mayusculas**
## Comandos mas importantes de las bases de datos

<ul>
  <li><code class="w3-codespan">SELECT</code> - extrae datos de una base de datos</li>
  <li><code class="w3-codespan">UPDATE</code> - actualiza datos en una base de datos</li>
  <li><code class="w3-codespan">DELETE</code> - elimina datos de una base de datos</li>
  <li><code class="w3-codespan">INSERT INTO</code> - inserta nuevos datos en una base de datos</li>
  <li><code class="w3-codespan">CREATE DATABASE</code> - crea una nueva base de datos</li>
  <li><code class="w3-codespan">ALTER DATABASE</code> - modifica una base de datos</li>
  <li><code class="w3-codespan">CREATE TABLE</code> - crea una nueva tabla</li>
  <li><code class="w3-codespan">ALTER TABLE</code> - modifica una tablas</li>
  <li><code class="w3-codespan">DROP TABLE</code> - elimina una tabla</li>
  <li><code class="w3-codespan">CREATE INDEX</code> - crea un índice (clave de búsqueda)</li>
  <li><code class="w3-codespan">DROP INDEX</code> - elimina un índice</li>
</ul>

## Seleccion en SQL

Sintaxis:
```sql
SELECT column1, column2, ...
FROM table_name;
```
### Seleccionar todas las columnas de la base de datos

```sql
SELECT * FROM Customers;
```
### Seleccionar todos los elementos pero sin repetir ningun elemento

```sql
SELECT DISTINCT column1, column2, ...
FROM table_name;
```

### Seleccion para devolver el numero de los diferentes elementos sin repetir (COUNT DISTINCT)

```sql
SELECT COUNT(DISTINCT Country) FROM Customers;
```

### Seleccion con condición WHERE

Se utiliza para extraer registros con una condicion especifica

```sql
SELECT * FROM Customers
WHERE Country='Mexico';
```
### Operadores de Seleccion

<table class="ws-table-all notranslate">
  <tbody><tr>
    <th style="width:20%">Operator</th>
    <th style="width:70%">Description</th>
    
  </tr>
  <tr>
    <td>=</td>
    <td>Equal</td>
    
  </tr>
  <tr>
    <td>&gt;</td>
    <td>Greater than</td>
 
  </tr>
  <tr>
    <td>&lt;</td>
    <td>Less than</td>
 
  </tr>
  <tr>
    <td>&gt;=</td>
    <td>Greater than or equal</td>
 
  </tr>
  <tr>
    <td>&lt;=</td>
    <td>Less than or equal</td>
  
  </tr>
  <tr>
    <td>&lt;&gt;</td>
    <td>Not equal. <b>Note:</b> In some versions of SQL this operator may be written as !=</td>
 
  </tr>
  <tr>
    <td>BETWEEN</td>
    <td>Between a certain range</td>
  
  </tr>
  <tr>
    <td>LIKE</td>
    <td>Buscar un Patron</td>
  
  </tr>
  <tr>
    <td>IN</td>
    <td>Para especificar multiples valores posibles en una columna</td>
 
  </tr>
</tbody></table>

### Seleccion Ordenada

Sintaxis:
```sql
SELECT column1, column2, ...
FROM table_name
ORDER BY column1, column2, ... ASC|DESC;
```
Para cadenas lo ordena alfabeticamente

#### Ordenar por varias columnas

Esto es util por si dos tuplas tienen un mismo elemento para que las desempate un segundo elemento
```sql
SELECT * FROM Customers
ORDER BY Country, CustomerName;
```

### Operador AND

Sintaxis:
```sql
SELECT column1, column2, ...
FROM table_name
WHERE condition1 AND condition2 AND condition3 ...;
```

Todas las condiciones deben ser verdaderas

### Operador OR

Sintaxis:
```sql
SELECT column1, column2, ...
FROM table_name
WHERE condition1 OR condition2 OR condition3 ...;
```

Almenos una condicion debe ser cierta

### Operador NOT

Sintaxis:
```sql
SELECT column1, column2, ...
FROM table_name
WHERE NOT condition;
```

