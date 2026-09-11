**nomorlization**
```c
float angle = fmodf(angle_elec - motor->zero_offset_elec, TWO_PI);//  
if (angle < 0.0f)  angle += TWO_PI;
```

**inv_park_transform**
```c
foc_inv_park_transform(uq, ud, angle, &u_alpha, &u_beta);
```

**svpwm center align**
```c
float center = motor->cfg.voltage_supply / 2.0f;  
float ua = u_alpha + center;  
float ub = (SQRT3 * u_beta - u_alpha) / 2.0f + center;  
float uc = (-u_alpha - SQRT3 * u_beta) / 2.0f + center;
```