# Engenharia de hardware

> 🌐 Tradução não oficial para pt-BR do [original](https://github.com/fanhao375/microduck-replica). Em caso de divergência, vale o original. Licença: CC BY-NC-SA 4.0 (texto alterado: traduzido do chinês/inglês).

Ponto de entrada único para as três placas de circuito feitas por conta própria, o controlador principal Radxa e o HAT oficial de código aberto.

| Placa | Compra / projeto | Prévia do design | Materiais |
|---|---|---|---|
| **Placa IMU de barramento v2** | [imu_to_dxl_1V0A](新版/imu_to_dxl_1V0A/) | [3D](../assets/hw/imu-to-dxl-1v0a-3d.png) · [PCB](../assets/hw/imu-to-dxl-1v0a-pcb.png) | Projeto no EasyEDA (LCSC) edição Pro, PDF do esquemático, STEP do PCB; [descrição da nova versão](新版/README.md) |
| **HUB do barramento de servos** | [wire_hub_1V0A](新版/wire_hub_1V0A/) | [3D](../assets/hw/wire-hub-1v0a-3d.png) · [PCB](../assets/hw/wire-hub-1v0a-pcb.png) | Projeto no EasyEDA (LCSC) edição Pro, STEP do PCB; [descrição da nova versão](新版/README.md) |
| **Placa de interface da bateria** | [POGO_PIN_1V0A](新版/POGO_PIN_1V0A/) | [3D](../assets/hw/pogo-pin-1v0a-3d.png) · [PCB](../assets/hw/pogo-pin-1v0a-pcb.png) | Projeto no EasyEDA (LCSC) edição Pro, STEP do PCB; [descrição da nova versão](新版/README.md) |
| **Controlador principal Radxa ZERO 3W** | [Compra do controlador (Taobao)](https://item.taobao.com/item.htm?id=763812025816) | [Imagem oficial da placa](../assets/hw/radxa-zero3w-official.webp) | [Documentação oficial](https://docs.radxa.com/en/zero/zero3) · [Arquivos de projeto de hardware](https://docs.radxa.com/en/zero/zero3/download#hardware-design): esquemático, dimensões 2D e modelo 3D |
| **RPI Robot HAT oficial** | [Projeto KiCad da Pollen Robotics](https://github.com/pollen-robotics/elec_RPI_Robot_HAT) | [Imagem oficial do design](../assets/hw/rpi-robot-hat-official.png) | [Arquivos de produção e protótipo](https://github.com/pollen-robotics/elec_RPI_Robot_HAT/tree/main/production): Gerber, BOM, coordenadas de pick-and-place, esquemático e STEP |

O estado do design, a ligação e os registros de validação de cada placa estão na descrição do respectivo projeto.

A [placa IMU de barramento antiga e seu firmware](imu_to_dxl/README.md) preservam o projeto do circuito, o esquemático, a tabela de ligação, a BOM da placa e os materiais do firmware.

[Lista de compras da montagem do robô completo](../BOM/README.md) · [Introdução ao hardware](../docs/硬件入门.md) · [Voltar ao início](../README.md)
