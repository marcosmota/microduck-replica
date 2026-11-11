# Guia de tradução (pt-BR)

Regras compartilhadas por todos os arquivos traduzidos neste fork.

## Regras

1. Traduzir o texto para português do Brasil, tom técnico e direto. Manter a estrutura Markdown (títulos, tabelas, listas, âncoras, HTML) idêntica.
2. **Não traduzir**: blocos de código, comandos, caminhos, nomes de arquivos e pastas (inclusive os chineses), URLs, identificadores, nomes de variáveis, números de peça, SKUs.
3. Links internos continuam apontando para os mesmos arquivos. Se um título traduzido for destino de âncora (`#titulo`), ajuste a âncora para o novo título.
4. Quando houver versão `.en.md`, traduzir a partir do inglês e conferir contra o chinês; sem versão em inglês, traduzir do chinês.
5. Nomes de produtos e marcas ficam no original (Microduck, Feetech, Radxa Zero 3W, Raspberry Pi, Pollen Robotics, mjlab, etc.).
6. Valores, datas e medidas não mudam. Datas no formato original.
7. No topo de cada arquivo traduzido, logo após o título H1, inserir exatamente este aviso:

```
> 🌐 Tradução não oficial para pt-BR do [original](https://github.com/fanhao375/microduck-replica). Em caso de divergência, vale o original. Licença: CC BY-NC-SA 4.0 (texto alterado: traduzido do chinês/inglês).
```

   Em arquivos de `software/training/` (Apache-2.0), usar o mesmo aviso mas sem a menção à licença CC.
8. Não traduzir arquivos LICENSE nem textos de licenças de terceiros.
9. Não fazer commit, não trocar de branch, não tocar em arquivos fora da lista atribuída.

## Glossário

| Termo (zh/en) | pt-BR |
|---|---|
| 复刻 / replica | réplica |
| 整机 / full robot | robô completo |
| 舵机 / servo | servo (servomotor) |
| 执行器 / actuator | atuador |
| 镜像 / system image | imagem do sistema |
| 烧录 / flash | gravar (flash) |
| 装配 / assembly | montagem |
| 紧固件 / fasteners | fixadores |
| 打印 / print | impressão (3D) |
| 物料清单 / BOM | lista de materiais (BOM) |
| 标定 / 校准 / calibration | calibração |
| 步态 / gait | marcha |
| 前滚翻 / forward roll | cambalhota para frente |
| 起身 / stand up | levantar |
| 急停 / e-stop | parada de emergência |
| 手柄 / gamepad | controle (gamepad) |
| 上机实测 / on-hardware test | teste no robô real |
| 仿真 / simulation | simulação |
| 训练 / training | treinamento |
| 策略 / policy | política |
| 强化学习 / RL | aprendizado por reforço (RL) |
| 中控 / Studio web console | console web (Studio) |
| 热点 / hotspot | hotspot |
| 总线 / bus | barramento |
| 接线表 / wiring table | tabela de ligação |
| 掉电 / power loss | queda de energia |
| 踩坑记录 | registro de armadilhas (problemas encontrados) |
| 调试记录 | registro de depuração |
| 构建日志 / build log | diário de construção |
| 上游 / upstream | upstream |
| 勘误 / erratum | errata |
| IMU, HAT, firmware, Wi-Fi, SSH, GPIO, CAD, STL, MJCF | manter |
