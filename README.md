# SMARTJET

>Adrián García Padilla

>Francisco Quesada Muñoz

>Antony Franco Rivero Meza

>David Robles del Viso

<br/>


## 1. Introducción  

El presente documento corresponde a la memoria del trabajo de la asignatura Sistemas Inteligentes el cual detalla el estudio y desarrollo de un controlador difuso, en Unity 3D, para el control automático de vuelo de un avión.

Para la realización de este trabajo se ha dividido el desarrollo en dos fases. 
La primera se centra en estudiar los principios básicos de vuelo, identificando las partes o módulos de una aeronave que se deberá manejar para lograr el objetivo.

En la segunda fase se desarrolla el código en C# utilizando ténicas de lógica difusa.

<br/>


## 2. Objetivo

El objetivo de este trabajo es lograr el control automático de la estabilidad de un avión durante su vuelo acercándonos a un situación lo más real posible, es decir, teniendo en cuenta el baremo del que se rige una aeronave en un vuelo real.

<br/>

## 3. Estudio previo

Básicamente, un piloto maneja las fuerzas de vuelo, dirección y el movimiento de la aeronave para su control y tomando en cuenta ciertas características que varían dependiendo del tipo de avión. 
En este caso, hemos tomado como referencia un Asset que proporciona Unity, concretamente un jet.

Nuestro objetivo es lograr la estabilidad del avión, por lo que estudiaremos la composición del sistema estabilizador.

Sistema estabilizador: compuesto por un estabilizador vertical y un estabilizador horizontal, es decir, conseguir la estabilidad manipulando sus ejes vertical y horizontal.

Los movimientos que se realizan alrededor de los ejes son los siguientes:

+ __Eje longitudinal:__ es un eje imaginario que va desde el morro del avión hasta la cola del mismo. El movimiento que se ejerce alrededor de este eje se le denomina alabeo (“roll” en inglés). Concretamente, se trata de levantar un ala bajando la otra. 
+ __Eje lateral:__ es un eje imaginario que va desde el extremo de un ala al extremo de la otra. El movimiento alrededor de este eje se le denomina cabeceo (“pitch” en inglés). Concretamente, se trata de levantar el morro arriba o bajarlo.
+ __Eje vertical:__ es un eje imaginario que atraviesa el centro del avión. El movimiento en torno a este eje se le denomina guiñada (“yaw” en inglés). Concretamente, vira el morro hacia la derecha o a la izquierda.

En la parte de desarrollo en un entorno 3D, el eje longitudinal, el lateral y el vertical serán las coordenadas x, y, z, respectivamente.

![alt text](http://2.bp.blogspot.com/-_d_Y51vcJnU/UIxBT81gbzI/AAAAAAAAB7Q/Jf5B9nV_bEU/s1600/EjesMovimiento.png "Ejes")

<br/>

## 4. Funcionamiento de la práctica

A continuación se explica el funcionamiento al momento de ejecutar el programa.

+ __Caída libre:__ como punto de partida, el avión aparecerá en el espacio creado, con una velocidad igual a cero y una dirección aleatoria que se define mediante en el método start(). El avión, al encontrarse en esa situación, inicia su desplazamiento hasta llegar a una velocidad definida en el sistema para luego continuar su trayecto de manera uniforme.
+ __Control manual:__ se establecen una serie de teclas las cuales pueden alterar el movimiento del avión, simulando la manipulación que un piloto realiza a la hora de realizar la navegación.
	
    Pulsando el botón izquierdo del ratón se activarán los frenos, reduciendo así la velocidad del avión tanto como cuán prolongada sea esa pulsación. Se puede apreciar cómo el avión va descendiendo por falta de potencia y si está realizando una maniobra de cambio de dirección, al existir poca potencia y aceleración, le cuesta mucho más volver al estado de estabilidad.

    Las teclas direccionales sirven para variar la dirección de vuelo, tanto para ascender, descender, girar a la izquierda o girar a la derecha. En cualquier acción, en el momento de dejar de pulsar las teclas, el avión detectará la posición en la que se encuentra e intentará reincorporarse para situarse en su estado de estabilidad y continuar su vuelo.

    Las teclas direccionales izquierda y derecha afectan al eje longitudinal, es decir, manipula el alabeo. En el código, a esta variable se le identifica por su término en inglés: roll.
Las teclas direccionales arriba y abajo afectan al eje lateral, es decir, manipula el cabeceo. En el código, a esta variable se le identifica por su término en inglés: pitch.
 	
<br/>

## 5. Lógica y código del controlador

### Variables de control
```c#

        //mathematical variables
        private double maxRoll = Math.PI / 6;
        private double mediumRoll = Math.PI / 12;
        private double lowRoll = Math.PI / 24;
        private double maxPitch = Math.PI / 4;
        private double mediumPitch = Math.PI / 8;
        private double lowPitch = Math.PI / 16;
        private int cont;

        //Input values
        private Dictionary<string, double> roll = new Dictionary<string, double>();
        private Dictionary<string, double> pitch = new Dictionary<string, double>();

        //output values
        private Dictionary<string, double> force = new Dictionary<string, double>();
        private double exit;
```



### KnowledgeBase()
Definiremos los valores que tienen las variables de entrada y de salida. Estas variables se han implementado mediante el uso de diccionarios clave-valor.

```c#
private void KnowledgeBase()
        {
            roll.Add("TooMuchLeft", -maxRoll);
            roll.Add("TooMuchRight", maxRoll);
            roll.Add("MediumLeft", -mediumRoll);
            roll.Add("MediumRight", mediumRoll);
            roll.Add("LowLeft", -lowRoll);
            roll.Add("LowRight", lowRoll);

            pitch.Add("TooMuchUpside", -maxPitch);
            pitch.Add("TooMuchDownside", maxPitch);
            pitch.Add("MediumUpside", -mediumPitch);
            pitch.Add("MediumDownside", mediumPitch);
            pitch.Add("LowUpside", -lowPitch);
            pitch.Add("LowDownside", lowPitch);

            force.Add("Prestissimo", 2.5f);
            force.Add("Presto", 1.5f);
            force.Add("Allegro", 0.8f);
            force.Add("Piano", 0.3f);
        }
```



### DeFuzzify()
En este método crearemos las reglas que serán utilizadas posteriormente por el controlador.

```c#
        private void DeFuzzify()
        {
            float exit;
            if (PitchInput == 0f && RollInput == 0f)
            {
                exit = Fuzzify();
                PitchInput = -PitchAngle * exit;
                RollInput = -RollAngle * exit;
            }
        }

```
### Fuzzify()
En este método se aplicarán las reglas definidas anteriormente para poder controlar la estabilidad del avión.

```c#
        private float Fuzzify()
        {
            double exit = this.exit;
            if ((PitchAngle <= this.getValue(pitch, "TooMuchUpside")) ||
                (PitchAngle >= this.getValue(pitch, "TooMuchDownside")) ||
                (RollAngle <= this.getValue(roll, "TooMuchLeft")) ||
                (RollAngle >= this.getValue(roll, "TooMuchRight")))
            {
                exit = force["Prestissimo"];
            }
            else if ((PitchAngle > this.getValue(pitch, "TooMuchUpside") && PitchAngle <= this.getValue(pitch, "MediumUpside") || RollAngle > this.getValue(roll, "TooMuchLeft") && RollAngle <= this.getValue(roll, "MediumLeft")) ||
                     (PitchAngle > this.getValue(pitch, "TooMuchUpside") && PitchAngle <= this.getValue(pitch, "MediumUpside") || RollAngle < this.getValue(roll, "TooMuchRight") && RollAngle >= this.getValue(roll, "MediumRight")) ||
                     (PitchAngle < this.getValue(pitch, "TooMuchDownside") && PitchAngle >= this.getValue(pitch, "MediumDownside") || RollAngle > this.getValue(roll, "TooMuchLeft") && RollAngle <= this.getValue(roll, "MediumLeft")) ||
                     (PitchAngle < this.getValue(pitch, "TooMuchDownside") && PitchAngle >= this.getValue(pitch, "MediumDownside") || RollAngle < this.getValue(roll, "TooMuchRight") && RollAngle >= this.getValue(roll, "MediumRight")))
            {
                exit = force["Presto"];
            }
            else if ((PitchAngle > this.getValue(pitch, "MediumUpside") && PitchAngle <= this.getValue(pitch, "LowUpside") || RollAngle > this.getValue(roll, "MediumLeft") && RollAngle <= this.getValue(roll, "LowLeft")) ||
                     (PitchAngle > this.getValue(pitch, "MediumUpside") && PitchAngle <= this.getValue(pitch, "LowUpside") || RollAngle < this.getValue(roll, "MediumRight") && RollAngle >= this.getValue(roll, "LowRight")) ||
                     (PitchAngle < this.getValue(pitch, "MediumDownside") && PitchAngle >= this.getValue(pitch, "LowDownside") || RollAngle > this.getValue(roll, "MediumLeft") && RollAngle <= this.getValue(roll, "LowLeft")) ||
                     (PitchAngle < this.getValue(pitch, "MediumDownside") && PitchAngle >= this.getValue(pitch, "LowDownside") || RollAngle < this.getValue(roll, "MediumRight") && RollAngle >= this.getValue(roll, "LowRight")))
            {
                exit = force["Allegro"];
            }
            else {
                exit = force["Piano"];
            }

            return (float)exit;

        }
```





