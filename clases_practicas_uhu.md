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


-- practica 4 Join


select *
from mf.telefono tlf inner join mf.cliente cli ON(tlf.cliente = cli.dni)


-- S2.1 Mostrar el código y coste de las tarifas junto con el nombre de la compañía que las ofrecen, de aquellas
-- tarifas cuya descripción indique que otras personas deben estar también en la misma compañía
select tar.coste, tar.tarifa, com.nombre
from mf.tarifa tar inner join mf.compañia com ON(tar.compañia = com.cif)
where tar.descripcion like '%en la compañía'


-- S2.2 Nombre y número de teléfonos de aquellos abonados con contrato que tienen tarifas inferiores a 0,20 €.
select tlf.numero, cli.nombre
from mf.telefono tlf inner join mf.tarifa tar using(tarifa, compañia)
inner join mf.cliente cli ON(tlf.cliente = cli.dni)
where tar.coste < 0.2 and tlf.tipo = 'C'



-- S2.3 Obtener el código de las tarifas, el nombre de las compañías, los números de teléfono y los puntos, de
-- aquellos teléfonos que se contrataron en el año 2006 y que hayan obtenido más de 200 puntos
select tlf.tarifa, com.nombre, tlf.puntos, tlf.numero
from mf.telefono tlf inner join mf.compañia com ON(tlf.compañia = com.cif)
where extract(year from f_contrato) = '2006' and tlf.puntos > 200


-- S2.4 Obtener los números de teléfono (origen y destino), así como el tipo de contrato, de los clientes que alguna
-- vez hablaron por teléfono entre las 8 y las 10 de la mañana
select cal.tf_origen, cal.tf_destino, tlf.tipo
from mf.llamada cal inner join mf.telefono tlf ON(cal.tf_origen = tlf.numero)
where extract(hour from fecha_hora) BETWEEN '8' and '9'



-- S2.5 Interesa conocer los nombres y números de teléfono de los clientes (origen y destino) que, perteneciendo
-- a compañías distintas, mantuvieron llamadas que superaron los 15 minutos. Se desea conocer, también, la
-- fecha y la hora de dichas llamadas así como la duración de esas llamadas
select cli_or.nombre,cal.tf_origen,cli_des.nombre, cal.tf_destino, cal.fecha_hora, cal.duracion
from mf.llamada cal inner join mf.telefono tlf on(cal.tf_origen = tlf.numero)
                    inner join mf.telefono tf_des on(cal.tf_destino = tf_des.numero)
                    inner join mf.cliente cli_or on(tlf.cliente = cli_or.dni)
                    inner join mf.cliente cli_des on(tf_des.cliente = cli_des.dni)
    where tlf.compañia <> tf_des.compañia and cal.duracion > 15



-- Practica 5 consultas anidadas

select *
from mf.telefono tlf
where tlf.compañia IN (select cif from mf.compañia where nombre ='Kietostar' or nombre = 'Petafón')

-- S3.1 Obtener la fecha (día-mes-año) en la que se realizó la llamada de mayor duración
select to_char(ll.fecha_hora, 'dd-mm-yyyy') as fecha
from mf.llamada ll
where ll.duracion >= ALL(select l2.duracion
                        from mf.llamada l2)


-- S3.2 Obtener el nombre de los abonados de la compañía ‘Aotra’ con el mismo tipo de tarifa que la del telefono “654123321”
select cli.nombre from mf.cliente cli inner join mf.telefono tlf ON(tlf.cliente = cli.dni)
where tlf.compañia In(select cif from mf.compañia where nombre = 'Aotra')
and tlf.tarifa = (select tarifa from mf.telefono where numero = '654123321')

-- S3.3 Mostrar, utilizando para ello una subcobsulta, el número de teléfono, fecha de contrato y tipo de los
-- abonados que han llamado a teléfonos de clientes de fuera de la provincia de La Coruña durante el mes de
-- octubre de 2006
select distinct tlf2.numero, tlf2.f_contrato, tlf2.tipo
from mf.llamada ll inner join mf.telefono tlf2 ON(ll.tf_origen = tlf2.numero)
where ll.tf_destino IN(select tlf.numero from mf.cliente cli inner join mf.telefono tlf ON(tlf.cliente = cli.dni) where cli.provincia <> 'La Coruña') 
and to_char(ll.fecha_hora, 'MM/YYYY' )= '10/2006'


--S3.4 Se necesita conocer el nombre de los clientes que tienen teléfonos con tarifa “dúo” pero no “autónomos”.
-- Utiliza subconsultas para obtener la solución.
select nombre
from mf.cliente
where dni IN(
    select tlf.cliente
    from mf.telefono tlf
    where tlf.tarifa = 'dúo'
)
and dni not IN(
select tlf.cliente
from mf.telefono tlf
where tlf.tarifa = 'autónomos'
)


-- S4.1 Utilizando consultas correlacionadas, mostrar el nombre de los abonados que han llamado el día ‘16/10/06’
select nombre
from mf.cliente cl inner join mf.telefono tf on(cl.dni = tf.cliente)
where exists(select *
from mf.llamada ll where to_char(ll.fecha_hora, 'dd-mm-yy') = '16-10-06' and ll.tf_origen = tf.numero)


-- S4.2 Utilizando consultas correlacionadas, obtener el nombre de los abonados que han realizado llamadas de menos de 1 minuto y medio
select nombre
from mf.cliente cl inner join mf.telefono tf on(cl.dni = tf.cliente)
where exists(select *
from mf.llamada ll
where tf.numero = ll.tf_origen and ll.duracion < 90)



-- S4.3 Utilizando consultas correlacionadas, obtener el nombre de los abonados de la compañía ‘KietoStar’ que no hicieron ninguna llamada el mes de septiembre
select nombre
from mf.cliente cl inner join mf.telefono tf ON(cl.dni = tf.cliente)
where tf.compañia = (select cif
from mf.compañia where nombre = 'Kietostar')
and not exists(select * from mf.llamada ll
where ll.tf_origen = tf.numero and extract(month from ll.fecha_hora) = '09')


-- S4.4 Utilizando consultas correlacionadas, mostrar todos los datos de los telefonos que hayan llamado al número 654234234 pero no al 666789789
select *
from mf.telefono tf 
where exists(select *
from mf.llamada ll
where tf.numero = ll.tf_origen and ll.tf_destino = 654234234) 
and not exists(select *
from mf.llamada ll where tf.numero = ll.tf_origen and  tf_destino = 666789789)


-- S4.5 Utilizando consultas correlacionadas, obtener el nombre y número de teléfono de los clientes de la compañía Kietostar que no han hecho llamadas a otros teléfonos de la misma compañía
select cl.nombre, tf.numero
from mf.cliente cl inner join mf.telefono tf ON(cl.dni = tf.cliente)
where exists(select *
from mf.compañia cm
where tf.compañia = cm.cif and cm.nombre = 'Kietostar')
and not exists(select *
from mf.llamada ll inner join mf.telefono tfd On(ll.tf_destino = tfd.numero)
where ll.tf_origen = tf.numero and tf.compañia = tfd.compañia)


-- Sesion 5
-- Ejercicio 1
select comp.nombre
from mf.llamada ll inner join mf.telefono tf on(ll.tf_origen = tf.numero)
inner join mf.compañia comp on(tf.compañia = comp.cif)
where to_char(ll.fecha_hora,'dd/mm/yy') = '16/10/06'
group by comp.nombre
having count(*) = (select MAX(count(*))
                    from mf.llamada llam inner join mf.telefono tlf on llam.tf_origen = tlf.numero
                    where to_char(llam.fecha_hora,'dd/mm/yy') = '16/10/06'
                    group by tlf.compañia)


-- Obtener los nombres distintos de los clientes que residen en la provincia de "Granada"
-- y que realizaron llamadas desde un teléfono asociado a la compañía "Kietostar" hacia un teléfono de la compañía "Aotra".
Select distinct cli.nombre
from mf.cliente cli 
inner join mf.telefono tfor on(cli.dni = tfor.cliente) 
inner join mf.llamada ll on(tfor.numero = ll.tf_origen)
inner join mf.compañia cmor on(cmor.cif = tfor.compañia)
inner join mf.telefono tfdes on(ll.tf_destino = tfdes.numero)
inner join mf.compañia cmdes on(cmdes.cif = tfdes.compañia)
where cli.provincia = 'Granada' 
and cmor.nombre = 'Kietostar'
and cmdes.nombre = 'Aotra'



```

[Pastebin](https://pastebin.com/Y9N31KWv)

[Ejercicios](https://github.com/JuanmaFranco/Ejercicios-SQL/blob/main/ejercicios/empleados.MD)
