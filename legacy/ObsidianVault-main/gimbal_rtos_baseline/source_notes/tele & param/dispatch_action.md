```c
case TELE_CMD_MOTOR_RUN:
    if (cmd->value) {
        if (app_safety_is_faulted()) break;

        foc_reset_controller_state(&gimbal_pitch);
        foc_reset_controller_state(&gimbal_yaw);
        foc_enable(&gimbal_pitch);
        foc_enable(&gimbal_yaw);
        system_runflag = 1;
    }
```