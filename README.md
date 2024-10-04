--Se crea la tabla usuarios con los encabezados de sus columnas y tipos de datos para cada una
create table usuarios (
	id serial primary key, --se incrementara con cada nuevo registro, id unico
	email varchar (50), --puede almanacenar hasta 50 caracteres
	nombre varchar (20), --puede almanacenar hasta 20 caracteres
	apellido varchar (50), --puede almanacenar hasta 50 caracteres
	rol varchar check (rol in ('administrador', 'usuario')) --puede ser rol o administrador y no es limitado en caracteres
	);

--Se insertar los registros para cada una de las columnas para la tabla usuarios
insert into usuarios (email, nombre, apellido, rol)
values
('vanesa.martinez@gmail.com', 'vanesa', 'martinez', 'administrador'),
('ana123@gmail.com', 'ana', 'martinez', 'administrador'),
('edgar@hotmail.com', 'edgar','perez', 'usuario'),
('carolina45@yahoo.es', 'carolina', 'lopez', 'usuario'),
('doriagh_9@gmail.com', 'doria', 'marquez', 'usuario')
;

--Se crea la tabla posts con los encabezados de sus columnas y tipos de datos para cada una
create table posts (
	id serial primary key,
	titulo varchar, --no esta limitado (tener en cuenta limitación max de 255 caracteres para tipo de dato varchar)
	contenido text, --almacena texto largo
	fecha_creacion timestamp, --almacenará fecha y hora de creación
	fecha_actualizacion timestamp, --almacená fecha y hora de actualización
	destacado boolean, --será TRUE or FALSE
	usuario_id bigint --puede ser NULL para posts sin usuario asignado
);

--Se insertar los registros para cada una de las columnas de la tabla posts
insert into posts 
	(titulo, contenido, fecha_creacion, fecha_actualizacion, destacado, usuario_id)
	values
	('post del administrador 1', 'este es el contenido del primer post del administrador', '2020-05-01 14:30:00', '2023-12-28 09:53:00', TRUE, 1), 
	('post del administrador 2', 'este es el contenido del segundo post del administrador', '1999-01-30 16:50:00', '2005-04-10 15:00:09', TRUE, 1),
	('post del usuario 1', 'este es el contenido del primer post del usuario', '2022-12-12 08:00:00', '2023-03-15 19:14:00', FALSE, 3),
	('post del usuario 2', 'este es el contenido del segundo post del usuario', '2007-08-09 10:00:00', '2009-09-09 18:59:00', FALSE, 4),
	('post sin usuario', 'este es el contenido del post sin usuario asignado', '2010-10-05 23:09:00', '2013-09-27 03:01:00', TRUE, NULL)
	;

--Se crea la tabla comentarios con los encabezados de sus columnas y tipos de datos para cada una
create table comentarios (
	id serial primary key,
	contenido text,
	fecha_creacion timestamp,
	usuario_id bigint,
	post_id bigint
	);

--Se insertar los registros para cada una de las columnas de la tabla comentarios
insert into comentarios 
	(id, contenido, fecha_creacion, usuario_id, post_id)
	values
	(1, 'contenido del usuario con id 1', '2006-08-30 19:56:00', 1, 1),
	(2, 'contenido del usuario con id 2', '2020-07-09 09:49:00', 2, 1),
	(3, 'contenido del usuario con id 3', '2003-02-10 18:44:00', 3, 1),
	(4, 'contenido del usuario con id 4', '2024-01-01 04:50:00', 1, 2),
	(5, 'contenido del usuario con id 5', '2000-09-30 15:22:00', 2, 2)
	;

--Verificamos que los datos se hayan insertado correctamente
select * from usuarios;
select * from posts;
select * from comentarios;

--cruzamos datos de la tabla usuarios con la tabla posts
--realizamos el cruce por medio de inner join para obtener registros coincidentes en ambas
--mostramos nombre e email para usuarios, titulo y contenido para posts
--utilizamos alias para ambas tablas
select u.nombre, u.email, p.titulo, p.contenido from usuarios u inner join posts p
on u.id = p.usuario_id;

--Este resultado muestra los posts que fueron creados por usuarios y tiene el rol de administrador
--Tenemos en cuenta el alias 
select p.id, p.titulo, p.contenido from usuarios u inner join posts p
on u.id = p.usuario_id
where u.rol = 'administrador';

--conteo de la cantidad de post de cada usuario
select u.id, u.email, count (p.id) as cantidad_posts
from usuarios u left join posts p
on u.id = p.usuario_id
group by u.id, u.email
order by id asc
;

--A continuación, se muestra la querry para obtener el email del usuario que ha creado mas posts
select u.email
from usuarios u join posts p
on u.id = p.usuario_id
group by u.id, u.email
order by count (p.id) desc
limit 1;

--fecha del ultimo post por cada usuario
select u.id, u.email, max (p.fecha_creacion) as fecha_ultimo_post
from usuarios u left join posts p
on u.id = p.usuario_id
group by u.id, u.email
order by id asc;

--mostramos el titulo y contenido del post con mas comentarios
select p.titulo, p.contenido
from posts p join comentarios c
on p.id = c.post_id
group by p.id, p.titulo, p.contenido
order by count(c.id) desc
limit 1;

--Para este caso, he decidido no usar alias sin embargo arriba se muestra la idea de utilizarlo para varios casos
--Aqui se muestra en una tabla el título de cada post, el contenido de cada post y el contenido  de cada comentario asociado a los posts mostrados, junto con el email del usuario  que lo escribió
select posts.titulo as post_titulo, 
posts.contenido as post_contenido,
comentarios.contenido as comentario_contenido,
usuarios.email as usuario_email
from posts join comentarios on posts.id = comentarios.post_id 
join usuarios on comentarios.usuario_id = usuarios.id;

--contenido del ultimo comentario de cada usuario
with ultimo_comentario_de_cada_usuario as (
select usuario_id, max(fecha_creacion) as ultima_fecha
from comentarios
group by usuario_id)
select c.usuario_id, u.email, c.contenido as comentario_del_contenido, c.fecha_creacion
from ultimo_comentario_de_cada_usuario ucu
join comentarios c on ucu.usuario_id = c.usuario_id and ucu.ultima_fecha = c.fecha_creacion
join usuarios u on c.usuario_id = u.id
order by usuario_id asc;

--emails de los usuarios que no han escrito ningun comentario usando having
select u.email from usuarios u left join comentarios c
on u.id = c.usuario_id
group by u.id, u.email
having count (c.id) = 0;








