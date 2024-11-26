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


--PRACTICA 2 SESION 5
 
--Consulta 1
/*1*/       SELECT distinct co.nombre, count(*) num_llamadas
            FROM MF.llamada ll 
                inner join MF.telefono t on ll.tf_origen=t.numero
                inner join MF.compañia co on t.compañia=co.cif
            WHERE to_char(fecha_hora, 'dd/mm/yy') = '16/10/06' --Compañias que han hecho llamadas ese día
            GROUP BY co.nombre
 /*2*/      HAVING COUNT(*) =  (SELECT MAX(COUNT(*))
                                FROM MF.llamada ll inner join MF.telefono t on tf_origen=t.numero
                                WHERE to_char(fecha_hora, 'dd/mm/yy') = '16/10/06'
                                GROUP BY t.compañia); --En el having, filtramos para que devuelva la agrupacion 
                                                      --cuyo count se iguala al max valor de counts 
                                                      
--consulta 2
SELECT T.numero, C.nombre
FROM MF.CLIENTE C 
    INNER JOIN MF.TELEFONO T ON C.dni=T.cliente
WHERE NOT EXISTS (SELECT * -- Seleccionar los teléfonos que recibieron llamadas del 654345345 en octubre de 2006
                    FROM MF.LLAMADA L1
                    WHERE L1.tf_origen= '654345345' AND
                    to_char(L1.fecha_hora, 'mm/yy') = '10/06'
                    AND NOT EXISTS (SELECT * --Seleccionar todas las llamadas realizadas desde los teléfonos considerados y establecer la correlación entre subconsultas
                                    FROM MF.LLAMADA L2
                                    WHERE T.numero = L2.tf_origen AND L2.tf_destino = L1.tf_destino
                                    AND L2.tf_origen<> '654345345'));
                                    
--consulta 3
select cl.nombre as cliente ,c.nombre as compañia, sum(ll.duracion/60*ta.coste) as COSTE --multiplicamos porque el la duracion es en segundos, y el coste en minutos
from MF.cliente cl 
    inner join MF.telefono t on t.cliente=cl.dni
    inner join MF.tarifa ta on ta.tarifa=t.tarifa and ta.compañia=t.compañia
    inner join MF.compañia c on c.cif=ta.compañia
    inner join MF.llamada ll on ll.tf_origen=t.numero
group by cl.nombre,c.nombre
order by cliente DESC, compañia ASC;
 
 
 
 
 
--consulta 4
select cl.nombre as CLIENTE, sum (ll.duracion) AS DURACION
from MF.llamada ll 
    inner join MF.telefono t on ll.tf_origen=t.numero
    inner join MF.cliente cl on t.cliente=cl.dni
where ll.tf_origen in (select te.numero --llamadas realizadas por clientes de coruña
                    from MF.telefono te inner join MF.cliente ci on te.cliente=ci.dni
                    where ci.provincia='La Coruña')
AND ll.tf_destino IN (SELECT TEL.numero
                        from  mf.telefono tel inner join mf.cliente clte on tel.cliente = clte.dni 
                        where clte.provincia = 'Jaén')
group by cl.nombre;
 
--consulta 5
select ci.nombre, count(*) as numLlamadas
from MF.llamada ll 
    inner join MF.telefono t on ll.tf_origen=t.numero
    inner join MF.cliente ci on ci.dni=t.cliente
    group by ci.nombre
    having count(*) > 5;
 
--consulta 6
select cl.nombre as cliente ,avg(ta.coste) as media
from MF.cliente cl 
    inner join MF.telefono t on t.cliente=cl.dni
    inner join MF.tarifa ta using(tarifa, compañia)
group by cl.nombre
having avg(ta.coste)>= ALL (select avg(coste)
                            from MF.tarifa);
                            
--consulta 7
 
select cl.nombre as cliente, sum(ll.duracion/60*ta.coste) as costeLlamadas
from MF.llamada ll 
    inner join MF.telefono t on ll.tf_origen=t.numero
    inner join MF.tarifa ta using(tarifa, compañia)
    inner join MF.cliente cl on t.cliente=cl.dni
where ll.tf_destino in (select t.numero
                        from MF.telefono t inner join MF.compañia c on t.compañia=c.cif
                        where c.nombre='Kietostar')
group by cl.nombre
having sum(ll.duracion/60*ta.coste) < 100;


/*Listar los nombres de clientes, sus números de teléfono, y la compañía asociada a dichos teléfonos, siempre y cuando cumplan con las siguientes condiciones:

El cliente utiliza una tarifa denominada "dúo".
El cliente no ha realizado llamadas en el mes de octubre de 2006 hacia teléfonos que pertenezcan a la compañía "Petafón".*/

select cli.nombre, tel_o.numero, cia_o.nombre as compañia
from mf.cliente cli 
                inner join mf.telefono tel_o on (cli.dni=tel_o.cliente)
                inner join mf.compañia cia_o on (tel_o.compañia=cia_o.cif)
where exists (select * from mf.tarifa tar
                where tar.tarifa='dúo'
                and cia_o.cif=tar.compañia)
 and not exists (select * from mf.llamada ll 
                inner join mf.telefono tel_d on (ll.tf_destino=tel_d.numero)
                inner join mf.compañia cia_d on (tel_d.compañia=cia_d.cif)
                 where to_char(ll.fecha_hora, 'mm/yy')='10/06'
                 and cia_d.nombre='Petafón'
                 and tel_o.numero=ll.tf_origen);


-- Sesion 1 PL SQL
-- script hola nombre
set serveroutput on;

create or replace
procedure hola (nombre varchar) is
begin
    dbms_output.put_line('Hola ' || nombre);
    end;
-- llamar funcion
call hola('Ruben')


-- eje 1
create or replace
function facturacion(origen LLAMADA.tf_origen%TYPE, p_año INTEGER)
return float is
Importe_total number(10,2);
facturacionBaja EXCEPTION;
begin
    select TRUNC(SuM((ll.duracion * tar.coste)/60),2) INTO Importe_total
    from(telefono t inner join tarifa tar using(tarifa,compañia))
    inner join llamada ll on ll.tf_origen = t.numero
    where extract(year from ll.fecha_hora) = p_año
        and ll.tf_origen = origen
        group by ll.tf_origen;
        if(Importe_total < 1)then
            raise facturacionBaja;
            end if;
            return Importe_total;

exception
    when facturacionBaja then
        dbms_output.put_line('Facturacion demasiado baja!!');
        Return -1;
    when NO_DATA_FOUND then
        dbms_output.put_line('El telefono introducido no existe');
        return -1;
    when OTHERS then
        dbms_output.put_line('Ha ocurrido un error');
        return -1;
end facturacion;


call dbms_output.put_line( facturacion('654123321',2006));


create or replace
procedure LlamadaFacturacion(p_año INTEGER) is
cursor c_telefonos is
select distinct tf_origen from llamada
where extract(year from fecha_hora) = p_año; 

begin

dbms_output.put_line('Nº teléfono' || ' ' ||'Importe (en €)');
dbms_output.put_line('----------- -----------------');
for r_telefono IN c_telefonos LOOP
dbms_output.put_line(r_telefono.tf_origen || ' ' || facturacion(r_telefono.tf_origen, p_año));
end loop;


exception
when others  then
dbms_output.put_line('Ha ocurrido un error!!'); 

end;

call LlamadaFacturacion(2006);


```


[Pastebin](https://pastebin.com/Y9N31KWv)

[Pastebin 2](https://pastebin.com/Zwp1KGKX)

contraseña: lentejasconchorizo

[Ejercicios](https://github.com/JuanmaFranco/Ejercicios-SQL/blob/main/ejercicios/empleados.MD)
