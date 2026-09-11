1. sense_callback 
```c
   motor->hw.get_current_cb(motor->hw.ctx, &raw_ia, &raw_ib);
```
2. update motor
```c
motor->ia = raw_ia;  
motor->ib = raw_ib;  
motor->ic = -motor->ia - motor->ib;
```