# verification
```c
if (!t || !t->motor) return; 
if (!b->enabled || !b->triggered) return;  
if (++b->decim_counter < b->decimation) return;  
b->decim_counter = 0;  
```

# foc_motor_t copy
```c
  foc_motor_t *m = t->motor;  
  tune_sample_t *s = &b->buf[b->head]; 
```

# snap shot
```c
s->abs_angle     = m->abs_angle;  
s->target_angle  = m->planned_target_prev;  
s->velocity      = m->velocity;  
/* 电压模式: vel_pid_out = Uq; 电流模式: uq = iq_pid.output_ramp.prev_out */  
s->uq            = m->vel_pid.output_ramp.prev_out;  
s->angle_pid_out = m->angle_pid.output_ramp.prev_out;  
s->vel_pid_out   = m->vel_pid.output_ramp.prev_out;  
s->iq = m->iq;  
s->id = m->id;  
s->tick = b->count;  
  
b->head = (b->head + 1) % TUNE_BUF_SAMPLES;  
b->count++;
```