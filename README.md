
# Laboratorio: Índice Pletismográfico Quirúrgico (SPI)

## Objetivos

- Revisar la literatura para definir matemáticamente el Índice Pletismográfico Quirúrgico (SPI).
- Implementar un algoritmo en MATLAB para calcular el SPI a partir de la señal PPG.
- Evaluar la variación del SPI durante un estímulo nociceptivo (Cold Pressor Test).
- Analizar el uso del SPI como indicador de dolor y activación autonómica.

---

# Marco teórico

## Actividad autonómica y nocicepción

El sistema nervioso autónomo regula funciones involuntarias como la frecuencia cardíaca y el tono vascular. Ante estímulos nociceptivos, se activa el sistema simpático, produciendo vasoconstricción periférica y aumento de la frecuencia cardíaca [1].

La nocicepción corresponde al proceso fisiológico de detección de estímulos potencialmente dañinos, mientras que el dolor es una experiencia subjetiva que involucra componentes emocionales y cognitivos [2].

---

## Fotopletismografía (PPG)

La fotopletismografía es una técnica óptica que permite medir cambios en el volumen sanguíneo periférico mediante la absorción de luz en los tejidos [3]. Esta señal contiene información relevante sobre:

- Frecuencia cardíaca  
- Variabilidad del pulso  
- Cambios en la perfusión periférica  

Durante activación simpática, la amplitud de la señal PPG disminuye debido a vasoconstricción [4].

---

## Definición del Índice Pletismográfico Quirúrgico (SPI)

El SPI es un índice desarrollado para cuantificar el balance entre nocicepción y analgesia durante procedimientos quirúrgicos [5].

Se calcula a partir de dos variables derivadas de la señal PPG:

- PPGA (Photoplethysmographic Pulse Wave Amplitude)  
- HBI (Heartbeat Interval)  

Su expresión matemática es:


$$SPI = 100 - (0.33 \cdot PPGA_{norm} + 0.67 \cdot HBI_{norm})$$


Donde:
- $PPGA_{norm}$: amplitud normalizada del pulso  
- $HBI_{norm}:$ intervalo entre latidos normalizado

Valores típicos:
- 20 – 50 analgesia adecuada  
- Cuando el índice supera los 50, hay una posible activación nociceptiva [5]

---

## Fundamento fisiológico

El SPI refleja cambios en el balance autonómico:

- Aumento del simpático → disminución de la amplitud PPG y aumento de la frecuencia cardíaca  
- Aumento del parasimpático → aumento de la amplitud PPG y disminución de la frecuencia cardíaca  

Esto lo convierte en un indicador indirecto de la actividad nociceptiva [6].

---

# Metodología

## Adquisición de señal

- Sensor: PPG óptico en dedo  
- Duración: 120 segundos  
- Protocolo:
  - 0–40 s: reposo  
  - 40–80 s: Cold Pressor Test (CPT)  
  - 80–120 s: recuperación  

El CPT consiste en sumergir la mano en agua fría para inducir activación simpática [7].

---

# Implementación en MATLAB

## Captura y cálculo del SPI


1. Entrada del usuario
Solicita la duración de la captura en segundos y valida que sea un valor positivo.

2. Comunicación serial
Abre la conexión con Arduino en el puerto COM10 a 115200 baudios, define el terminador de línea y limpia el buffer de datos.

3. Filtro pasa banda
Diseña un filtro Butterworth entre 0.5 y 3 Hz para aislar la señal cardíaca y eliminar ruido de baja y alta frecuencia.

4. Configuración de la gráfica
Crea dos líneas animadas: una para la señal PPG y otra para los picos detectados, además define etiquetas y límites del eje.

5. Inicialización del tiempo
Guarda el tiempo inicial para calcular el eje temporal en segundos durante la adquisición.

6. Variables de detección
Inicializa variables para calcular derivadas, controlar el tiempo entre picos y evitar detecciones múltiples (periodo refractario).

7. Variables para SPI
Define buffers para almacenar los últimos valores de intervalo entre latidos (HBI) y amplitud del pulso (PPGA), usados en el cálculo del índice.

8. Baseline
Crea un buffer para estimar dinámicamente el nivel base de la señal.

9. Bucle principal
Ejecuta la adquisición y procesamiento de datos hasta completar el tiempo definido por el usuario.

10. Lectura de datos
Lee valores desde el puerto serial, los convierte a número y descarta datos inválidos.

11. Conversión a voltaje
Escala el valor leído a voltaje suponiendo un ADC de 10 bits.

12. Filtrado de la señal
Aplica el filtro pasa banda para limpiar la señal antes de analizarla.

13. Suavizado
Aplica una media móvil para mejorar la visualización, sin afectar la detección de picos.

14. Cálculo de derivadas
Calcula la primera y segunda derivada para identificar cambios en la pendiente y curvatura de la señal.

15. Detección de picos
Detecta un pico cuando la señal pasa de subir a bajar, presenta curvatura negativa y se respeta el tiempo mínimo entre latidos.

16. Registro del pico
Guarda el valor y tiempo del pico detectado y lo muestra en la gráfica.

17. Cálculo de HBI
Calcula el intervalo de tiempo entre dos picos consecutivos.

18. Cálculo del baseline
Estima el nivel base como el promedio de las muestras recientes.

19. Cálculo de PPGA
Obtiene la amplitud del pulso como la diferencia entre el pico y el baseline.

20. Actualización de buffers
Almacena los valores recientes de HBI y PPGA en ventanas deslizantes de tamaño fijo.

21. Normalización
Escala HBI y PPGA a un rango de 0 a 100 usando los valores mínimos y máximos del buffer.

22. Cálculo del SPI
Combina PPGA y HBI normalizados para obtener un índice de perfusión ajustado.

23. Salida en consola
Imprime el valor del SPI en tiempo real.

24. Actualización del baseline buffer
Mantiene un conjunto de muestras recientes para recalcular continuamente la línea base.

25. Visualización de la señal
Muestra la señal filtrada y suavizada en función del tiempo, después de un periodo inicial de estabilización.

26. Gráfica final del SPI
Al finalizar, muestra la evolución del SPI en el tiempo
<img width="1410" height="788" alt="image" src="https://github.com/user-attachments/assets/fc0d39a6-d558-4217-bb2e-456f17c1ee8c" />


<img width="1410" height="1284" alt="image" src="https://github.com/user-attachments/assets/b2d69883-5ac3-42f5-a534-584e77531358" />
```

```

# Resultados y análisis

## Comparación con valores clínicos

Los resultados obtenidos durante la práctica muestran un comportamiento coherente del Índice Pletismográfico Quirúrgico (SPI) en relación con las distintas fases del protocolo experimental. Durante la fase inicial de reposo, los valores del SPI se mantuvieron relativamente estables y dentro de un rango bajo, lo cual es consistente con un estado de baja activación simpática y predominio parasimpático. Este comportamiento concuerda con los rangos reportados en la literatura clínica, donde valores entre 20 y 50 se consideran indicativos de un nivel adecuado de analgesia en pacientes sometidos a anestesia general [5].

Al iniciar el Cold Pressor Test (CPT), se evidenció un incremento significativo en los valores del SPI. Este aumento puede explicarse por la activación del sistema nervioso simpático frente al estímulo frío, el cual induce vasoconstricción periférica y un aumento de la frecuencia cardíaca. Desde el punto de vista de la señal PPG, esto se traduce en una disminución de la amplitud del pulso (PPGA) y una reducción del intervalo entre latidos (HBI). Dado que el SPI se calcula a partir de estas dos variables normalizadas, dichos cambios provocan un incremento en el índice, reflejando una mayor actividad nociceptiva. Este comportamiento ha sido ampliamente documentado en estudios clínicos donde el SPI responde de forma sensible a estímulos dolorosos agudos [5,6].

Durante la fase de recuperación, una vez finalizado el estímulo, los valores del SPI mostraron una tendencia decreciente, aproximándose gradualmente a los niveles basales. Este fenómeno refleja la disminución de la actividad simpática y la restauración del equilibrio autonómico, evidenciando la capacidad del SPI para capturar dinámicamente cambios en la fisiología del individuo en función del contexto experimental.

---

## Alcance y limitaciones del SPI

El SPI representa una herramienta valiosa para la estimación indirecta de la nocicepción debido a su carácter no invasivo y su capacidad de proporcionar información en tiempo real. Su implementación a partir de señales PPG lo hace especialmente atractivo, ya que permite utilizar sensores de bajo costo y fácil integración en sistemas portátiles. En el contexto clínico, su principal aplicación se encuentra en la monitorización del balance entre nocicepción y analgesia durante procedimientos quirúrgicos, donde la evaluación subjetiva del dolor no es posible [6].

Sin embargo, es importante destacar que el SPI no mide el dolor como experiencia subjetiva, sino la respuesta fisiológica asociada a la activación del sistema nervioso autónomo. Esto implica que factores distintos al dolor, como el estrés emocional, la ansiedad o cambios en la temperatura, pueden generar respuestas similares en el índice. Por lo tanto, su interpretación debe realizarse con cautela y en conjunto con otros parámetros clínicos.

Adicionalmente, la señal PPG es susceptible a artefactos de movimiento, variaciones en la presión de contacto del sensor y cambios en la perfusión periférica, lo que puede afectar la calidad de la señal y, por ende, la precisión del SPI. Estas limitaciones son particularmente relevantes en entornos no controlados, donde el movimiento del usuario es inevitable. A pesar de ello, el SPI sigue siendo una herramienta útil cuando se emplea bajo condiciones adecuadas y con un procesamiento de señal robusto.

---

# Preguntas de discusión

## Relación entre volumen sanguíneo periférico y balance autonómico

Las variaciones del volumen sanguíneo periférico están directamente relacionadas con el balance entre los sistemas simpático y parasimpático del sistema nervioso autónomo. En situaciones de activación simpática, como las inducidas por el dolor o el estrés, se produce vasoconstricción periférica, lo que reduce el flujo sanguíneo en los capilares y disminuye la amplitud de la señal PPG. Por el contrario, en estados de relajación, el predominio parasimpático favorece la vasodilatación, incrementando el volumen sanguíneo periférico y la amplitud de la señal.

Este comportamiento permite utilizar la señal PPG como un indicador indirecto del estado autonómico, ya que los cambios en la amplitud reflejan modificaciones en el tono vascular. En este contexto, el SPI integra esta información junto con la variabilidad cardíaca para estimar el nivel de activación simpática, lo que lo convierte en un indicador útil de la respuesta fisiológica ante estímulos nociceptivos [6].

---

## Comparación con otros índices

El SPI se diferencia de otros índices utilizados en anestesia, como el índice de nocicepción-analgesia (ANI) y el índice de perfusión, en los mecanismos fisiológicos que evalúa. Mientras que el ANI se basa en la variabilidad de la frecuencia cardíaca y refleja principalmente la actividad parasimpática, el SPI se centra en la actividad simpática mediante el análisis de la señal PPG. Por su parte, el índice de perfusión proporciona información sobre el flujo sanguíneo periférico, pero no está diseñado específicamente para evaluar la nocicepción.

Estas diferencias implican que cada índice ofrece una perspectiva distinta del estado fisiológico del paciente. En la práctica clínica, la combinación de múltiples indicadores puede proporcionar una evaluación más completa del balance autonómico, permitiendo una mejor toma de decisiones en la administración de anestesia y analgesia [6].

---

# Conclusiones

El desarrollo de esta práctica permitió comprender en profundidad el funcionamiento del Índice Pletismográfico Quirúrgico (SPI) tanto desde su formulación matemática como desde su fundamento fisiológico. A partir de la implementación del algoritmo en MATLAB, se evidenció que es posible calcular este índice utilizando únicamente la señal PPG, extrayendo información relevante como la amplitud del pulso y el intervalo entre latidos.

Los resultados obtenidos durante el Cold Pressor Test demostraron que el SPI es sensible a cambios en la activación del sistema nervioso simpático, reflejando de manera adecuada la respuesta del organismo ante un estímulo nociceptivo. Este comportamiento confirma su utilidad como herramienta de monitoreo en contextos donde no es posible evaluar el dolor de manera subjetiva, como en pacientes bajo anestesia.

No obstante, también se identificaron limitaciones importantes asociadas a la naturaleza del índice. En particular, el SPI no permite distinguir entre diferentes fuentes de activación simpática, lo que implica que su interpretación debe realizarse dentro de un contexto adecuado. Asimismo, la dependencia de la señal PPG introduce vulnerabilidad frente a artefactos y condiciones de medición no ideales.

Finalmente, esta práctica permitió establecer una distinción clara entre los conceptos de dolor y nocicepción. Mientras que el dolor es una experiencia subjetiva influenciada por factores emocionales y cognitivos, la nocicepción corresponde a un proceso fisiológico objetivo que puede ser medido mediante indicadores como el SPI. Esta diferencia es fundamental en la práctica clínica, ya que resalta la necesidad de complementar las mediciones instrumentales con una evaluación integral del paciente.

---

# Referencias

[1] Guyton, A. C., & Hall, J. E. *Textbook of Medical Physiology*, 13th ed.  
[2] International Association for the Study of Pain (IASP), 2020.  
[3] Allen, J., “Photoplethysmography and its application,” Physiol Meas, 2007.  
[4] Shelley, K. H., “Photoplethysmography: Beyond the calculation of arterial oxygen saturation,” Anesth Analg, 2007.  
[5] Huiku, M. et al., “Assessment of surgical stress during general anaesthesia,” Br J Anaesth, 2007.  
[6] Logier, R. et al., “Physiological signals for nociception monitoring,” IEEE Trans Biomed Eng, 2010.  
[7] Mitchell, L. A. et al., “Cold pressor test,” J Vis Exp, 2004.  
