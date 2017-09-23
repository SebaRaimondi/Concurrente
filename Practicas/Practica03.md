CONSIDERACIONES PARA RESOLVER LOS EJERCICIOS:
• Los monitores utilizan el protocolo signal and continue.
• A una variable condition SÓLO pueden aplicársele las operaciones SIGNAL, SIGNALALL y WAIT.
• NO puede utilizarse el wait con prioridades.
• NO se puede utilizar ninguna operación que determine la cantidad de procesos encolados en una variable condition o si está vacía.
• La única forma de comunicar datos entre monitores o entre un proceso y un monitor es por medio de invocaciones al procedimiento del monitor del cual se quieren obtener (o enviar) los datos.
• No existen variables globales.
• En todos los ejercicios debe maximizarse la concurrencia.
• En todos los ejercicios debe aprovecharse al máximo la característica de exclusión mutua que brindan los monitores.
• Debe evitarse hacer busy waiting.
• En todos los ejercicios el tiempo debe representarse con la función delay


1. Se dispone de un puente por el cual puede pasar un solo auto a la vez. Un auto pide permiso para pasar por el puente, cruza por el mismo y luego sigue su camino. Nota: no importa el orden en que han llegado al puente.
a. ¿El código funciona correctamente? Justifique su respuesta.
b. ¿Se podría simplificar el programa? En caso afirmativo, rescriba el código.
c. Si hubiese que respetar el orden de llegada de los vehículos, ¿La solución original lo respeta? Si rescribió el código en el punto b), ¿esa solución lo respeta?.

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

a.  No funciona porque cuando hay un auto en el puente deja esperando muchas veces al mismo auto en la cola. Se soluciona cambiando el while por un if.

b.  Se puede pensar a la variable cant como un boolean que determina si en un momento dado se puede cruzar o no, ya que la cantidad de autos permitidos en un momento dado es solo 1, pero el funcionamiento seria el mismo. Ademas de cambiar el while por un if.

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
    
c.  Ambas lo respetan, asumiendo que la primera funcionara.


2. Implementar el acceso a una base de datos de solo lectura que puede atender a lo sumo 5 consultas simultáneas.

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


3. En un laboratorio de genética se debe administrar el uso de una máquina secuenciadora de ADN. Esta máquina se puede utilizar por una única persona a la vez. Existen 100 personas en el laboratorio que utilizan repetidamente esta máquina para sus estudios, para esto cada persona pide permiso para usarla, y cuando termina el análisis avisa que termino. Cuando la máquina está libre se le debe adjudicar a aquella persona cuyo pedido tiene mayor prioridad (valor numérico entre 0 y 100).

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



