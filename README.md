# martin-informatica
Nombre del Sistema Operativo: Athelon OS (derivado de "Atleta" y "On" - encendido/activo)

Propósito: Ser el sistema operativo de base para dispositivos wearables inteligentes (biosensores, ropa técnica, etc.) y sistemas de entrenamiento inmersivos (VR/AR).

Su función principal es recolectar, procesar y analizar en tiempo real datos biométricos, biomecánicos y ambientales para generar prescripciones de entrenamiento y alertas predictivas de riesgo de lesión con una latencia mínima.

2. Tipo de núcleo elegido y justificación
Tipo de Núcleo Elegido: Microkernel
Justificación:
Estabilidad y Fiabilidad Crítica: En el deporte de alto rendimiento, un fallo del sistema (un crash) mientras se monitoriza una carga crítica o se realiza un diagnóstico puede ser muy peligroso. En un microkernel, si un controlador de sensor (módulo) falla, no tumba el núcleo completo, garantizando la continuidad de las funciones vitales (como la alerta de paro cardíaco o lesión inminente).
Seguridad Modular: El núcleo solo maneja las funciones más básicas (comunicación interproceso, gestión de memoria básica). Los servicios críticos (Driver de GPS, Driver de ECG, Algoritmo de Predicción de Lesiones por IA) se ejecutan como procesos separados. Esto aísla los componentes, mejorando la seguridad y el control de acceso a datos sensibles.

Modularidad y Adaptabilidad: Facilita añadir o actualizar rápidamente nuevos módulos de sensores (por ejemplo, un nuevo tipo de biosensor de lactato) sin tener que recompilar o modificar el núcleo principal.

4. Gestión de procesos: planificación, estados, concurrencia
Planificación (Scheduling): Planificación por Prioridades Preemptiva y Planificación por Tiempos Compartidos (Time-Sharing).
Alta Prioridad (Preemptiva): Procesos de análisis de datos en tiempo real (IA/ML) y sistemas de alerta de seguridad (ej: monitorización del ritmo cardíaco, detección de pisada incorrecta). Estos interrumpen inmediatamente cualquier otra tarea.
Baja Prioridad (Time-Sharing/Round Robin): Procesos de sincronización de datos en la nube, actualización de interfaz gráfica o cálculo de estadísticas históricas.
Estados de Procesos:
Ejecución (Running): El proceso está utilizando la CPU (ej: el modelo de IA está procesando los datos).
Listo (Ready): El proceso está esperando ser asignado a la CPU (ej: un nuevo paquete de datos de acelerómetro ha llegado y está listo para ser procesado).
Bloqueado (Blocked/Waiting): El proceso está esperando una operación de E/S (ej: esperando que el módulo GPS reciba una señal, esperando escribir un bloque de datos en el sistema de archivos).
Suspensión Crítica (Critical Suspend): Un estado exclusivo para reducir el consumo energético en periodos de baja actividad, manteniendo solo los procesos de monitorización de seguridad esenciales.
Concurrencia: Uso intensivo de Multithreading para paralelizar la ingesta de datos. Cada sensor (ritmo cardíaco, GPS, giroscopio, etc.) puede tener su propio thread de recolección para evitar cuellos de botella y garantizar que el análisis en tiempo real se realice sin retrasos. Uso de semáforos y monitores para proteger las regiones críticas donde se almacenan los datos brutos antes de ser procesados.

6. Gestión de memoria: segmentación, paginación, asignación
Estrategia Principal: Paginación con Asignación de Swapping (para sistemas con almacenamiento limitado)
Segmentación: Se utiliza para aislar lógicamente los diferentes tipos de procesos críticos:
Segmento del Núcleo: Código y datos esenciales del microkernel.
Segmento de Sensor-Data Buffer: Área de alta velocidad (DRAM) para el almacenamiento temporal de datos de sensores en tiempo real.
Segmento del Algoritmo de IA (Predictor): Código ejecutable y modelos de aprendizaje automático.
Paginación: Se usa para gestionar eficientemente la memoria física (RAM) y permitir que los procesos sean cargados en cualquier marco de página disponible. Esto es crucial para un sistema que maneja muchos procesos pequeños y concurrentes.
Asignación: Se empleará la asignación contigua mediante la estrategia Best-Fit o First-Fit solo para el Segmento de Sensor-Data Buffer para asegurar bloques de memoria grandes y contiguos que maximicen la velocidad de transferencia de datos de los sensores. Para el resto de procesos se usa la paginación.

8. Sistema de archivos: estructura, permisos, jerarquía
Estructura: Sistema de Archivos No Volátil Estructurado (NVFS - Non-Volatile Structured File System). Optimizado para la escritura rápida de grandes flujos de datos y con alta resistencia a fallos de energía.
Permisos (Basado en Roles):
Núcleo (Root): Acceso total.
Atleta: Solo puede leer sus propios archivos de entrenamiento y ejecutar aplicaciones de terceros.
Entrenador (Coach): Puede leer y escribir (modificar) planes de entrenamiento. Puede leer datos de rendimiento del atleta.
Equipo Médico (Med): Puede leer datos fisiológicos sensibles (ECG, historial de lesiones) y escribir protocolos de recuperación.
Jerarquía:
/sys: Configuración del sistema y drivers (solo root).
/data: Base de datos de rendimiento bruto (solo lectura para Entrenador y Med).
/apps: Aplicaciones y modelos de terceros (ejecutables).
/usr/athn_[ID]: Directorio individual del atleta (separación de usuarios).
/usr/athn_[ID]/bio: Datos fisiológicos sensibles.
/usr/athn_[ID]/plans: Planes de entrenamiento activos.
/usr/athn_[ID]/meta: Metadatos y configuraciones de dispositivos.

10. Mecanismos de seguridad: autenticación, control de acceso, cifrado
Autenticación:
Doble Factor Biométrico (Integrado): Requiere un PIN o patrón + verificación de huella dactilar o patrón de latido cardíaco (tomado del sensor ECG del propio dispositivo).
Control de Acceso: Se implementa un Control de Acceso Basado en Roles (RBAC) estricto, como se detalla en el punto 5, garantizando que el Entrenador no pueda acceder a archivos médicos críticos sin la aprobación explícita del Atleta o del Médico.
Cifrado:
Cifrado de disco completo (Full Disk Encryption): Todos los datos almacenados se cifran por defecto con un algoritmo robusto (ej: AES-256).
Cifrado en tránsito: Uso de protocolos de seguridad (TLS 1.3) y end-to-end encryption para la sincronización de datos con el servidor en la nube (la "Bóveda de Datos Deportivos").
Taint Checking (Análisis de Contaminación): El núcleo rastrea el origen de los datos de los sensores para prevenir la inyección de datos falsos por malware que podría llevar a un diagnóstico de entrenamiento erróneo o peligroso.

12. Interfaz de usuario: CLI, GUI, voz, gestos, etc.
GUI (Gráfica): Interfaz minimalista, basada en realidad aumentada (AR) para dispositivos de gafas y holoproyectores en el gimnasio. Muestra métricas clave, zonas de esfuerzo codificadas por colores y superpone indicaciones de corrección de técnica directamente en el campo de visión.
Voz: Es el mecanismo de interacción principal para la prescripción de entrenamiento.
Entrada: El atleta da órdenes como: "Athelon, iniciar secuencia de carrera L5" o "Athelon, revisar técnica de levantamiento".
Salida: Retroalimentación en tiempo real: "Frecuencia cardíaca alta, reducir ritmo 5%" o "Ángulo de rodilla incorrecto, ajustar a 160 grados".
Gestos (Para wearables): Uso de movimientos predefinidos de la mano o muñeca para tareas rápidas (ej: un toque para marcar un "lap", dos toques para detener la sesión).
CLI (Command Line Interface): Oculta y reservada exclusivamente para administradores del club y desarrolladores del sistema para tareas de diagnóstico, mantenimiento y parches de seguridad de bajo nivel.
9En resumen, la actividad me obligó a trasladar los conceptos teóricos (paginación, semáforos, tipos de kernel) a soluciones concretas y justificadas para un problema real y demandante: la optimización del rendimiento humano.
