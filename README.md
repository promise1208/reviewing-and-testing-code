# reviewing-and-testing-code

嵌入式软件审查与测试 Skill，供支持 Skill 机制的 Agent（Claude Code / Codex / OpenCode 等）加载使用。

## 用途

当需要审查、审计或验证涉及以下内容的嵌入式改动时加载本 Skill：

- 系统：嵌入式 Linux、FreeRTOS、RT-Thread、其他 RTOS、裸机、BSP、HAL、驱动
- 底层机制：ISR、DMA、cache、信号量、互斥锁、原子操作、共享内存、内存边界、整数溢出、资源耗尽
- 通信与外设：GPIO、UART、I2C、SPI、CAN、USB、SDIO、以太网、MQTT、HTTP、TLS、Wi-Fi、BLE、ADC、DAC、PWM、RTC、看门狗、Flash、显示与 MIPI 等

## 目录结构

| 路径 | 说明 |
| --- | --- |
| `SKILL.md` | Skill 主入口（触发条件与工作流程） |
| `agents/openai.yaml` | OpenAI Agent 配置 |
| `references/` | 分主题审查参考（embedded-linux、freertos、rt-thread、connectivity、gui、services、bare-metal、embedded） |

## 安装

复制整个目录到对应 Agent 的 skills 目录即可，例如 Codex：

```bash
cp -r reviewing-and-testing-code ~/.codex/skills/
```
