```

pkg load arduino;
ar = arduino("/dev/ttyACM0");

duty=0.5;
loop=2;
counter=1;
loop_values=[0];
while(counter>10) 
duty=input("entery duty:");
writePWMDutyCycle(ar,'D3',duty);
end 

#SERVO: MG996R pwm 1-2ms

#servono 2 bottoni.
# button1L gira a sx. PWM 0.5
# button1R gira a dx. PWM 0.9

#funzione: button pressed
button1r = 0; #collegato a 3.3V con 10K a terra mi da tensione 677 -> ovvero????
button1l = 0;
duty=0; #duty sarà 0.9 per andare a sx, 0.5 per destra, 0 per stare fermo
#NOTA: lo imposto manualmente nella funzione perchè tanto sono solo 3 casi!
while (true)
  
  button1r = readAnalogPin (ar, "A0");
  button1l = readAnalogPin (ar, "A7");
  if (button1r > 600 && button1l > 600)
    button1r=button1l=0;
  endif  
  
  if (button1r == 0 && button1l == 0)
    writePWMDutyCycle(ar,'D3',0);      
  endif
  if (button1r > 600 )
    writePWMDutyCycle(ar,'D3',0.6);
    #pause(3);break;
  endif
  
  if (button1l > 600 )
    writePWMDutyCycle(ar,'D3',0.9);
    #pause(3);break;
  endif
  if (button1l == 0 && button1r == 0)
    writePWMDutyCycle(ar,'D3',0);      
  endif 
end
```
