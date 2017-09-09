# Práctica 2 – Semáforos

### 1. 	Existen N personas que deben ser chequeadas por un detector de metales antes de poder ingresar al avión.

#### 	a. 	Implemente una solución que modele el acceso de las personas a un detector (es decir si el detector está libre la persona lo puede utilizar caso contrario debe esperar).

	Var
		sem libres = 1
	Process Detector[i = 1..N]::
	{	llega una persona.
		P(semaforo)
		pasa una persona.
		V(semaforo)
		ingresa al avion o le sacan la bomba.
	}


#### 	b. 	Modifique su solución para el caso que haya tres detectores.

	Var
		sem libres = 3
	Process Detector[i = 1..N]::
	{	llega una persona.
		P(semaforo)
		pasa una persona.
		V(semaforo)
		ingresa al avion o le sacan la bomba.
	}


### 2.	Un sistema operativo mantiene 5 instancias de un recurso almacenadas en una cola, cuando un proceso necesita usar una instancia del recurso la saca de la cola, la usa y cuando termina de usarla la vuelve a depositar.

	Var
		sem instLibres = 5
		sem colaLibre = 1
	Process Operativo[i = 1..N]::
	{	P(instLibres)
		recurso = cola.desencolar			# Necesito semaforo para usar la cola???
		el proceso usa la instancia del recurso.
		cola.encolar(recurso)				# Necesito semaforo para usar la cola???
		V(instLibres)
	}

### 3.	Suponga que existe una BD que puede ser accedida por 6 usuarios como máximo al mismo tiempo. Además los usuarios se clasifican como usuarios de prioridad alta y usuarios de prioridad baja. Por último la BD tiene la siguiente restricción:
* no puede haber más de 4 usuarios con prioridad alta al mismo tiempo usando la BD.
* no puede haber más de 5 usuarios con prioridad baja al mismo tiempo usando la BD.
### 	Indique si la solución presentada es la más adecuada. Justifique la respuesta.

		Var
			sem: semaphoro := 6;
			alta: semaphoro := 4;
			baja: semaphoro := 5;

		Process Usuario-Alta [I:1..L]::				
			{ 	P (sem);								
				P (alta);									
				//usa la BD									
				V(sem);										
				V(alta);									
			}											

		Process Usuario-Baja [I:1..K]::
			{ 	P (sem);
				P (baja);
				//usa la BD
				V(sem);
				V(baja);
			}

	No es la forma correcta de declarar semaforos, se declaran con "sem nombreDeSemaforo = valorInicial"
	Se tiene que hacer P(sem) luego del P(alta/baja), ya que sino podrian pasar el P(sem) 6 prio baja, 
		cuando solo 5 pasan el P(baja) y no puede ejecutarse ningun prioridad alta
	

### 4. 	Se tiene un curso con 40 alumnos, la maestra entrega una tarea distinta a cada alumno, luego cada alumno realiza su tarea y se la entrega a la maestra para que la corrija, esta revisa la tarea y si está bien le avisa al alumno que puede irse, si la tarea está mal le indica los errores, el alumno corregirá esos errores y volverá a entregarle la tarea a la maestra para que realice la corrección nuevamente, esto se repite hasta que la tarea no tenga errores.

	Var
		cola porCorregir	#Asumo que encolar y desencolar son atomicas, sino agregaria un semaforo.
		int seFueron = 0
		sem entregados = 0
		array[1..40] tareaPara of sem = 0
		array[1..40] corregido of sem = 1
		array[1..40] puedoIrme of boolean = false

	Process Alumno[i = 1..40]::
	{	P(tareaPara[i])					# Espero a que la maestra me de la tarea.
		while(!puedoIrme[i]) {
			hago mi tarea
			porCorregir.encolar(i)		# Le entrego mi tarea a la maestra para que la corrija.
			V(entregados)				# Aviso que hay una tarea entregada mas.
			P(corregido[i])				# Espero hasta que mi tarea haya sido corregida.
		}
	}

	Process Maestra::
	{	for i = 1 to 40 do V(tareaPara[i])	# Le doy a los su tarea a cada alumno.
		while (seFueron < 40) {
			P(entregados)					# Espero hasta que haya algun entregado.
			alumnoActual = porCorregir.desencolar()		# Agarro de la cola una tarea.
			Corrijo la tarea del alumnoActual
			if laTareaEstabaBien then 
				puedoIrme[alumnoActual] = true 
				seFueron++
			endif
			V(corregido[alumnoActual])	# Le aviso al alumnoActual que ya corregi su trabajo.
		}
	}


### 5.	Suponga que se tiene un curso con 50 alumnos. Cada alumno elije una de las 10 tareas para realizar entre todos. Una vez que todos los alumnos eligieron su tarea comienzan a realizarla. Cada vez que un alumno termina su tarea le avisa al profesor y si todos los alumnos que tenían la misma tarea terminaron el profesor les otorga un puntaje que representa el orden en que se terminó esa tarea.
### Nota: Para elegir la tarea suponga que existe una función elegir que le asigna una tarea a un alumno (esta función asignará 10 tareas diferentes entre 50 alumnos, es decir, que 5 alumnos tendrán la tarea 1, otros 5 la tarea 2 y así sucesivamente para las 10 tareas).

	Var
		sem eligieron = 0					# max 50
		sem tareasPorAtender = 0			# max 10
		array[1..10] orden of int			# orden en el que termino cada una de las 10 tareas.
		array[1..10] termine of sem = 0		# max 5, cantidad de alumnos que termino cada tarea.
		array[1..10] semOrden of sem = 0	# indica cuando el profesor asigno el orden para mi tarea. 
		array[1..50] arranquen of sem = 0	# las posiciones seran 1 cuando todos los alumnos elijan tarea.
	
	Process Alumno[i = 1..50]:: 
	{	tarea = elegirTarea()				# Cada alumno elije su tarea.
		V(eligieron)						# Aumento la cantidad de alumnos que ya eligieron.
		if (eligieron == 50) then for i = 1 to 50 do V(arranquen[i])	# Si eligieron 50, digo que se puede arrancar. Aca tengo que poner semaforo no?
		P(arranquen[i])						# Espero a que pueda arrancar.
		Realizo mi tarea.					
		V(termine[tarea])					# Aumento la cantidad que termino la tarea que tengo asignada.
		if (termine[tarea] == 5) then 		# Si terminamos 5 de mi tarea:				Aca tengo que poner semaforo no???
			cola.encolar(tarea)					# Encolo la tarea que tengo asignada.
			V(tareasPorAtender)					# Aumento que hay una mas para atender.
		endif
		P(semOrden[tarea])					# Espero a que se le asigne un orden a mi tarea.
		mirarMiOrden(orden[tarea])			# Veo que orden se me asigno.
		Me voy.								# Bai.
	}

	Process Profesor::
	{	for tareasAtendidas = 1 to 10 do {
			P(tareasPorAtender)
			tareaActual = cola.desencolar()
			orden[tareaActual] = tareasAtendidas
			semOrden[tareaActual] = 5	# Se puede esto? Seria atomico porque solo es una asignacion.
										# Lo que quiero hacer es dejar que pasen los 5 alumnos que estan
										# esperando su resultado (orden en el que terminaron la tarea).
										# Sino hago 5 veces V(semOrden[tareaActual]) re villa.
		}
	}


### 6.	A una empresa llegan E empleados y por día hay T tareas para hacer (T>E), una vez que todos los empleados llegaron empezaran a trabajar. Mientras haya tareas para hacer los empleados tomaran una y la realizarán. Cada empleado puede tardar distinto tiempo en realizar cada tarea. Al finalizar el día se le da un premio al empleado que más tareas realizó.

	Var
		sem semTareas = 1
		sem llegue = 1
		int tareas = T
		array[1..E] terminadas of int
		array[1..E] arrancar of sem = 0

	Process Empleado[i = 1..E]::
	{	P(llegue)
		cant++
		if (cant == E) then for j = 1 to E do V(arrancar[j])
		V(llegue)
		P(arrancar[i])
		P(semTareas)
		while (tareas > 0) {
			tareas--
			V(semTareas)
			realizar tarea
			terminadas[i]++
			P(semTareas)
			terminadasTotales++
		}
		if terminadasTotales == E {
			premio = terminadas.max
			for k = 1 to E do V(terminamos[k])
		}
		V(semTareas)
		P(terminamos[k])
		if (terminadas[i] == premio) then recibirPremio
	}
