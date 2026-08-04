# MONITOREO DEL PATRON Y FRECUENCIA RESPIRATORIA

**Luz Marina Valderrama-5600741**

## INTRODUCCION
La respiración es un proceso fisiológico indispensable para el intercambio de oxígeno y dióxido de carbono entre el organismo y el medio ambiente. La frecuencia respiratoria constituye uno de los signos vitales más importantes, ya que permite evaluar el estado fisiológico de una persona y detectar posibles alteraciones clínicas. En esta práctica se desarrolló un sistema de adquisición de la señal respiratoria utilizando un sensor resistivo sensible a la fuerza (FSR400), una placa Arduino Uno y MATLAB, con el propósito de obtener el patrón respiratorio y analizar las diferencias existentes entre la respiración en reposo y durante una actividad de verbalización.
## OBJETIVOS

### Objetivo General: 
Evaluar la influencia del habla o verbalización sobre el patrón respiratorio.
### Objetivos Específicos
• Reconocer las variables físicas principalmente involucradas en el proceso
respiratorio.

• Desarrollar un sistema que extraiga el patrón respiratorio y la frecuencia
respiratoria.

• Identificar tareas de verbalización a partir del patrón y/o la frecuencia
respiratoria.

## METODOLOGIA
Inicialmente se seleccionó un sensor FSR400 debido a que permite detectar cambios de presión causados por el movimiento de la caja torácica durante la respiración. El sensor se conectó mediante un divisor de voltaje alimentado con 5 V utilizando una resistencia fija de 47 kΩ. Durante las primeras pruebas se observó que valores menores de resistencia producían una variación muy pequeña en la salida del sensor, por lo que fue necesario aumentar dicho valor para mejorar la sensibilidad del sistema.

Con el fin de incrementar aún más la respuesta del sensor a los pequeños movimientos producidos por la respiración, se colocó una pequeña espuma alrededor de la zona sensible del FSR. Esta espuma permitió mantener una presión inicial constante sobre el sensor, haciendo que pequeñas expansiones y contracciones del tórax generaran cambios más notorios en la señal registrada.

Posteriormente, el punto medio del divisor de voltaje se conectó a la entrada analógica A0 de una placa Arduino Uno. Para la adquisición de la señal se desarrolló un programa sencillo en Arduino IDE, mostrado en el Código 1.

### Programa en Arduino UNO

    void setup() {
    Serial.begin(9600);
    }

    void loop() {
    int dato = analogRead(A0);
    Serial.println(dato);
    delay(80);
    }
En la función setup() se inicializó la comunicación serial a una velocidad de 9600 baudios, permitiendo el envío constante de datos hacia MATLAB. Posteriormente, dentro de la función loop(), el Arduino realizó la lectura del sensor mediante analogRead(A0) y transmitió cada muestra utilizando Serial.println(). Finalmente, la instrucción delay(80) estableció un intervalo aproximado de 80 ms entre muestras, una frecuencia de adquisición cercana a 10 Hz, suficiente para registrar la respiración humana.

Antes de colocar el sensor sobre el cuerpo, se verificó el funcionamiento del circuito presionando manualmente el FSR y observando la respuesta mediante el Serial Plotter de Arduino IDE. Una vez comprobado su funcionamiento, el sensor fue ubicado sobre una de las costillas del sujeto de prueba y asegurado mediante una banda elástica para mantener un contacto constante durante toda la adquisición.

Se realizaron dos registros experimentales. El primero correspondió a la respiración en reposo, mientras que el segundo se obtuvo durante una lectura en voz alta con el propósito de evaluar la influencia de la verbalización sobre el patrón respiratorio.

Posteriormente se desarrolló un programa en MATLAB para capturar, procesar y analizar la información enviada por el Arduino. Las principales etapas del procesamiento se describen a continuación.

### Comunicación serial
    puerto = serialport("COM3",9600);
    pause(2)
    
    senal = [];
    
    for i = 1:300
        texto = readline(puerto);
        senal(i) = str2double(texto);
    end
Inicialmente se abrió el puerto serial utilizando la función serialport(), especificando el puerto COM y la velocidad de transmisión de 9600 baudios. Después de una pausa de dos segundos para garantizar la comunicación, se ejecutó un ciclo for que recibió 300 muestras provenientes del Arduino mediante la función readline(). Cada dato recibido fue convertido de texto a formato numérico utilizando str2double() y almacenado dentro del vector senal.

### Almacenamiento de la información
    save("senal_hablando.mat","senal")
Este procedimiento permitió conservar los datos experimentales para realizar el procesamiento y el análisis sin necesidad de repetir la adquisición.

### Correcion de polaridad y filtrado
    senal = 1023 - senal;
    senal_filtrada = movmean(senal,5);
La inversión de la señal se realizó porque la configuración utilizada en el divisor de voltaje producía una respuesta invertida respecto al movimiento respiratorio; de esta forma, durante la inspiración los valores aumentaban en lugar de disminuir, facilitando la interpretación de los resultados. Posteriormente se aplicó un filtro de media móvil con una ventana de cinco muestras mediante la función movmean(), reduciendo pequeñas fluctuaciones y suavizando el comportamiento de la señal sin modificar significativamente su patrón respiratorio.

Finalmente, ambas señales fueron representadas gráficamente para comparar el efecto del filtrado.

    figure
    plot(senal)
    hold on
    plot(senal_filtrada,'LineWidth',2)
### Analisis en frecuencia
    fs = 10;
    N = length(senal_filtrada);
    senal_fft = senal_filtrada - mean(senal_filtrada);
    Y = abs(fft(senal_fft));
    f = (0:N-1)*fs/N;
En primer lugar se definió una frecuencia de muestreo aproximada de 10 Hz, la cual corresponde al intervalo de adquisición utilizado por el Arduino. Posteriormente se eliminó el valor promedio de la señal para reducir la componente continua y mejorar el análisis espectral. Después se calculó la FFT mediante la función fft(), obteniendo el espectro de magnitud correspondiente.

Finalmente, se identificó automáticamente la frecuencia dominante y se calculó la frecuencia respiratoria en respiraciones por minuto.

    [valor,pos] = max(Y(2:N/2));
    frecuencia = f(pos+1);
    disp("Frecuencia dominante (Hz):")
    disp(frecuencia)
    disp("Respiraciones por minuto:")
    disp(frecuencia*60)
La frecuencia dominante corresponde al componente principal presente en el espectro de la señal respiratoria. Multiplicando este valor por 60 se obtuvo la frecuencia respiratoria expresada en respiraciones por minuto, permitiendo comparar los resultados obtenidos durante la respiración en reposo y durante la verbalización.
## DISCUSION
Durante las primeras pruebas el sistema presentó poca sensibilidad para detectar los movimientos respiratorios cuando se utilizó un divisor de voltaje con una resistencia de menor valor. En estas condiciones la señal permanecía cercana al valor máximo del conversor análogo-digital (1023) y únicamente cambiaba cuando el sensor era presionado directamente con los dedos. Debido a esto fue necesario modificar el circuito, reemplazando la resistencia por una de 47 kΩ, lo que incrementó la sensibilidad del divisor de voltaje y permitió obtener una mayor variación de la señal durante el movimiento del tórax [5].

A pesar de esta mejora, las primeras señales obtenidas continuaban siendo poco representativas del patrón respiratorio esperado, ya que el sensor solamente respondía cuando existían cambios grandes de presión. Por esta razón se realizó un ajuste adicional, incorporando una pequeña espuma alrededor de la zona sensible del FSR400. Esta modificación permitió mantener una presión inicial constante sobre el sensor, haciendo que pequeñas variaciones producidas por la expansión y contracción del tórax fueran detectadas con mayor facilidad. Como consecuencia, la señal adquirida presentó una transición mucho más continua y una forma considerablemente más cercana al comportamiento esperado para un ciclo respiratorio [5].

<img width="682" height="772" alt="image" src="https://github.com/user-attachments/assets/b061bab2-51eb-4c10-bd1a-c656bd0991de" />

Las señales registradas mediante el Serial Plotter muestran la diferencia entre ambas condiciones experimentales. Durante la respiración en reposo se observaron ciclos repetitivos y relativamente periódicos, donde cada inspiración produjo una variación amplia del valor entregado por el sensor. En contraste, durante la verbalización el patrón dejó de ser periódico, apareciendo inspiraciones más espaciadas y periodos prolongados de exhalación [1].

Posteriormente, las señales fueron procesadas en MATLAB. Debido a la configuración empleada en el divisor de voltaje, la salida del sensor presentaba un comportamiento invertido, es decir, durante la inspiración el valor disminuía en lugar de aumentar. Para facilitar la interpretación de los resultados se realizó la inversión de la señal mediante la operación:

    Señal invertida=1023−Señal original
De esta manera las inspiraciones quedaron representadas por máximos de la señal, permitiendo una interpretación más intuitiva sin modificar la información contenida en la medición.

<img width="563" height="842" alt="image" src="https://github.com/user-attachments/assets/599613dd-dd36-4e5d-aaad-d88273a425d0" />

En la condición de reposo se obtuvo un patrón respiratorio periódico, observándo aproximadamente cinco ciclos completos durante el tiempo de adquisición. La frecuencia dominante obtenida mediante la Transformada Rápida de Fourier se ubicó alrededor de 0.17–0.20 Hz, equivalente aproximadamente a 10–12 respiraciones por minuto, valor que se encuentra muy cercano al rango fisiológico normal reportado para adultos sanos en reposo, comprendido entre 12 y 20 respiraciones por minuto, según la American Thoracic Society y Guyton & Hall [1][2]. La ligera diferencia puede atribuirse a que el sujeto realizó respiraciones profundas y controladas durante la prueba, disminuyendo naturalmente la frecuencia respiratoria.

<img width="496" height="762" alt="image" src="https://github.com/user-attachments/assets/245fb991-5481-4df1-90bb-6a45f2f9cfaa" />

Posteriormente se aplicó un filtro de media móvil con una ventana de cinco muestras utilizando la función movmean() de MATLAB. Este filtrado redujo pequeños picos producidos por el ruido del sensor sin alterar significativamente el comportamiento general de la señal, permitiendo identificar con mayor claridad cada ciclo respiratorio.
Durante la verbalización, la señal presentó un comportamiento claramente diferente. Los intervalos entre inspiraciones aumentaron debido a que la exhalación fue utilizada para producir el habla, observándose ciclos menos periódicos y una mayor variabilidad entre respiraciones consecutivas. En el dominio de la frecuencia esta condición produjo un espectro con mayor dispersión energética, ya que la respiración dejó de ser estrictamente periódica. Aunque continuó existiendo una frecuencia dominante correspondiente al ritmo respiratorio, aparecieron componentes adicionales de baja frecuencia asociadas a las pausas y variaciones naturales del habla [1][4].

Este comportamiento coincide con lo descrito por Guyton y Hall, quienes indican que durante la fonación la espiración deja de ser un proceso pasivo y pasa a ser controlada voluntariamente para regular el flujo de aire necesario para la producción de la voz. Como consecuencia, las inspiraciones se vuelven más rápidas y menos frecuentes, mientras que las exhalaciones se prolongan considerablemente, modificando tanto el patrón temporal como el contenido espectral de la señal respiratoria [1].

Aunque la forma de la señal obtenida no corresponde exactamente a una onda senoidal como la que suele observarse utilizando bandas respiratorias extensiométricas o sensores neumáticos especializados, los resultados fueron consistentes con las características propias del sensor FSR400. Según la hoja de datos del fabricante Interlink Electronics, este sensor presenta una respuesta no lineal, histéresis y fue diseñado principalmente para detectar cambios de fuerza [5], no desplazamientos continuos de pequeña amplitud. Por esta razón la señal adquirida presenta regiones relativamente planas junto con cambios pronunciados durante las variaciones de presión ejercidas por el tórax. Sin embargo, estas características no impidieron identificar correctamente los ciclos respiratorios ni estimar la frecuencia respiratoria.

Respecto al empleo de múltiples sensores para el monitoreo respiratorio, estos podrían mejorar significativamente la calidad de la medición al registrar simultáneamente el movimiento del tórax y del abdomen, reduciendo la influencia de la posición del sensor o de movimientos involuntarios del paciente. Sin embargo, esta alternativa incrementaría el costo del sistema, la complejidad electrónica y computacional, además de disminuir la comodidad del usuario. Para los objetivos planteados en esta práctica, un único sensor FSR400 fue suficiente para diferenciar el patrón respiratorio en reposo y durante la verbalización, así como para estimar adecuadamente la frecuencia respiratoria [3][4].
## CONCLUSION
En esta práctica se logró evaluar la influencia de la verbalización sobre el patrón respiratorio mediante un sistema de adquisición basado en un sensor FSR400, Arduino Uno y MATLAB. El sistema permitió obtener el patrón respiratorio y estimar la frecuencia respiratoria, evidenciando diferencias entre la respiración en reposo y durante el habla, por lo que se cumplieron los objetivos planteados. Finalmente, se concluye que variables físicas como el patrón respiratorio, la frecuencia respiratoria y el movimiento de expansión y contracción de la caja torácica son las más adecuadas para detectar posibles anomalías respiratorias, ya que cualquier alteración en su amplitud, periodicidad o frecuencia puede indicar cambios en la función respiratoria.
##REFERENCIAS
[1] J. E. Hall, Guyton y Hall. Tratado de Fisiología Médica, 14.ª ed. Barcelona, España: Elsevier, 2021.

[2] American Thoracic Society, ATS Patient Education Series: Pulmonary Function Tests. New York, NY, USA: American Thoracic Society, 2021.

[3] N. Massaroni, A. Nicolò, D. Lo Presti, M. Sacchetti, S. Silvestri and E. Schena, "Contact-Based Methods for Measuring Respiratory Rate," Sensors, vol. 19, no. 4, Art. no. 908, 2019.

[4] A. Nicolò, N. Massaroni and E. Schena, "The Importance of Respiratory Rate Monitoring: From Healthcare to Sport and Exercise," Sensors, vol. 20, no. 21, Art. no. 6396, 2020.

[5] Interlink Electronics, FSR® 400 Series Force Sensing Resistor Integration Guide and Evaluation Parts Catalog, Camarillo, CA, USA, Interlink Electronics.
