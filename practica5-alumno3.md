Alumno 3:

## ORACLE:

**1. Muestra los objetos a los que pertenecen las extensiones del tablespace TS2 (creado por Alumno 2) y el tamaño de cada una de ellas.**

El tablespace TS2, del alumno 2 es el que vemos a continuación:

	Create tablespace TS2
	Datafile '/home/oracle/TS2.dbf'
	size 1M,
	'/home/oracle/TS2-2.dbf'
	size 1M
	autoextend off;
	
	Create table Pruebon
	(
	Prueba char(2000),
	Prueba2 char(2000),
	Prueba3 char(2000)
	)
	Storage
	(
		Initial 10K
	)
	tablespace TS2;



Realizamos una consulta para ver la información como espacio que ocupa, comprobar la ubicación, nombre y tipo del segmento :


![](https://github.com/alvarocn/practica5-almacenamiento-alumno3/blob/master/imagenes/1.png)


	SQL> select file_name,tablespace_name
	from dba_data_files
	where tablespace_name='TS2';  2    3  

	FILE_NAME
	--------------------------------------------------------------------------------
	TABLESPACE_NAME
	------------------------------
	/home/oracle/TS2.dbf
	TS2

	/home/oracle/TS2-2.dbf
	TS2


	SQL> 
	



	SQL> select OWNER, SEGMENT_NAME, SEGMENT_TYPE, TABLESPACE_NAME,BYTES
	from dba_extents
	where tablespace_name='TS2';  2    3  

	OWNER
	--------------------------------------------------------------------------------
	SEGMENT_NAME
	--------------------------------------------------------------------------------
	SEGMENT_TYPE	   TABLESPACE_NAME		       BYTES
	------------------ ------------------------------ ----------
	SYS
	PRUEBON
	TABLE		   TS2				       65536


	SQL> 

![](https://github.com/alvarocn/practica5-almacenamiento-alumno3/blob/master/imagenes/2.png)


![](https://github.com/alvarocn/practica5-almacenamiento-alumno3/blob/master/imagenes/3.png)


**2. Borra la tabla que está llenando TS2 consiguiendo quevuelvan a existir extensiones libres. Añade después otro fichero de datos a TS2.**


	SQL> select OWNER, SEGMENT_NAME, SEGMENT_TYPE, TABLESPACE_NAME,BYTES
	from dba_extents
	where tablespace_name='TS2';  2    3  

	OWNER
	--------------------------------------------------------------------------------
	SEGMENT_NAME
	--------------------------------------------------------------------------------
	SEGMENT_TYPE	   TABLESPACE_NAME		       BYTES
	------------------ ------------------------------ ----------
	SYS
	PRUEBON
	TABLE		   TS2				       65536


	SQL>
	
	
	SQL> spool drop.sql
	select 'DROP TABLE '||owner||'.'||segment_name||';'
	from dba_segments
	where segment_type='TABLE' and segment_name in (select table_name
	 						from dba_all_tables
							where tablespace_name = 'TS2');
	spool off
	@dropSQL>   2    3    4    5  

	'DROPTABLE'||OWNER||'.'||SEGMENT_NAME||';'
	--------------------------------------------------------------------------------
	DROP TABLE SYS.PRUEBON;
	
	Tabla borrada.

	
	
	SQL> select OWNER, SEGMENT_NAME, SEGMENT_TYPE, TABLESPACE_NAME,BYTES
	from dba_extents
	where tablespace_name='TS2';  2    3  

	ninguna fila seleccionada

	SQL> spool drop.sql
	select 'DROP TABLE '||owner||'.'||segment_name||';'
	from dba_segments
	where segment_type='TABLE' and segment_name in (select table_name
	 						from dba_all_tables
							where tablespace_name = 'TS2');SQL>   2    3    4    5  
	
	ninguna fila seleccionada

	SQL> 

	


![](https://github.com/alvarocn/practica5-almacenamiento-alumno3/blob/master/imagenes/55.png)

Comprobación de que se ha borrado.

	select 'DROP TABLE '||owner||'.'||segment_name||';'
	from dba_segments
	where segment_type='TABLE' and segment_name in (select table_name
							from dba_all_tables
							where tablespace_name = 'TS2')
							and bytes = (select max(bytes)
								     from dba_segments
								     where tablespace_name = 'TS2');
	
	ninguna fila seleccionada
	
	SQL> 

![](https://github.com/alvarocn/practica5-almacenamiento-alumno3/blob/master/imagenes/45.png)

Añadimos de nuevo otro fichero a TS2:

	**alter tablespace TS2 add datafile '/home/oracle/TS2.dbf' size 10M;**
	**alter tablespace TS2 add datafile '/opt/oracle/product/12.2.0.1/dbhome_1/dbs/c:fichero3-TS2.dbf' size 10M;**


	SQL> select file_name,tablespace_name
	from dba_data_files
	where tablespace_name='TS2';  2    3  
	
	FILE_NAME
	--------------------------------------------------------------------------------
	TABLESPACE_NAME
	------------------------------
	/home/oracle/TS2.dbf
	TS2
	
	/home/oracle/TS2-2.dbf
	TS2
	
	/opt/oracle/product/12.2.0.1/dbhome_1/dbs/c:fichero3-TS2.dbf
	TS2
	
	
	SQL>


**3. Crea el tablespace TS3 gestionado localmente con un tamaño de extension uniforme de 128K y un fichero de datos asociado. Cambia la ubicación del fichero de datos y modifica la base de datos para que pueda acceder al mismo. Crea en TS3 dos tablas e inserta registros en las mismas. Comprueba que segmentos tiene TS3, qué extensiones tiene cada uno de ellos y en qué ficheros se encuentran.**



Creamos el tablespaces TS3 y el fichero como nos pide:

	SQL> CREATE TABLESPACE TS3 DATAFILE '/home/oracle/TS3.dbf' SIZE 20M EXTENT MANAGEMENT LOCAL UNIFORM SIZE 128K;
	
	Tablespace creado.

Comprobamos que se ha creado:


	SQL> select file_name,tablespace_name
	from dba_data_files
	where tablespace_name='TS3';  2    3  
	
	FILE_NAME
	--------------------------------------------------------------------------------
	TABLESPACE_NAME
	------------------------------
	/home/oracle/TS3.dbf
	TS3
	
	
	SQL> 

![](https://github.com/alvarocn/practica5-almacenamiento-alumno3/blob/master/imagenes/6.png)


Para poder mover un fichero, el cual está en uso tenemos que ponerlo en "offline":

	"ALTER TABLESPACE TS3 OFFLINE ;"

A continuación cambiamos la ubicacion del fichero:

	"alter tablespace TS3 rename datafile '/home/oracle/TS3.dbf' to '/home/oracle/practica5/TS3.dbf';"


Y activamos de nuevo el tablespace.

	"ALTER TABLESPACE TS3 ONLINE;"


	SQL> select file_name,tablespace_name
	  2  	from dba_data_files
		where tablespace_name='TS3';   3
	
	FILE_NAME
	--------------------------------------------------------------------------------
	TABLESPACE_NAME
	------------------------------
	/home/oracle/practica5/TS3.dbf
	TS3
	
	
	SQL> 

![](https://github.com/alvarocn/practica5-almacenamiento-alumno3/blob/master/imagenes/7.png)

	Creamos las tablas e insertamos registros:
	
	SQL> create table interpretes
	(
	id_interprete varchar2(10),
	nombre_interprete varchar2(50),
	constraint pk_interpretes primary key (id_interprete),
	constraint contar check(length(id_interprete)>='5'))tablespace TS3;
	  2    3    4    5    6  

	Tabla creada.
	
	SQL> SQL> insert into interpretes values ('123456','MAURICE ANDRE');
	insert into interpretes values ('1er456','CLAUDIO ARRAU');
	insert into interpretes values ('654tgb','DANIEL BARENBOIM');
	insert into interpretes values ('543frtyh','YURI BASHMET');
	insert into interpretes values ('12654f','HEINZ HOLLIGER');
	insert into interpretes values ('qazwsxedc','VLADIMIR HOROWITZ');
	insert into interpretes values ('1mkonji','YEHUDI MENUHIN');
	insert into interpretes values ('1234567','MAURICE ANDRE');
	insert into interpretes values ('123345','DAVID OISTRAKH');
	insert into interpretes values ('123321','ITZHAK PERLMAN');
	insert into interpretes values ('123565','MSTISLAV ROSTROPOVICH');
	insert into interpretes values ('123422','Marc-André Hamelin ');
	insert into interpretes values ('1234777','PIERRE RAMPAL');	
	insert into interpretes values ('124223','RUBEN FERNANDEZ');
	insert into interpretes values ('12bgtyhn','Martha Argerich');
	1 fila creada.
	
	SQL> 
	1 fila creada.
	
	SQL> 
	1 fila creada.
		
	SQL> 
	


	SQL> create table solistas
	(
	IDinterprete_solista varchar2(10),
	nombre_artistico varchar2(50),
	instrumento varchar2(100),
	constraint pk_solistas primary key (IDinterprete_solista),
	constraint solistas_interpretes foreign key (IDinterprete_solista) references 		interpretes(id_interprete))tablespace TS3;

	  2    3    4    5    6    7  
	Tabla creada.

	SQL> SQL> 
	SQL> insert into solistas values ('124223','RUBEN FERNANDEZ','flauta');
	insert into solistas values ('123456','MAURICE ANDRE','trompeta');
	insert into solistas values ('1er456','CLAUDIO ARRAU','piano');
	insert into solistas values ('654tgb','BARENBOIM','piano');
	insert into solistas values ('543frtyh','YURI BASHMET','violin');
	insert into solistas values ('12654f','HEINZ HOLLIGER','oboe');
	insert into solistas values ('qazwsxedc','HOROWITZ','piano');
	insert into solistas values ('1mkonji','MENUHIN','violin');
	insert into solistas values ('1234567','PIERRE RAMPAL','flauta');
	insert into solistas values ('123345','OISTRAKH','violin');
	

	1 fila creada.
	
	SQL> 
	1 fila creada.
	


Comprobamos los segmentos, extensiones y la ubicación que se encuentra:


	SQL> select de.segment_name,de.extent_id,df.file_name,de.file_id
	from dba_data_files  df, dba_extents de
	where de.file_id = df.file_id
	and de.tablespace_name = 'TS3'; 
	
	SEGMENT_NAME
	--------------------------------------------------------------------------------
	 EXTENT_ID
	----------
	FILE_NAME
	--------------------------------------------------------------------------------
	   FILE_ID
	----------
	SOLISTAS
		 0
	/home/oracle/practica5/TS3.dbf
		19
	

	SEGMENT_NAME
	--------------------------------------------------------------------------------
	 EXTENT_ID
	----------
	FILE_NAME
	--------------------------------------------------------------------------------
	   FILE_ID
	----------
	INTERPRETES
		 0
	/home/oracle/practica5/TS3.dbf
		19
	

	SEGMENT_NAME
	--------------------------------------------------------------------------------
	 EXTENT_ID
	----------
	FILE_NAME
	--------------------------------------------------------------------------------
	   FILE_ID
	----------
	PK_INTERPRETES
		 0
	/home/oracle/practica5/TS3.dbf
		19
	

	SEGMENT_NAME
	--------------------------------------------------------------------------------
	 EXTENT_ID
	----------
	FILE_NAME
	--------------------------------------------------------------------------------
	   FILE_ID
	----------
	PK_SOLISTAS
		 0
	/home/oracle/practica5/TS3.dbf
		19


	SQL> 

![](https://github.com/alvarocn/practica5-almacenamiento-alumno3/blob/master/imagenes/8.png)

![](https://github.com/alvarocn/practica5-almacenamiento-alumno3/blob/master/imagenes/9.png)


**4. Redimensiona los ficheros asociados a los tres tablespaces que has creado de forma que ocupen el mínimo espacio posible para alojar sus objetos.**


	A continuación vamos a mostrar el espacio que ocupan:

	SQL> select sum(bytes)/1024||'KB', tablespace_name
	from dba_segments
	where tablespace_name like 'TS%'
	group by tablespace_name;  2    3    4  
	
	SUM(BYTES)/1024||'KB'			   TABLESPACE_NAME
	------------------------------------------ ------------------------------
	64KB					   TS2
	512KB					   TS3
	
	SQL> 


	SQL> select file_name,tablespace_name,(bytes/1024)||'KB'
	from dba_data_files
	where tablespace_name like 'TS%';  2    3  
	
	FILE_NAME
	--------------------------------------------------------------------------------
	TABLESPACE_NAME 	       (BYTES/1024)||'KB'
	------------------------------ ------------------------------------------
	/home/oracle/TS2.dbf
	TS2			       1024KB
	
	/home/oracle/TS2-2.dbf
	TS2			       1024KB
	
	/opt/oracle/product/12.2.0.1/dbhome_1/dbs/c:fichero3-TS2.dbf
	TS2			       10240KB
	
	
	FILE_NAME
	--------------------------------------------------------------------------------
	TABLESPACE_NAME 	       (BYTES/1024)||'KB'
	------------------------------ ------------------------------------------
	/home/oracle/practica5/TS3.dbf
	TS3			       20480KB
	

	SQL> 


![](https://github.com/alvarocn/practica5-almacenamiento-alumno3/blob/master/imagenes/10.png)

Con el siguiente comando redimensionamos los ficheros creados anteriormente y posteriormente comprobaremos que el espacio que ocupa es menor.


	SQL> alter database datafile '/home/oracle/TS2.dbf' resize 150K;
	
	Base de datos modificada.
	
	SQL> alter database datafile '/home/oracle/TS2-2.dbf' resize 150K;
	
	Base de datos modificada.
	
	SQL> alter database datafile '/home/oracle/practica5/TS3.dbf' resize 2M;
	
	Base de datos modificada.
	SQL> select file_name,tablespace_name,(bytes/1024)||'KB'
	from dba_data_files
	where tablespace_name like 'TS%';  2    3 
	
	FILE_NAME
	--------------------------------------------------------------------------------
	TABLESPACE_NAME 	       (BYTES/1024)||'KB'
	------------------------------ ------------------------------------------
	/home/oracle/TS2.dbf
	TS2			       152KB
	
	/home/oracle/TS2-2.dbf
	TS2			       152KB
	
	/opt/oracle/product/12.2.0.1/dbhome_1/dbs/c:fichero3-TS2.dbf
	TS2			       10240KB
	
	
	FILE_NAME
	--------------------------------------------------------------------------------
	TABLESPACE_NAME 	       (BYTES/1024)||'KB'
	------------------------------ ------------------------------------------
	/home/oracle/practica5/TS3.dbf
	TS3			       2048KB
		

	SQL> 


![](https://github.com/alvarocn/practica5-almacenamiento-alumno3/blob/master/imagenes/11.png)

**5. Realiza un procedimiento llamado InformeRestricciones que reciba el nombre de una tabla y muestre los nombres de las restricciones que tiene, a qué columna o columnas afectan y en qué consisten exactamente.**


	create or replace procedure externa(p_table varchar2,p_nombreColumna varchar2)
	is
	
	  v_nombreClomuna varchar2(50);
	  v_nombreTabla varchar2(50);
	  v_nombre_ext varchar2(50);
	  v_nombrerestriccion varchar2(50);
	
	begin
	
		
	 SELECT  a.column_name, a.constraint_name, c_fk.table_name , c_fk.constraint_name into 		v_nombreClomuna,v_nombrerestriccion,v_nombreTabla,v_nombre_ext
	 FROM all_cons_columns a
 	JOIN all_constraints c ON a.owner = c.owner
 	AND a.constraint_name = c.constraint_name
 	JOIN all_constraints c_fk ON c.r_owner = c_fk.owner
 	AND c.r_constraint_name = c_fk.constraint_name
 	WHERE c.constraint_type = 'R'
 	AND a.table_name = p_table
 	AND a.column_name = p_nombreColumna;

 	 dbms_output.put_line(rpad(v_nombrerestriccion,30)||rpad(v_nombreClomuna,30)||		'foreign key '||v_nombreTabla||': '||v_nombre_ext);

	end externa;
	/

	create or replace procedure InformeRestricciones(p_table varchar2)
	is

	  cursor c_restricciones is
	  select distinct cc.CONSTRAINT_NAME,COLUMN_NAME,SEARCH_CONDITION_VC,decode(CONSTRAINT_TYPE, 'C', 'check','R', 'foreign key', 'U', 'unique 	key','P', 'primary key','V','check view','O','R/O View') CONSTRAINT_TYPE
	  from DBA_CONS_COLUMNS cc,dba_constraints c
	  where cc.CONSTRAINT_NAME=c.CONSTRAINT_NAME
	  and c.table_name=p_table;

	begin
	  notable(p_table);
	  dbms_output.put_line(rpad('Nombre',30)||rpad('COLUMNA',15)||rpad('TIPO',10)||'CONSISTE EN');
	  for x in c_restricciones loop
	   if x.CONSTRAINT_TYPE = 'foreign key' then
	    externa(p_table,X.COLUMN_NAME);
	  elsif x.CONSTRAINT_TYPE = 'check' then
	    dbms_output.put_line(rpad(x.CONSTRAINT_NAME,30)||rpad(x.COLUMN_NAME,15)||rpad(x.CONSTRAINT_TYPE,10)||x.SEARCH_CONDITION_VC);
	  else
	    dbms_output.put_line(rpad(x.CONSTRAINT_NAME,30)||rpad(x.COLUMN_NAME,15)||rpad(x.CONSTRAINT_TYPE,10));
	  end if;

 	  end loop;
	  end InformeRestricciones;
	  /

	create or replace procedure notable(p_table VARCHAR2)
	is

	 v_table number;

	begin

	  select count(*) into v_table
	  from dba_tables
	  where table_name = p_table;
	  if v_table=0 then
	    raise_application_error(-20006,'La tabla introducida no existe');
	  end if;

	end notable;
	/
![](https://github.com/alvarocn/practica5-almacenamiento-alumno3/blob/master/imagenes/12.png)
![](https://github.com/alvarocn/practica5-almacenamiento-alumno3/blob/master/imagenes/15.png)
![](https://github.com/alvarocn/practica5-almacenamiento-alumno3/blob/master/imagenes/14.png)

 ## PostgreSQL
**7. Averigua si es posible establecer cuotas de uso sobre los tablespaces en Postgres.**


PostgreSQL no tiene la opción de administrar cuotas como presentan tener los Sistemas Gestores de Bases de Datos como por ejemplo Oracle Database.Lo que se puede hacer es asignar una cuota maxima de tamaño al usuario en la particion donde se aloja el tablespace, como  veremos a continuación.


Como usuario root,  modificamos /etc/fstab para añadir las opciones usrquota.
Instalamos "quota"

	"apt install quota"
Modificamos el fichero "/etc/fstab"

	"/dev/sda1 /home/alvaro ext4 defaults,usrjquota=aquota.user 1 2"

Montamos las particiones con el siguiente comando:

	"mount -o remount /home/alvaro"


Usamos el comando quotacheck que nos define y analiza los sistemas de archivos que llevan quotas además de añadir tabla del uso del disco por sistema de archivo. Esta tabla es necesaria para que se actualice la copia del uso del disco del sistema operativo. PostgreSQL lo que hace es limitar el tamaño de un tablespace al espacio máximo del dispositivo de bloques.

Para hacer un checkeo utilizamos el siguiente comando:

	"quotacheck -avugcm"

Para iniciar y actualizar archivos de configuración de las cuotas que hemos montado lanzamos estos comandos:

	"quotaon /home/alvaro"

Si queremos cambiar la cuota de un usuario:

	"edquota "usuarios""

![](https://github.com/alvarocn/practica5-almacenamiento-alumno3/blob/master/imagenes/13.png)

## MySQL:

**8. Averigua si existe el concepto de extensión en MySQL y si coincide con el existente en ORACLE.**

InnoDB es un motor de almacenamiento que usa el gestor de base de datos, en el podemos encontrar extensiones, segmentos y páginas. 

	File system              <-> InnoDB
	----------------------------------------------
	disk partition           <-> tablespace
	file                     <-> segment ---------> UN FICHERO ES UN SEGMENTO.
	inode                    <-> fsp0fsp.c 'inode'
	fs space allocation unit <-> extent---------> UN ESPACIO ES UNA EXTENSIÓN.
	disk block               <-> page (16 kB)-----------> UN BLOQUE ES UNA PÁGINA.

En Oracle:

Extensión es un número específico de bloques de datos contiguos en el disco.
El segmento es un conjunto de extensiones, no necesariamente consecutivas  en un mismoconsecutivas, enunmismotablespace.
Bloque de datos es el número específico de bytes contiguos de espacio físico en el disco. 

## MongoDB
**9. Averigua si en MongoDB puede saberse el espacio disponible para almacenar nuevos documentos.**

Mongodb tiene una serie de funciones con las cuales podemos obtener diferentes datos sobre almacenamiento como por ejemplo el espacio disponible con la funcion db.collection.storageSize(), dicha función nos proporciona la reserva de espacio incluido el espacio no utilizado.

Otras funciones serían las siguientes:

    db.collection.dataSize(): el tamaño de los datos en la colección.
    db.collection.index.stats().indexSizes: Ver tamaño de un índice.
    db.collection.totalSize(): El tamaño de los datos más el de los índices.
    db.collection.totalIndexSize(): el tamaño de los índices.








