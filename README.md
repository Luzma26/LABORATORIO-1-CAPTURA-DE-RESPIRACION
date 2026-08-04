# MONITOREO DEL PATRON Y FRECUENCIA RESPIRATORIA

**Luz Marina Valderrama-5600741**

## INTRODUCCION
La respiración es un proceso fisiológico indispensable para el intercambio de oxígeno y dióxido de carbono entre el organismo y el medio ambiente. La frecuencia respiratoria constituye uno de los signos vitales más importantes, ya que permite evaluar el estado fisiológico de una persona y detectar posibles alteraciones clínicas. En esta práctica se desarrolló un sistema de adquisición de la señal respiratoria utilizando un sensor resistivo sensible a la fuerza (FSR400), una placa Arduino Uno y MATLAB, con el propósito de obtener el patrón respiratorio y analizar las diferencias existentes entre la respiración en reposo y durante una actividad de verbalización.

## METODOLOGIA
Inicialmente se seleccionó un sensor FSR400 debido a que permite detectar cambios de presión ocasionados por el movimiento de la caja torácica durante la respiración. El sensor se conectó mediante un divisor de voltaje alimentado con 5 V utilizando una resistencia fija de 47 kΩ. Durante las primeras pruebas se observó que valores menores de resistencia producían una variación muy pequeña en la salida del sensor, por lo que fue necesario aumentar dicho valor para mejorar la sensibilidad del sistema.

Con el fin de incrementar aún más la respuesta del sensor frente a los pequeños movimientos producidos por la respiración, se colocó una pequeña espuma alrededor de la zona sensible del FSR. Esta espuma permitió mantener una presión inicial constante sobre el sensor, haciendo que pequeñas expansiones y contracciones del tórax generaran variaciones más evidentes en la señal registrada.

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
En la función setup() se inicializó la comunicación serial a una velocidad de 9600 baudios, permitiendo el envío continuo de datos hacia MATLAB. Posteriormente, dentro de la función loop(), el Arduino realizó la lectura permanente del sensor mediante analogRead(A0) y transmitió cada muestra utilizando Serial.println(). Finalmente, la instrucción delay(80) estableció un intervalo aproximado de 80 ms entre muestras, equivalente a una frecuencia de adquisición cercana a 10 Hz, suficiente para registrar la respiración humana.

Antes de colocar el sensor sobre el cuerpo se verificó el funcionamiento del circuito presionando manualmente el FSR y observando la respuesta mediante el Serial Plotter de Arduino IDE. Una vez comprobado su funcionamiento, el sensor fue ubicado sobre una de las costillas del sujeto de prueba y asegurado mediante una banda elástica para mantener un contacto constante durante toda la adquisición.

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
Inicialmente se abrió el puerto serial utilizando la función serialport(), especificando el puerto COM correspondiente y la velocidad de transmisión de 9600 baudios. Después de una pausa de dos segundos para garantizar el establecimiento de la comunicación, se ejecutó un ciclo for que recibió 300 muestras provenientes del Arduino mediante la función readline(). Cada dato recibido fue convertido de texto a formato numérico utilizando str2double() y almacenado dentro del vector senal.

### Almacenamiento de la información
    save("senal_hablando.mat","senal")
Este procedimiento permitió conservar los datos experimentales para realizar posteriormente el procesamiento y el análisis sin necesidad de repetir la adquisición.

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
En primer lugar se definió una frecuencia de muestreo aproximada de 10 Hz, correspondiente al intervalo de adquisición utilizado por el Arduino. Posteriormente se eliminó el valor promedio de la señal para reducir la componente continua y mejorar el análisis espectral. Después se calculó la FFT mediante la función fft(), obteniendo el espectro de magnitud correspondiente.

Finalmente, se identificó automáticamente la frecuencia dominante y se calculó la frecuencia respiratoria en respiraciones por minuto.

    [valor,pos] = max(Y(2:N/2));
    frecuencia = f(pos+1);
    disp("Frecuencia dominante (Hz):")
    disp(frecuencia)
    disp("Respiraciones por minuto:")
    disp(frecuencia*60)
La frecuencia dominante corresponde al componente principal presente en el espectro de la señal respiratoria. Multiplicando este valor por 60 se obtuvo la frecuencia respiratoria expresada en respiraciones por minuto, permitiendo comparar los resultados obtenidos durante la respiración en reposo y durante la verbalización.
## DISCUSION
Durante las primeras pruebas el sistema presentó poca sensibilidad para detectar los movimientos respiratorios cuando se utilizó un divisor de voltaje con una resistencia de menor valor. En estas condiciones la señal permanecía cercana al valor máximo del conversor análogo-digital (1023) y únicamente cambiaba cuando el sensor era presionado directamente con los dedos. Debido a esto fue necesario modificar el circuito, reemplazando la resistencia por una de 47 kΩ, lo que incrementó la sensibilidad del divisor de voltaje y permitió obtener una mayor variación de la señal durante el movimiento del tórax.

A pesar de esta mejora, las primeras señales obtenidas continuaban siendo poco representativas del patrón respiratorio esperado, ya que el sensor solamente respondía cuando existían cambios relativamente grandes de presión. Por esta razón se realizó un ajuste mecánico adicional, incorporando una pequeña espuma alrededor de la zona sensible del FSR400. Esta modificación permitió mantener una presión inicial constante sobre el sensor, haciendo que pequeñas variaciones producidas por la expansión y contracción del tórax fueran detectadas con mayor facilidad. Como consecuencia, la señal adquirida presentó una transición mucho más continua y una forma considerablemente más cercana al comportamiento esperado para un ciclo respiratorio.


## CONCLUSION
Se implementó un sistema de adquisición de la señal respiratoria utilizando un sensor FSR400, un divisor de voltaje con una resistencia de 47 kΩ, una placa Arduino Uno y MATLAB para el procesamiento de la información. El sistema permitió registrar la señal respiratoria, aplicar un filtrado sencillo y obtener su representación en el dominio de la frecuencia. Los resultados mostraron diferencias entre la respiración en reposo y durante la verbalización, evidenciando que el habla modifica el patrón respiratorio. A pesar de las limitaciones propias del sensor FSR, el sistema resultó adecuado para cumplir los objetivos planteados en la práctica.
