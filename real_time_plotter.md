```

# # # # # # # # # # BLOCCO 1
#{

#creo un segnale. onda seno
# caratteristiche: voglio che faccia 1000 giri/s
#nel plot considererò 0.002s in cui ci saranno 2 giri
frequency=1000; #1000 giri al sec
period=1/frequency; #0.001 per un giro. 1ms!
amplitude=1;
dcoffset=0;
t= 0:0.00001:2*period; #asse dei tempi.arriva fino a 2*period: 2 ms, 2 giri.
out= dcoffset+amplitude*sin(2*pi*frequency*t); #funzione del tempo.

plot(t,out); #plot base. se volessi le ordinate in ms: moltiplica t*1000
# asse x: da 0 a 0.002 a passi di 0.00001;
xlabel("time [ms]");
ylabel("signal amplitude");#perchè xlabel mi da "out of bound" come errore??? boh, non me lo da più
#
#ridimensiono intervallo assignin
axis([0 period -1.5 +1.5]); #asse x: da 0 a 0.001

hold on; #hold on si usa per mantenere le impostazioni del plot aggiungendo altra roba!
x=5; #nota: loop disattivato!
while(x<4)
  pause(1);
  if rem (x++,2)==0
  plot(t,out,"r");
  else plot (t,out,"k");
endif
#PROBLEMA: COSÌ CREA TROPPI PLOT UNO SULL'ALTRO??
end
hold off;

# PARTE 2: PROVO A FAR SCORRERE L'ASSE DELLE X.
# scorro di period/10, per 10 volte. voglio metterci 1 sec -> scorro ogni 0.1s!

#dichiaro variabili, estremi assex
loop=0; xmin=0;xmax=period;

# # DICHIARO NEL TITOLO CHE STO PER SCORRERE
title("about to flow in: 3");pause(0.5);
title("about to flow in: 2");pause(0.5);
title("about to flow in: 1");pause(0.5);
title("go!");

loop=0;
while(loop<=10)
  pause(0.333); #VELOCITÀ DI SCORRIMENTO! voglio 1 giro in 3s -> 10*x=3
  axis([xmin xmax -1.5 +1.5]); #nota: axis() non richiede hold on/off
  xmin+=0.1*period;xmax+=0.1*period; loop+=1;
end
title("bodacious!");

#} 
# # # # # # # # # # FINE BLOCCO 1


# # # # # # # # # # BLOCCO 2

### PRENDO SEGNALE DA UNA PORTA
## 1 loop in cui stabilisco la frequenza di prelievo. va avanti fino a ctrl-c! per semplicità, facciamo 10s 
## 2 funzione: prendi tensione da pin dell'arduino e scrivila da qualche parte
## 3 dopo un tot di acquisizioni (10/s?) aggiungi al plot.

#NOTA: voglio 10 acquisizioni al secondo! ne ho 48. posso averle più precise???

##### SETUP DEL GRAFICO
# come dimensiono l'asse x? ?????????????????????????????????????
# plotto signal inizialmente. man mano che va avanti, aggiungo punti.


##### SETUP THE ARDUINO
pkg load arduino;
ar = arduino("/dev/ttyACM0");
#error handling: if no arduino found: print it and fail gracefully. how???????????????????

#select pin to read: https://octave.org/doc/v4.2.1/Terminal-Input.html
# pin = input("select pin: ");
pin="a0"; 
#leggerò un valore da 0 a 5V! e lo memorizza in un intero? ?????????????????

#creo array di valori letti:
signal = [0];
control=0; #uso per controllare le esecuzioni al secondo!
loop=0;

#inizio a plottare il segnale
plot(signal); hold on; pause(1);
xlabel("amplitude [V]");

xlimits=[0 70]; ylimits=[0 60000]; #defaults: 0 60 0 60000 
tempoacquisiz=6; # default: 6
axis([xlimits(1) xlimits(2) ylimits(1) ylimits(2)]); #asse x: da 0 a acquisizioni che mi aspetto (circa); asse y: valore max (5V)
tic; #inizio a contare. quando tok > 10, interrompi

while((control)>=0)
  pause(0.05);  #VELOCITÀ DI ACQUISIZIONE!
  analog_input = 4.89*readAnalogPin (ar, pin); 
      # ritorna un valore da 0 a 1023 corrispondenti a 0 5 V. 10bit integer value!!
      # ogni punto è 4.8876 mV (errore del 0.00001)
      # ogni punto è 4.888 mv (errore 0.0001)
      # 4.89 mV , errore 0.001 accettabile 
      # moltiplico il valore per 4.89 e ottendo il risultato in mV!!
      # moltiplico il valore per 0.00489 e ottengo in V!
      ## stessa cosa sopra per il limite dell'asse y!
  control+=1;
  
  # aggiorno il vettore di dati
  signal = [signal analog_input]; # QUANTO È SCOMODO FARE COSÌ? COPIA E INCOLLA OGNI VOLTA IL VETTORE?

  # aggiorno il plot
  plot(signal);
  
  
  
  #condizione di uscita
  if (toc>tempoacquisiz) #NOTA per avere 5 secondo, metto >6 perchè perde del tempo ed arriva a 5.001 al 4 giro ed esce prima!
    disp("10 seconds have passed. terminating acquisition.\n");
    break; 
## IMPORTANTE: decommenta il break sopra per 1 acquisizione, commentando il blocco successivo!

#{close
signal = [0];
control=0; 
loop=0;
plot(signal); hold on; pause(1);
xlabel("amplitude [V]");
axis([xlimits(1) xlimits(2) ylimits(1) ylimits(2)]); 
tic; 
#}  
  endif
end


# # # # # 
#commento finale. numero di acquisizioni (la prima è 0 default)

disp("acquisizioni completate:"); disp(control); #questo mi ritorna il numero di acquisizioni
disp("tempo di acquisizione: "); disp (toc);

##PROBLEMI FINORA
### 1 non riesce a printare su promp durante l'esecuzione. dev'essere perchè il promp è occupato
   #quindi "10 seconds have passed..." viene printato all'uscita!
### 2 quali sono i limiti di tensione del pin analogo? cosa mi ritorna analogread? millivolts? arriva fino a 700!
   # -> sono 1024 steps che identificato 0-5 Volts! 
   # -> 4.8876 millivolts per unità! 
   # -> cosa centra il pullup? è automatico? qual'è la tensione default?
### 3 perchè il prot cambia colore di continuo? perchè sovrascrive un plot sull'altro? mi sa di sì!!
    # se tolgo l'hold on ,devo formattare gli assi di volta in volta? che palle!
    ## ma arebbe senso, dato che di volta in volta potrei dover spostare in avanti nel tempo tutto!!!
    
### 4 tempistiche
  # 0.173s per plottare una volta. un po' tanto!
  # ma alla fine poi per comandare il servo non mi serve plottare
  # esperimenta: num di acquisizioni con e senza plot
  # aggiungendo codice l'esecuzione rallenta! come mi assicuro di avere 10 acquisizioni al secondo????
### 5 limiti di corrente
  # quanto può prendere di corrente massimo l'input analogico?

### 6 FLUTTUAZIONI
  # leggendo il pin è stabile. connettendo un cavo e lasciandolo libero fluttua troppo, fino a 5V! perchè?????
  # https://forum.arduino.cc/index.php?topic=50189.0 fluttuazioni random?
  # SOLUZIONE: RESISTENZA PULLUP
  ## la voglio 1/10 volte resistenza input. leggo 100Mohm di impedenza? ha senso?.> verifica con tester!
  ## -> https://learn.sparkfun.com/tutorials/pull-up-resistors/all
  ### nota: il valore dipende da condizioni di potenza che voglio avere: assorbe più o meno corretne, leggi il link di sopra

### acquisizione segnale: sarà dell'ordine dei 10mV -> riscala l'asse y per arrivare a 10 max 
### servono degli elettrodi veri.
```
