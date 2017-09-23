# Práctica 3 – Monitores 

##### CONSIDERACIONES PARA RESOLVER LOS EJERCICIOS:
* Los monitores utilizan el protocolo signal and continue.
* A una variable condition SÓLO pueden aplicársele las operaciones SIGNAL, SIGNALALL y WAIT.
* NO puede utilizarse el wait con prioridades.
* NO se puede utilizar ninguna operación que determine la cantidad de procesos encolados en una variable condition o si está vacía.
* La única forma de comunicar datos entre monitores o entre un proceso y un monitor es por medio de invocaciones al procedimiento del monitor del cual se quieren obtener (o enviar) los datos.
* No existen variables globales.
* En todos los ejercicios debe maximizarse la concurrencia.
* En todos los ejercicios debe aprovecharse al máximo la característica de exclusión mutua que brindan los monitores.
* Debe evitarse hacer busy waiting.
* En todos los ejercicios el tiempo debe representarse con la función delay

---

### Ejercicio 1.
### Se dispone de un puente por el cual puede pasar un solo auto a la vez. Un auto pide permiso para pasar por el puente, cruza por el mismo y luego sigue su camino. Nota: no importa el orden en que han llegado al puente.
a. ¿El código funciona correctamente? Justifique su respuesta.

b. ¿Se podría simplificar el programa? En caso afirmativo, rescriba el código.

c. Si hubiese que respetar el orden de llegada de los vehículos, ¿La solución original lo respeta? Si rescribió el código en el punto b), ¿esa solución lo respeta?.
```
Monitor Puente
    cond cola;
    int cant= 0;

    Procedure entrarPuente (int au)
        while ( cant > 0) wait (cola);
        cant = cant + 1;
    end;

    Procedure salirPuente (int au)
        cant = cant – 1;
        signal(cola);
    end;
End Monitor;

Process Auto [a:1..M]
    Puente. entrarPuente (a);
    “el auto cruza el puente”
    Puente. salirPuente(a);
End Process;
```

a.  No funciona porque cuando hay un auto en el puente deja esperando muchas veces al mismo auto en la cola. Se soluciona cambiando el while por un if.

b.  Se puede pensar a la variable cant como un boolean que determina si en un momento dado se puede cruzar o no, ya que la cantidad de autos permitidos en un momento dado es solo 1, pero el funcionamiento seria el mismo. Ademas de cambiar el while por un if.

```
Monitor Puente
    cond cola;
    boolean ocupado = false;

    Procedure entrarPuente (int au)
        if (ocupado) wait (cola);
        else ocupado = true;
    end;

    Procedure salirPuente (int au)
        ocupado = false;
        signal(cola);
    end;
End Monitor;
```

c.  Ambas lo respetan, asumiendo que la primera funcionara.

---

### Ejercicio 2.
### Implementar el acceso a una base de datos de solo lectura que puede atender a lo sumo 5 consultas simultáneas.

```
Monitor BD
    cond cola;
    int lectores = 0;

    Procedure leer(int l)
        if ( lectores > 5 ) wait (cola);
        else lectores = lectores + 1;
    end;

    Procedure liberar(int l)
        lectores = lectores – 1;
        signal(cola);
    end;
End Monitor;

Process Lector [l: 1..M]
    BD.leer(l);
    “el lector accede a la base y lee la informacion que necesita”
    BD.liberar(l);
End Process;
```

---

### Ejercicio 3.
### En un laboratorio de genética se debe administrar el uso de una máquina secuenciadora de ADN. Esta máquina se puede utilizar por una única persona a la vez. Existen 100 personas en el laboratorio que utilizan repetidamente esta máquina para sus estudios, para esto cada persona pide permiso para usarla, y cuando termina el análisis avisa que termino. Cuando la máquina está libre se le debe adjudicar a aquella persona cuyo pedido tiene mayor prioridad (valor numérico entre 0 y 100).

```
Monitor Maquina {
    bool libre = true
    cond turno[N]
    cola espera

    Procedure pedir(int p) {
        if (libre) libre = false
        else {
            insertar_ordenado(espera, p, p)
            wait turno[p]
        }
    }

    Procedure liberar() {
        if ( empty(espera) ) libre = true
        else {
            sacar(espera, prox)
            signal(turno[prox])
        }
    }

}

Process Persona [p: 1..100] {
    Maquina.pedir(p)
    "La persona usa la maquina"
    Maquina.liberar()
}
```

---

### Ejercicio 4.
### Suponga que N personas llegan a la cola de un banco. Una vez que la persona se agrega en la cola no espera más de 15 minutos para su atención, si pasado ese tiempo no fue atendida se retira. Para atender a las personas existen 2 empleados que van atendiendo de a una y por orden de llegada a las personas.

```
By Borre ft. Seba

Monitor Banco {
    cond cola
    array pos[1..N] of cond
    cola clientes
    array atendido[1..N] of bool = false
    int s[1..N] of bool = false

    Procedure esperar (int au) {
        encolar(clientes, au)
        wait (cola)
    }


    Procedure atender () {
        if ( !empty(clientes) ) {
            aux = desencolar(clientes) 
            atendido[aux] = true
            signal(cola)
        }
    }

    Procedure irse(int au) {
        if (!atendido[au]) {
            desencolar(clientes)
            signal(cola)
        }
    }

    Procedure P (int i) {
        if (s[i] == false) wait(pos[i])
    }

    Procedure V (int i) {
        s[i]=true
        signal(pos[i])
    }
}


Process Empleado [e: 1..2] {
	While(true) {
		Banco.atender()
		"Atender Cliente"
    }
}

Process Cliente [c: 1..N] {
    Banco.V(c)
    Banco.esperar()
    "Me voy"
}

Process Timer [t: 1..N] {
    Banco.P(t)
    delay (15)
    Banco.irse(t)
}
```

---

### Ejercicio 5.
### Se tienen 50 empleados de una empresa petrolera que se reúnen para ir en grupos de a 5 a verificar 1 de los 10 pozos de petróleo existentes, cuando los empleados llegan se les asigna un numero de grupo, luego deberán esperar a sus compañeros de grupo para ir a la verificación.

```
Process Empleado [e: 1..50] {
    Empresa.llegue()
    "Se va a verificar un pozo con su grupo."
}

Monitor Empresa {
    int actual = 1
    int[10] grupos = 0
    cond llegaronTodosGrupo

    Procedure llegue() {
        grupos[actual]++
        if (grupos[actual] < 5) wait(llegaronTodosGrupo)
        else {
            signal_all(llegaronTodosGrupo)
            actual++
        }
    }
}
```

---

### Ejercicio 6.
### En un entrenamiento de futbol hay 20 jugadores que forman 4 equipos (cada jugador conoce el equipo al cual pertenece llamando a la función DarEquipo()). Cuando un equipo está listo (han llegado los 5 jugadores que lo componen), debe enfrentarse a otro equipo que también esté listo (los dos primeros equipos en juntarse juegan en la cancha 1, y los otros dos equipos juegan en la cancha 2). Una vez que el equipo conoce la cancha en la que juega, sus jugadores se dirigen a ella. Cuando los 10 jugadores del partido llegaron a la cancha comienza el partido, juegan durante 50 minutos, y al terminar todos los jugadores del partido se retiran (no es necesario que se esperen para salir).

```
Process Jugador [j: 1..20] {
	equipo = DarEquipo()
	Entrenamiento.llegue(equipo)
	delay(50)  // Juega durante 50min
}

Monitor Entrenamiento {
	int[4] llegaronEquipo
	cond[4] equipo
	queue completos
	int cancha = 1

	Procedure llegue(int i) {
        llegaronEquipo[i]++
        if (llegaronEquipo[i] < 5) wait(equipo[i])
        else {
            if(completos.empty) {
                completos.encolar(i)
                wait(equipo[i])
            }
            else {
                signal_All(equipo[i])
                signal_All(equipo[completos.next])
            }
        }
    }
}
```

---

### Ejercicio 7.
### Resolver la siguiente situación. Suponga una comisión con 50 alumnos. Cuando los alumnos llegan forman una fila, una vez que están los 50 en la fila el jefe de trabajos prácticos les entrega el número de grupo (número aleatorio del 1 al 25) de tal manera que dos alumnos tendrán el mismo número de grupo (suponga que el jefe posee una función DarNumero() que devuelve en forma aleatoria un número del 1 al 25, el jefe de trabajos prácticos no guarda el número que le asigna a cada alumno). Cuando un alumno ha recibido su número de grupo comienza a realizar la práctica. Al terminar de trabajar, el alumno le avisa al jefe de trabajos prácticos y espera la nota. El jefe de trabajos prácticos, cuando han llegado los dos alumnos de un grupo les devuelve a ambos la nota del GRUPO (el primer grupo en terminar tendrá como nota 25, el segundo 24, y así sucesivamente hasta el último que tendrá nota 1).

---

### Ejercicio 8.
### Suponga que en una fábrica de camisas trabajan 40 operarios que deben realizarse 5000 camisas. Para realizar una camisa se requieren 8 materiales diferentes, por lo que existe un depósito para cada uno de estos donde se almacenan.
### Cuando todos los operarios han llegado el encargado los agrupa de a cuatro (les asigna un número de grupo de 1 a 10). Los 4 operarios de grupo deben juntarse y luego comenzar a fabricar las camisas.
### Para realizar cada camisa, entre los empleados del grupo deben buscar los 8 materiales necesarios, cuando lo han conseguido, los 4 la fabrican conjuntamente.
### Luego de que todas las camisas han sido fabricadas los operarios deben retirarse.
### Nota: no se deben fabricar camisas de más. No se puede suponer nada sobre los tiempos, es decir, el tiempo en que un operario tarda en buscar los elementos, ni el tiempo en que tarda un grupo en fabricar una camisa. 

