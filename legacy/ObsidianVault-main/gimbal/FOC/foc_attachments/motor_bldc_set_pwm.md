```c
__HAL_TIM_SET_COMPARE(m->htim, TIM_CHANNEL_1,  
                      (uint32_t)(ua / m->bus_voltage * m->pwm_period));  
__HAL_TIM_SET_COMPARE(m->htim, TIM_CHANNEL_2,  
                      (uint32_t)(ub / m->bus_voltage * m->pwm_period));  
__HAL_TIM_SET_COMPARE(m->htim, TIM_CHANNEL_3,  
                      (uint32_t)(uc / m->bus_voltage * m->pwm_period));
```