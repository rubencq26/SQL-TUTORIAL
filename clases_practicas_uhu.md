# Codigo clases practicas de la universidad
```sql
--Crear tablas
  CREATE TABLE llamada(
    tf_origen char(9),
    tf_destino char(9),
    fecha_hora timestamp,
    duracion number(5,0),
    CONSTRAINT llamada_pk PRIMARY KEY(tf_origen, fecha_hora),
    CONSTRAINT llamada_fk
    FOREIGN KEY(tf_origen) REFERENCES TELEFONO(numero),
    CONSTRAINT llmada2
    FOREIGN KEY(tf_destino) REFERENCES TELEFONO(numero),
    CONSTRAINT llamada_unica UNIQUE (tf_destino, fecha_hora));

--modificar tablas
  ALTER TABLE TARIFA
  DROP CONSTRAINT COMPAÑIA_FK;
  
  ALTER TABLE TARIFA
  ADD CONSTRAINT COMPAÑIA_FK FOREIGN KEY(COMPAÑIA) references COMPAÑIA(CIF) ON DELETE CASCADE;

  --Insertar en tablas

  INSERT INTO COMPAÑIA 
VALUES('A00000001', 'Kietostar', 'http://www.kietostar.com');

INSERT INTO COMPAÑIA
VALUES('B00000002','Aotra','http://www.aotra.com');

INSERT INTO TARIFA
VALUES('familiar','A00000001','la pareja también está en la compañía',0.15);

INSERT INTO TARIFA
VALUES('autónomos','B00000002','trabajador autónomo',0.12);

INSERT INTO TARIFA
VALUES('dúo','B00000002','la pareja también está en la compañía',0.15);

INSERT INTO COMPAÑIA
VALUES('A00000001','Petafón','http://www.petafón.com '); --No funciona porque se incumple   que el ID ya existe

-- CONSULTAS DE TABLAS
select * from mf.cliente

select nombre, ciudad from mf.cliente
WHERE DNI = '35000001P'

select *
from mf.cliente
where nombre like  'R%M%'

select *
from mf.cliente
where nombre like  '% M%'

select *
from mf.cliente
where nombre  like  'R% M% S%'

select *
from mf.cliente
order by f_nac

select nombre, direccion --2
from mf.cliente
where extract(year from f_nac) IN ('1973','1985') and cp like '15%' 
order by nombre asc, direccion desc

select tf_destino --3
from mf.llamada
where tf_origen = '666010101' and extract(year from fecha_hora) = '2006'

select tf_origen --4
from mf.llamada
where tf_destino = '666010101' and extract(hour from fecha_hora) between 10 and 12

select tarifa --5
from mf.telefono
where cliente like '%2%'  and tipo like 'C' and puntos between 10000 and 20000

select numero, tarifa --6
from mf.telefono
where extract(month from f_contrato) = '05' and tarifa not like 'joven'
and numero like '%9'
order by puntos desc

select tf_destino --7
from mf.llamada
where tf_origen = '654345345' and extract(month from fecha_hora) between '10' and '11'
and duracion > 250

select nombre --8
from mf.cliente
where extract(year from f_nac) between '1970' and '1985'
and provincia = 'Huelva'
order by ciudad asc, provincia desc

``` 
