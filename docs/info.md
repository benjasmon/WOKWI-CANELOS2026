<!---

This file is used to generate your project datasheet. Please fill in the information below and delete any unused
sections.

You can also include images in this folder and reference them in the markdown. Each image must be less than
512 kb in size, and the combined size of all images must be less than 1 MB.
-->

## How it works

Sistema digital encargado de 3 etapas distintas:
1- Muestreo de una señal análoga a una señal digital de 3 bits.
2- Filtrado IIR exponencial.
3- Muestra de salida en un display de 7 segmentos.

1. ADC Flash de 3 bits:
A partir del DIP Switch con entrada V_cc se considera como "codificación termómetro":
SW[0]->0000000
SW[1]->1000000
SW[2]->1100000
SW[3]->1110000
SW[4]->1111000
SW[5]->1111100
SW[6]->1111110
SW[7]->1111111
Donde la salida es de la forma T[X].

Para poder lograr la conversión a 3 bits, se utilizó la siguiente tabla:
- B[2] = T[4]
- B[1] = T[6] + T[2] * ~T[4]
- B[2] = T[7] + T[1] * ~T[2] + T[3] * ~T[4] + T[5] * ~T[6]
Inmedaiatamente después de convertir, se muestrea con 3 flip-flops D, con frecuencia de muestreo fs.

2. Filtrado Exponencial:
En cada flanco positivo del reloj se calcula lo siguiente;
y[n]={[x(n-1)+y(n-1)]/2}
Donde x(n-1) es la muestra guardada en el registro e y(n-1) es la salida anterior realimentada, dado que dividir en 2
es simplemente eliminar el LSB solo se trabaja con las salidas S[3], S[2], y S[1], utilizando un semisumador en S[0] y dos
sumadores completos en S[1], S[2], y S[3].

Finalmente se muestrea de nuevo la salida en los registros Y[2], Y[1], e Y[0], además, particularmente en este circuito ocurre un error de
truncamiento; dada la naturaleza del filtro es posible llegar a salidas cercanas al valor final pero sin llegar a este (un ejemplo
se verá en "How to Test"). Dicho lo anterior, la salida se estabiliza en X-1.

3. Decodificador 7 Segmentos:

| Y | Y2 Y1 Y0 | a | b | c | d | e | f | g | Salida|
|---|----------|---|---|---|---|---|---|---|-------|
| 0 | 0 0 0    | 1 | 1 | 1 | 1 | 1 | 1 | 0 |---0---|
| 1 | 0 0 1    | 0 | 1 | 1 | 0 | 0 | 0 | 0 |---1---|
| 2 | 0 1 0    | 1 | 1 | 0 | 1 | 1 | 0 | 1 |---2---|
| 3 | 0 1 1    | 1 | 1 | 1 | 1 | 0 | 0 | 1 |---3---|
| 4 | 1 0 0    | 0 | 1 | 1 | 0 | 0 | 1 | 1 |---4---|
| 5 | 1 0 1    | 1 | 0 | 1 | 1 | 0 | 1 | 1 |---5---|
| 6 | 1 1 0    | 1 | 0 | 1 | 1 | 1 | 1 | 1 |---6---|
| 7 | 1 1 1    | 1 | 1 | 1 | 0 | 0 | 0 | 0 |---7---|


## How to test

1. Dejar todas las entradas a 0 y dar 2 flancos de reloj.
2. Activar rst_n para dejar display en 0.
3. Para activar, por ejemplo, un nivel 3, se debe encender ui[0], ui[1], y ui[2].
4. Apretar el botón de flancos de reloj para probar salidas esperadas:
- ENTRADA SIN CAMBIAR (ESCALÓN):
X | E1 E2 E3 E4 E5 E6
--+-------------------
7 |  0  3  5  6  6  6
6 |  0  3  4  5  5  5
5 |  0  2  3  4  4  4
4 |  0  2  3  3  3  3
3 |  0  1  2  2  2  2
2 |  0  1  1  1  1  1
1 |  0  0  0  0  0  0
- BAJADA A CERO (DE X=7 A X=0 EN UN CANTO Y  CON Y=6 ESTABLE):
Flanco | E1 E2 E3 E4 E5
-------+---------------
Y      |  6  3  1  0  0
- PULSO AISLADO (DE X=7 A X=0 EN UN ESTADO CUALQUIERA):
Flanco     | E1 E2 E3 E4 E5 E6 E7
-----------+---------------------
X aplicada |  0  0  7  0  0  0  0
Y          |  0  0  0  3  1  0  0
- ENTRADA ALTERNADA (CAMBIO ENTRE X=0 Y X=7):
Flanco     | E1 E2 E3 E4 E5 E6 E7 E8
-----------+------------------------
X aplicada |  0  7  0  7  0  7  0  7
Y          |  0  0  3  1  4  2  4  2


## External hardware

- DIP SWITCH: Entradas ui[0] a ui[7] que emulan salida de ADC Flash.
- Display de 7 segmentos: Conectados a uo[0] hasta uo[7].
- Botones: Se utiliza el botón rst_n y el pulsador de reloj para ver cambios en diferentes estados.
