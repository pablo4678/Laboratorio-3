
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

<img width="843" height="422" alt="image" src="https://github.com/user-attachments/assets/49406095-0cce-46b5-ab24-0693b0f1341e" />

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

Para tomar y analizar la señal, se usó el sensor MAX30102 usado para fotopletismografía. Para el procesamiento de la señal se empleó un filtro pasa banda Butterworth entre 0.5 y 3 Hz, adecuado para aislar el rango típico de la frecuencia cardíaca y eliminar tanto la deriva de baja frecuencia como el ruido de alta frecuencia. La detección de picos se realizó mediante un método basado en derivadas, identificando los máximos locales cuando la señal cambia de pendiente positiva a negativa y presenta curvatura descendente, junto con un periodo refractario que evita múltiples detecciones por latido, logrando así una identificación más precisa frente al ruido en comparación con otros métodos basados únicamente en umbrales. 

<img width="736" height="302" alt="image" src="https://github.com/user-attachments/assets/0e391537-b3f9-48a6-8b8d-2cbf7694c810" />
<img width="443" height="118" alt="image" src="https://github.com/user-attachments/assets/1b9cde40-2177-47c7-a8dd-4a4272197f93" />

A continuación se describe cómo funciona el código de procesamiento y detección:

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

<img width="451" height="594" alt="image" src="https://github.com/user-attachments/assets/7fa2fad2-fb75-4340-a8c4-a143166c71a5" />


# Resultados y análisis

## Comparación con valores clínicos
<img width="767" height="449" alt="image" src="https://github.com/user-attachments/assets/27fdbf9a-83a3-4717-8436-555db3d48c60" />

Los resultados obtenidos evidencian que el comportamiento del Índice Pletismográfico Quirúrgico (SPI) no fue completamente estable durante la fase inicial de reposo, presentando una variabilidad considerable en los primeros segundos del registro. Esta fluctuación puede atribuirse a factores como la adaptación del sensor, el movimiento del sujeto o la estabilización del procesamiento de la señal (filtrado y normalización), lo cual afecta directamente el cálculo del SPI.

A partir de aproximadamente el segundo 40, coincidiendo con la aplicación del estímulo frío, se observa un incremento sostenido en los valores del SPI. Este aumento es consistente con la activación del sistema nervioso simpático ante un estímulo nociceptivo, lo que genera vasoconstricción periférica y cambios en la señal PPG, particularmente una disminución de la amplitud del pulso (PPGA) y una reducción del intervalo entre latidos (HBI). Estos cambios se reflejan en un incremento del SPI, evidenciando su sensibilidad frente a estímulos dolorosos.[5,6]

Tras la retirada del estímulo, se aprecia una disminución en los valores del SPI, lo que sugiere una reducción de la activación simpática. Sin embargo, los valores no retornan completamente a los niveles iniciales observados al inicio del experimento. Este comportamiento puede estar relacionado con una recuperación fisiológica incompleta, efectos residuales del estímulo o la variabilidad inherente al sistema de medición.

En conjunto, los resultados muestran que el SPI logra capturar la respuesta al estímulo nociceptivo, aunque su comportamiento en reposo y recuperación puede verse afectado por la variabilidad de la señal y las condiciones experimentales.

---

## Alcance y limitaciones del SPI

El SPI representa una herramienta valiosa para la estimación indirecta de la nocicepción debido a su carácter no invasivo y su capacidad de proporcionar información en tiempo real. Su implementación a partir de señales PPG lo hace especialmente atractivo, ya que permite utilizar sensores de bajo costo y fácil integración en sistemas portátiles. En el contexto clínico, su principal aplicación se encuentra en la monitorización del balance entre nocicepción y analgesia durante procedimientos quirúrgicos, donde la evaluación subjetiva del dolor no es posible [6].

Sin embargo, es importante destacar que el SPI no mide el dolor como experiencia subjetiva, sino la respuesta fisiológica asociada a la activación del sistema nervioso autónomo. Esto implica que factores distintos al dolor, como el estrés emocional, la ansiedad o cambios en la temperatura, pueden generar respuestas similares en el índice. Por lo tanto, su interpretación debe realizarse con cautela y en conjunto con otros parámetros clínicos.

Adicionalmente, la señal PPG es susceptible a artefactos de movimiento, variaciones en la presión de contacto del sensor y cambios en la perfusión periférica, lo que puede afectar la calidad de la señal y, por ende, la precisión del SPI. Estas limitaciones son particularmente relevantes en entornos no controlados, donde el movimiento del usuario es inevitable. A pesar de ello, el SPI sigue siendo una herramienta útil cuando se emplea bajo condiciones adecuadas y con un procesamiento de señal adecuado.

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

Los resultados obtenidos durante el Cold Pressor Test demostraron que el SPI es sensible a cambios en la activación del sistema nervioso simpático, reflejando de manera adecuada la respuesta del organismo ante un estímulo nociceptivo. En particular, se observó que, tras la aplicación del estímulo frío, los valores del SPI aumentaron de forma notable, lo cual es consistente con la respuesta fisiológica esperada. Sin embargo, durante la fase inicial de reposo se evidenció una alta variabilidad en el índice, y tras la retirada del estímulo no se presentó un retorno completo a los valores iniciales. Este comportamiento puede atribuirse a procesos de estabilización de la señal, variabilidad fisiológica del sujeto y posibles limitaciones en el procesamiento de la señal.

No obstante, también se identificaron limitaciones importantes asociadas a la naturaleza del índice. En particular, el SPI no permite distinguir entre diferentes fuentes de activación simpática, lo que implica que su interpretación debe realizarse dentro de un contexto adecuado. Asimismo, la dependencia de la señal PPG introduce vulnerabilidad frente a artefactos, ruido y errores en la detección de características clave como picos y valles.

En este sentido, se evidenció que el uso de métodos básicos de detección de picos, como los implementados mediante funciones genéricas como el find_peaks, puede no ser suficiente para señales PPG en tiempo real. Por ello, es recomendable emplear algoritmos de detección más  especializados para este tipo de señales, como el método del alpinista, el cual permite un seguimiento más fiable de la morfología de la señal y mejora la identificación de máximos y mínimos incluso en presencia de ruido. La implementación de este tipo de enfoques puede contribuir significativamente a la estabilidad y precisión del cálculo del SPI.

---

# Referencias

[1] Guyton, A. C., & Hall, J. E. *Textbook of Medical Physiology*, 13th ed.  
[2] International Association for the Study of Pain (IASP), 2020.  
[3] Allen, J., “Photoplethysmography and its application,” Physiol Meas, 2007.  
[4] Shelley, K. H., “Photoplethysmography: Beyond the calculation of arterial oxygen saturation,” Anesth Analg, 2007.  
[5] Huiku, M. et al., “Assessment of surgical stress during general anaesthesia,” Br J Anaesth, 2007.  
[6] Logier, R. et al., “Physiological signals for nociception monitoring,” IEEE Trans Biomed Eng, 2010.  
[7] Mitchell, L. A. et al., “Cold pressor test,” J Vis Exp, 2004.  
