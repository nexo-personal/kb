---
layout: default
title: Hardware
parent: main
nav_order: 3
parent: nX3
---

# Hardware

## Collegamenti

<div style="text-align:center"><img src="nx3/img/top_small.png" width="40%"/></div>


|*Simbolo*	 		|*Funzione* 							|*Alternativa*	|
|:-----------:		|:-------------: 						|:-:			|
|**+**		     	| power supply +24VDC        			|				|
|**-**	         	| power supply 0VDC          			|				|
|**PE**		     	| power supply ground connection		|				|
|**-->**			| watchdog relè output					|   			|				
|**-->**         	| watchdog relè output		  			|				|
|****ETH0****		| ethernet for realtime					|       		|				
|***ETH1***		    | ethernet 								|       		|				
|**CAN H** [^1]	    | CAN High (in CODESYS CAN0)			|      			|				
|**CAN L**		    | CAN low  								|     			|				
|**1** [^2]			| CAN High 								| RS485 +      	|
|**2**		     	| CAN Low  								| RS485 -   	|
| **°**				| jumper for CAN termination resistor 	|				|


[^1]: In CODESYS corrisponde all'interfaccia CAN0
[^2]: In CODESYS corrisponde all'interfaccia CAN1 vedi impostazioni al capitolo [Parametri nX3][Parametri nX3]


## Led frontale

Il led frontale multicolore è posizionato a destra della microSD card e assume un colore diverso a seconda degli stati del controllore. Nella tabella di seguito i possibili stati del controllore e relativa segnalazione.

<div style="text-align:center"><img src="nx3/img/led.png" width="50%"/></div>



|*colore del led*	|*significato*								|
|:--------------	|------------:								|
|**bianco**			| OS in fase di boot						|
|**arancio**		| applicazione CODESYS in stop o assente	|
|**verde**			| applicazione CODESYS in run				|
|**rosso**			| applicazione CODESYS in exception			|
|**spento**			| tensione sotto i 19 volt					|


## Memoria ritentiva

nX3 effettua la memorizzazione automatica delle variabili dichiarate come ritentive all'interno dell'applicativo CODESYS, la memoria ritentiva massima utilizzabile è di 128kbyte.

Qui un esempio di dichiarazione di variabile ritentiva in un POU

```python
VAR RETAIN
	iRem1 : INT;
END_VAR
```

Qui un esempio di dichiarazione di variabile ritentiva globale in GVL

```python
	VAR_GLOBAL RETAIN
	gvarRem1 : INT;
END_VAR
```

Quando allo spegnimento la tensione di alimentazione scende sotto i 19VDC entra in funzione il micro UPS integrato di nX3 e le variabili ritentive vengono salvate automaticamente. In questa fase il led frontale viene spento per risparmiare corrente e garantire un tempo utile per il salvataggio.