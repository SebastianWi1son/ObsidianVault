**adc_scale_calc**
1. $Raw \rightarrow I_{phase} = (Raw - Offset) \times \frac{V_{ref} / Resolution}{R_{shunt} \times Gain}$
	```c
	#define ADC_SENSE_A_PER_LSB(cfg) \ 
	((cfg)->vref / (cfg)->curr_resolution) / ((cfg)->shunt_res * (cfg)->amp_gain)
	```

2. CONVERT从注入组获取原始值并转换
	```c
	raw = (uint16_t)HAL_ADCEx_InjectedGetValue(inj->hadc, inj->rank_ia);
	inj->axis->cached_ia = ((float)raw - inj->axis->offset_ia) * ADC_SENSE_A_PER_LSB(inj->cfg);
	```

**adc_injected_fetch**  
	Q: Why use injected ADC?  
	A: The injected group has higher priority and can be triggered precisely by TIM TRGO. This lets us sample the phase current at the PWM midpoint, where the low-side MOSFETs are conducting, so the current reading is cleaner and more reliable.
1. ADC_FLAG_JEOC
	```c
	if (__HAL_ADC_GET_FLAG(inj->hadc, ADC_FLAG_JEOC)) {  
        // 获取 ia, ib 缓存 ...
        __HAL_ADC_CLEAR_FLAG(inj->hadc, ADC_FLAG_JEOC);  
    }  
	```
2.  cached_i
	```c
	raw = (uint16_t)HAL_ADCEx_InjectedGetValue(inj->hadc, inj->rank_ia);  
	inj->axis->cached_ia =  adc_sense_raw_to_current(inj->cfg, raw, inj->axis->offset_ia);  
	```