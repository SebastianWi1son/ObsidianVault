```c
tele_data[4] -> memcpy 到 tele_tx_buf[4]
追加 tail
HAL_UART_Transmit_DMA()
```