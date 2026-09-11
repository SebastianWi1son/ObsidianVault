
**foc_transform**
1. $i_a, i_b \rightarrow$ Clarke 变换 $\rightarrow i_\alpha, i_\beta$
    ```c
    foc_clarke_transform(motor->ia, motor->ib,
                         &motor->i_alpha, &motor->i_beta);
    ```
2. $i_\alpha, i_\beta \rightarrow$ Park 变换 $\rightarrow i_d, i_q$
	```c
	float angle_dq = angle_elec - motor->zero_offset_elec;   // Park依赖angle_elec
	foc_park_transform(motor->i_alpha, motor->i_beta, angle_dq,  
                   &iq_raw, &id_raw);
	```
	
**cur_lpf**
```c
motor->iq = dsp_lpf_calc(&motor->iq_lpf, iq_raw, dt);
motor->id = dsp_lpf_calc(&motor->id_lpf, id_raw, dt);
```

**cur_pid_calc**

```c
float uq = pid_calculate(&motor->iq_pid, iq_ref, motor->iq, dt);
float ud = pid_calculate(&motor->id_pid, 0.0f, motor->id, dt);
```