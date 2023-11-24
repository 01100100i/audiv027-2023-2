Sonidos desde el movimiento

 Materiales:
 Arduino con sensor IMU
 Processing
 Datos de sonido
 

A partir del código del ejemplo de Tensorflow llamado “Magic Wand”, aplicado al lenguaje de Arduino, se pretendió asignar una respuesta de sonido al ejecutar sus funciones. Para ello era necesario establecer comunicación con un software flexible que permitiera la reproducción de pistas de sonido ya grabadas, y asignar estos sonidos a los outputs del código de “Magic Wand” en Arduino. De la investigación realizada se obtuvieron 3 resultados particulares del código creado: se determinó la forma efectiva de comunicar el código de “Magic Wand” en Arduino con el código creado en Processing, se pudo crear una representación gráfica en Processing del output mostrado en el Monitor Serial de Arduino, y no se pudo lograr asignar a los valores leídos por Processing un sonido asociado en el código. Sin embargo, sí se pudo ejecutar sonidos no condicionados en el código de Processing, demostrando que es posible asociar ambas variables con comandos correspondientes aún por encontrar. Se hicieron múltiples pruebas sobre las posibles soluciones sobre esta función investigada, tomados de distintos referentes:

Minim Library
Librería Serial para Arduino y Processing
Librería Minim para Processing
SoundClassification p5.js

![image](https://github.com/01100100i/audiv027-2023-2/assets/142625648/47517fcf-8482-4bff-9d68-43d543973da3)

El objetivo era crear una experiencia kinestésica sonora a través de movimientos que actúan como comandos para generar distintos sonidos y asociados al cambio de una imagen. En el caso de Magic Wand, se pueden establecer muchos gatillantes kinestésicos, pero se usó la secuencia numérica que incluía los enteros del 0 al 9. El sistema IMU incluido en el Arduino Nano BLE 33 Sense incluye los sensores acelerómetro y giroscopio, que determinan la figura dibujada en el espacio cuando se mueve el equipo Arduino. Si este movimiento coincide con lo que el código luego interpreta como números del 0 al 9, se traduce la información de los sensores en el output del Monitor Serial de Arduino según una base de datos de inteligencia artificial alimentada con información de secuencia de movimientos de formas de números del 0 al 9.

![image](https://github.com/01100100i/audiv027-2023-2/assets/142625648/23419a75-c190-41b0-a4d4-fe78cff8bf24)


Se pensó originalmente traducir la información leída por los sensores del sistema IMU directamente para luego producir un sonido en Processing, pero esto presentó las siguientes complicaciones:

La comunicación de valores IMU entre Arduino y Processing usando el código de Magic Wand no se podía establecer directamente debido a la cantidad de variables que el código debía interpretar primero para luego dar outputs de valores de medidas.

Aún si se logra establecer una comunicación con cada variable, construir una lectura discreta o en intervalos en Processing es contraproducente, ya que el código contiene la inteligencia artificial que interpreta los movimientos y entrega valores de outputs discretos asociados.

Establecer una comunicación de valores IMU sin usar el código de Magic Wand, sino el rescatado de los ejemplos de los sensores de la biblioteca del Arduino Nano BLE 33 Sense, implicaba dispensar de las funciones de inteligencia artificial que le entregaban el interés al proyecto.

Por ende, se optó por comunicar la lectura del Monitor Serial del código de Arduino por Processing, añadiendo al código de Arduino la siguiente función:

     void loop () {
       int8_t max_score;        // llama al String definido en void setup () de números del 0 al 9
       int max_index;             // define la variable output de función for()/if() determinada por la AI

       Serial.print(max_index);       // print agregado para que Processing lea el output
       delay(300);
     }

  ![image](https://github.com/01100100i/audiv027-2023-2/assets/142625648/9e14ce2d-d35d-45da-84c8-d1d8430d0ced)

     
![image](https://github.com/01100100i/audiv027-2023-2/assets/142625648/310744d3-0bea-46c5-8c3c-da36d6b520d7)


Luego en Processing se importó la librería Sound, definiendo el siguiente código:

       import processing.serial.*;

       Serial mySerial;      	                              // crea un objeto de Clase serial
       String myString = null;                                   // variable para colectar data del serial
       float myVal;            	                              // float para el data serial de ASCII guardado

       void setup () {
            String myPort = Serial.list()[0];        	              // ocupar puerto correcto
            mySerial =new Serial(this, myPort, 9600);      // conectar Processing a el puerto serial
       }

       void draw () {
	while ( mySerial.available() >0){                      // mientras los datos estén disponibles
	myString = mySerial.readStringUntil('\n');       // lo guarda en val
  	println(myString);
       }

De esta forma se logró comunicar los outputs de Magic Wand en Processing. El objetivo de esto es usar la interfaz de Arduino para reproducir sonidos por Processing, debido a que, si bien Arduino posee librerías de archivos de sonido disponibles, no puede reproducir archivos de reproducción de sonido de rango alto debido al límite en su memoria. Por ende, se utiliza la reproducción de sonido por computador, siendo un parlante asociado a Processing.

![image](https://github.com/01100100i/audiv027-2023-2/assets/142625648/43c970d3-e995-40e3-befa-897c5a3e938a)
![image](https://github.com/01100100i/audiv027-2023-2/assets/142625648/1f5ffd51-9df7-4c69-8e92-05a9469dd205)

A partir de este código también pudimos desarrollar otras respuestas visuales creando un lienzo y modificando las dimensiones de su output respecto del valor que Magic Wand reconoce: si reconoce un valor 0, el lienzo no se ve afectado, pero a medida que reconoce valores más grandes del 1 al 9, el lienzo negro muestra una barra blanca que va en crecimiento directamente proporcional. La inclusión al código es:


       void setup () {
       size(200, 400);
       }

       void draw () {
	if (myString !=null) {
  	background(0);   //si mi dato es nulo, rellena con 0 el fondo
  	myVal = float(myString); //toma los datos del serial y los convierte en números
	}
  	myVal = myVal/10 *height;
  	rectMode(CENTER);
  	rect(width/2, height-(myVal/2), 100, myVal);
        }

Luego decidimos probar cómo reproducir música en Processing.

![image](https://github.com/01100100i/audiv027-2023-2/assets/142625648/56bd9628-5281-41a0-ac9d-f3ffa56509a8)

Descargamos pequeños samples en freesound.org  para agregar a Processing y que este reproduzca tales archivos, según la librería Sound de Processing. Se evaluó usar la librería Minim también, pero debido a las limitantes por la vigencia de tal librería, se optó por usar la librería Sound, según el código:

  import processing.sound.*;

       SoundFile file;       	//importa el archivo de sonido a reproducir

       void setup () {
            file = new SoundFile(this, "wawa.wav"); 
  	file.play();  // reproduce automáticamente el archivo de sonido al ejecutar código
       }

Donde wawa.wav es el archivo descargado de freesound.org a probar. Inicialmente el código es insuficiente porque se debe indicar a Processing la dirección donde se encuentra el archivo. Para ello se puede cargar una librería en el menú de Processing que incluya una carpeta con los archivos de sonido a usar. La otra forma es arrastrar con el cursor del mouse los archivos de sonido sobre el display de Processing y dejar que el programa reconozca la dirección. Este último método presenta el problema de que no se puede llamar a otros archivos de sonido nuevos tal que, si se vuelve a cargar con el cursor nuevos archivos, estos reemplazan a los archivos antiguos cargados. Por ello, se recomienda cargar librerías con archivos. Aún así, se debe llamar en el void setup () a todo archivo a reproducir en el código. Sin embargo, usar la función .play() en el void setup () reproduce automáticamente las pistas de sonido, sin ningún condicionante más que el inicio del código, por lo que se propuso evitar esta función en el void setup () y buscar una manera de condicionar en base a los resultados del reconocimiento de Magic Wand (números del 0 al 9) una forma de reproducir el sonido de manera variable.

![image](https://github.com/01100100i/audiv027-2023-2/assets/142625648/6f23f80a-3d98-4945-901a-b6955d87d2b5)

![image](https://github.com/01100100i/audiv027-2023-2/assets/142625648/b09b2bd7-000a-4e4f-8e65-293a020b7897)

Conclusiones:
No es facil traspasar los datos de sensores incluidos en la placa desde arudino a processing., sin emabrgo logramos hacerlo.
Nos costó identificar los valores o resultados pra asociarlos a distints operaciones en el codigo.

Sonidos a usar freesound.org
https://freesound.org/people/FreqMan/sounds/42899/
https://freesound.org/people/edwardszakal/sounds/516400/
https://freesound.org/people/anthonychartier2020/sounds/560189/
https://freesound.org/people/Xiko__/sounds/711251/
https://freesound.org/people/melack/sounds/9158/
https://freesound.org/people/digitl/sounds/336875/


