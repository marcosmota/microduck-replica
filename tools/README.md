# Índice de ferramentas

> 🌐 Tradução não oficial para pt-BR do [original](https://github.com/fanhao375/microduck-replica). Em caso de divergência, vale o original. Licença: CC BY-NC-SA 4.0 (texto alterado: traduzido do chinês/inglês).

[Início](../README.md#navegação-da-documentação) · [Toda a documentação](../docs/README.md) · [English](README.en.md)

Use o [Studio](../software/studio/README.md) para a configuração rotineira de Wi-Fi, calibração e controle. Esta página agrupa ferramentas de gravação, depuração serial, referências da Feetech e análise para desenvolvedores. A maioria dos guias linkados está em chinês.

## 1. Gravação e ferramentas para Windows

| Tarefa | Download ou ponto de entrada | Observações |
|---|---|---|
| Gravar a imagem do pato | **[Rufus 4.15 portátil (EXE)](rufus/rufus-4.15p.exe)** · [Instruções](rufus/README.md) | Windows x64; fonte original, licença e checksums incluídos |
| Depurar servos Feetech | **[Arquivo do FD 1.9.8.5](飞特/FD1.9.8.5%28250729%29.7z)** · [Instruções](飞特/README.md) | Extraia e execute o FD para configurar IDs e parâmetros e ler o estado |
| Enviar dados seriais brutos | [UartAssist.zip no repositório da Feetech](https://gitee.com/ftservo/fddebug/blob/master/UartAssist.zip) | Assistente serial genérico; envie dados conforme o protocolo do dispositivo |
| Corrigir o reconhecimento do adaptador | [Links dos drivers CH343 / CH340](飞特/README.md#驱动与串口助手) | Primeiro confirme o modelo do chip do adaptador no Gerenciador de Dispositivos |

### Configurações seriais

| Conexão | Configuração típica | Guia |
|---|---|---|
| Terminal serial USB da placa principal | 115200, 8N1 | [Guia de conexão da placa](radxa/镜像使用说明.md) |
| Barramento Feetech configurado para este projeto | 1000000, 8N1 | [Guia dos servos](飞特/README.md); use um adaptador TTL adequado e alimentação separada para os servos |

8N1 significa 8 bits de dados, sem paridade e 1 bit de parada. Use o baud rate real do dispositivo caso ele tenha sido alterado.

## 2. Referências e SDKs da Feetech

O [índice de referências da Feetech](../docs/飞特资料/README.md) reúne links para os manuais oficiais atuais, cópias offline datadas, especificações dos modelos e SDKs.

- Protocolos: [SCS](http://doc.feetech.cn/#/f?q=2506ba160b93) · [mapa de registradores STS](http://doc.feetech.cn/#/f?q=2506abfb1cb0) · [mapa de registradores SMS](http://doc.feetech.cn/#/f?q=2602a06ca658). Versões oficiais verificadas em 2026-10-25; as cópias offline mantêm datas de snapshot próprias.
- SDKs oficiais: [Python](https://gitee.com/ftservo/FTServo_Python) · [Linux](https://gitee.com/ftservo/FTServo_Linux) · [Arduino](https://gitee.com/ftservo/FTServo_Arduino) · [STM32 HAL](https://gitee.com/ftservo/FTServo_stm32HAL).

## 3. Ferramentas de depuração do repositório

| Ferramenta | Local no repositório | Finalidade |
|---|---|---|
| Console web de servos | [tools/servo-web/](servo-web/README.md) | Depuração serial, leitura de estado, poses, sequências e exibição 3D; inclui modo de simulação |
| Testador HD1910M | [tools/FeeTech_HD1910M_Servo/](FeeTech_HD1910M_Servo/README.md) | CLI em Rust para varredura, registradores, configuração e monitoramento; usa o mapa de registradores deste modelo |
| Testador STS3215 | [tools/sts3215Servo_testtool/](sts3215Servo_testtool/README.md) | CLI em Rust para varredura, leitura de estado e testes individuais ou em lote; voltado ao STS3215 |
| Verificação do protocolo do IMU de barramento | [tools/imu200/](imu200/) | [imu200.py](imu200/imu200.py): simulação, vetores de teste e verificações na placa real para o bloco de 15 bytes no endereço 56 da Feetech; veja o [protocolo do barramento](../hardware/imu_to_dxl/总线协议.md) |

## 4. Gravação de dados e análise

| Ferramenta | Local no repositório | Finalidade |
|---|---|---|
| Análise de marcha | [tools/gait-analysis/](gait-analysis/README.md) | Analisa no computador gravações JSONL: erro de rastreamento, vibração e movimento dos pés |
| Gravação e análise a bordo | [tools/radxa/onboard-analysis/](radxa/onboard-analysis/README.md) | Grava juntas, IMU e comandos para depuração no robô real |
| Análise de contato de dobra | [tools/fold_contact/](fold_contact/README.md) | Investiga o contato entre peças usando a geometria da simulação do upstream |
| Desenhos de montagem e análise de furos | [scripts/](../scripts/) | Gera desenhos, exporta malhas de análise e varre características dos furos; veja os [comandos de reprodução](../docs/项目研究归档.md#reprodução) |

## 5. Scripts antigos de preparação do cartão

[tools/radxa/](radxa/README.md) preserva scripts antigos de preparação do cartão, bootloader e depuração de rede. A imagem completa atual é construída a partir de [software/image/](../software/image/README.md); ao gravar um cartão, comece pelo [guia de primeiro uso](../docs/上手指南/README.md).
