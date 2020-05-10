---
layout: default
title: nX3
nav_order: 2
has_children: true
permalink: /docs/nx3
---

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

# Generale

nX3 è un controllore avanzato per automazione industriale programmabile con CODESYS. È stato pensato per soddisfare una vasta gamma di esigenze applicative, può essere utilizzato come semplice controllore logico o come motion controller avanzato grazie alla completa integrazione delle librerie di Softmotion, CNC e robotica di CODESYS.
È governato da un sistema operativo realtime che consente di impostare il tempo di switching delle task di CODESYS sotto al milliscondo (tipicamente fino a 250microsec.) con un jitter normalmente contenuto entro i 20 microsec.


Questo manuale descrive le funzioni hardware e software specifiche del controllore neXo nX3.

Per qualsiasi indicazione riguardo la programmazione CODESYS è possibile consultare l'help online su help.codesys.com.

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
|****ETH0****		    | ethernet for realtime					|       		|				
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

# Funzioni software specifiche


## Parametri nX3

Un volta creato un progetto per nX3 è possibile, editando il nodo device e selezionando la sheet
*nX3 CPU Parameters*, visualizzare e modificare alcuni parametri che
consentono di impostare alcuni funzionamenti del controllore.


<div style="text-align:center"><img src="nx3/img/ParametrinX3.jpg" /></div>


**BusSpeed**

Consente di impostare la velocità del bus di comunicazione con i moduli di I/O locali. E' impostato per default al valore massimo di 8Mhz. Non dovrebbe mai
essere modificato.

**Enable Can1**

Abilita il funzionamento della seconda linea CAN (CAN1 in CODESYS). In condizioni di default il connettore superiore a due pin identificato con 1 e 2 (vd. [Collegamenti][Collegamenti]) non è utilizzato, mettendo TRUE in questo parametro il connettore è connesso alla seconda linea CAN (CAN1 in CODESYS).

**Enable RS485 line**

Abilita il funzionamento della linea RS485. In condizioni di default il connettore superiore a due pin identificato con 1 e 2 (vd. [Collegamenti][Collegamenti]) non è utilizzato, mettendo TRUE in questo parametro il connettore è connesso alla RS485. Questa impostazione non ha effetto nel caso in cui sia abilitata la precendente.

## Watchdog

Il modulo nX3 dispone di un Watchdog connesso ad un rele statico le cui connessioni sono disponibili sul connettore di alimentazione e che permette di verificare
costantemente il corretto funzionamento di hardware e software.

Editando il nodo device e selezionando la sheet *nX3 CPU I/O Mapping* è possibile verificare che è mappata automaticamente una variabile ti tipo BOOL con nome Watchdog. Se lo si desidera in questa sheet è possibile cambiare nome alla varabile.


<div style="text-align:center"><img src="nx3/img/Watchdog.jpg" /></div>


Per attivare il watchdog è sufficiente aggiungere la seguente linea di codice nel task che si ritiene rilevante per l'esecuzione della propria automazione.
```python
WATCHDOG := TRUE;
```
Se in seguito ad un guasto o un errore nel software dell'applicazione la linea contenente il codice non è più eseguita il rele di watchdog si apre.



## Linee Ethernet

Il modulo dispone di due linee Ethernet: ***eth0*** e *eth1*. È inoltre supportato la connessione WiFi inserendo un adattatore USB esterno nella porta USB Host disponibile.

Attualmente è supportato il modello USB LM816 di LM Tecnologies.

La linea ***eth0*** è impostata per default con questi valori:

**IP=192.168.2.100, mask=255.255.255.0**

La linea *eth1* è impostata per default con questi valori:

**IP=192.168.3.100, mask=255.255.255.0**

Le linee ethernet sono entrambe utilizzabili sia con l'ambiente di sviluppo CODESYS per la programmazione e il debug delle applicazioni sia come etherCAT master.
La linea **eth0** è quella con minor latenza e consente un jitter leggermente inferiore rispetto alla *eth1*, ne è pertanto consigliato l'utilizzo per la connessione etherCAT ini particolare quando si sviluppano applicazioni di Motion Control, CNC e robotica.

La libreria *nX3CpuSetup* contiene alcune funzioni che permettono di configurare entrambe le linee Ethernet e il WiFi.

\begin{figure}[htbp]
\includegraphics[width=1\textwidth,keepaspectratio]{assets/Funzioni.jpg}
\caption{Funzioni networking}
\end{figure}

\newpage

### Funzione SetEthIPAddress

La funzione *SetEthIPAddress* permette di impostare l'indirizzo IP e la Subnet Mask per entrambe le linee Ethernet.

\begin{figure}[htbp]

	\centering\includegraphics[width=1\textwidth,keepaspectratio]{assets/SetEthIPAddress.jpg}

	\caption{Impostazione indirizzo IP porte ethernet}

	\label{fig:}

\end{figure}


Di seguito un esempio di codice che consente di impostare un indirizzo IP fisso e Subnet Mask alla linea **eth0**.

```python
VAR
Error: nX3.ERROR;
END\_VAR
Error := nX3.SetEthIPaddress( 0, 192, 168, 10, 123, 255, 255, 255, 255);
```

Di seguito un esempio di codice che consente di impostare un indirizzo IP fisso e Subnet Mask alla linea *eth1*.
```python
VAR
Error: nX3.ERROR;
END\_VAR
Error := nX3.SetEthIPaddress( 1, 192, 168, 20, 123, 255, 255, 255, 255);
```

Qualora si desideri impostare la linea con indirizzo assegnato tramite DHCP è possibile utilizzare questo esempio di codice.

```python
VAR
Error: nX3.ERROR;
END\_VAR
Error := nX3.SetEthIPaddress( 0, 0, 0, 0, 0, 0, 0, 0, 0 );
```

**Attenzione**
La funzione impiega qualche secondo per effettuare la configurazione, è opportuno chiamarla da un task con priorità più bassa di
quello utilizzato per l'automazione in modo da non bloccare l'esecuzione del programma principale.

\newpage


### Funzione SetWirelessNetwork

La funzione SetWirelessNetwork permette di selezionare la rete WiFi cui ci si vuole connettere, la relativa password, l'indirizzo IP e la Subnet Mask.



\begin{figure}[htbp]

	\centering\includegraphics[width=1\textwidth,keepaspectratio]{assets/SetWirelessNetwork.jpg}

	\caption{Set Wireless Network}

	\label{fig:}

\end{figure}




Per assegnare le rete è possibile utilizzare questo esempio di codice.
\begin{lstlisting}
VAR
Error: nX3.ERROR;
END\_VAR
Error := nX3. SetWirelessNetwork ( \'NomeReteWIFI\', \'PasswordWiFI\',192, 168, 10, 123, 255, 255, 255, 255 );
\end{lstlisting}

Qualora si desideri impostare la linea con indirizzo assegnato tramite DHCP è possibile utilizzare questo esempio di codice.
\begin{lstlisting}
VAR
Error: nX3.ERROR;
END\_VAR
Error := nX3. SetWirelessNetwork ( \'NomeReteWIFI\', \'PasswordWiFI\',0, 0, 0, 0, 0, 0, 0, 0 );
\end{lstlisting}


**Attenzione**
La funzione impiega qualche secondo per effettuare la configurazione, è opportuno chiamarla da un task con priorità più bassa di
quello utilizzato per l'automazione in modo da non bloccare l'esecuzione del programma principale.

\newpage


### Funzione RemoveWirelessNetwork

La funzione *RemoveWirelessNetwork* consente di dissociare il dispositivo dalla rete WiFi precedentemente selezionata.


\begin{figure}[htbp]

	\centering\includegraphics[width=1\textwidth,keepaspectratio]{assets/RemoveWirelessNwtwork.jpg}

	\caption{Remove Wireless Network}

	\label{fig:}

\end{figure}



Per rimuovere le rete è possibile utilizzare questo esempio di codice.
\begin{lstlisting}VAR
Error: nX3.ERROR;
END\_VAR
Error := nX3. RemoveWirelessNetwork ( \'NomeReteWIFI\' );
\end{lstlisting}

**Attenzione**
La funzione impiega qualche secondo per effettuare la configurazione, è opportuno chiamarla da un task con priorità più bassa di
quello utilizzato per l'automazione in modo da non bloccare l'esecuzione del programma principale.

## Codici di errore.

Le funzioni restituiscono eventuali codici di errore appartenenti alla enumerazione ERROR contenuta nella libreria stessa.



\begin{figure}[htbp]

	\centering\includegraphics[width=1\textwidth,keepaspectratio]{assets/Errori.jpg}

	\caption{Errori}

	\label{fig:}

\end{figure}


\newpage

## Accesso remoto

Grazie al framewrok di comunicazione e ai sewrvizi di connessione realizzati da [nabto](www.nabto.com), è possibile realizzare una connessione peer to peer con nX3 ovunque sia installato. Grazie all'aperture di un tunnel TCP sarà possibile connettersi a nX3 con l'Engineering di CODESYS.

Una volta effettuata la registrazione del device al server nabto e ottenuti il **DeviceId** e la **Key** sarà possibile utilizzare il function block di seguito per effettuare la connessione. Non serve nessun'altra configurazione.

\begin{figure}[htbp]

	\centering\includegraphics[width=1\textwidth,keepaspectratio]{assets/nabto1.png}

	\caption{Configure and enable the TCP tunnel for the remote connection}

	\label{fig:}

\end{figure}



Il tunnel TCP rimane attivo anche se non è presente la boot application di CODESYS. Questo consente di raggiungere e programmare nX3 anche in assenza di applicativo CODESYS.

Qualora si volesse disabilitare la connessione è possibile utilizzare questo functon block.

\begin{figure}[htbp]

	\centering\includegraphics[width=1\textwidth,keepaspectratio]{assets/nabto2.png}

	\caption{Disable the TCP tunnel}

	\label{fig:}

\end{figure}



## Moduli I/O

Il controllore nX3 dispone di un bus locale che ne consente l'espansione con i moduli di I/O ProCard realizzati da UFG. Questi possono essere inseriti nel progetto selezionando il device nX3, attivando il menu di contesto (tasto destro del mouse). Appare la finestra di inserimento dispositivi dove sotto la voce Miscellaneous sono elencati i moduli supportati.



\begin{figure}[htbp]

	\centering\includegraphics[width=1\textwidth,keepaspectratio]{assets/AddDevice.jpg}

	\caption{Aggiunta I/O locali}

	\label{fig:}

\end{figure}




Ciascun modulo, una volta inserito nel progetto, dispone di un proprio editor dove la sheet nX3CPU I/O mapping permette di assegnare il nome al punto I/O gestito dal modulo stesso secondo lo standard CODESYS.



\begin{figure}[htbp]

	\centering\includegraphics[width=1\textwidth,keepaspectratio]{assets/IOMapping.jpg}

	\caption{Mapping I/O locali}

	\label{fig:}

\end{figure}

\newpage

## Dispositivi di memoria rimovibili.

nX3 permette di inserire sul frontale una scheda micro SD e una memoria USB.

Tramite le funzioni standard di CODESYS per l'accesso ai file è possibile leggere e scrivere file contenuti in questi dispositivi.

Per accedere al directory principale della scheda miscro SD è necessario utilizzare il percorso:
\begin{lstlisting}
/media/mmcblk1p1/
\end{lstlisting}

Per accedere al directory principale della memoria USB è necessario utilizzare il percorso:
\begin{lstlisting}
/media/sda1/
\end{lstlisting}

<!--
# Storico revisioni


| data 		| versione 	| descrizione 		| autore 			|
|:-----		|:---------	|:------------		|:-------			|
| 2018/10/4 | 1.0		| Prima versione	| Davide Gallesi	|
-->
