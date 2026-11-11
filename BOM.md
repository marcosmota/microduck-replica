# Lista de materiais do robô completo (BOM)

> 🌐 Tradução não oficial para pt-BR do [original](https://github.com/fanhao375/microduck-replica). Em caso de divergência, vale o original. Licença: CC BY-NC-SA 4.0 (texto alterado: traduzido do chinês/inglês).

> **Peças impressas, materiais e quantidades de montagem valem conforme a versão correspondente no [repositório CAD](https://github.com/fanhao375/microduck-replica-cad#装配-bom).** As contagens de referências do MJCF e as estimativas geométricas deste documento são análises do upstream, não a lista de impressão Feetech mais recente.

> **O repositório tem quatro listas e uma nota de impressão, cada uma com seu papel:**
>
> | Documento | O que cobre | O que não cobre |
> |---|---|---|
> | **BOM** (este documento) | **Tabela geral**: todos os materiais, quantidades (contadas a partir das referências geom do MJCF), ordem de grandeza de custo, lacunas ainda não resolvidas | Onde comprar cada item, como imprimir |
> | [Lista de compras da eletrônica](docs/电控采购清单.md) | **Onde comprar** os itens de eletrônica: placa principal / câmera / ToF / alimentação / duas PCBs / cabos, com links reais do Taobao + armadilhas a evitar | Base das quantidades, peças mecânicas |
> | [Lista de compras da mecânica](docs/机械采购清单.md) | **Onde comprar** as peças mecânicas: rolamentos / fixadores / consumíveis, com links reais do Taobao | Itens de eletrônica |
> | [Consulta rápida de especificações de hardware](docs/硬件规格速查.md) | **Parâmetros**: interfaces, tensões, protocolos, valores que o software precisa respeitar | Preços e lojas |
> | [print/README](print/README.md) | **Ponto de entrada da impressão**: escolher no repositório CAD a versão, o material e as quantidades | Compras |
>
> Os números de peça e a correspondência de parafusos para a montagem estão no [repositório CAD](https://github.com/fanhao375/microduck-replica-cad#装配-bom).


**Português (BR)** · [English](BOM.en.md)

Tudo o que é preciso para replicar um Microduck. As quantidades vêm da contagem de referências geom do `robot_walk.xml` do upstream
(**38 tipos de malha / 75 instâncias**); não são estimadas.

> ⚠️ **A Pollen nunca publicou uma BOM.** Esta lista foi reconstruída a partir do MJCF, dos STL, do código-fonte Rust e do projeto KiCad públicos.
> Cada item indica a sua base. Os preços variam muito conforme a região e a época; **use-os apenas como ordem de grandeza**.
>
> Data de conferência: **2026-09-04**. ⚠️ **Baseline do upstream a conferir**: os antigos STL de `print/` deste repositório (já removidos) eram idênticos byte a byte à
> cópia de trabalho local de `microduck_rl`, que é um clone raso (depth-1) com HEAD = **`d424a0c` (2026-08-27)**.
> A baseline registrada anteriormente, `29e887e` / reexportação `8dfc08f`, não existe no banco de objetos local; **portanto a afirmação "a contagem de geom é a mesma antes e depois da
> reexportação" não pode ser verificada**. Todas as contagens abaixo vêm de `d424a0c`.

---

## 1. Custo em resumo

| Categoria | Qtd. | Ordem de grandeza |
|---|---|---|
| **Servos** | 15 | **$359 – €629** (ver abaixo; a diferença entre regiões é enorme) |
| Placa principal e sensores | 4 tipos | cerca de $80 – 120 |
| Bateria e alimentação | 2 itens | cerca de $30 – 50 |
| Rolamentos | 14 | cerca de $15 – 30 |
| Fixadores | cerca de 325 peças | cerca de $15 – 25 |
| **Fabricação de PCB (2 placas)** | 2 | cerca de $60 – 150 (com SMT) |
| Filamento de impressão | — | cerca de $15 – 30 |

**Os servos respondem pela maior parte do custo**, e a diferença de preço entre canais é grande: no site internacional da ROBOTIS, $23,90 × 15 ≈ **$359** (abaixo do preço de venda do robô completo),
no site dos EUA, $27,49 × 15 ≈ **$412** (aproximadamente igual ao preço de venda) e, na Europa com impostos, **€603–629** (muito acima).

> $399 é o preço de produção em escala da Seeed + cadeia de suprimentos oficial; **uma réplica individual não consegue chegar a esse valor**.

---

## 2. Servos (o núcleo do custo)

| Modelo | Qtd. | Base |
|---|---|---|
| **Dynamixel XL330-M288-T** | **15** | a malha `xl330` é referenciada 15 vezes em `robot_walk.xml` |

- 14 entram no espaço de ações da política; **o 15º é a boca** (`JOINT_NAMES[9] = "mouth"`, `MOUTH_INDEX = 9`),
  e não entra no espaço de ações da política (todas as políticas alpha têm observação de 61 dimensões / ação de 14 dimensões).
  ⚠️ **Essa junta nem existe no MJCF** — `robot_walk.xml` tem apenas 14 juntas hinge / 14 atuadores;
  em todos os MJCF a boca é fixada rigidamente ao corpo `jaw_soft`; a default class `passive_*` tem zero referências no modelo de marcha
- Atribuição de IDs: perna esquerda 20–24 ／ pescoço-cabeça-boca 30–34 ／ perna direita 10–14
- ⚠️ **O modelo é uma inferência**: o código-fonte só tem `motor_name="xl330"`, sem sufixo. A base para M288-T está em
  [seleção de atuadores](docs/执行器选型.md#como-o-modelo-foi-determinado)

**Referência de preços** (2026-09-04):

| Canal | Preço unitário | 15 unidades |
|---|---|---|
| Site oficial da Robotis nos EUA | **$27,49** | **$412** |
| Generation Robots (Europa, sem impostos) | €33,50 | €503 |
| Generation Robots (Europa, com impostos) | €40,20 | €603 |
| MyBotShop | €41,95 | €629 |

> ⚠️ **Aviso de operação em sobretensão**: o XL330 tem tensão nominal de 3,7–6,0 V, e o Microduck o alimenta com 6,6–8,2 V.
> No esquemático da HAT oficial, o conector Dynamixel liga diretamente em `+BATT` (tensão bruta da bateria); o único buck de 5 V da placa é para o Raspberry Pi.
> Essa é uma escolha oficial, não um erro de digitação. Detalhes em [a verdade sobre a tensão](docs/执行器选型.md#-a-verdade-sobre-a-tensão-o-xl330-opera-em-sobretensão).

> 🛒 **Links reais do Taobao para compras na China**: [lista de compras da eletrônica](docs/电控采购清单.md) (placa principal/câmera/ToF/alimentação/PCB/cabos),
> [lista de compras da mecânica](docs/机械采购清单.md) (rolamentos/fixadores/porcas de inserção térmica/trava-rosca/consumíveis).

---

## 3. Componentes eletrônicos

| Peça | Modelo | Qtd. | Base / observações |
|---|---|---|---|
| Placa principal | **Radxa Zero 3W** | 1 | device tree `compatible = "radxa,zero-3w"`. Existem várias combinações de RAM/eMMC. Os materiais oficiais informam **1 GB de memória, 32 GB de armazenamento** (sem dizer se é eMMC); **para a réplica, basta uma placa nua de 1G ou 2G sem eMMC + cartão microSD de 32 GB**, comprando só por canais oficiais, veja [lista de compras da eletrônica](docs/电控采购清单.md#qual-configuração-comprar) (corrigido em 2026-10-12; a recomendação anterior de 2G/16G eMMC era desnecessária); a imagem do sistema precisa ter o kernel do fabricante Rockchip (família Armbian), senão a NPU não existe |
| Câmera | Raspberry Pi Camera v2 (**IMX219**) | 1 | `setup-board.sh` usa o overlay `radxa-zero3-rpi-camera-v2`. ⚠️ **Montada torta, um quarto de volta; exige compensação por software**: no código atual `mediad/src/main.rs:82`, o `--rotate` padrão é **90**, e o comentário original diz «the head camera is mounted **a quarter turn off**, and this is the one place that fact is written down». Os 180° de `media-bringup.md:472` são dados da **máquina alpha**. **Vale o código = 90°** |
| Profundidade | Módulo **VL53L5CX** (ou L8CX) | 1 | O firmware só reconhece esses dois chips (revision `0x02` / `0x0C`); **o L7CX é rejeitado diretamente**; recomenda-se o L5CX: a placa tem 15,8 × 10 mm, já o formato comum do L8CX, de 33,4 mm, não cabe na carcaça do rosto ([medição real](踩坑记录.md#compras)); endereço `0x29` ou `0x52`; conector Stemma/Qwiic |
| Bateria | **Sony NP-F550** (2S, 7,4 V) | 1 | ⚠️ ver errata abaixo |
| Suporte da bateria | Suporte genérico da série NP-F | 1 | O upstream só tem a peça impressa `power_support` e **nenhum modelo de contatos** — a solução de alimentação precisa ser resolvida por conta própria |
| Alto-falante | Alto-falante pequeno | 1 | malha `speaker` ×1; a HAT tem amplificador PAM8406 e terminais Wago |

### ⚠️ Errata do modelo da bateria: é NP-F550, não F970

O nome do arquivo de malha no upstream é `np_f970`, **mas isso é enganoso**:

| | |
|---|---|
| Caixa delimitadora medida da malha | **70,8 × 38,6 × 20,6 mm** ← são as dimensões da **NP-F550/F570** |
| Uma NP-F970 de verdade | cerca de **60 mm** de espessura, cerca de 300 g |
| Modelos que aparecem no código-fonte | **apenas NP-F550** (`model.rs`, `robotd-design.md`); F970 não aparece em lugar nenhum do repositório |

**Uma F970 não cabe e, além disso, os 300 g consumiriam mais de um terço do orçamento de 800 g do robô completo.**
Este repositório escrevia «NP-F970» em vários lugares; foi tudo corrigido de forma unificada.

---

## 4. Placas de circuito (2 placas, ambas a mandar fabricar)

### Placa 1: RPI Robot HAT — já aberta pelo fabricante, basta baixar e mandar fabricar

| | |
|---|---|
| Origem | [`pollen-robotics/elec_RPI_Robot_HAT`](https://github.com/pollen-robotics/elec_RPI_Robot_HAT) (Apache-2.0) |
| Arquivos de produção | `production/`: Gerber, BOM, coordenadas de pick-and-place, PDF do esquemático, STEP |
| **Camadas** | **4 camadas** (`F.Cu / In1.Cu / In2.Cu / B.Cu`) |
| **Espessura da placa** | **1,0 mm** (KiCad `(thickness 1)`. Os 0,84 mm medidos no STL são uma aproximação da malha de simulação; para o pedido, vale o KiCad) |
| Dimensões | **65,0 × 30,9 mm** (medido no `Edge.Cuts` do KiCad, cantos arredondados R3,5). 0,9 mm mais larga que uma Pi Zero |

> ⚠️ **Não use o STL, nem o STEP convertido dele, para encaixes mecânicos.** A malha de simulação mede **65,025 × 30,025 × 0,840 mm**,
> uma diferença para o contorno real da placa de **0,875 mm na largura e 0,16 mm na espessura** — essa diferença na largura é de 3% num lado de 30 mm, e
> encaixes, colunas e aberturas não vão bater. **Para encaixes mecânicos, sempre use 65,0 × 30,9 × 1,0; o contorno da placa é definido pela camada `Edge_Cuts` do Gerber.**
> O STL/STEP só serve para verificação de interferência 3D e posicionamento aproximado.
>
> (Para comparação: a malha `pcb__raspberry_pi_zero_2_w` mede 65,000 × 30,000 × 1,600 mm, que é o tamanho padrão da Pi Zero, e portanto está correta — mas é só um bloco de marcação de posição; o robô real usa a Radxa Zero 3W.)
| BOM | 47 linhas / **123 componentes**, dos quais **5 linhas são DNP (não montadas)** |
| Posições de montagem | `POS.csv` com **117 linhas**, incluindo 3 pontos de referência FID1–3 e H3 → **113 componentes reais** |
| Viável soldar à mão? | ❌ **Impossível** — tem codec VQFN-32 e IMU LGA-16, é obrigatório usar serviço de SMT |

**Componentes-chave** (para estimar preço e substituir):

| Ref. | Componente | Função |
|---|---|---|
| U2 | TLV320AIC3104IRHBR | Codec de áudio (I²C `0x18`) |
| U1 | PAM8406D | Amplificador classe D |
| MK1 | Microfone MEMS (LCSC **C7587901**) | Integrado à placa. ⚠️ O Value na BOM oficial é só `Microphone_MEMS`, e ao lado do esquemático só há «Onboard Mic.», **sem indicar o modelo concreto**; o LMA2718 que este repositório escrevia antes não tinha fonte e foi retirado |
| U11 | BMI088 | IMU — **montado, mas não usado pelo software** (o chamado «segundo IMU») |
| U8 | SIT3088E | Transceptor RS-485 |
| U10 | LM5050-1 | Diodo ideal OR-ing ＋ detecção de desligamento. ⚠️ Fica no trilho `+5V` depois do buck, não é proteção contra polaridade invertida na entrada da bateria |
| U9 | AP63205 | Regulador buck |
| U4 | CAT24C32 | EEPROM — **DNP, não montada**, por isso esta não é uma HAT autoidentificável |
| J13/J14 | JST EH 3P | Dynamixel **TTL** |
| J3/J11 | JST EH 4P | Dynamixel **RS-485** |
| J5–J8 | JST SH 1mm 4P | Qwiic / Stemma (liga o ToF) |

⚠️ **Leitura obrigatória antes de fabricar**:

1. **Esta placa não tem circuito de carga nem entrada USB-C.** O `pwr_supply_charge.kicad_sch` no repositório
   é uma **folha órfã** (o `main.kicad_sch` não a instancia, e o bloco de título ainda traz o nome de outro projeto). A bateria precisa ser carregada com um carregador externo.
2. **Para abrir o projeto KiCad é preciso instalar antes a [`lib_KiCAD`](https://github.com/pollen-robotics/lib_KiCAD)**,
   senão tudo aparece como símbolos não resolvidos. Se for só fabricar, não é necessário; use diretamente o Gerber.

### Placa 2: `imu_to_dxl` — não existe projeto público em lugar nenhum, é preciso desenhar a sua

| | |
|---|---|
| Estado | 🔧 **sendo reconstruída neste repositório**, veja [engenharia reversa da solução de hardware §3](docs/硬件方案逆向.md) |
| Função | Faz o IMU se passar por um escravo Dynamixel (ID **200**) no barramento dos servos |

**BOM de referência** (projeto deste repositório, não oficial):

| Ref. | Componente | Código LCSC | Descrição |
|---|---|---|---|
| U1 | **STM32G031F8P6** | `C529334` | MCU, TSSOP-20. ⚠️ **Esta é uma sugestão de escolha deste repositório, não algo obtido por engenharia reversa** — o modelo do MCU da placa oficial não tem como ser reconstituído (veja [engenharia reversa da solução de hardware](docs/硬件方案逆向.md#e-que-mcu-há-na-placa-imu_to_dxl)). A escolha do F8 (64 KB de Flash) em vez do F6 (32 KB) deixa folga para um firmware de protocolo duplo.<br>**O motivo de escolher ST não é «bom ecossistema», são duas características concretas**: ① é do mesmo fabricante do IMU, e o driver [`lsm6dsv16x-pid`](https://github.com/STMicroelectronics/lsm6dsv16x-pid) e o exemplo de SFLP [`lsm6dsv16x_sensor_fusion.c`](https://github.com/STMicroelectronics/STMems_Standard_C_drivers/blob/master/lsm6dsv16x_STdC/examples/lsm6dsv16x_sensor_fusion.c) já existem prontos — a inicialização do SFLP exige configurar FIFO, taxa de fusão e faixa de medição, e se a ordem estiver errada não sai dado; ② a USART do STM32G0 tem **DE (Driver Enable) por hardware**, com comutação automática de direção no half-duplex; a 1 Mbps um bit dura apenas 1 µs, e o jitter de uma comutação por software é dessa ordem de grandeza |
| U2 | **LSM6DSV16XTR** | `C5267406` | IMU de seis eixos + fusão SFLP por hardware, LGA-14 |
| U3 | **SN74LVC2G241DCUR** | `C10430` | Buffer de três estados, para fazer o half-duplex em fio único |
| U4 | **HT7533-1** | `C14289` | LDO de 3,3 V, **tensão suportada de 30 V** (biblioteca básica, sem taxa de setup) |
| J1/J2 | B3B-EH-A(LF)(SN) | `C160259` | Dynamixel 3P, **2,5 mm**, **mesmo código de peça da HAT oficial**, cabos intercambiáveis |
| **J4/J5** | **B3B-PH-K-S(LF)(SN)** | a definir | **Feetech 3P, 2,0 mm vertical** (adicionado em 2026-09-08). O `AMP2.0-3P` da Feetech e o JST EH têm **passos diferentes e não encaixam um no outro**, por isso os dois tipos de conector coexistem, e usa-se o grupo correspondente ao servo.<br>⚠️ **A ordem dos pinos é exatamente oposta à de J1/J2**: `1=DXL_BUS / 2=VDD_BUS / 3=GND`, veja a [tabela de ligação](hardware/imu_to_dxl/imu_to_dxl-接线表.md).<br>⚠️ **Código de peça a definir** — a folha de dados da Feetech só informa o modelo do lado do cabo, não o do lado da placa. O padrão de pads é de três furos com passo de 2,0 mm, comum a outras séries de 2,0 mm; trocar de série só muda o footprint, sem mexer nos pads.<br>O corpo tem altura de cerca de **6,0 mm**, mais adequado à cavidade interna do tronco de 6,65 mm do que os 8 mm do EH vertical |
| **J3** | **PZ254V-11-06P** (6P, 2,54 mm) | `C492405` | **SWD + printf pela porta serial em um só**. Ordem dos pinos `1=GND 2=SWDCLK 3=SWDIO 4=UART_TX 5=UART_RX 6=+3V3` — os 4 primeiros pinos não mudam, e o flat antigo de SWD com 4 pinos ainda funciona nos pinos 1–4.<br>A UART vem dos pinos 16/17 do U1 (`PA11[PA9]`/`PA12[PA10]`), remapeados por `SYSCFG_CFGR1` para **PA9 = `USART1_TX` / PA10 = `USART1_RX`** (DS12992 Rev 3, Table 12, nota 4).<br>**Sem USB**: o G031 não tem periférico USB (zero ocorrências no manual inteiro; Development support só tem SWD), e adicionar USB exigiria mais um CH340/CP2102, o que não compensa numa placa de 45×22 mm. A depuração usa SWD + RTT (o `probe-rs` suporta RTT rodando sobre CMSIS-DAP, sem parar a CPU); para um printf comum, ligue os pinos 4/5.<br>⚠️ O Cortex-M0+ **não tem SWO/ITM**, não conte com printf por SWO |
| C1–C4, C8 | **100 nF 0603** 50 V | `C14663` | Desacoplamento (mesmo código de peça da HAT oficial). C1/C2 junto aos pinos 8/5 do U2, C3 junto ao pino 8 do U3, C4 junto ao pino 4 do U1 |
| **C5** | **4,7 µF / 16 V / 0603 / X5R**<br>Samsung `CL10A475KO8NNNC` | `C19666` | **Reserva de energia local do MCU**; junto com C4 forma o `100 nF + 4,7 µF` exigido pela ST.<br>Base: **DS12992 Rev 3, Figure 13 «Power supply scheme» (p.44)**: o ramo `VDD/VDDA` indica expressamente `1 x 100 nF + 1 x 4.7 μF`, com Caution exigindo «o mais próximo possível do pino, ou no verso da PCB diretamente oposto ao pino».<br>**Biblioteca básica** da LCSC.<br>⚠️ Esta tabela escrevia antes o C5 também como 100 nF — dois 100 nF idênticos em paralelo apenas dobram a capacitância, a curva de resposta em frequência não muda e a faixa de baixa frequência continua vazia, o que não atende ao manual |
| **C6** | **10 µF / 25 V / 0805 / X5R**<br>Samsung `CL21A106KAYNNNE` | `C15850` | **Entrada do LDO**, ligada diretamente ao barramento de 8,4 V. **Biblioteca básica** da LCSC (sem taxa de setup).<br>⚠️ Sob polarização DC de 8,4 V, o X5R perde cerca de 30–40% de capacitância, ficando em cerca de 6–7 µF — suficiente para a entrada de um LDO |
| C7 | 10 µF 0603 10 V | `C19702` | Saída do LDO (3,3 V), 10 V é suficiente |
| **R1/R2/R4/R5/R7** | **10 kΩ 0603** | `C25804` | Cinco pull-ups, todos ligados a 3,3 V:<br>**R1 = SPI_CS** (obrigatório) — durante a energização e o reset, PA4 fica em alta impedância, e um CS flutuante pode ser interpretado pelo IMU como modo I²C<br>**R2 = NRST** — o chip já tem um pull-up fraco interno de ~40 kΩ; um externo adicional melhora a imunidade a ruído<br>**R4 = DE** (obrigatório, adicionado em 2026-09-06) — `1OE#` em nível baixo habilita a transmissão. Durante o reset, PA1 fica em alta impedância; se esse pino flutuar para nível baixo, o buffer de transmissão abre e esta placa despeja dados no barramento, colidindo com os 15 servos. Antes só se havia colocado pull-up no CS, e o DE foi esquecido; apontado pela revisão da comunidade<br>**R5 = DXL_DATA** (adicionado em 2026-09-06) — com o barramento ocioso e ninguém o acionando, a entrada CMOS `2A` flutuando gera corrente de passagem, e a UART também pode tomar ruído por bit de início. Esta placa deve definir ela mesma o nível de repouso, sem depender do host<br>**R7 = RX_EN** (adicionado em 2026-09-07) — o `2OE` estava originalmente ligado direto a 3,3 V (recepção sempre ativa). Mudou-se para pull-up via R7, com um fio também levado a `U1.1 (PB7/PB8)`: o comportamento padrão não muda em nada, mas o firmware ganha a opção de «desligar a recepção durante a transmissão, evitando o eco». Durante o reset do MCU, PB7 fica em alta impedância, e o R7 garante `2OE` em nível alto |
| **R6** | **150 Ω 0603** | `C22808` | **Resistor série na origem da linha de dados** (adicionado em 2026-09-06). Junto com o D2, forma a mesma proteção de linha de dados da HAT: limita a corrente de curto-circuito e suaviza as bordas para reduzir EMI.<br>A 1 Mbps, o atraso formado por 150 Ω e a capacitância do barramento ocupa cerca de 3% do tempo de bit, o que é aceitável.<br>Base: o `dynamixel.kicad_sch` da HAT oficial usa `R33 150R + TH1 100R` em série, totalizando 250 Ω |
| **D2** | **BZT52C5V1-7-F**<br>Zener de 5,1 V, **SOD-123** | `C151588` | **Grampeamento da linha de dados** (adicionado em 2026-09-06). É o **mesmo modelo, mas com outro encapsulamento** do `D4` da HAT oficial — a HAT usa o `BZT52C5V1S` (SOD-323, `C151348`), e esta placa usa **SOD-123** (`C151588`) para facilitar o retrabalho.<br>Catodo em `DXL_BUS`, anodo no terra. A especificação do nível alto do sinal da Feetech é 2–5 V, e a entrada do 74LVC tolera 5,5 V e por si só não queima, mas antes **não havia nenhum componente contra picos na linha**; plugar servos com a alimentação ligada é rotina |
| **D1** | **TVS `SMF12A`**, SOD-123FL | `C2943870` | ⚠️ **Deve ser montado junto ao conector 3P**. Tensão de corte **12 V** > 8,4 V com carga cheia (sem fuga em operação normal); ruptura 13,3–14,7 V; **grampeamento 19,9 V** < os 25 V do C6. Potência de pico 200 W.<br>Encapsulamento de 3,7 × 1,8 mm, metade do SMA. Para maior capacidade, pode-se trocar pela versão SMA `SMAJ12A` (400 W): ST `C132952` / Littelfuse `C148213` |
| **F1** | **PPTC `MF-NSMF020X-2`**, BOURNS, 1206 | `C210358` | Se esta placa falhar, não derruba todo o barramento dos servos (o termistor TH1 de 100R da HAT oficial segue a mesma ideia).<br>**Tensão nominal 24 V** (barramento 8,4 V, margem de 2,9×) · corrente de retenção **200 mA** · disparo 460 mA · tempo de disparo 0,6 s · AEC-Q200.<br>⚠️ **Não escolha os modelos comuns em 0805** — nessa faixa a maioria só tem tensão nominal de **6 V** (como o `0805L150SLYR`, que além disso é de 1,5 A e não 150 mA), e ligados a um barramento de 8,4 V são simplesmente inadequados. **A classe de tensão de um PPTC é uma restrição rígida** |
| J4 | Barra de pinos 2×2 ou 1×4 (opcional, DNP) | — | Leva para fora os quatro fios SPI do IMU. Se no futuro se mudar para SPI direto à placa principal, o MCU não é montado e basta usar fios soltos, sem refazer a placa inteira |

#### Clock: este encapsulamento não aceita cristal passivo

Consultando a **DS12992 Rev 3, Table 12 «Pin assignment and description» (p.36)**, coluna TSSOP20:

| Pino | Nome | Função adicional |
|---|---|---|
| 2 | `PC14-OSC32_IN` | `OSC32_IN`, **`OSC_IN`** |
| 3 | `PC15-OSC32_OUT` | `OSC32_OUT` (**não há `OSC_OUT`**); entre as funções alternativas há `OSC_EN` |

`PF0-OSC_IN` / `PF1-OSC_OUT` aparecem como `-` nessa coluna do TSSOP20, ou seja, **não estão expostos**. Um cristal passivo precisa dos dois pinos
`OSC_IN` + `OSC_OUT` para oscilar, portanto:

| Solução | Viabilidade |
|---|---|
| Cristal passivo de alta velocidade (8/16 MHz + dois capacitores) | ❌ não há `OSC_OUT` |
| **Oscilador ativo em bypass de HSE** (clock single-ended injetado no pino 2) | ✅ o pino 3 ainda pode servir de `OSC_EN` para habilitá-lo |
| Cristal passivo de 32,768 kHz (LSE) | ✅ os pinos 2/3 existem, mas o **LSE não pode alimentar o PLL** (manual p.21: o PLL só aceita HSE ou HSI16); só serve para calibrar o HSI16 |

**Usa-se HSI16 por padrão.** Dados da **Table 41 (p.64)** do manual:

| Item | Condição | Faixa |
|---|---|---|
| Frequência de fábrica | V<sub>DD</sub> = 3,0 V, T<sub>A</sub> = 30 °C | 15,88–16,08 MHz, ou seja, **−0,75% / +0,5%** |
| Deriva térmica | 0 ~ 85 °C | **±1%** |
| Deriva térmica | −40 ~ 125 °C | −2% / +1,5% |
| Deriva de tensão | 1,62–3,6 V | ±0,1% (desprezível) |

Somando tudo, no pior caso entre 0–85 °C é cerca de **−1,85% / +1,55%**. A tolerância típica de uma UART assíncrona a 1 Mbps é da ordem de ±3%,
e o lado do host (Raspberry Pi) tem precisão de cristal e quase não consome da margem — **funciona, mas consome cerca de dois terços da folga**.

→ A solução original era **deixar o footprint do cristal, por padrão não montado (DNP)**, e soldá-lo com ferro de solda se surgir algum problema.
**Em 2026-09-07 passou a ser montado por padrão**; o motivo está abaixo em «Por que mudou de DNP para montado».

##### Posição do clock (a partir de 2026-09-07, **montado por padrão**)

| Ref. | Componente | Código LCSC | Descrição |
|---|---|---|---|
| **X1** | **F322516MUBCE2O** (Yajingxin)<br>**Oscilador ativo** de 16 MHz · CMOS · 1,8–3,3 V · ±10 ppm à temperatura ambiente · **5 mA** · SMD3225-4P | `C5917307` | Usa **bypass de HSE**. Pinagem `1=OE / 2=GND / 3=OUT / 4=VDD`.<br>**Foi escolhido porque 5 mA é o menor consumo entre os candidatos no mesmo encapsulamento** — veja abaixo «Por que escolher o de 5 mA» |
| **C9** | 100 nF 0603 | `C14663` | Desacoplamento do VDD do X1, **colocado junto ao pino 4 do X1** (medido: 2,39 mm) |
| **R3** | 33 Ω 0603 | `C23140` | **Amortecimento série na origem** da saída do clock. A onda quadrada de 16 MHz tem bordas abruptas, e um trecho de trilha já basta para gerar ringing e EMI. Para economizar, pode-se trocar por 0 Ω |

> A escolha original era o `C387369` (Xingtong Shipin S3D16.000000B20F30T, 10 mA). Ao fazer o pedido, descobriu-se que
> **o estoque de montagem SMT da LCSC não tem esse item** (ele é classificado em «componentes que usam estoque privado», com estoque privado 0), por isso foi trocado.

###### Por que mudou de DNP para montado

A razão original do DNP em si continua válida: o HSI16 no pior caso é −1,85%/+1,55%, a tolerância da UART a 1 Mbps é ±3%, o que basta;
o firmware ainda poderia reajustar o `HSITRIM` com captura de entrada de timer, reduzindo o erro para menos de 0,3%.

**Mas esse firmware de autocalibração do `HSITRIM` ainda não teve uma única linha escrita.** A primeira versão da placa nunca foi energizada, e se aparecerem erros reais de comunicação,
«foi o HSI16 que derivou ou o firmware está errado» seria uma bifurcação que só desperdiça tempo. Um cristal de ¥1,9 elimina essa bifurcação de uma vez
— **toda incerteza que pode ser comprada com dinheiro na primeira versão deve ser comprada.**

###### Por que escolher o de 5 mA

A precisão de todos os candidatos no mesmo encapsulamento é mais do que suficiente (o HSI16 a ser substituído é de ±18500 ppm, qualquer um deles é mil vezes melhor),
**o único parâmetro que interage com o projeto desta placa é a corrente de operação**:

```
板上原本总电流 5–10 mA  →  LDO 压降 5.1 V  →  耗散约 51 mW
选 5 mA 晶振  → 总 11 mA → 56 mW → SOT-89 结温升 6~9 °C
选 10 mA 晶振 → 总 16 mA → 82 mW → 结温升 8~13 °C
```

Ambos são seguros (o F1 retém 200 mA, muito longe disso), mas **dobrar a dissipação se soma diretamente ao ponto levantado na revisão
sobre «o aquecimento do LDO afetando o IMU»**. O de 5 mA custa só ¥0,4 a mais, não há motivo para escolher o de 10 mA.

###### ⚠️ Armadilha ao trocar o componente: mesmo encapsulamento de quatro pinos, definição de pinos oposta

Entre os resultados da busca por `3225` na LCSC, **a grande maioria são cristais passivos**, e entre eles o `C13738` (YSX321SL, biblioteca básica,
sem taxa de troca de componente, centenas de milhares em estoque) parece o mais semelhante — mesmo SMD3225-4P, mesmos quatro pads.
**Mas, se for montado, ao energizar será um curto de 3,3 V para o terra.**

| Pino | Oscilador ativo (esta placa foi desenhada assim) | Cristal passivo de 4 pinos |
|---|---|---|
| 1 | Habilitação OE | Terminal do cristal |
| 2 | GND | **Carcaça metálica (aterrada)** |
| 3 | Saída OUT | Terminal do cristal |
| 4 | **VDD** | **Carcaça metálica (aterrada)** |

Os pinos 2 e 4 de um cristal passivo são a mesma tampa metálica, em curto internamente. Nesta placa, `X1.2 = GND` e `X1.4 = 3.3V`,
e montá-lo equivale a usar aquela tampa para ligar os 3,3 V ao terra.

**Ao trocar o componente, aceite somente o rótulo «oscilador ativo / oscillator / OSC»**; os que dizem `XTAL` / `ressonador de cristal` /
`passivo` não servem em hipótese alguma. E isto não é opcional — o TSSOP-20 do G031 **não expõe `OSC_OUT`**,
então um cristal passivo simplesmente não pode ser ligado a este MCU.

Ligação:

```
                     3.3V
                       │
   3.3V ── 1 OE   VDD 4 ┴── C9 100nF ── GND
           ┌───────────┐
           │    X1     │
           └───────────┘
    GND ── 2 GND  OUT 3 ── R3 33Ω ── OSC_IN ── U1 脚2 (PC14/OSC_IN)
```

O `OE` também pode ser ligado ao pino 3 do U1 (função alternativa `OSC_EN` do `PC15`) para habilitação por software, mas isso obrigaria o firmware a cuidar de mais uma coisa;
por padrão é ligado direto aos 3,3 V, sempre ativo.

> ⚠️ **Estes três não entram na BOM, mas entram na PCB** — ao fazer o pedido na JLC, eles não serão montados nem cobrados como material,
> mas os pads continuam na placa. No esquemático já estão configurados como `Add into BOM = no` / `Convert to PCB = yes`.

#### Critério de encapsulamento: resistores e capacitores sempre 0603, nunca 0402

**Decisão de 2026-09-06**: todos os resistores e capacitores passam de 0402 para **0603**, e o D2 de SOD-323 para **SOD-123**.

O motivo não é «não dá para soldar 0402 à mão» — esta placa tem o **LSM6DSV16X em LGA-14** (pads na parte inferior,
3 × 2,5 mm), que de qualquer forma exige ar quente ou refusão, e com essas ferramentas o 0402 também não é difícil.

**O motivo real é o retrabalho na fase de depuração**: o resistor série precisa ser testado de 150 Ω a 33 Ω, algum pull-up precisa ser removido,
um capacitor de desacoplamento precisa ser adicionado — essas mudanças levam alguns segundos com ferro de solda em 0603, e em 0402 seria preciso montar o ar quente de novo.
Uma placa que nunca foi fabricada e validada inevitavelmente passa por algumas modificações na primeira versão.

O custo é pequeno: trocar 11 componentes 0402 da placa por 0603 ocupa cerca de 16 mm² a mais,
menos de 2% da área de 45 × 22 mm da placa. E **todos esses valores em 0603 estão na biblioteca básica da LCSC**
(10 kΩ `C25804`, 150 Ω `C22808`, 33 Ω `C23140`, 100 nF `C14663`),
o que economiza ainda mais taxa de setup do que os componentes da biblioteca estendida em 0402 usados antes.

> O C6 (10 µF/25 V) permanece em 0805; o C5 (4,7 µF) e o C7 (10 µF) já eram 0603.

#### Contorno da placa e fixação

O fabricante **não publicou nenhuma geometria desta placa** (não há malha dela no MJCF). A única evidência física é a peça impressa que a fixa,
`banana_pcb_locker`, e a engenharia reversa a partir de medições dá:

| Grandeza | Valor | Fonte |
|---|---|---|
| Comprimento total da trava | **54,05 mm** | medição da malha |
| **Distância entre centros das duas orelhas de posicionamento** | **≈ 34 mm** | posição das orelhas estimada por cortes no eixo X |
| Cada orelha | ≈ 4 × 2,3 × 2,0 mm | idem |
| Altura do arco do corpo | 6,7 mm (Z 48,97→55,63) | é daí que vem o formato de «banana» |

→ Dedução para a placa original: **dois pontos de fixação a 34 mm de distância, lado maior ≤ 54 mm, formato em arco**.

> 💡 **Mas a própria trava é uma peça impressa** (os arquivos e as quantidades mais recentes estão no [repositório CAD](https://github.com/fanhao375/microduck-replica-cad)) — portanto **o contorno da placa e a trava podem ser redesenhados juntos**,
> sem precisar se adaptar ao arco original. O original foi feito em arco provavelmente para acompanhar a parede interna do tronco e economizar espaço; para uma réplica,
> **placa reta + trava modificada** é muito mais simples, pois o roteamento e a panelização de PCB em arco são trabalhosos.

**Contorno recomendado: 40 × 18 mm (folgado: 45 × 22), dois furos M2 de fixação a 34 mm de distância, placa de 2 camadas.**

- A área líquida dos componentes é de cerca de 230 mm² (dominada pelos conectores: dois JST EH 3P de cerca de 10 × 6 mm cada),
  uma placa de duas camadas, calculando 2–3 vezes, precisa de 460–700 mm² → 40 × 18 = 720 mm² serve na medida
- Mantendo a distância de furos em 34 mm, **dá para usar diretamente a trava impressa original**
- ⚠️ **Não reduza as dimensões para economizar**: na JLC, placas de 2 camadas até 100 × 100 mm custam o mesmo, desenhar 25×20 ou 50×50 custa igual.
  O tamanho só deve ser determinado por restrições de montagem
- ⚠️ **Atenção à altura dos componentes**: a cavidade interna da trava tem 6,65 mm de altura, e o conector JST EH de inserção reta tem corpo de cerca de 8 mm —
  ou se usa um encapsulamento horizontal (side entry), ou esta placa simplesmente não é prensada por aquela trava

#### Pontos de layout (funcionam melhor do que adicionar componentes, e não custam nada)

**Tipo de placa: 2 camadas, 45 × 22 mm, todos os componentes SMD na face superior.**

| Decisão | Por quê |
|---|---|
| **2 camadas bastam** | Uma dúzia de componentes, e 1 Mbps não é alta velocidade. **O que mais se precisa é de um plano de terra completo**, que numa placa de 2 camadas se obtém preenchendo toda a camada inferior com terra, muito mais importante que ter mais duas camadas. Em custo, 2 camadas saem várias vezes mais baratas que 4 |
| **Todos os SMD na face superior** | ① A montagem SMT de protótipo cobre uma face só, **o custo cai pela metade** (dupla face passa duas vezes pelo forno) ② A camada inferior não é interrompida por componentes e vias, e o **plano de terra fica completo** |
| **Conectores THT, alinhados na mesma borda** | O `B3B-EH-A` e a barra de pinos SWD são de inserção. Soldá-los à mão economiza o custo de montagem THT (pinos grandes, passo de 2,5 mm, muito fáceis de soldar).<br>⚠️ Mas as vias abrem buracos no plano de terra inferior — **os três conectores ficam do mesmo lado**, concentrando os buracos de um lado só, sem retalhar o plano de terra |
| 45 × 22 e não 40 × 18 | O layout de face única precisa de 2–3 vezes a área líquida dos componentes (cerca de 230 mm²) → 460–700 mm². 45×22 ＝ 990 mm² é folgado.<br>**De qualquer forma, na JLC o preço é o mesmo até 100 × 100, não há por que reduzir** |

**Três pontos para o IMU (exigência da folha de dados + bom senso, sem custo):**

1. **`VDD` e `VDDIO` cada um com 100 nF de desacoplamento próximo** — exigência expressa do manual
2. **Plano de terra completo, sem linhas de alta velocidade logo abaixo do IMU**
3. **Separar a trilha de alimentação do IMU da parte de transmissão/recepção da UART**

> **Sem isolamento elétrico.** O codec de áudio, o BMI088 e o ToF da HAT oficial estão todos ligados diretamente à mesma alimentação, zero isolamento,
> e o robô é capaz de andar. Mais importante ainda: isolamento de verdade exige uma alimentação independente, e então esta placa não poderia mais tirar energia do barramento —
> **a premissa de projeto de «ser o 16º dispositivo no barramento» deixaria de existir.**
>
> 💡 Deixe uma posição para **jumper de 0 Ω** na entrada de alimentação: por padrão monta-se 0 Ω direto; se no futuro medições mostrarem que o ruído da alimentação afeta a leitura do giroscópio,
> basta trocar por uma ferrite bead. Custo igual, risco zero — colocar a ferrite diretamente pode, em vez disso, formar uma ressonância LC com o capacitor grande e amplificar certa faixa de frequência.

#### ⚠️ Proteção contra sobretensão: ela divide o mesmo barramento com os servos

**Este é o risco mais facilmente ignorado nesta placa.** Os 15 servos estão pendurados no mesmo `+BATT`; ao desacelerar, inverter e liberar um travamento,
a **força contraeletromotriz eleva a tensão do barramento**; esta placa está no fim do barramento, e a indutância do cabo somada à comutação rápida faz os picos vistos na entrada serem piores que no lado da bateria.

> Note que a natureza da ameaça é **sobretensão direta, e não polaridade invertida** — por isso um diodo em série não resolve o problema.

Três linhas de defesa, em ordem de importância:

| Defesa | Componente | Contra o quê | Estado |
|---|---|---|---|
| **① LDO de tensão ampla** | `HT7533-1` (operação 30 V / máximo absoluto 33 V) | Sobretensão contínua. Um barramento de 8,4 V, mesmo elevado a 17 V, não se abala, **margem de 3,5×** | ✅ já escolhido |
| **② TVS** | `SMAJ12A`, **junto ao conector** | Picos de µs. Corte 12 V > 8,4 V com carga cheia (sem fuga), grampeamento 19,9 V < os 25 V do C6 | ⚠️ adicionado agora |
| **③ PPTC** | hold 100–200 mA | Quando esta placa entra em curto, não derruba o barramento inteiro — os 15 servos perderiam energia junto e o robô cairia | ⚠️ adicionado agora |

> **A posição do TVS importa mais que o modelo**: ele deve ficar ao lado do conector 3P, quanto mais perto da entrada, melhor. Colocá-lo junto ao LDO já é tarde —
> o pico já percorreu a placa.
>
> ⚠️ **A amplitude real da força contraeletromotriz não foi medida neste repositório**; depende da indutância do cabo e da velocidade de comutação. Depois que os servos chegarem, meça uma vez com osciloscópio
> no barramento e volte aqui para conferir a escolha.

> ⚠️ **Mais duas armadilhas, ambas queimam a placa:**
>
> **1. Tensão suportada do LDO.** Com o barramento a 8,4 V na carga cheia, todos os modelos comuns são insuficientes (abaixo estão **limite da tensão de operação / máximo absoluto**):
> AP2112K **6,0 / 6,5 V**, ME6211 **6,0 / 6,5 V**, TLV75533 **5,5 / 6,0 V**.
> É obrigatório escolher o HT7533-1 (operação 30 V, máximo absoluto 33 V) ou equivalente.
>
> **2. Tensão suportada do capacitor de entrada — esta tabela escolheu errado antes.** O `C19702` é um X5R de **10 V**; ligado a um barramento de 8,4 V,
> sobra apenas 1,2× de margem, e o X5R sob polarização DC de 8,4 V tem **capacitância real inferior à metade da nominal**, além de ter de suportar os transientes
> de partida e parada dos servos. **O C6 deve ser trocado por um componente de ≥25 V** (0805 é mais seguro); o C7, no lado da saída de 3,3 V, com 10 V basta.

---

## 5. Peças mecânicas

### Rolamentos (14 no total)

| Especificação | Qtd. | Base |
|---|---|---|
| **Ø22 × 16 × 4** | **11** | `seeed_bearing__configuration__22x16x4` referenciado 11 vezes |
| **Ø15 × 10 × 3** | **3** | `seeed_bearing__configuration_default` referenciado 3 vezes |

### Fixadores (principalmente M2, cerca de 325 peças)

Os furos de classe M2 do robô completo (Ø1,9–2,5 mm, ângulo de envolvimento ≥300°), **ponderados pelo uso no conjunto montado, somam 237**,
dos quais **60 estão no corpo dos servos** (15 × 4), 21 em peças padrão como rolamentos/PCB/bateria
e **cerca de 156 nas peças estruturais**.

> ⚠️ **Nota sobre o critério**: o «213 furos» escrito no início neste repositório não pôde ser recalculado e foi corrigido para 237.
> A diferença vem de dois pontos: a contagem inicial **não foi ponderada pelo uso no conjunto** (`leg` na verdade é ×4, `hip_l` ×2 etc.),
> e **incluiu malhas não utilizadas**. A tabela de distribuição de diâmetros usa um critério diferente deste número, veja [reconstrução dos fixadores](docs/紧固件反推.md).
>
> As quantidades de compra abaixo são dadas com base em 325 peças e, para 237 furos, ainda deixam **1,37×** de folga, **sem afetar o pedido**.

| Especificação | Quantidade sugerida | Uso |
|---|---|---|
| M2×4 cabeça cilíndrica com sextavado interno | 60 | paredes finas |
| M2×6 cabeça cilíndrica com sextavado interno | **80** (principal) | |
| M2×8 cabeça cilíndrica com sextavado interno | 40 | profundidade de furo de 3–5 mm |
| M2×12 cabeça cilíndrica com sextavado interno | 15 | poucos furos profundos |
| Porcas M2 | 50 | onde não há rosca aberta |
| **Porcas de inserção térmica M2** | **60** | recomendadas para peças impressas, muito mais resistentes do que abrir rosca direto |
| M2.5×6 | 20 | poucos furos Ø2,7 |

O processo de dedução está em [reconstrução dos fixadores](docs/紧固件反推.md).

### Peças impressas e variante com rodas

**Modelos, quantidades, materiais e instruções de montagem: consulte sempre o [repositório CAD](https://github.com/fanhao375/microduck-replica-cad#装配-bom).** Escolha primeiro Feetech HD-1910 ou XL330 e depois confira a versão específica do anexo. Este repositório não oferece mais uma segunda cópia dos STL de impressão, nem trata a quantidade de malhas duplicadas do MJCF do upstream como a quantidade real a imprimir.

A versão Feetech v2.1 já trocou o aro e o pneu independentes da variante com rodas por um pneu integrado em TPU; o 3MF antigo de 09-15 ainda não foi sincronizado com essa mudança. Veja os detalhes em [ponto de entrada da impressão e avisos de versão](print/README.md). As peças para andar e as peças de substituição para rodas também não podem ser misturadas.

---

## 6. Lacunas ainda não resolvidas

No final da réplica você vai travar nestes dois pontos, para os quais os materiais públicos não dão resposta:

1. **Contatos de alimentação da bateria.** No CAD só existe a peça impressa `power_support` (×2), **sem nenhum modelo de PCB de contatos ou de lâminas de mola**.
   Não dá para saber se o fabricante usa lâminas metálicas ou um adaptador NP-F comercial. O jeito mais prático é comprar um adaptador de bateria NP-F pronto.
2. **Chicote de cabos.** O comprimento dos cabos Dynamixel 3P entre os servos e o do flat MIPI da câmera não têm nenhum desenho oficial.
   Os servos costumam vir com cabos curtos, mas o comprimento da fiação interna do robô precisa ser medido na prática.

---

## Fontes dos dados

| Dado | Fonte |
|---|---|
| Quantidade de peças | contagem de referências geom `mesh=` do `robot_walk.xml` do upstream |
| Quantidade da variante com rodas | `robot_groundcontact_rollers.xml` do upstream |
| Quantidade de parafusos | reconstrução a partir da geometria dos furos nos STL, veja [reconstrução dos fixadores](docs/紧固件反推.md) |
| Modelos dos componentes eletrônicos | código-fonte Rust, device tree, `robotd.toml`, veja [engenharia reversa da solução de hardware](docs/硬件方案逆向.md) |
| Parâmetros da placa HAT | projeto KiCad oficial e arquivos de produção em `production/` |
| Especificação dos servos | [Robotis e-manual](https://emanual.robotis.com/docs/en/dxl/x/xl330-m288/) |
| Códigos LCSC | consulta em tempo real à biblioteca de componentes do EasyEDA da JLC (嘉立创) |
