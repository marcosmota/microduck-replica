# Tabela de ligação do imu_to_dxl

> 🌐 Tradução não oficial para pt-BR do [original](https://github.com/fanhao375/microduck-replica). Em caso de divergência, vale o original. Licença: CC BY-NC-SA 4.0 (texto alterado: traduzido do chinês/inglês).

> Para ligação manual. Os designadores dos componentes coincidem com o `microduck-replica/BOM.md`.
> Estado: rascunho da solução, sem verificação em hardware real.

## Lista de componentes (com códigos LCSC)

| Posição | Componente | Encapsulamento | Código LCSC |
|---|---|---|---|
| U1 | STM32G031F8P6 | TSSOP-20 | C529334 |
| U2 | LSM6DSV16XTR | LGA-14 | C5267406 |
| U3 | SN74LVC2G241DCUR | VSSOP-8 | C10430 |
| U4 | HT7533-1 (entrada de 30V, 100mA) | SOT-89 | C14289 |
| F1 | PPTC MF-NSMF020X-2 (24V / retenção de 200mA) | 1206 | C210358 |
| D1 | TVS SMF12A | SOD-123FL | C2943870 |
| R1, R2, R4, R5 | 10kΩ | **0603** | C25804 |
| R6 | 150Ω | **0603** | C22808 |
| D2 | Diodo zener BZT52C5V1 5,1V | **SOD-123** | C151588 |
| C1–C4, C8 | 100nF 50V | **0603** | C14663 |
| C5 | 4,7µF / 16V X5R | 0603 | C19666 |
| C6 | 10µF / 25V X5R | 0805 | C15850 |
| C7 | 10µF / 10V | 0603 | C19702 |
| J1, J2 | B3B-EH-A (3P **2,5mm**, cabo Dynamixel / HAT oficial) | Furo passante | C160259 |
| **J4, J5** | **B3B-PH-K-S (3P 2,0mm vertical, compatível com o `AMP2.0-3P` da Feetech)** | Furo passante | A definir |
| J3 | Barra de pinos 6P SWD+serial PZ254V-11-06P | Furo passante | C492405 |

## Netlist (23 redes)

### VDD_BUS —— tensão bruta do barramento dos servos (bateria de lítio 2S 6,0–8,4V)
J1.2 · J2.2 · **J4.2** · **J5.2** · F1.1

> O `VDD_BUS` dos quatro conectores segue por **cobre de passagem direta**: `J1.2 → J2.2 → J5.2 → J4.2`, com 0,76 mm em todo o percurso.

### VDD_F —— após o fusível
F1.2 · cátodo do D1 · C6.1 · C8.1 · U4.2 (Vin)

### +3V3
U4.3 (Vout) · C7.1 · U1.4 (VDD/VDDA) · C4.1 · C5.1 · R2.2 ·
U2.5 (Vdd_IO) · U2.8 (Vdd) · C1.1 · C2.1 · R1.2 ·
U3.8 (VCC) · C3.1 · J3.6 · **R7.2**

### GND
Ânodo do D1 · C6.2 · C8.2 · U4.1 (GND) · C7.2 ·
U1.5 (VSS/VSSA) · C4.2 · C5.2 ·
U2.2 (SDx) · U2.3 (SCx) · U2.6 (GND) · U2.7 (GND) · C1.2 · C2.2 ·
U3.4 (GND) · C3.2 · J1.1 · J2.1 · J3.1 · **J4.3** · **J5.3** · C9.2 · X1.2

### Reset
| Rede | Pontos de conexão |
|---|---|
| NRST | U1.6 (PF2-NRST) · R2.1 |

A outra extremidade do R2 (R2.2) vai a +3V3, formando um pull-up de 10k.

### Quatro resistores de pull-up (R1/R2/R4/R5, com a outra extremidade toda em +3V3)

| Posição | Puxa qual rede | Por quê |
|---|---|---|
| **R1** | `SPI_CS` | Na energização e no reset o PA4 fica em alta impedância, e o CS flutuando pode ser tomado pelo IMU como modo I²C ou ler ruído do barramento como comando. **Obrigatório** |
| **R2** | `NRST` | O chip já tem um pull-up interno fraco de ~40 kΩ; os 10 kΩ externos aumentam a imunidade a interferência. Reforço |
| **R4** | `DE` | O `1OE#` **habilita a transmissão em nível baixo**. Durante o reset o PA1 fica em alta impedância; se este pino flutuar para nível baixo, o buffer de transmissão abre e esta placa despeja dados no barramento, colidindo com os 15 servos. **Obrigatório** |
| **R5** | `DXL_DATA` | Quando o barramento está ocioso e ninguém aciona, a entrada CMOS `2A` flutuando tem corrente de passagem, e a UART também pode tomar ruído por bit de início. Esta placa define por conta própria o nível de repouso, sem depender do host |

> R4 e R5 foram acrescentados em 2026-09-06 conforme comentários da revisão da comunidade. Antes só havia pull-up no CS,
> e **o mesmo raciocínio não foi estendido ao DE, o que foi uma omissão**.

### IMU (SPI de quatro fios + duas interrupções)
| Rede | U1 (STM32) | U2 (LSM6DSV16X) | Outros |
|---|---|---|---|
| SPI_CS   | 11 · PA4 | 12 · CS       | R1.1 (R1.2 vai a +3V3, pull-up) |
| SPI_SCK  | 12 · PA5 | 13 · SCL      | |
| SPI_MISO | 13 · PA6 | 1 · SDO/SA0   | |
| SPI_MOSI | 14 · PA7 | 14 · SDA      | |
| IMU_INT1 | 7 · PA0  | 4 · INT1      | |
| IMU_INT2 | 15 · PB0/PA8 | 9 · INT2  | |

### Barramento half-duplex (U3 = buffer duplo SN74LVC2G241)
| Rede | Pontos de conexão |
|---|---|
| MCU_TX   | U1.9 (PA2 · USART2_TX) · U3.2 (1A) |
| MCU_RX   | U1.10 (PA3 · USART2_RX) · U3.3 (2Y) |
| DE       | U1.8 (PA1 · USART2_DE) · U3.1 (1OE#) · **R4.1** |
| DXL_DATA | U3.6 (1Y) · U3.5 (2A) · **R5.1** · **R6.1** —— lado do buffer |
| **DXL_BUS** | **R6.2** · **cátodo do D2** · J1.3 · J2.3 · **J4.1** · **J5.1** —— lado dos conectores |
| **RX_EN** | **U1.1 (PB7/PB8)** · **U3.7 (2OE)** · **R7.1** —— habilitação do buffer de recepção, acrescentada em 2026-09-07 |

O `U3.7 (2OE)` era originalmente ligado direto a +3V3 e, em 2026-09-07, passou a ser a **rede `RX_EN`**: com **pull-up de R7 10 kΩ a
+3V3**, e ao mesmo tempo um fio levado ao **U1.1 (PB7/PB8)**.

**O comportamento padrão não muda** —— durante o reset do MCU o PB7 fica em alta impedância, o R7 puxa o `2OE` para alto e o buffer de recepção continua aberto,
e, sem escrever uma linha de firmware, fica idêntico ao que era antes da alteração. É uma **reserva**: para desligar o eco durante a transmissão, basta configurar o PB7 como
saída push-pull em nível baixo e ativar o pull-up interno no pino `MCU_RX`, evitando que o RX fique flutuando com o `2Y` em alta impedância.

### Porta de depuração J3 (6P, `PZ254V-11-06P` / `C492405`)

| Pino | Rede | Origem |
|---|---|---|
| 1 | GND | |
| 2 | SWDCLK | U1.19 (PA14) |
| 3 | SWDIO | U1.18 (PA13) |
| 4 | **UART_RX** | **U1.17** `PA12[PA10]` → remapeado para **PA10** = `USART1_RX`; liga ao TX do USB-TTL (nível de 3,3V) |
| 5 | **UART_TX** | **U1.16** `PA11[PA9]` → remapeado para **PA9** = `USART1_TX`; liga ao RX do USB-TTL (nível de 3,3V) |
| 6 | +3V3 | VTref do depurador, referência de tensão; não ligar a 5V |

**Errata de 2026-09-21:** conferindo com o [PDF do esquemático](imu_to_dxl-原理图.pdf) de 2026-09-08 deste diretório,
o SWD usa os pinos **1, 2, 3 e 6**; os pinos 4/5 são a serial. O cabo flat antigo de quatro pinos não pode ser encaixado inteiro nos pinos 1–4,
e o pino 4 não pode ser tratado como 3,3V nem como VTref. Antes de ligar, confira a marca do pino 1 na placa real e a definição da interface do depurador.

O remapeamento dos pinos 16/17 depende do `SYSCFG_CFGR1` (DS12992 Rev 3, Table 12, nota de rodapé 4).

**Por que sem USB**: o STM32G031 **não tem periférico USB** (zero ocorrências de USB no manual inteiro; Development support
só menciona "serial wire debug (SWD)"). Acrescentar USB exigiria mais um CH340/CP2102, o que não compensa em uma placa de 45×22 mm.
Para depurar, o SWD basta:

| Método | O que é necessário | Descrição |
|---|---|---|
| **RTT** (recomendado) | Pinos 1, 2, 3 e 6 do J3 | O `probe-rs` suporta executar RTT sobre CMSIS-DAP, lendo diretamente a RAM do alvo, **sem parar a CPU**, sem afetar a malha de controle de 50 Hz |
| printf pela serial | Pinos 4/5 do J3 + qualquer USB-TTL | Usa o USART1 |
| Semihosting | Apenas o J3 | Cada printf para a CPU, a malha de controle desmorona, **não use** |
| ~~SWO / ITM~~ | — | **O Cortex-M0+ não tem**; só do M3 em diante |

### Interface dos servos —— dois conjuntos coexistem, ⚠️ **a ordem dos pinos é oposta**

A placa tem quatro conectores de barramento de servos, em dois pares, com as três linhas ligadas uma a uma em paralelo (o mesmo nó elétrico),
**mas a ordem dos pinos dos dois grupos é totalmente oposta** —— porque a ordem dos fios dos dois fabricantes é de fato diferente.

**`J1` / `J2` —— `B3B-EH-A`, 2,5 mm, para chicote Dynamixel / HAT oficial**

| Pino | Rede |
|---|---|
| 1 | GND |
| 2 | VDD_BUS |
| 3 | **DXL_BUS** |

**`J4` / `J5` —— `B3B-PH-K-S`, 2,0 mm, para chicote Feetech HD-1910**

| Pino | Rede |
|---|---|
| 1 | **DXL_BUS** |
| 2 | VDD_BUS |
| 3 | **GND** |

> ⚠️ **Jamais copie as redes do `J1` para o `J4`.** A Feetech, em *Especificação em série HD-1910-C001* A/0,
> página 3/7, itens 6-7, informa que a ponta do cabo é `1=Signal / 2=Vcc / 3=GND`, de ponta-cabeça em relação ao lado Dynamixel.
> Felizmente **2,0 mm e 2,5 mm não se encaixam um no outro**, o que impede fisicamente a ligação errada, restando apenas a possibilidade de "desenhar errado".
> A ordem dos pinos dos quatro conectores foi conferida em 2026-09-08 relendo a netlist do PCB, e todos estão corretos.
>
> Observe também que **`AMP2.0` é 2,0 mm e `JST EH` é 2,5 mm**, e o cabo da Feetech não entra em `J1`/`J2`,
> ⚠️ Teste real da primeira placa (2026-09-17): o **corpo** do `B3B-PH-K-S` não combina com o plugue da Feetech e não entra; lixando o corpo e deixando só os pinos, funciona. A v2 troca para um conector que combine de fato com o cabo da Feetech.
> E vice-versa —— use o grupo que corresponde ao servo que estiver usando.

> 📌 **O pino 3 era registrado por engano nesta tabela como `DXL_DATA`**. Depois de acrescentado, em 2026-09-06, o resistor em série `R6` de 150 Ω,
> o lado do buffer (`DXL_DATA`) e o lado dos conectores (`DXL_BUS`) passaram a ser duas redes, e no conector é `DXL_BUS`.
> Corrigido em 2026-09-08.

## Pinos flutuantes

- U1: **3 (PC15-OSC32_OUT), 20 (PB3/PB4/PB5/PB6)**
  - O pino 3 já está levado a um pad, e junto com o pino 2 forma a posição reservada para o oscilador ativo (não montado por padrão), veja "Relógio" abaixo
  - O pino 20 é o único GPIO totalmente livre (agrega quatro portas, PB3/PB4/PB5/PB6, o mais flexível), reservado para uso futuro
  - O pino 1 (PB7/PB8) estava originalmente livre e, desde 2026-09-07, é usado como `RX_EN`
  - O pino 2 (PC14-OSC32_IN) liga ao `OSC_IN`, e os pinos 16/17 (PA9/PA10) ligam a `UART_TX`/`UART_RX`
    —— antes esta tabela os listava por engano como flutuantes, corrigido em 2026-09-07
- U2: 10 (OCS_Aux), 11 (SDO_Aux) —— SPI auxiliar não usado; o DS13510 da ST permite deixar flutuando; a prática conservadora é ligar o OCS_Aux ao Vdd_IO

**U2.2 (SDx) e U2.3 (SCx) devem ser aterrados**, não podem ficar flutuando —— ST DS13510 Rev 1, Table 1.

## Relógio: usar o HSI16 interno, posição do cristal reservada e não montada

**Este encapsulamento não permite ligar um cristal passivo.** DS12992 Rev 3, Table 12 (p.36): o pino 2 do TSSOP20
(`PC14-OSC32_IN`) tem entre as funções adicionais o `OSC_IN`, mas o pino 3 (`PC15-OSC32_OUT`) só tem `OSC32_OUT`,
**sem `OSC_OUT`**; `PF0`/`PF1` nem sequer são expostos neste encapsulamento. Um cristal passivo precisa de dois pinos para oscilar.

| Opção | Viabilidade |
|---|---|
| Cristal passivo de alta velocidade | ❌ Falta o `OSC_OUT` |
| Oscilador ativo em bypass HSE (entra pelo pino 2, o pino 3 pode servir de `OSC_EN`) | ✅ |
| Cristal passivo LSE de 32,768 kHz | ✅ Mas não pode fornecer relógio ao PLL, só calibrar o HSI16 |

Precisão do HSI16 (Table 41, p.64): de fábrica −0,75%/+0,5%, deriva térmica de 0–85 °C de ±1%, deriva de tensão de ±0,1%
→ pior caso de cerca de **−1,85% / +1,55%**. A tolerância típica da UART a 1 Mbps é de ±3%, basta, mas com pouca folga.

**Abordagem**: levar os pinos 2 e 3 a pads, deixando uma posição para oscilador ativo 3225 + uma posição de desacoplamento. Inicialmente não montado por padrão; desde 2026-09-08, X1 / R3 / C9 passaram a ser montados por padrão;
o firmware usa primeiro o HSI16, e só se habilita o bypass HSE se a medição real a 1 Mbps mostrar erros de bit.
Atenção: o pino de habilitação `Tri-state` do X1 no desenho está **fixo em 3,3 V**, e o "pino 3 (PC15) pode servir de `OSC_EN`" mencionado acima não foi desenhado, com o PC15 vazio. Se a primeira placa precisar que o MCU controle o oscilador, ligue um fio PC15 → X1.1; veja o README "Registro de revisão · 2026-09-16".

## Nota sobre a polaridade de habilitação

O `1OE#` habilita o buffer de transmissão em nível baixo, e o `2OE` em nível alto mantém o buffer de recepção sempre aberto.

- O PA1 é o pino DE em hardware do USART2; configura-se `CR3.DEP = 1` (DE ativo em baixo), e a comutação transmissão/recepção é feita pelo hardware
- Recepção sempre aberta significa que, ao transmitir, o RX recebe o próprio eco, e o firmware basta descartá-lo conforme o comprimento transmitido
- A alternativa é ligar também o `2OE` ao DE (desligando o buffer de recepção durante a transmissão), mas então o pino RX do MCU fica flutuando durante a transmissão e, ao reabilitar, pode surgir um falso bit de início; não recomendada
- **Melhoria de 2026-09-07**: o `2OE` deixou de ser ligado rigidamente a +3V3, passando por um pull-up de R7 10 kΩ e sendo levado também ao `U1.1 (PB7/PB8)`.
  O padrão continua sendo sempre aberto (comportamento inalterado), mas o firmware ganha uma opção —— para fechar a recepção, puxa-se o PB7 para baixo e ativa-se o
  pull-up interno do pino RX do MCU para resolver a flutuação. É mais flexível do que ligar ao DE: se e quando fechar fica a critério do próprio firmware

## Parâmetros reais do HD-1910-C001 (página de produto oficial da Feetech, período de pré-venda)

Um modelo que a Feetech fez especialmente para o pato; o título da página é "**Servo do Pato de código aberto** —— servo de barramento serial, desenvolvido sob medida para o boneco de IA Pato".
O corpo do servo traz a indicação: **"interface de três pinos · barramento serial TTL"**.

| Item | Valor |
|---|---|
| Faixa de tensão | **5 V – 8,4 V** |
| Torque de travamento | **10 kg·cm = 0,98 N·m** |
| Carga nominal | **3,0 kg·cm = 0,294 N·m** |
| Velocidade sem carga | 0,09 s/60° (110 RPM) |
| **Corrente de repouso** | **21 mA** |
| **Corrente sem carga** | **140 mA** |
| **Corrente nominal** | **500 mA** |
| **Corrente de travamento** | **1,8 A** |
| Dimensões | 34 × 20 × 23 mm |
| Peso | 22,5 ± 2 g |
| Carcaça | PA + Fiber |

As dimensões estão na mesma faixa do XL330 (20×34×26 mm), mecanicamente quase intercambiáveis; o torque é quase o dobro do XL330-M288 (0,52 N·m).

> ⚠️ Não aplique os parâmetros elétricos da especificação do `HL-2915-C002` a este modelo —— aquele é um modelo de 12 V (9–14 V).
> O que se aproveita é só a definição do conector e a parte do protocolo.

### ⚠️ Sobrecorrente no caminho de passagem direta J1↔J2

As três linhas de J1 / J2 estão ligadas uma a uma em paralelo, **e se esta placa ficar no meio da cadeia, a corrente de todos os servos a jusante passa pelo conector e pelo cobre desta placa**:

| Condição | Individual | ×15 |
|---|---|---|
| Repouso | 21 mA | 0,3 A |
| Giro sem carga | 140 mA | 2,1 A |
| **Carga nominal** | **500 mA** | **7,5 A** |
| Travamento | 1,8 A | 27 A |

Já a **corrente nominal do JST EH é de apenas 3 A**.

> O F1 (PPTC de 200 mA) **não é afetado** —— ele só está em série no ramo em que a própria placa retira energia
> (`VDD_BUS → F1 → VDD_F → U4`), e não no caminho de passagem direta J1→J2.

**Escolha uma das duas:**

1. **Deixar esta placa na ponta da cadeia** (nó folha), usando só o J1, com o J2 vazio ou ligado a apenas 1 servo —— passa pela placa só cerca de 12 mA. **Recomendado**
2. Se for obrigatório ficar no meio da cadeia, no PCB passar `VDD_BUS` e `GND` de J1 a J2 por um **cobre largo** (≥ 2 mm ou diretamente em plano de cobre),
   e verificar que o número de servos a jusante não passe de 3 A

## Definição da interface Feetech (confirmada pela especificação oficial)

**Origem**: Feetech, *Especificação em série HL-2915-C002* A/0, 2026-02-27, página 4, itens 6-7.

```
6-7  连接器和电缆 Connector and Cable
     类型 Type        AMP-3
     材质 Material    PP
     线长 Length      15CM / 25CM
     引脚定义         1  GND
     Pin Definition   2  Vcc
                      3  Signal/TTL
```

**`1=GND / 2=Vcc / 3=Signal`, exatamente igual ao Dynamixel → a ligação atual de J1/J2 está correta.**

Evidência relacionada: no conjunto `HD-1910-C001.stp`, o nome da peça do conector é **`AMP-3P`**, e esse conjunto inclui
`HL-2915-C002-20260409_ASM` —— os dois modelos compartilham a plataforma, e a definição da interface é comum.

### Outros itens aproveitáveis da mesma especificação

| Item | Valor | Significado |
|---|---|---|
| 7-2 Tipo de protocolo | Half Duplex Asynchronous, **8bit / 1stop / No Parity** | Consistente com esta placa |
| 7-4 Baud rate | 38400 ~ 1 Mbps, **padrão de fábrica 1 Mbps** | Mesma velocidade do Dynamixel |
| 7-3 Faixa de ID | 0–253, **ID padrão de fábrica 1** | Os 15 servos precisam ter o ID alterado um a um; esta placa usa 200, sem conflito |
| 12 **Nível alto do sinal** | **2 V ~ 5 V** | Veja abaixo |
| 12 Nível baixo do sinal | 0,0 V ~ 0,45 V | |
| 6-6 Sensor de ângulo | Encoder magnético de 12 bits | Encoder magnético ⇒ **ordem de bytes little-endian** |

**Os níveis de sinal precisam ser vistos nos dois sentidos**:

1. Esta placa acionando o barramento —— o `SN74LVC2G241`, com alimentação de 3,3 V, emite cerca de 3,3 V, dentro da janela de nível alto de 2–5 V exigida pelo servo ✅
2. O servo acionando o barramento —— ele pode emitir até **5 V**; com alimentação de 3,3 V, a série 74LVC **tolera 5,5 V na entrada** ✅
   (escolher LVC em vez de LV/HC é justamente o que satisfaz este ponto)

> ⚠️ Esta especificação é do **HL-2915-C002 (12 V, 9–14 V)**, não do HD-1910 (5–8,4 V).
> **Os parâmetros elétricos não podem ser misturados** —— faixa de alimentação, corrente de travamento e limiar de proteção contra sobretensão são diferentes.
> O que é comum é só a parte da **definição do conector e do protocolo**.

### Contexto: a camada física varia entre as séries da Feetech

Texto original dos manuais de mapa de memória de cada série oficial (<http://doc.feetech.cn>, interface `getQr/{q}`):

| Série | Nível | Baud rate padrão |
|---|---|---|
| **STS** (encoder magnético) | **Barramento único TTL** | **1M** |
| SCS (potenciômetro) | Barramento único TTL | 1M / 500k |
| SMS (encoder magnético) | **RS485** | 115200 |
| HLS (encoder magnético) | Não indicado | 1M |
| SHC | CANopen | 125k |

"Feetech = TTL de fio único" **só vale para o ramo STS/SCS/HL**; a SMS é RS485. Antes de trocar de série, consulte o manual dela.

## Reconhecimento automático do protocolo (sem jumper, sem DIP switch)

Os cabeçalhos dos pacotes dos dois fabricantes são naturalmente distinguíveis, e o ponto de discriminação são os **bytes 3 e 4**:

| | Cabeçalho | Em seguida | Checksum |
|---|---|---|---|
| Dynamixel V2 | `FF FF` | **`FD 00`** | CRC-16 (poly 0x8005) |
| Feetech STS/SCS | `FF FF` | `ID` + `LENGTH` | `~(ID+LENGTH+INST+parâmetros)` |

Sem ambiguidade: o byte 3 do pacote Feetech é o ID (que pode ser 0xFD), mas **o byte 4, LENGTH, nunca é 0**
(o mínimo é 2 = instrução + checksum). Portanto `FF FF FD 00` só pode ser V2.

```
状态：未知 / 已锁 V1 / 已锁 V2

未知时，收到 FF FF 开头的包：
    if 字节[2]==0xFD && 字节[3]==0x00:
        按 V2 拆，算 CRC-16 → 过了记一票 V2
    else:
        按 V1 拆，算 ~sum   → 过了记一票 V1
    连续 3 票一致 → 锁定

锁定后：只按锁定协议解析，回复也用同一种
兜底：连续 100 包全解析失败 → 解锁重新识别
```

O que segue toma como base o **texto original oficial da Feetech, *Protocolo de comunicação dos servos SCS***:

- Formato do quadro: **1 bit de início + 8 bits de dados + sem paridade + 1 bit de parada**, 10 bits no total
- Cabeçalho: dois `0xFF` consecutivos
- Faixa de ID **0～253** (`0x00`～`0xFD`), **ID de broadcast = 254 (`0xFE`)**
- Comprimento dos dados **Length = número de parâmetros N + 2**, portanto **o mínimo é 2 e nunca é 0** ← a base de que a discriminação é inequívoca
- Checksum = `~(ID + Length + Instruction + Param1 + … + ParamN)`, tomando o byte baixo se passar de 255
- Leitura síncrona `0x82`, escrita síncrona `0x83`, PING `0x01`, leitura `0x02`, escrita `0x03`
- ⚠️ **A ordem de bytes dos parâmetros multibyte depende do tipo de servo**: "servos do tipo potenciômetro são **big-endian** (byte alto primeiro), servos do tipo encoder magnético são **little-endian** (byte baixo primeiro)"

Três pontos essenciais:

1. **É preciso travar, não julgar pacote a pacote** —— o barramento tem ruído, e se um pacote for corrompido e ficar parecido com `FD 00` haverá oscilação de um lado para o outro
2. **A resposta usa o mesmo protocolo** —— não é só a análise que precisa de ramificação, a montagem do pacote também
3. **Destravar como último recurso** —— ao trocar de marca de servo não é preciso regravar o firmware

O ID do escravo é **200** nos dois lados, sem conflito com os 15 servos (normalmente numerados de 1 a 15).

> O `Return Delay Time` do STS da Feetech é 0 por padrão, e o manual indica "Not functional on STS" ——
> o registrador não tem efeito, e o atraso da resposta é decidido inteiramente pelo firmware.

## Critério de encapsulamento: resistores e capacitores sempre em 0603

Todos os resistores e capacitores usam **0603**, e o D2 usa **SOD-123**, sem 0402/SOD-323.

O motivo não é a soldagem manual —— a placa tem o LSM6DSV16X em LGA-14, que de qualquer modo exige ar quente.
**O motivo real é o retrabalho na fase de depuração**: testar o resistor em série de 150 Ω a 33 Ω, retirar algum pull-up, acrescentar um desacoplamento;
em 0603, com o ferro de solda, alterar leva segundos. Uma primeira versão de placa não validada inevitavelmente será alterada várias vezes.

Custo: 11 componentes ocupam cerca de 16 mm² a mais, menos de 2% da área da placa. E estes valores em 0603 estão todos na **biblioteca básica** do LCSC
(10 kΩ `C25804` · 150 Ω `C22808` · 33 Ω `C23140` · 100 nF `C14663`), economizando até a taxa de carga de peças em relação à biblioteca estendida do 0402.

> O C6 (10 µF/25 V) permanece em 0805; o C5 (4,7 µF) e o C7 (10 µF) já eram 0603.

## Proteção da linha de dados (seguindo o que o HAT faz)

```
U3.6 (1Y) ──┬─ DXL_DATA ─┬── R5 10k → 3.3V
U3.5 (2A) ──┘            │
                         └── R6 150Ω ──┬── DXL_BUS ── J1.3 / J2.3
                                       │
                                    D2 5V1
                                       ⏚
```

| Componente | Função |
|---|---|
| **R6 150 Ω** | Resistor em série na fonte. Limita a corrente de curto-circuito e suaviza as bordas para reduzir EMI. A 1 Mbps o atraso ocupa cerca de 3% do tempo de bit |
| **D2 5V1** | Grampo. Cátodo em `DXL_BUS`, ânodo no terra |

**O pull-up fica no lado do buffer** (R5 antes do R6): assim, mesmo sem o barramento conectado, a entrada CMOS `U3.5 (2A)`
tem um nível definido e não flutua. Na mesma posição do `R32` do HAT.

> O HAT oficial usa `R33 150R + TH1 100R` em série, totalizando 250 Ω, além do grampo `D4 5V1`
> (`dynamixel.kicad_sch`). **O D2 é do mesmo modelo que o D4 do HAT, com encapsulamento diferente** —— o do HAT é SOD-323 (`C151348`), e esta placa usa SOD-123 (`C151588`) para facilitar o retrabalho.
> Esta placa usa só 150 Ω, porque o lado do HAT já tem 250 Ω, e colocar demais no barramento retardaria as bordas.

## Três linhas de defesa contra sobretensão

1. **F1** PPTC de 200mA de retenção / 24V —— limita a corrente, protege contra curto-circuito
2. **D1** TVS SMF12A —— grampeia, protege contra picos de força contraeletromotriz dos servos
3. **U4** HT7533-1 com tolerância de 30V —— mesmo que as duas primeiras falhem, o próprio LDO aguenta

## Pontos do layout

- Placa de 2 camadas, 45 × 22mm, componentes SMD todos na face frontal
- 3 conectores (J1/J2/J3) dispostos na mesma borda, de furo passante
- Dois furos de fixação M2, com espaçamento de 34mm (compatível com o `banana_pcb_locker` original)
- C1/C2 rente aos pinos Vdd / Vdd_IO do U2, com trilhas o mais curtas possível
- C4 (100nF) e C5 (4,7µF) rente ao pino 4 do U1 —— a ST DS12992 Fig.13 exige "o mais próximo possível do pino, ou no verso do PCB bem em frente a esse pino"
- A trilha DXL_DATA é engrossada e mantida afastada do SPI
