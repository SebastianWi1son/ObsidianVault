1. rx_buf[size] = '\0'
2. 如果没有 pending 命令:
      copy tele_rx_buf -> tele_cmd_buf
      cmd_ready = 1
3. 立即重启 RX DMA