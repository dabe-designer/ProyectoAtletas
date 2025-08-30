# ProyectoAtletas
Realizar diagrama de clases Subir código a gitlab o github Desarrollar aplicación con programación orientada a objetos en Java Revisión en clase de funcionamiento

<p align="center">
<img width="467" height="747" alt="RegistroAtletas" src="https://github.com/user-attachments/assets/afd81ab5-6b59-4722-9cda-84398451ae04" />
</p>

```java

import java.util.*;
import java.time.LocalDate;
import java.time.format.DateTimeParseException;
import java.io.*;

public class RegistroAtletas {
    // -------------------- CLASES INTERNAS --------------------
    static class SesionEntrenamiento {
        private final LocalDate fecha;
        private String tipoEntrenamiento;
        private double valorRendimiento;

        public SesionEntrenamiento(LocalDate fecha, String tipoEntrenamiento, double valorRendimiento) {
            this.fecha = fecha;
            this.tipoEntrenamiento = tipoEntrenamiento;
            this.valorRendimiento = valorRendimiento;
        }

        public LocalDate getFecha() { return fecha; }
        public String getTipoEntrenamiento() { return tipoEntrenamiento; }
        public double getValorRendimiento() { return valorRendimiento; }
    }

    static class Atleta {
        private String nombre;
        private int edad;
        private String disciplina;
        private String departamento;
        private List<SesionEntrenamiento> sesiones;

        public Atleta(String nombre, int edad, String disciplina, String departamento) {
            this.nombre = nombre;
            this.edad = edad;
            this.disciplina = disciplina;
            this.departamento = departamento;
            this.sesiones = new ArrayList<>();
        }

        public void agregarSesion(SesionEntrenamiento sesion) { sesiones.add(sesion); }
        public List<SesionEntrenamiento> obtenerHistorial() { return sesiones; }

        public double calcularPromedio() {
            if (sesiones.isEmpty()) return 0;
            double suma = 0;
            for (SesionEntrenamiento s : sesiones) suma += s.getValorRendimiento();
            return suma / sesiones.size();
        }

        public double obtenerMejorMarca() {
            return sesiones.stream()
                    .mapToDouble(SesionEntrenamiento::getValorRendimiento)
                    .max()
                    .orElse(0);
        }

        public List<SesionEntrenamiento> obtenerEvolucion() {
            List<SesionEntrenamiento> copia = new ArrayList<>(sesiones);
            copia.sort(Comparator.comparing(SesionEntrenamiento::getFecha));
            return copia;
        }

        public String getNombre() { return nombre; }
        public int getEdad() { return edad; }
        public String getDisciplina() { return disciplina; }
        public String getDepartamento() { return departamento; }
    }

    static class SistemaMonitoreo {
        private List<Atleta> atletas;
        public SistemaMonitoreo() { atletas = new ArrayList<>(); }

        public void registrarAtleta(Atleta atleta) { atletas.add(atleta); }

        public Atleta buscarAtletaPorNombre(String nombre) {
            for (Atleta a : atletas) {
                if (a.getNombre().equalsIgnoreCase(nombre)) return a;
            }
            return null;
        }

        public List<Atleta> getAtletas() { return atletas; }

        // Guardar datos en formato CSV
        public void guardarDatosCSV(String archivo) throws IOException {
            try (PrintWriter pw = new PrintWriter(new FileWriter(archivo))) {
                for (Atleta atleta : atletas) {
                    for (SesionEntrenamiento sesion : atleta.obtenerHistorial()) {
                        pw.println(atleta.getNombre() + "," +
                                atleta.getEdad() + "," +
                                atleta.getDisciplina() + "," +
                                atleta.getDepartamento() + "," +
                                sesion.getFecha() + "," +
                                sesion.getTipoEntrenamiento() + "," +
                                sesion.getValorRendimiento());
                    }
                }
            }
        }

        // Cargar datos desde formato CSV
        public void cargarDatosCSV(String archivo) throws IOException {
            atletas.clear();
            Map<String, Atleta> mapa = new HashMap<>();
            try (Scanner sc = new Scanner(new File(archivo))) {
                while (sc.hasNextLine()) {
                    String[] partes = sc.nextLine().split(",");
                    if (partes.length < 7) continue; // Evitar errores si hay línea corrupta
                    String nombre = partes[0];
                    int edad = Integer.parseInt(partes[1]);
                    String disciplina = partes[2];
                    String departamento = partes[3];
                    LocalDate fecha = LocalDate.parse(partes[4]);
                    String tipo = partes[5];
                    double valor = Double.parseDouble(partes[6]);

                    Atleta atleta = mapa.getOrDefault(nombre,
                            new Atleta(nombre, edad, disciplina, departamento));
                    atleta.agregarSesion(new SesionEntrenamiento(fecha, tipo, valor));
                    mapa.put(nombre, atleta);
                }
                atletas.addAll(mapa.values());
            }
        }
    }

    // -------------------- MAIN Y MENÚ --------------------
    public static void main(String[] args) {
        SistemaMonitoreo sistema = new SistemaMonitoreo();
        Scanner scanner = new Scanner(System.in);
        int opcion;
        do {
            System.out.println("\n--- Sistema de Monitoreo de Atletas de Alto Nivel ---");
            System.out.println("1. Registrar nuevo atleta");
            System.out.println("2. Registrar sesión de entrenamiento");
            System.out.println("3. Mostrar historial y estadísticas de atleta");
            System.out.println("4. Guardar datos en archivo CSV");
            System.out.println("5. Cargar datos desde archivo CSV");
            System.out.println("0. Salir");
            System.out.print("Seleccione una opción: ");
            try {
                opcion = Integer.parseInt(scanner.nextLine());
            } catch (Exception e) {
                opcion = -1;
            }

            switch(opcion) {
                case 1: registrarAtleta(sistema, scanner); break;
                case 2: registrarSesion(sistema, scanner); break;
                case 3: mostrarHistorialYEstadisticas(sistema, scanner); break;
                case 4: guardarDatosCSV(sistema); break;
                case 5: cargarDatosCSV(sistema); break;
                case 0: System.out.println("¡Hasta luego!"); break;
                default: System.out.println("Opción inválida.");
            }
        } while(opcion != 0);

        scanner.close();
    }

    private static void registrarAtleta(SistemaMonitoreo sistema, Scanner scanner) {
        try {
            System.out.print("Nombre: ");
            String nombre = scanner.nextLine();
            System.out.print("Edad: ");
            int edad = Integer.parseInt(scanner.nextLine());
            System.out.print("Disciplina: ");
            String disciplina = scanner.nextLine();
            System.out.print("Departamento: ");
            String departamento = scanner.nextLine();
            sistema.registrarAtleta(new Atleta(nombre, edad, disciplina, departamento));
            System.out.println("Atleta registrado correctamente.");
        } catch (Exception e) {
            System.out.println("Error: datos inválidos.");
        }
    }

    private static void registrarSesion(SistemaMonitoreo sistema, Scanner scanner) {
        System.out.print("Nombre del atleta: ");
        String nombre = scanner.nextLine();
        Atleta atleta = sistema.buscarAtletaPorNombre(nombre);
        if (atleta == null) {
            System.out.println("Atleta no encontrado.");
            return;
        }
        try {
            System.out.print("Fecha (YYYY-MM-DD): ");
            LocalDate fecha = LocalDate.parse(scanner.nextLine());
            System.out.print("Tipo de entrenamiento: ");
            String tipo = scanner.nextLine();
            System.out.print("Valor de rendimiento (tiempo, peso, etc.): ");
            double valor = Double.parseDouble(scanner.nextLine());
            atleta.agregarSesion(new SesionEntrenamiento(fecha, tipo, valor));
            System.out.println("Sesión registrada correctamente.");
        } catch (DateTimeParseException e) {
            System.out.println("Formato de fecha inválido. Use YYYY-MM-DD.");
        } catch (NumberFormatException e) {
            System.out.println("Valor de rendimiento inválido.");
        }
    }

    private static void mostrarHistorialYEstadisticas(SistemaMonitoreo sistema, Scanner scanner) {
        System.out.print("Nombre del atleta: ");
        String nombre = scanner.nextLine();
        Atleta atleta = sistema.buscarAtletaPorNombre(nombre);
        if (atleta == null) {
            System.out.println("Atleta no encontrado.");
            return;
        }
        if (atleta.obtenerHistorial().isEmpty()) {
            System.out.println("No hay sesiones registradas para este atleta.");
            return;
        }
        System.out.println("\nHistorial de sesiones:");
        for (SesionEntrenamiento sesion : atleta.obtenerEvolucion()) {
            System.out.println(sesion.getFecha() + " | " +
                    sesion.getTipoEntrenamiento() + " | " +
                    sesion.getValorRendimiento());
        }
        System.out.println("Promedio de rendimiento: " + atleta.calcularPromedio());
        System.out.println("Mejor marca: " + atleta.obtenerMejorMarca());
    }

    private static void guardarDatosCSV(SistemaMonitoreo sistema) {
        try {
            sistema.guardarDatosCSV("atletas.csv");
            System.out.println("Datos guardados correctamente en atletas.csv");
        } catch (Exception e) {
            System.out.println("Error al guardar los datos: " + e.getMessage());
        }
    }

    private static void cargarDatosCSV(SistemaMonitoreo sistema) {
        try {
            sistema.cargarDatosCSV("atletas.csv");
            System.out.println("Datos cargados correctamente desde atletas.csv");
        } catch (Exception e) {
            System.out.println("Error al cargar los datos: " + e.getMessage());
        }
    }
}


https://github.com/dabe-designer/ProyectoAtletas.git

gh repo clone dabe-designer/ProyectoAtletas








