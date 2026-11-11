# Firmware 0.2.0 do imu_to_dxl

> 🌐 Tradução não oficial para pt-BR do [original](https://github.com/fanhao375/microduck-replica). Em caso de divergência, vale o original. Licença: CC BY-NC-SA 4.0 (texto alterado: traduzido do chinês/inglês).

Destinado à placa **STM32G031F8P6 + LSM6DSV16X** feita por um colega de equipe. A base de hardware é o
commit `a1f4979c425bd9228622be89f2bfa4d1de7fbde3` de `fanhao375/microduck-replica`
(2026-09-08, com J4/J5 e RX_EN). Em 2026-09-20 foi concluída a primeira inicialização em placa real: conexão SWD com J-Link,
gravação do firmware com verificação de leitura, Keil parando em `main`, clock de 64MHz e amostragem do IMU, todos verificados.
No teste curto, a identificação do IMU foi `0x70`, ready=1 e erros de SPI/FIFO iguais a 0; o barramento externo e os servos ainda não foram testados em campo.

**A 0.2.0 (2026-09-22) troca o protocolo de barramento para Feetech SCS/STS**, implementado item por item conforme o [protocolo de barramento](../总线协议.md): ID 200, bloco de 15 bytes no endereço 56, enfileiramento do sync_read. O IMU, a camada de placa e o projeto Keil não foram alterados. Os testes de host no computador e a aceitação "código real do protocolo sobre barramento simulado" (`Tests/bus_sim_test.py`, usando os critérios do `check` de [`tools/imu200`](../../../tools/imu200)) passaram integralmente; já foi gravado na placa de desenvolvimento original e o IMU funciona normalmente. **O barramento real e os servos ainda não foram testados em campo.**
Os limites entre compilação, testes de desktop e a primeira validação em placa real estão em [VALIDATION.md](VALIDATION.md).

Este diretório fica junto do esquemático e do PCB, fornecendo o código-fonte, o projeto Keil e o único artefato pré-compilado mantido,
[`Build/imu_to_dxl.hex`](Build/imu_to_dxl.hex). O console de depuração serial [`tools/servo-web`](../../../tools/servo-web) consegue, em uma mesma página web, sobrepor os ângulos das juntas dos servos e a atitude do tronco dada pelo IMU, com três formas de leitura: `--imu-bus` (barramento dos servos, no mesmo sync_read do controlador principal), `--imu-swd` (ST-Link / DAPLink), `--imu-jlink` (J-Link).

O observador SWD / J-Link decodifica do HEX a imagem de Flash de **10.724 bytes** a partir de `0x08000000`, confere o SHA-256
`66bc7c532f5a1ad87e3dfff71a4b51f8c4f20840a8f62436874fa5b7c665938e` e valida a imagem na placa.
Somente essa base pode usar o layout de RAM já revisado: `sample=0x2000006C` (60 bytes),
`tick=0x20000A34` (4 bytes), conferidos contra o MAP da 0.2.0. Para usar apenas a página de atitude não é preciso recompilar; depois de alterar o firmware é preciso revisar de novo
o valor de verificação da Flash e o endereço e tamanho desses dois símbolos, sem reaproveitar endereços antigos para ler um programa novo.

Para depurar o código-fonte com pontos de parada, é preciso reconstruir o `.axf` no seu próprio ambiente; a compilação ainda gera `.map` e `.bin` para conferência durante o desenvolvimento,
e esses arquivos e os artefatos de teste são excluídos pelo `.gitignore`, sem ir para o repositório.

Fluxo de dados: LSM6DSV16X → SPI1 → STM32 → USART2 + buffer externo → barramento half-duplex de fio único → controlador principal Linux.
Implementa o **protocolo Feetech SCS/STS, ID 200, 1Mbps**, fingindo ser um servo no barramento dos servos (a 0.1.0 era Dynamixel Protocol 2.0).
O código não aciona movimento de servos, não escreve nos Option Bytes do MCU e não grava a configuração do firmware na Flash.

**Aceitação no barramento real**: ligue o adaptador half-duplex (URT-2) e rode `python tools/imu200/imu200.py check --port COMx`; os 13 critérios foram escritos conforme o código-fonte do controlador principal, e só depois de passar é que se pode ir para o robô completo. As verificações elétricas (liberação do DE, níveis, medição real do T_resp) ainda exigem um analisador lógico.

## 1. Começar direto

### Keil

1. Ao usar o J-Link, abra `MDK-ARM/imu_to_dxl_jlink.uvprojx`.
2. No seu Keil, escolha o Arm Compiler 6 instalado legalmente (verificados 6.24 e 6.22; versões antigas como o MDK 5.24, com o 6.7, não compilam: os cabeçalhos do CMSIS 6 exigem ≥ 6.10, e dará erro de `cmsis_clang.h` não encontrado) e o pacote do dispositivo `STM32G031F8Px`.
3. Pressione **F7** para compilar; `Build/imu_to_dxl.hex` é o arquivo de gravação, e o `.axf` serve para depuração do código-fonte com pontos de parada.
4. Em Debug → Settings, escolha o seu J-Link e confira SWD, o algoritmo de Flash STM32G0xx de 64KB e Run to main.
   O `JLinkSettings.JLinkScript` define a velocidade de transferência em **100kHz** após a conexão e o reset; mantenha esse arquivo.
   A primeira placa teve falha de transferência de dados ao baixar a 4MHz; na primeira conexão, recomenda-se usar primeiro os 100kHz já verificados.
5. Pressione **Ctrl+F5** para entrar na depuração e parar em `main`; **F10** passa por cima, **F11** entra na função, **F5** executa.
   Pausar altera a amostragem e a temporização do barramento; em testes de comunicação contínua, execute em velocidade total.

A configuração ST-LINK do `MDK-ARM/imu_to_dxl.uvprojx` original foi mantida. Ao trocar para um J-Link, em Debug → Settings
escolha de novo a sonda.

### VS Code

Abra esta pasta no VS Code para editar o código-fonte e execute os comandos de compilação abaixo no terminal integrado.
Esse caminho usa o VS Code para edição e o Keil para compilação e depuração em hardware; não é preciso instalar CubeMX, CubeIDE nem GCC para compilar.
O código-fonte já traz os cabeçalhos CMSIS necessários e o driver do IMU da ST, então a primeira compilação não precisa baixar dependências pela rede.

Também é possível executar, em um terminal PowerShell neste diretório:

```powershell
powershell -NoProfile -ExecutionPolicy Bypass -File .\Tools\build.ps1
```

O caminho padrão do compilador `E:\Keil_v5\ARM\ARMCLANG` é apenas um exemplo. Em outros computadores, use `-ToolchainRoot` para indicar o Arm Compiler instalado legalmente.
O diretório temporário de compilação fica em `Build`, dentro deste projeto; o sistema operacional e o cache de licenças das ferramentas existentes continuam sendo gerenciados por cada software.
A compilação substitui o firmware pré-compilado em `Build`; para manter a base já verificada da página web, guarde antes o artefato original.

## 2. Conteúdo do projeto

| Arquivo | Conteúdo |
|---|---|
| `Core/Inc/board_config.h` | Chaves de nível de placa centralizadas, clock, baud rate, taxa do SPI |
| `Core/Src/board.c` | Clock, GPIO, SPI, USART, reserva do SWD, base de tempo em ms/µs, watchdog |
| `Core/Src/imu.c` | Verificação de identidade, releitura da inicialização, FIFO, aceleração/giroscópio brutos, SFLP, nova tentativa em caso de falha |
| `Core/Src/protocol.c` | Escravo Feetech SCS/STS: análise de quadros (ressincronização pelo checksum), resposta a cada instrução, enfileiramento do sync_read, temporização da resposta |
| `Core/Src/control_table.c` | Bloco de 15 bytes no endereço 56, bits de estado, registradores de diagnóstico |
| `Core/Src/main.c` | Escalonamento do laço principal, comandos seriais e log de estado |
| `Drivers/` | Código-fonte ST/CMSIS em versão fixa e licenças |
| `MDK-ARM/` | Projeto Keil, layout de link e script de conexão em baixa velocidade do J-Link |
| `Tests/` | Testes de protocolo, sensor simulado e registradores que rodam sem a placa; o `bus_sim_test.py` compila o código real do protocolo em uma DLL e o conecta a um barramento simulado para executar a aceitação |

Sem memória dinâmica, sem RTOS, sem dependência de HAL. O código de nível de placa usa as definições de registradores oficiais do CMSIS da ST; o sensor usa o driver oficial independente de plataforma.
A recepção do barramento externo do STM32 usa uma fila circular por interrupção, guardando o instante de recepção de cada byte; o protocolo, a amostragem e a formatação do log rodam no laço principal.
O envio do log usa uma fila por interrupção à parte, sem bloquear a recepção do barramento.

## 3. Configuração de hardware padrão

| Função | Pino do chip / configuração |
|---|---|
| MCU | STM32G031F8P6, TSSOP20, 64KB de Flash / 8KB de RAM |
| Clock do sistema | HSI16 → PLL, M=1, N=8, R=2, 64MHz, Flash com 2 wait states |
| Opção de clock externo | `BOARD_USE_HSE_BYPASS=1`, clock ativo de 16MHz do X1 ligado ao pino 2 do encapsulamento; se a partida falhar, volta à rota HSI e registra o estado |
| SPI do IMU | SPI1 mode3, 8bit, CS por software; 2MHz a 64MHz; PA4=CS, PA5=SCK, PA6=MISO, PA7=MOSI |
| INT do IMU | Na primeira versão faz polling do FIFO; o sinal de interrupção do PA0/pino 15 do encapsulamento não é usado por enquanto |
| Serial do barramento | USART2, PA2=TX, PA3=RX, AF1, 1Mbps, 8N1; sem habilitar o HDSEL de pino único do USART |
| DE do barramento | PA1 → U3.1 `1OE#`, **ativo em baixo**; controlado por GPIO, liberado só depois do USART TC |
| RX_EN do barramento | PB7 → U3.7 `2OE`, **ativo em alto**; desligado durante a transmissão, PA3 com pull-up; o PB8 do mesmo pino do encapsulamento fica como entrada analógica |
| Serial de depuração | USART1, PA9/PA10 remapeados via SYSCFG para os pinos 16/17 do encapsulamento, 115200, 8N1; `BOARD_LOG_UART_SWAP=0` |
| SWD | PA13=SWDIO, PA14=SWCLK, mantendo as funções de depuração do reset padrão |
| Watchdog | Ligado por padrão, cerca de 2 segundos, com o erro do LSI afetando a duração real; congelado quando o depurador pausa |

Se a partida do PLL falhar, o HSI16 continua em operação, e o BRR da serial e os temporizadores são calculados pela frequência real.
Nesse caso o SPI também cai para 0,5MHz,
e a latência de serviço aumenta; trata-se de um modo de diagnóstico de falha, que exige verificar o estado do clock e não serve como configuração para a aceitação de desempenho.

### A ligação do J3 deve seguir o [PDF do esquemático](../imu_to_dxl-原理图.pdf) e a rede da placa real

| Pino do J3 | Sinal do esquemático mais recente | Ligação |
|---|---|---|
| 1 | GND | Terra comum do J-Link/ST-LINK/USB-TTL |
| 2 | SWDCLK | SWCLK do J-Link/ST-LINK |
| 3 | SWDIO | SWDIO do J-Link/ST-LINK |
| 4 | UART_RX, pino 17 do encapsulamento do U1 | TX do USB-TTL, nível de 3,3V |
| 5 | UART_TX, pino 16 do encapsulamento do U1 | RX do USB-TTL, nível de 3,3V |
| 6 | 3,3V | Referência de tensão do depurador; não ligue 5V aqui |

Os quatro fios do SWD do J3 usam os pinos **1, 2, 3 e 6**; não encaixe o cabo flat antigo de quatro pinos inteiro nos pinos 1–4,
o pino 4 é entrada serial e não pode ser tratado como alimentação ou referência de tensão. Na primeira energização, confira antes pelos pads / pela rede do EDA.
O J3 não expõe o NRST; se precisar de conexão under-reset, ligue separadamente no ponto de teste real do NRST.
Se o depurador tiver saída de alimentação, não a coloque às cegas em paralelo com outra fonte; confirme antes se é entrada VTref ou saída de alimentação.

J1/J2: 1=GND, 2=VDD_BUS, 3=DATA; J4/J5: 1=DATA, 2=VDD_BUS, 3=GND.
Os testes de barramento exigem um adaptador USB half-duplex de fio único; um USB-TTL comum serve apenas para o log do J3.

## 4. Comportamento do IMU

- O WHO_AM_I deve ser `0x70`. Após o reset por software, configura-se passo a passo e relêem-se ODR, faixa e a habilitação do SFLP.
- Aceleração ±4g, 0,122mg/LSB; giroscópio ±500dps, 17,5mdps/LSB.
- Aceleração, giroscópio, SFLP e processamento em lote do FIFO são todos a 120Hz, para a leitura a 50Hz do controlador principal.
- O FIFO distingue aceleração, giroscópio e quaternion por etiqueta; no máximo 16 registros por rodada, sem supor que os três venham no mesmo registro.
- Na inicialização, em falha de SPI, estouro do FIFO ou dados obsoletos, `ready=0`; após cerca de 1 segundo, nova tentativa automática.
- Se o giroscópio ou o quaternion ficar mais de 100ms sem atualização, é invalidado. Mesmo que o laço principal esteja ocupado e sem executar o polling de amostragem, a leitura do snapshot confere a idade de novo.
- A interface de leitura mantém um snapshot consistente, sem atualizar a atitude dentro da interrupção. A rotação de montagem do sensor fica com o controlador principal, evitando repetir a transformação de coordenadas oficial.
- Esta versão não guarda viés medido; o SFLP começa a estimar do estado padrão de energização. A atitude de seis eixos não fornece referência absoluta de rumo.

A pose unitária legítima `xyz=0,0,0` coincide com a sentinela de "não pronto" toda zero do upstream. Quando já se recebeu uma atitude válida do FIFO, apenas a codificação de meia precisão de x
é definida como o zero negativo IEEE `0x8000`, com valor matemático ainda 0; o verdadeiro "não pronto" sempre devolve tudo zero. Detalhes na descrição do driver e nos testes de regressão.

**Limite do tratamento de falhas em tempo de execução:** o controlador principal oficial examinado, ao receber atitude toda zero, pode manter a última atitude, e o seu `ready()` não é revogado automaticamente por isso.
Antes da integração formal em pé/em movimento, o controlador principal deve ler adicionalmente o `ready`, o contador de amostras e a idade do diagnóstico desta versão, e executar o tratamento de invalidação.
O script de coleta para PC desta versão já lê esses campos; não se deve tomar "a placa isolada devolve tudo zero" como se o robô completo já tivesse comportamento automático de parada.

## 5. Comunicação e registradores

Formato do quadro `FF FF ID LEN INSTR parâmetros… CHK`, `CHK = ~(ID+LEN+INSTR+Σparâmetros)`, sem byte stuffing, de modo que `FF FF` também pode aparecer na carga útil: a recepção termina de receber conforme o comprimento e só então verifica o checksum; com checksum errado, descarta um byte e volta a procurar o cabeçalho do quadro, sem ser desviada por falsos cabeçalhos na carga útil. O comportamento de cada instrução está em [protocolo de barramento §8](../总线协议.md#8-como-responder-às-outras-instruções):

| Instrução | Comportamento |
|---|---|
| `SYNC_READ 0x82` (broadcast) | Estando na posição k da lista: k=0 envia 50 µs depois de receber a instrução; k>0 espera os quadros de resposta dos k dispositivos anteriores (ID da frente, comprimento certo, 5º byte que não é código de instrução), e se um dispositivo da frente ficar 1 ms sem se manifestar, considera-o fora de linha e o pula |
| `PING` / `READ` (unicast) | Responde com quadro de estado / dados, ERROR sempre 0; PING em broadcast não responde |
| `WRITE` / `REG_WRITE` (unicast) | Responde com ACK, ignorando por completo o conteúdo (o controlador principal nunca escreve nesta placa) |
| `ACTION` `0x06` `0x09` `0x0A` `0x0B` (unicast) | ACK, sem executar ação |
| `REBOOT 0x08` | Não responde, reinicia de verdade |
| `SYNC_WRITE`, outros broadcasts | Não responde |

Temporização (todos valores inferidos, a medir; veja protocolo de barramento §5/§12; os 50 µs são o número da lógica do protocolo, e somando o tempo de CPU de análise e montagem do pacote o valor real precisa ser medido com um analisador lógico): T_resp 50 µs; descarta-se o quadro residual se a lacuna entre bytes dentro de um quadro passar de 500 µs; no enfileiramento, 1 ms de silêncio do barramento faz considerar o dispositivo anterior fora de linha; desiste se esperar 25 ms sem chegar a vez; se, passada a hora, ainda não conseguir enviar após 2 ms de atraso, desiste, para não disputar o barramento na transação seguinte. Antes de enviar, verifica-se de novo atomicamente os novos dados RX recebidos, o USART BUSY e a flag de falha; se ocupado, abandona essa resposta.
Na interrupção de recepção, só o **estouro (ORE)** é tratado como falha do barramento e esvazia a fila de recepção; as flags de ruído / erro de quadro só contam, e os bytes são recebidos normalmente, com o erro sendo descartado pelo checksum —— a 0.1.0 esvaziava sempre, e um byte com flag de ruído jogava fora todo o sync_read, perdendo junto os 15 servos desse tick (alterado após a revisão da 0.2.0).

**Coloque o ID200 na primeira posição da lista do Sync Read**, como no `bus.rs` oficial; no meio também consegue responder (regras de enfileiramento), mas quando o dispositivo da frente está ausente a análise do lado do controlador principal desalinha, veja protocolo de barramento §2.

Todos os campos multibyte são little-endian.

| Endereço | Nº de bytes | Conteúdo / permissão |
|---|---:|---|
| 0 / 1 | 1 cada | Versão do firmware 0.2 (maior.menor), sem se passar pela 3.46 dos servos |
| 2 | 1 | END = 0 (little-endian) |
| 3 | 2 | Modelo personalizado `0x4D44` |
| 5 / 6 / 8 | 1 cada | ID 200 / baud rate 0 (1Mbps) / nível de estado da resposta 1 |
| **56** | **15** | **O bloco que o controlador principal lê a cada tick**: gyro XYZ (int16, 17,5 mdps/LSB), quaternion XYZ (binary16), contador de amostras (u8), bits de estado, reservado 0. Ordem de eixos bruta, sem transformação de coordenadas |
| 124 | 6 | gyro XYZ, três int16, ±500dps (diagnóstico, a mesma cópia do bloco) |
| 130 | 6 | quaternion XYZ, três IEEE binary16; o W é reconstruído pelo controlador principal |
| 136 | 6 | Aceleração bruta XYZ, três int16, ±4g |
| 142 | 1 | 8 bits baixos do contador de amostras de atitude |
| 143 | 1 | bit0=ready, bit1=configured, bit2=error |
| 144 / 145 | 1 cada | WHO_AM_I / código de erro |
| 146 / 147 | 1 cada | Estado do clock / chave do watchdog |
| 148 / 152 / 156 | 4 cada | Contadores de amostras de atitude / gyro / accel |
| 160 / 164 / 168 | 4 cada | uptime_ms / idade do gyro em ms / idade da atitude em ms |
| 172 / 176 / 180 / 184 | 4 cada | Erros de SPI / estouros do FIFO / quaternions inválidos / número de recuperações |
| 188 / 192 / 196 / 200 | 4 cada | Estouro da fila RX / erros de UART (ruído/erro de quadro/estouro, todos contam) / erros de checksum / pacotes inválidos |
| 204 / 208 / 212 / 216 | 4 cada | Timeouts de recepção / pacotes recebidos válidos / pacotes realmente enviados / respostas canceladas |
| 220 / 224 / 228 / 232 | 4 cada | Valor bruto da causa de reset do RCC / frequência principal em Hz / timeouts de TX / logs descartados |
| 236 | 1 | Versão 2 da tabela de registradores de diagnóstico deste projeto (protocolo Feetech, bloco em 56) |
| 240 | 4 | Número de vezes que, no enfileiramento do sync_read, se julgou o dispositivo da frente fora de linha e se o pulou |
| 244 | 4 | Número de vezes que se detectou barramento ocupado antes de enviar e se abandonou a resposta |

Os demais endereços 0~255 são lidos como 0; qualquer escrita dá ACK mas não surte efeito (para reinicializar o IMU, use o comando `r` da serial de log do J3). 56~70 é a interface do controlador principal; 124 em diante é uma **extensão de diagnóstico personalizada deste projeto**, sem afirmar compatibilidade com os registradores de tensão/temperatura dos servos.

Bits de estado no bloco (byte 13): BIT0 fusão não pronta (quaternion todo zero); BIT1 falha de comunicação com o IMU (SPI/timeout de reset/configuração/dado obsoleto/FIFO); BIT2 falha no autoteste (WHO_AM_I incorreto); BIT3 mais de 4 atualizações entre duas leituras do bloco (com amostragem a 120 Hz e leitura a 50 Hz, o normal é 2~3 por leitura).
Os códigos de erro estão em `imu.h`: 0 normal, 1 partida, 2 SPI, 3 identidade, 4 timeout de reset, 5 configuração, 6 obsoleto, 7 FIFO, 8 quaternion.
Estado do clock: 0=HSI PLL, 1=HSE PLL, 2=falha do HSE, volta ao HSI PLL, 3=falha do PLL, HSI16, 4=falha na troca, volta atrás.

## 6. Ordem de trabalho quando a placa chegar

1. Confirme a versão do hardware, a direção do J3 e a alimentação; primeiro energize a placa isolada com limitação de corrente, verificando 3,3V, curto-circuito, aquecimento e reset.
2. Reconheça o STM32 por SWD, baixe o HEX, confirme que há ponto de parada em main e que a placa inicia tanto por reset quanto por religamento da energia.
3. Ligue a serial de log do J3, 115200/8N1. Deve haver informação de versão; a cada segundo saem ready, id, erros e dados.
   `s` imprime o estado, `l` alterna o log periódico, `r` reinicializa o IMU, `?` mostra a ajuda.
4. Verifique WHO_AM_I=0x70, ready=1, contador de amostras crescendo; vire/gire nos três eixos para conferir a aceleração e a direção da atitude.
5. Ligue só o barramento do IMU (adaptador half-duplex como o URT-2), execute `python tools/imu200/imu200.py check --port COMx --only-imu` (pergunta só pelo 200), depois ligue os servos e execute o `check` completo.
6. Use um analisador lógico para verificar 1Mbps, polaridade do DE, liberação após o último bit de parada, recuperação do RX e o pior atraso de resposta.
7. Ligue um servo e depois o barramento completo; ID200 na primeira posição. Registre erros, perda de pacotes, FIFO e idade das amostras.
8. Depois de concluir o tratamento de falhas do controlador principal e a conferência da direção de montagem, faça a integração do robô completo em estado apoiado.

### Ferramenta de aceitação no computador

A aceitação do barramento usa [`tools/imu200/imu200.py`](../../../tools/imu200/imu200.py) (o `Tools/imu_probe.py` da 0.1.0 falava o protocolo Dynamixel e foi removido junto com a troca de protocolo). Requer Python 3.10+ e pyserial:

```powershell
python tools/imu200/imu200.py check --port COM5   # 13 项逐条 PASS/FAIL，判据照主控源码
python tools/imu200/imu200.py demo                # 不接硬件，看假小板怎么答
```

Substitua COM5 pela porta do **adaptador de barramento half-duplex**; a porta de log do J3 não consegue responder a essas instruções de protocolo. O log é gravado em `tools/imu200/logs/` (TX/RX brutos em hexadecimal + uma linha JSONL por quadro).
É preciso ter um Python executável instalado; o atalho de espaço reservado da Microsoft Store no Windows não consegue executar esses scripts; compilar o firmware em si não exige Python.

## 7. Validação e limitações conhecidas

O código de teste é executado com um compilador C real; os testes de SPI/FIFO usam registradores e tempo simulados, não equivalendo a medição no chip.
Depois de instalar o MSVC, execute o `Tools/test.ps1` no **x64 Native Tools Command Prompt / Developer PowerShell** dele para repetir os testes de host.
Essa entrada exige que o `cl` já esteja no PATH; pode-se passar `-PythonExe` para indicar o seu próprio Python. Ao executar separadamente os dois `.cmd` de testes em C,
também se pode passar como primeiro argumento o caminho completo do seu próprio `vcvars64.bat`. Os scripts de teste não instalam compilador nem conectam hardware.
O registro de validação atual está em `VALIDATION.md`, e as versões das fontes de terceiros em `DEPENDENCIES.md`.

A confirmar em placa real: alimentação e montagem, erro de baud rate do HSI, modo/borda/temporização de seleção de chip do SPI, temporização de habilitação do TX, níveis elétricos do barramento,
taxa real de atualização do FIFO, convergência inicial do SFLP e direção das coordenadas, deriva térmica, recuperação do watchdog e operação prolongada na mesma linha.
A aprovação nos testes de software não substitui essas aceitações.

## 8. Fontes de referência

- [Hardware desta placa](https://github.com/fanhao375/microduck-replica/tree/a1f4979c425bd9228622be89f2bfa4d1de7fbde3/hardware/imu_to_dxl)
- [Datasheet do STM32G031](https://www.st.com/resource/en/datasheet/stm32g031f8.pdf)
- [Datasheet do LSM6DSV16X](https://www.st.com/resource/en/datasheet/lsm6dsv16x.pdf)
- [Driver de IMU da ST](https://github.com/STMicroelectronics/lsm6dsv16x-pid)
- Feetech, *Protocolo de comunicação SCS* e mapa de memória: [`docs/飞特资料/`](../../../docs/飞特资料/) do repositório
- [Protocolo de barramento desta placa (contrato de interface)](../总线协议.md)
- [Interface do controlador principal upstream examinada](https://github.com/pollen-robotics/microduck/blob/5984efb770855432b03dafd3d879e9929981e45b/duck-control/src/bus.rs)

O código novo escrito por este projeto é fornecido sob a licença MIT; os arquivos de terceiros mantêm seus próprios direitos autorais e licenças, veja cada diretório Drivers.
