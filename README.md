package modelo;

public class Estudiante {
    private String legajo;
    private String nombre;

    public Estudiante(String legajo, String nombre) {
        this.legajo = legajo;
        this.nombre = nombre;
    }

    public String getLegajo() {
        return legajo;
    }

    public String getNombre() {
        return nombre;
    }

    @Override
    public String toString() {
        return "Estudiante{" + "legajo='" + legajo + '\'' + ", nombre='" + nombre + '\'' + '}';
    }
}

package modelo;

public class Sala {
    private int id;
    private String nombre;

    public Sala(int id, String nombre) {
        this.id = id;
        this.nombre = nombre;
    }

    public int getId() {
        return id;
    }

    public String getNombre() {
        return nombre;
    }

    @Override
    public String toString() {
        return "Sala ID: " + id + " - " + nombre;
    }
}

package modelo;

import java.time.LocalDate;

public class Inscripcion {
    private LocalDate fecha;
    private String estado;
    private Estudiante estudiante;

    public Inscripcion(Estudiante estudiante, LocalDate fecha, String estado) {
        this.estudiante = estudiante;
        this.fecha = fecha;
        this.estado = estado;
    }

    public LocalDate getFecha() {
        return fecha;
    }

    public String getEstado() {
        return estado;
    }

    public Estudiante getEstudiante() {
        return estudiante;
    }

    @Override
    public String toString() {
        return "Inscripción [" + fecha + " | Estado: " + estado + "] -> " + estudiante.getNombre() + " (Legajo: " + estudiante.getLegajo() + ")";
    }
}

package modelo;

import java.time.LocalDate;
import java.util.ArrayList;
import java.util.List;

public abstract class Actividad {
    private int id;
    private String titulo;
    private int cupoMaximo;
    public static final int CUPO_MINIMO = 5;
    
    private List<Inscripcion> inscripciones;

    public Actividad(int id, String titulo, int cupoMaximo) {
        this.id = id;
        this.titulo = titulo;
        this.cupoMaximo = cupoMaximo;
        this.inscripciones = new ArrayList<>();
    }

    public Inscripcion inscribir(Estudiante estudiante) {
        if (inscripciones.size() < cupoMaximo) {
            Inscripcion inscripcion = new Inscripcion(estudiante, LocalDate.now(), "CONFIRMADA");
            inscripciones.add(inscripcion);
            return inscripcion;
        } else {
            System.out.println("Cupo lleno para la actividad: " + titulo);
            return null;
        }
    }

    public void mostrarInscripciones() {
        System.out.println("   Inscriptos en " + titulo + ":");
        if (inscripciones.isEmpty()) {
            System.out.println("   (Sin inscripciones)");
        } else {
            for (Inscripcion ins : inscripciones) {
                System.out.println("     - " + ins);
            }
        }
    }

    public final void mostrarIdentificacion() {
        System.out.println("  [Actividad #" + id + "] Tipo: " + getTipo() + " | Título: " + titulo + " | Cupo Máx: " + cupoMaximo);
    }

    public abstract double calcularCostoMateriales();
    public abstract String getTipo();

    public int getId() { return id; }
    public String getTitulo() { return titulo; }
    public int getCupoMaximo() { return cupoMaximo; }
    public List<Inscripcion> getInscripciones() { return inscripciones; }
}

package modelo;

public class Charla extends Actividad {
    private String disertante;

    public Charla(int id, String titulo, int cupoMaximo, String disertante) {
        super(id, titulo, cupoMaximo);
        this.disertante = disertante;
    }

    @Override
    public double calcularCostoMateriales() {
        return 0.0;
    }

    @Override
    public String getTipo() {
        return "Charla (Disertante: " + disertante + ")";
    }

    public String getDisertante() {
        return disertante;
    }
}

package modelo;

public class Taller extends Actividad {
    private boolean requiereNotebook;

    public Taller(int id, String titulo, int cupoMaximo, boolean requiereNotebook) {
        super(id, titulo, cupoMaximo);
        this.requiereNotebook = requiereNotebook;
    }

    @Override
    public double calcularCostoMateriales() {
        return requiereNotebook ? 5000.0 : 2000.0;
    }

    @Override
    public String getTipo() {
        return "Taller (Notebook: " + (requiereNotebook ? "Sí" : "No") + ")";
    }

    public boolean isRequiereNotebook() {
        return requiereNotebook;
    }
}

package modelo;

import java.util.ArrayList;
import java.util.List;

public class EventoUniversitario {
    private final String id;
    private String titulo;
    private double costoBase;
    private boolean gratuito;
    private static int cantidadEventos = 0;

    private Sala sala; // Relación de Agregación
    private List<Actividad> actividades; // Relación de Composición

    public EventoUniversitario(String id, String titulo, double costoBase, boolean gratuito) {
        this.id = id;
        this.titulo = titulo;
        this.costoBase = costoBase;
        this.gratuito = gratuito;
        this.actividades = new ArrayList<>();
        cantidadEventos++;
    }

    // Constructor de copia (Ejercicio 1 y 2)
    public EventoUniversitario(EventoUniversitario otro) {
        this.id = otro.id + "_copia";
        this.titulo = otro.titulo + " (Copia)";
        this.costoBase = otro.costoBase;
        this.gratuito = otro.gratuito;
        this.sala = otro.sala; // Agregación: comparte la referencia de la Sala
        this.actividades = new ArrayList<>(otro.actividades); // Composición/referencia
        cantidadEventos++;
    }

    public void asignarSala(Sala sala) {
        this.sala = sala;
    }

    public void crearActividad(int id, String titulo, int cupo, String tipo, String extra) {
        if ("Charla".equalsIgnoreCase(tipo)) {
            actividades.add(new Charla(id, titulo, cupo, extra));
        } else {
            System.out.println("Tipo no reconocido para creación simplificada.");
        }
    }

    public void crearActividad(int id, String titulo, int cupo, String tipo, boolean requiereNotebook) {
        if ("Taller".equalsIgnoreCase(tipo)) {
            actividades.add(new Taller(id, titulo, cupo, requiereNotebook));
        } else {
            System.out.println("Tipo no reconocido para creación simplificada.");
        }
    }

    public void agregarActividad(Actividad actividad) {
        this.actividades.add(actividad);
    }

    public double calcularCostoEstimado() {
        if (gratuito) {
            return 0.0;
        }
        double costoTotal = costoBase;
        for (Actividad actividad : actividades) {
            costoTotal += actividad.calcularCostoMateriales();
        }
        return costoTotal * 1.21; // Recargo del 21% de impuestos
    }

    public void mostrarDatos() {
        System.out.println("==================================================");
        System.out.println("EVENTO UNIVERSITARIO: " + titulo + " [ID: " + id + "]");
        System.out.println("Gratuito: " + (gratuito ? "Sí" : "No"));
        System.out.println("Costo Base: $" + costoBase);
        System.out.println("Costo Estimado Final (c/impuestos): $" + calcularCostoEstimado());
        System.out.println("Sala Asignada: " + (sala != null ? sala.getNombre() : "Sin asignar"));
        System.out.println("Actividades Programadas:");
        for (Actividad act : actividades) {
            act.mostrarIdentificacion(); // Llamada polimórfica a método final
            act.mostrarInscripciones();
        }
        System.out.println("==================================================");
    }

    public static int getCantidadEventos() {
        return cantidadEventos;
    }

    public List<Actividad> getActividades() {
        return actividades;
    }
}

import modelo.*;

public class App {
    public static void main(String[] args) {
        System.out.println("--- EJECUCIÓN TP1 - PARADIGMAS DE PROGRAMACIÓN ---\n");

        // a. Registrar 3 estudiantes
        Estudiante est1 = new Estudiante("E101", "Juan Pérez");
        Estudiante est2 = new Estudiante("E102", "María Gómez");
        Estudiante est3 = new Estudiante("E103", "Carlos López");

        // b. Construir 1 Evento
        EventoUniversitario evento = new EventoUniversitario("EVT-01", "Jornadas de Inteligencia Artificial", 15000.0, false);

        // c. Crear y asignar 1 Sala al evento (Agregación)
        Sala sala1 = new Sala(101, "Aula Magna - Bloque A");
        evento.asignarSala(sala1);

        // d. Crear 2 actividades para el evento: una Charla y un Taller (Composición & Herencia)
        Charla charla = new Charla(1, "Introducción a LLMs", 30, "Dr. Roberto García");
        Taller taller = new Taller(2, "Hands-on PyTorch", 20, true);

        evento.agregarActividad(charla);
        evento.agregarActividad(taller);

        // e y f. Inscribir estudiantes en las actividades
        charla.inscribir(est1);
        charla.inscribir(est2);

        taller.inscribir(est2);
        taller.inscribir(est3);

        // Demostración constructor de copia (Ejercicio 1/2)
        EventoUniversitario eventoCopia = new EventoUniversitario(evento);

        // Muestra de resultados
        evento.mostrarDatos();

        System.out.println("\n--- TOTAL DE EVENTOS CREADOS EN MEMORIA ---");
        System.out.println("Total de instancias de EventoUniversitario: " + EventoUniversitario.getCantidadEventos());
    }
}

==================================================================================================
        STACK (Pila de Ejecución)       |                      HEAP (Montículo de Memoria)
==================================================================================================
 [ main() frame ]                       |
  │                                     |
  ├── est1 ──────────────(ref)─────────>│ [Estudiante @0x01] ── legajo: "E101", nombre: "Juan Pérez"
  ├── est2 ──────────────(ref)─────────>│ [Estudiante @0x02] ── legajo: "E102", nombre: "María Gómez"
  ├── est3 ──────────────(ref)─────────>│ [Estudiante @0x03] ── legajo: "E103", nombre: "Carlos López"
  │                                     |
  ├── sala1 ─────────────(ref)─────────>│ [Sala @0x10] ──────── id: 101, nombre: "Aula Magna - Bloque A"
  │                                     |
  ├── charla ────────────(ref)─────────┐| ┌─────────────────────────────────────────────────────────────┐
  │                                     │| │ [Charla @0x21] (Extiende de Actividad)                      │
  │                                     └┼─┼─> id: 1, titulo: "Introducción a LLMs", cupoMaximo: 30       │
  │                                      | │   disertante: "Dr. Roberto García"                         │
  │                                      | │   inscripciones ──(ref)──> [ArrayList<Inscripcion> @0x31] │
  │                                      | └──────────────────────────────────┬──────────────────────────┘
  ├── taller ────────────(ref)─────────┐|                                     │
  │                                     │| ┌──────────────────────────────────┴──────────────────────────┐
  │                                     │| │ [Taller @0x22] (Extiende de Actividad)                      │
  │                                     └┼─┼─> id: 2, titulo: "Hands-on PyTorch", cupoMaximo: 20          │
  │                                      | │   requiereNotebook: true                                    │
  │                                      | │   inscripciones ──(ref)──> [ArrayList<Inscripcion> @0x32] │
  │                                      | └──────────────────────────────────┬──────────────────────────┘
  │                                     |                                     │
  └── evento ────────────(ref)─────────>│ [EventoUniversitario @0x50]         │
                                        │   ├── id: "EVT-01"                  │
                                        │   ├── titulo: "Jornadas de IA"      │
                                        │   ├── costoBase: 15000.0            │
                                        │   ├── gratuito: false               │
                                        │   ├── sala ──────────(AGREGACIÓN)───┼─> [Sala @0x10]
                                        │   └── actividades ───(COMPOSICIÓN)──┼─> [ArrayList<Actividad> @0x40]
                                        │                                     │     ├── [0] ──> [Charla @0x21]
                                        │                                     │     └── [1] ──> [Taller @0x22]
                                        │                                     │
                                        │ [ArrayList<Inscripcion> @0x31]      │
                                        │   ├── [0] ──> [Inscripcion @0x61] ──┼──> estudiante: [Estudiante @0x01]
                                        │   └── [1] ──> [Inscripcion @0x62] ──┼──> estudiante: [Estudiante @0x02]
                                        │                                     │
                                        │ [ArrayList<Inscripcion> @0x32]      │
                                        │   ├── [0] ──> [Inscripcion @0x63] ──┼──> estudiante: [Estudiante @0x02]
                                        │   └── [1] ──> [Inscripcion @0x64] ──┴──> estudiante: [Estudiante @0x03]
==================================================================================================
