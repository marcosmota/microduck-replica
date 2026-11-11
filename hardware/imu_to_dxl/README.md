# Esquemático + PCB do imu_to_dxl · em busca de revisão

> 🌐 Tradução não oficial para pt-BR do [original](https://github.com/fanhao375/microduck-replica). Em caso de divergência, vale o original. Licença: CC BY-NC-SA 4.0 (texto alterado: traduzido do chinês/inglês).

> ⚠️ **Esta é uma peça de réplica de terceiros, não um design oficial.**
> O projeto oficial nunca publicou o esquemático, o Gerber nem o contorno desta placa (zero resultados na internet; veja
> [engenharia reversa da solução de hardware](../../docs/硬件方案逆向.md)). Este diretório é o resultado de inferência
> a partir do modelo MJCF, do código-fonte do runtime oficial e do projeto de HAT já aberto, mais a seleção de componentes por conta própria, **sem protótipo fabricado e sem verificação em hardware real**.
> Se encontrar problemas, abra uma [issue](https://github.com/fanhao375/microduck-replica/issues).

![Esquemático do imu_to_dxl](../../assets/hw/imu_to_dxl-原理图.png)

| Arquivo | Descrição |
|---|---|
| [`imu_to_dxl-原理图.pdf`](imu_to_dxl-原理图.pdf) | PDF vetorial, para ampliar e ver os detalhes |
| [**`imu_to_dxl-交互式BOM.html`**](imu_to_dxl-交互式BOM.html) | **Para soldagem**: BOM interativa exportada do EasyEDA (LCSC); baixe e abra diretamente no navegador (funciona offline). Clique no código da peça para destacar a posição correspondente na imagem da placa e marque cada tipo que soldar. Clicar direto na página do GitHub mostra só o código-fonte |
| [**`firmware/`**](firmware/) | Firmware STM32G031 + LSM6DSV16X, projeto Keil, testes no host e um HEX pré-compilado; a partir da 0.2.0 usa o protocolo Feetech SCS/STS (ID 200), ainda não testado em barramento real |
| [**`firmware-1.0.0/`**](firmware-1.0.0/) | Outra versão do firmware do protocolo Feetech, 1.0.0: derivada à parte da 0.1.0 (versão Dynamixel), **testada em campo na mesma linha com servos Feetech reais** (posição na ordem, leitura e escrita), com ferramentas do lado do host como varredura / monitor de barramento / curvas do IMU e instruções de uso em chinês. Como a 0.2.0, usa o mesmo bloco de 15 bytes e o mesmo modelo (0x4D44); o programa principal da imagem da réplica 20261021b em diante reconhece as duas versões |
| [**`总线协议.md`**](总线协议.md) | **Para quem escreve o firmware**: definição byte a byte do bloco de 15 bytes, requisitos de temporização, regras de enfileiramento, critérios do controlador principal, vetores de teste e método de aceitação |
| [`imu_to_dxl-接线表.md`](imu_to_dxl-接线表.md) | Lista cada pino rede por rede, para conferir diretamente |
| [`imu_to_dxl.eprj2`](imu_to_dxl.eprj2) | Projeto do EasyEDA (LCSC) edição Pro, que pode ser aberto e modificado diretamente |
| [`../../BOM.md`](../../BOM.md) | BOM completa, com os códigos LCSC e a justificativa de seleção de cada componente |

**Ponto de entrada do firmware e da visualização da atitude:** [`firmware/README.md`](firmware/README.md) registra a inicialização da placa isolada, a amostragem do IMU e o escopo da validação.

**Como escolher entre as duas versões do firmware**: ao ligar ao pato e usar com o programa principal do fork, as duas servem. O programa principal (imagem da réplica 20261021b em diante) reconhece que é esta placa e, ao ler o corpo inteiro de uma vez, a coloca **depois dos servos**: com a 1.0.0 na primeira posição, em 30 tentativas só respondeu 3 vezes (os servos seguintes tomam a palavra por temporização e abafam o byte de checksum dela); no meio ou por último, 30/30. Para ver a atitude no servo-web via SWD / J-Link, só se pode usar o HEX da 0.2.0 em `firmware/`: o observador confere pelo hash e pelo endereço de memória dele, e com a 1.0.0 se recusa a ler. Nenhuma das duas versões foi executada por longo tempo no robô completo.
Por meio de [`tools/servo-web`](../../tools/servo-web), a atitude da placa pequena pode ser sobreposta ao pato 3D do console de depuração dos servos: os ângulos das juntas vêm dos servos e o quaternion do tronco vem do IMU.
Três formas de leitura: `--imu-bus` (barramento dos servos, no mesmo sync_read do controlador principal, lendo o bloco de 15 bytes no endereço 56 do ID 200), `--imu-swd` (ST-Link / DAPLink), `--imu-jlink` (J-Link).
A aceitação com `imu200.py check` no barramento real ainda não foi feita.

---

## Para que serve esta placa

Os 15 servos do Microduck oficial ficam em um barramento half-duplex Dynamixel. Esta placa pequena
**entra no mesmo barramento como o 16º dispositivo**, apresentando-se externamente como um escravo Dynamixel (ID 200),
e o controlador principal lhe pede os dados de atitude lendo registradores.

O IMU da placa é o **LSM6DSV16X**, que tem internamente o **bloco de fusão SFLP em hardware**
e emite diretamente o quaternion de rotação de jogo: o cálculo da atitude não consome processamento do controlador principal nem largura de banda do barramento.

```
Controlador principal (Raspberry Pi/Taishan Pi)
      │  1 Mbps half-duplex, fio único
      ├── Servos ×15
      └── imu_to_dxl (esta placa, ID 200)
             ├── LSM6DSV16X   SPI de quatro fios + duas interrupções
             ├── STM32G031F8P6 recebe pacotes / monta pacotes / lê o IMU
             └── SN74LVC2G241 buffer de transceptor half-duplex
```

## Seleção dos componentes principais e motivos

| Posição | Componente | Por que foi escolhido |
|---|---|---|
| **U1** | STM32G031F8P6 | ① O USART tem **DE (Driver Enable) em hardware** (RS485): a 1 Mbps um bit dura apenas 1 µs, e o jitter de inverter o pino de direção por software está nessa ordem de grandeza. Em comparação: **o manual do STM32F401 não contém "Driver Enable" em lugar nenhum**, só permite inversão por software<br>② Do mesmo fabricante do IMU, com o driver `lsm6dsv16x-pid` da ST e exemplos de SFLP prontos |
| **U2** | LSM6DSV16XTR | Fusão SFLP interna, emite diretamente o quaternion |
| **U3** | SN74LVC2G241DCUR | Buffer duplo para fazer o half-duplex de fio único. O canal 1 entra à esquerda e sai à direita, o canal 2 entra à direita e sai à esquerda, alinhando-se bem com as trilhas. Com alimentação de 3,3 V, **a entrada tolera 5,5 V**, suportando o sinal de 5 V dos servos |
| **U4** | HT7533-1 | **Tolera 30 V**: continua vivo mesmo que as duas primeiras proteções falhem |
| **F1** | PPTC MF-NSMF020X-2 | **24 V / 200 mA de retenção**. Em caso de falha desta placa, ela se desliga primeiro, sem derrubar todo o barramento dos servos |
| **D1** | TVS SMF12A | Tensão de corte de 12 V > 8,4 V da bateria cheia, grampo de 19,9 V < tensão suportada de 25 V do C6 |
| **C6** | 10 µF / **25 V** | Ligado ao barramento de 8,4 V. O MLCC sofre queda de capacitância por polarização DC, e as margens de 6,3 V/10 V não bastam |
| **C5** | 4,7 µF | Forma, com os 100 nF do C4, a combinação exigida pela ST: **DS12992 Rev 3, Figure 13** indica explicitamente para `VDD/VDDA` `1 × 100 nF + 1 × 4.7 μF` |

## Algumas decisões que podem gerar controvérsia

### 1. `2OE` ligado fixo em alto: durante a transmissão ouve-se o próprio eco

`U3.1 (1OE#)` é ligado ao DE para controlar o buffer de transmissão, e `U3.7 (2OE)` **vai direto a 3,3 V, com o buffer de recepção sempre aberto**.

- Custo: ao transmitir, o MCU recebe no RX os próprios bytes enviados, e o firmware os descarta conforme o comprimento transmitido
- Benefício: o PA3 está sempre sendo acionado por alguém e **nunca fica flutuando**

A alternativa seria ligar também o `2OE` ao DE (desligando a recepção durante a transmissão), mas assim o RX fica flutuando durante a transmissão
e, ao reabilitar, pode surgir um falso bit de início. Escolhi a primeira opção, **contestações são bem-vindas**.

### 2. Sem cristal, só as pastilhas

O encapsulamento TSSOP-20 **não expõe o `OSC_OUT`** (DS12992 Rev 3, Table 12: no pino 2 `PC14` a função adicional é
`OSC_IN`, e no pino 3 `PC15` só há `OSC32_OUT`), portanto **um cristal passivo simplesmente não pode ser ligado**.

Precisão do HSI16 (Table 41): de fábrica −0,75%/+0,5%, deriva térmica de 0–85 °C de ±1% → pior caso de cerca de **−1,85%/+1,55%**.
A tolerância típica de uma UART assíncrona a 1 Mbps é de ±3%: basta, mas com pouca folga.

Na placa há o **X1 (oscilador ativo de 16 MHz, HSE em bypass) + C9 + R3**, inicialmente não montados por padrão e, desde 2026-09-08, **montados por padrão** (o firmware ainda parte do HSI16, e só se passa ao bypass HSE se a medição real a 1 Mbps mostrar erros de bit). O pino de habilitação do X1 está fixo em 3,3 V e o PC15 não está ligado a ele, de modo que o MCU não consegue desligar o oscilador; veja "Registro de revisão · 2026-09-16".
Além disso, o firmware pode medir o período de bit vindo do host por captura de entrada de um timer e reajustar o `HSITRIM` (passo de 0,2–0,4%/degrau),
reduzindo o erro a menos de 0,3%, sem custo.

### 3. J1 / J2 mantidos em dois, com a sobrecorrente resolvida pelo cobre

As três linhas de J1 e J2 estão ligadas uma a uma em paralelo. **Se esta placa ficar no meio da cadeia, a corrente de todos os servos a jusante passa pelo conector e pelo cobre desta placa.**

Parâmetros oficiais do Feetech HD-1910-C001: corrente nominal **500 mA** / travamento **1,8 A**.

Mas essa restrição **não é exclusiva desta placa**: os servos já formam uma cadeia em margarida entre si, e os próprios conectores deles suportam a mesma corrente.
O HAT tem **4 portas de servo**, os 15 servos se distribuem em 4 ramos, com no máximo 4 cada, e um ramo tem corrente nominal de cerca de **2 A**,
dentro de 3 A. O design oficial aceita isso, e esta placa, usando o mesmo tipo de conector, não ficará pior.

**Por isso foram mantidos dois conectores**: para ficar na ponta, não se monta o J2; para ficar no meio da cadeia, monta-se, e a flexibilidade custa praticamente nada.
No PCB basta passar `VDD_BUS` e `GND` de J1 a J2 por um **cobre largo** (≥ 2 mm, ou diretamente em plano de cobre).

> Houve uma sugestão de revisão de "manter apenas um conector 3P". Isso vale quando esta placa é com certeza um nó folha,
> mas **se ela for o primeiro nó da cadeia** (saindo do HAT, entra nesta placa e depois sai para os servos), são necessários dois.
> Não há como confirmar a topologia oficial, e manter dois é mais seguro.

### 4. O protocolo é reconhecido automaticamente por software, sem jumper/DIP switch

A camada física do Feetech e a do Dynamixel são idênticas (half-duplex de fio único / 1 Mbps / 3 fios / GND-VCC-DATA);
a diferença está só no formato do pacote:

| | Cabeçalho | Em seguida | Checksum |
|---|---|---|---|
| Dynamixel V2 | `FF FF` | **`FD 00`** | CRC-16 |
| Feetech SCS | `FF FF` | `ID` + `LENGTH` | `~(ID+LEN+INST+parâmetros)` |

A documentação oficial do protocolo Feetech informa que **Length = número de parâmetros N + 2**, portanto o mínimo é 2 e **nunca é 0**;
o ID varia de 0 a 253. → `FF FF FD 00` só pode ser V2, **a discriminação é inequívoca**.

O firmware trava o protocolo ao receber o primeiro pacote com checksum válido e, dali em diante, envia e recebe conforme o protocolo travado, **sem destravar automaticamente durante a operação** (a ideia original era "reavaliar após 100 pacotes consecutivos com falha"; a revisão de 2026-09-17 apontou que o ruído no barramento faria a placa pequena ficar muda por 2 s e o controlador principal entrar em hold após 3 ticks sem leitura, então foi alterado; veja [`software/飞特适配架构.md` §3.5](../../software/飞特适配架构.md)).
É mais confiável que um DIP switch (em um robô que anda, o contato do DIP é um ponto de falha por vibração).

### 5. J3 com 6P, expondo também o printf serial

Conforme o PDF do esquemático de 2026-09-08, a ordem dos pinos é
`1=GND 2=SWDCLK 3=SWDIO 4=UART_RX 5=UART_TX 6=+3V3`.
O SWD usa os pinos **1, 2, 3 e 6**; o pino 6 fornece a referência de tensão ao depurador. **O cabo flat antigo de quatro pinos não pode ser encaixado inteiro nos pinos 1–4**:
o pino 4 é entrada serial e não pode ser tratado como 3,3 V. Antes de ligar, confira a marca do pino 1 na placa real e a definição dos pinos do depurador.

A UART vem dos pinos 16/17 do U1 (`PA11[PA9]` / `PA12[PA10]`), remapeados via `SYSCFG_CFGR1` para
`USART1_TX/RX`.

**Sem USB**: o G031 não tem periférico USB (o manual, em Development support, só menciona "serial wire debug (SWD)"),
e acrescentar USB exigiria mais um CH340/CP2102, o que não compensa em uma placa de 45 × 22 mm.
A depuração é feita por SWD + RTT (o `probe-rs` suporta executar RTT sobre CMSIS-DAP sem parar a CPU).

> O Cortex-M0+ **não tem SWO/ITM**; não espere printf via SWO.

---

## Contorno da placa e furos de fixação

| Item | Valor |
|---|---|
| Contorno | **45 × 22 mm**, placa de 2 camadas |
| Quatro cantos | **Cantos arredondados R2** |
| Furos de fixação | **2 × furos M2 não metalizados φ2,2 mm**, em diagonal |
| Posição dos furos (origem no canto inferior esquerdo da placa) | MH1 (2,667, 2,413), MH2 (42,672, 19,685) mm |
| Distância entre furos | Direção X 40,00 mm ／ direção Y 17,27 mm ／ diagonal 43,57 mm |
| Furo até a borda da placa | MH1 **1,31 mm** ／ MH2 **1,22 mm** (a JLCPCB exige ≥0,4 mm) |

O ponto médio dos dois furos (22,67, 11,05) difere do centro da placa (22,50, 11,00) em 0,17 mm, de modo que **girada 180° ainda se alinha praticamente** (desvio de 0,34 mm).

> ⚠️ **A posição dos furos foi alterada em 2026-09-08.** O design original era MH1 (5,5, 3,5) / MH2 (39,5, 18,5), com 34 mm na direção X,
> deliberadamente alinhado ao espaçamento entre as duas orelhas de posicionamento da peça impressa original `banana_pcb_locker`.
> **Ao adicionar o `J4`/`J5`, o círculo de folga do MH2 bloqueava exatamente a única faixa livre à direita onde cabiam componentes de furo passante**,
> então os dois furos foram deslocados para os cantos, e a direção X passou a 40 mm.
> **O custo é deixar de ser compatível com a trava original; do lado mecânico, a barra de pressão deve ser alterada em conjunto.**
> De qualquer forma, "o que a trava prende é o `imu_to_dxl`" na documentação de engenharia reversa era só uma inferência
> (veja [engenharia reversa da solução de hardware](../../docs/硬件方案逆向.md)); a única restrição rígida é "o IMU fixado rigidamente ao `trunk_base`".

Arquivo do contorno da placa: `imu_to_dxl-板框-45x22-R2.dxf` (unidade mm, 8 segmentos fechados: 4 retas + 4 arcos de 90°).

> **Por que o DXF é fornecido à parte**: nesta versão do EasyEDA (LCSC) edição Pro, a interface de primitivas só aceita camadas de cobre;
> criar linhas/arcos na camada de contorno (layer 11) sempre dá "parâmetro incorreto", e trocar de camada ou de fonte do documento é bloqueado,
> por isso o contorno é importado por DXF (`Arquivo → Importar → DXF`, camada de destino **camada de contorno**, unidade mm, origem alinhada a 0,0).
> Os dois furos M2 não estão no DXF: já foram criados no PCB como pads NPTH reais.

## PCB: posicionamento e roteamento

![Imagem de montagem da camada superior do PCB do imu_to_dxl](../../assets/hw/imu_to_dxl-PCB.png)

*Imagem de roteamento, renderizada a partir de dados medidos do projeto (camada superior em vermelho / inferior em azul / pads em amarelo / vias de GND em verde / vias de sinal em roxo, fundo azul é o plano de cobre GND da camada inferior, o quadro roxo é `J4`/`J5`).*
*Estado em 2026-09-08: 29 componentes · 227 segmentos de trilha · 31 vias · 172 teardrops, incluindo `R7`, `J4`, `J5`, com `X1`/`R3`/`C9` já alterados para montados.*
*Neste diretório, o `.eprj2` / PDF do PCB / PDF do esquemático / STEP e as duas imagens em `assets/` foram **todos atualizados para a versão final de 2026-09-08**.*

**45 × 22 mm, 2 camadas, cobre de 1 oz**, quatro cantos R2, dois furos M2 não metalizados na diagonal.
Os componentes SMD ficam todos na face frontal; os 3 conectores de furo passante estão em dois lados: `J1` `J2` (barramento dos servos) na borda inferior,
`J3` (SWD + serial) na borda superior.

### Zonas

| Zona | Conteúdo | Por quê |
|---|---|---|
| Borda inferior | Conectores de barramento `J1` `J2` (JST EH 2,5 mm) | O chicote passa pelo tronco, com saída de fio próxima |
| Inferior direita | **`J5` saída do barramento Feetech** (2,0 mm) | Do mesmo lado de `J1`/`J2`; o cobre de passagem direta do `VDD_BUS` dos quatro conectores forma uma só linha |
| Borda direita | **`J4` entrada do barramento Feetech** (2,0 mm) | Única faixa livre da placa onde cabe um componente de furo passante; o MH2 foi deslocado para dar passagem a ele |
| Lado direito | `F1` → `D1` → `C6`/`C8` → `U4` → `C7` | A cadeia de alimentação segue um só caminho até o fim, separada da zona digital |
| Centro | Buffer `U3` + grampo `D2` + resistor em série `R6` | Entre o MCU e os conectores do barramento, com o caminho de transmissão/recepção mais curto |
| Centro-superior | MCU `U1`, IMU `U2` e o desacoplamento de cada um | |
| Lado esquerdo | Posição do relógio `X1` / `R3` / `C9` | Já alterado para montado, colocado no canto sem ocupar um bom lugar |
| Borda superior | Barra de pinos de depuração `J3` | Direção de encaixe voltada para fora |

### Roteamento

| Item | Valor |
|---|---|
| Componentes | **29** (incluindo `R7`, adicionado em 09-07, e `J4`/`J5`, adicionados em 09-08) |
| Trilhas | **227 segmentos** (camada superior 197 / inferior 30) |
| Largura de trilha | Sinal principalmente 0,25 mm; **passagem direta do barramento 0,76 mm** |
| Vias | **31, φ0,6 / furo 0,3 mm** |
| Teardrops | 172 |
| Plano de cobre | **Camada inferior inteira em GND**, sem plano na superior |

As 30 trilhas da camada inferior deixam o **plano de terra praticamente completo**, o que dá caminho de retorno ao SPI e ao barramento half-duplex de 1 Mbps.
Os pads GND da camada superior têm, cada um, vias próximas descendo ao plano de terra.

### Resultados da revisão · 2026-09-08

Além do DRC próprio do LCSC, um script rasterizou novamente com a **forma real dos pads** (circular / oblonga / retangular)
e fez uma verificação independente:

| Item verificado | Resultado |
|---|---|
| DRC do LCSC | **0 violações** |
| **Ordem dos pinos dos conectores** | **Os quatro corretos**: `J1`/`J2` = `GND`/`VDD_BUS`/`DXL_BUS`; `J4`/`J5` = `DXL_BUS`/`VDD_BUS`/`GND` (**ordem inversa**, veja a seção sobre ordem dos pinos abaixo) |
| Conectividade das redes | **Todas as 23 redes conectadas** (domínios conexos rasterizados + vias/furos de componentes entre camadas, incluindo o plano de cobre da camada inferior) |
| Menor distância entre redes distintas | **0,152 mm ＝ 5,98 mil** (geometria exata, não raster). O limite inferior da JLCPCB para dupla face é 0,127 mm, folga de 20% |
| Cobre até a borda da placa | Mínimo **0,37 mm** |
| Furo a furo (entre paredes) | Mínimo **0,751 mm**, exigido ≥0,4 mm |
| Furo até a borda da placa | MH1 **1,31 mm** / MH2 **1,22 mm** |
| Folga do M2 | No círculo de folga de 2,5 mm do MH1 há apenas uma via de **GND** (1,81 mm): o parafuso pressionando terra continua sendo terra, inofensivo; o mais próximo do MH2 está a 4,59 mm |
| Especificação das vias | φ0,6 / furo 0,3 mm, de acordo com o limite inferior da JLCPCB para dupla face |
| Distância de desacoplamento | `U1.4`→`C4` 2,27, `U2.8`→`C1` 2,00, `U2.5`→`C2` 2,01, `U3.8`→`C3` 1,96, `X1.1`→`C9` 2,39 mm, todos <2,4 mm; `U4.3`→`C7` 2,94 mm. ⚠️ O `C8` de 100 nF no lado de entrada do LDO `U4.2` está a **6,64 mm** (um tanto frouxo, mas o `C6` de 10 µF, que cuida dos transientes, está a 4,29 mm, e o LDO é linear, sem chaveamento) |

**Cálculo de corrente**: a cadeia de passagem direta do `VDD_BUS` entre os quatro conectores
`J1.2 → J2.2 → J5.2 → J4.2` tem **0,76 mm em todo o percurso**, e com cobre de 1 oz suporta cerca de 2,0 A para ΔT=10K e cerca de 3,2 A para ΔT=30K.
O contato único do JST EH tem corrente nominal de 3 A, de modo que **a trilha e o conector estão praticamente na mesma ordem de grandeza**, e alargar a trilha não eleva o limite da cadeia inteira.
Há ainda um trecho de 0,51 mm e 4,7 mm no ramo `J1.2 → F1.1`, que **alimenta apenas esta própria placa**,
precedido por um PPTC de 200 mA de retenção, o que basta.

> **Um problema encontrado e corrigido em 2026-09-08**: ao rotear o `DXL_BUS` para `J4`/`J5`, uma via de φ0,3
> caiu exatamente sobre o furo passante φ1,0 do `J2.3` (distância entre centros de 0,001 mm). O DRC do LCSC acusou
> `Hole to Hole (DXL_BUS): J2_3 ↔ e75`: a furação é uma etapa mecânica, e nem na mesma rede sobreposição é permitida.
> O `J2.3` em si é um furo passante, já conectando topo e fundo, então a via era puramente redundante; ao removê-la, a conectividade ficou igual e o DRC zerou.

**Ainda não verificado**: **sem protótipo e sem placa real**. Tudo o que está acima são conferências de geometria e regras,
que não substituem medições reais.

### Pendências

- Antes de fazer o pedido, rodar uma vez a **padronização de componentes** (o DRC do esquemático acusou "propriedades do componente não correspondem ao código do fornecedor")
- Revisão da serigrafia: se os designadores sobrepõem os pads, e se a marca do pino 1 do `U1` / cátodo de `D1` `D2` / pino 1 do `J1` estão completas
- **Altura dos conectores**: `J1`/`J2`/`J3` são JST EH / barra de pinos 2,54 de furo passante, com corpo de cerca de **8 mm**. Se a cavidade do tronco
  realmente tiver só 6,65 mm, é preciso trocar por `S3B-EH-A` horizontal, e o footprint e a BOM têm de ser alterados juntos: basta comparar o STEP exportado
  na montagem para concluir.
  **Já `J4`/`J5` não preocupam**: o `B3B-PH-K-S` tem corpo de cerca de **6,0 mm**, cabendo na cavidade de 6,65 mm
- **Modelo do conector da placa a confirmar**: a especificação da Feetech só informa que a ponta do cabo é `AMP2.0-3P`, **sem indicar o modelo do lado da placa**.
  Esta placa usa provisoriamente o `B3B-PH-K-S` (JST PH, 2,0 mm vertical). A matriz de pads é de três furos com passo de 2,0 mm,
  comum a outras séries de 2,0 mm, de modo que **ao trocar de série no futuro só se altera o footprint, sem mexer nos pads nem rerrotear**
- **Reexportar os arquivos de produção antes do pedido**: os componentes passaram de 27 para 29, e o Gerber / BOM / coordenadas de pick-and-place no LCSC precisam ser reenviados

## Pontos que gostaria que você examinasse com mais atenção

1. **A decisão de deixar `2OE` sempre aberto** (item 1 acima): eco vs RX flutuando, qual é pior
2. **Sobrecorrente de J1/J2 e a largura do cobre** (item 3): 2 mm bastam?
3. **Margem do HSI16 a 1 Mbps** (item 2): se tiver experiência prática, avise
4. **As três linhas de defesa contra sobretensão bastam?** Barramento de 8,4 V + força contraeletromotriz dos servos
5. **Conectores**: veja abaixo "⚠️ A ordem dos pinos Feetech varia conforme o modelo, e é invertida". Os dois conectores de 2,0 mm `J4`/`J5`
   já foram posicionados no esquemático e no PCB (2026-09-08), mas **a série exata do conector do lado da placa ainda não foi comparada com a peça real**
   — a especificação da Feetech só informa `AMP2.0-3P` na ponta do cabo, e o modelo do lado da placa depende de confirmação do fabricante

## ⚠️ A ordem dos pinos Feetech varia conforme o modelo, e é invertida

**Este é o ponto mais fatal desta placa; as duas especificações do fabricante estão, cada uma, corretas: são dois modelos que realmente são opostos.**

| | Conector | Pino 1 | Pino 2 | Pino 3 |
|---|---|---|---|---|
| **`HD-1910-C001`** (o escolhido neste repositório) | **`AMP2.0-3P`** (2,0 mm) | **Signal/TTL** | Vcc | **GND** |
| `HL-2915-C002` (outro modelo, não copiar) | `AMP-3` | **GND** | Vcc | **Signal/TTL** |
| `J1`/`J2` desta placa (JST EH 2,5 mm, como o HAT oficial) | `B3B-EH-A` | GND | VDD_BUS | DXL_BUS |
| `J4`/`J5` desta placa (2,0 mm, compatível com a Feetech) | `B3B-PH-K-S` | **DXL_BUS** | VDD_BUS | **GND** |

A ordem dos pinos dos quatro conectores foi conferida relendo a netlist do PCB, e todos estão corretos (2026-09-08).
**Os dois passos não se encaixam um no outro (2,0 vs 2,5 mm), o que impede fisicamente a ligação errada; resta apenas a possibilidade de "desenhar errado".**

Origem: Feetech, *Especificação em série HD-1910-C001*, **A/0, 2026-09-07**, páginas 3/7, itens 6-7;
Feetech, *Especificação em série HL-2915-C002*, A/0, 2026-02-27, páginas 4/8, itens 6-7.
Ambas são especificações que o fabricante envia ao cliente para confirmação, e **este repositório, por convenção, não republica o PDF**; solicite à Feetech.

> Este repositório antes só registrava a linha do `HL-2915` e recomendava o `HD-1910` como modelo substituto, **o que era uma armadilha**,
> corrigida em 2026-09-08 conforme a nova especificação A/0.
>
> ⚠️ **O `HL-2915-C002` é um servo de 9–14 V (típico 12 V) e não funciona no barramento 2S desta solução (6,6–8,2 V)**
> — ele só começa a operar a 9 V. Usá-lo como amostra de "convenção de interface Feetech" foi um expediente; não o escolha por ele.
>
> ⚠️ **O passo também não coincide**: `AMP2.0` é 2,0 mm e `JST EH` é 2,5 mm, **não se encaixam um no outro**.
> Felizmente isso acaba sendo uma garantia: impede fisicamente a ligação errada, e resta apenas a possibilidade de "desenhar errado".

## Confirmado / não verificado

**Confirmado (fontes primárias)**

- **HD-1910-C001**, interface `1=Signal/TTL · 2=Vcc · 3=GND`, conector `AMP2.0-3P`, comprimento do cabo 15±0,5 cm
  — Feetech, *Especificação em série HD-1910-C001* A/0 (2026-09-07), páginas 3/7, itens 6-7
- O HD-1910-C001 usa **barramento TTL de três pinos**, **4–8,4 V**, torque de travamento 9/12/15 kg·cm @4,8/6/7,4 V,
  corrente nominal **500/690/900 mA**, corrente de travamento **1,2/1,6/2,0 A**, corrente de repouso 20 mA, encoder magnético de 12 bits,
  redução de 1/320 — mesma especificação, páginas 2/7 e 3/7
- **O motor traz três proteções eletrônicas** (mesma fonte, p. 4/7, 7-11):
  **sobrecorrente** (>0,5 A por 2 s desliga a saída, limiar e duração personalizáveis, **desativada de fábrica**) ·
  **sobretensão** (proteção acima de 10 V ou abaixo de 4 V, faixa personalizável, **desativada de fábrica**) ·
  **sobretemperatura** (>80 ℃ desliga a saída de torque, **ativada de fábrica**).
  **Antes de montar, as duas primeiras devem ser ativadas**: a detecção de anomalias é feita no motor, e o host não precisa fazer polling
- Seis itens de realimentação completos: tensão de entrada / carga / corrente de trabalho / velocidade de trabalho / temperatura de trabalho / posição (mesma fonte, 7-10).
  O runtime oficial realmente os usa: `present_current` é lido a cada tick junto com a posição, tensão e temperatura cerca de uma vez por segundo,
  e **a tensão é o único meio de medir a bateria do robô completo** (`duck-control/src/{bus,model}.rs`)
- O modo de operação padrão de fábrica é o **modo 4, "PD de posição pura (Sim2Real)"**, exatamente alinhado à implantação de aprendizado por reforço (mesma fonte, 7-12)
- Formato do pacote, `Length = N+2`, algoritmo de checksum, leitura síncrona `0x82` — Feetech, *Protocolo de comunicação dos servos SCS* oficial
- Formato do pacote, `Length = N+2`, algoritmo de checksum, leitura síncrona `0x82` — Feetech, *Protocolo de comunicação dos servos SCS* oficial
- STM32G031 sem pino HSE, precisão do HSI16, exigência de desacoplamento `100 nF + 4.7 µF` — ST DS12992 Rev 3
- Conectividade do esquemático — conferida rede por rede por script, sem pinos flutuantes, sem curtos, sem redes de nome duplicado

**Não verificado**

- **A placa inteira nunca foi fabricada em protótipo**, não há placa real
- O contorno de 45 × 22 mm e o espaçamento de 34 mm dos furos M2 na direção X foram inferidos da peça impressa `banana_pcb_locker`, não são dimensões oficiais
- O modelo do MCU é uma sugestão de seleção deste repositório, **não foi obtido por engenharia reversa**: não há como reconstituir qual MCU a placa oficial usa

---

## Registro de revisão

### 2026-09-16 · Oito pontos de revisão externa + autoverificação revelou que o pino de habilitação do X1 não está ligado

A placa já está em produção no LCSC; tudo isto conta como "o que validar quando a primeira placa voltar".

| Comentário da revisão | Conclusão | Base |
|---|---|---|
| ① Ligar o X1 a PC14/OSC32_IN está errado, o PC14 é só um pino LSE | **Não aceito, a revisão está errada** | DS12992 Rev 3 Table 12 (p.36): a função adicional do pino 2 do TSSOP20 é `OSC32_IN, OSC_IN`, ambas; p.60: no modo bypass HSE o oscilador fica desligado e o pino de entrada recebe o relógio externo em nível de GPIO comum. Encapsulamentos pequenos não expõem PF0/PF1, e a entrada de relógio externo HSE foi alocada no pino 2. Que um cristal passivo não pode ser ligado (falta o OSC_OUT) este diretório sempre disse |
| ② Os pinos OCS_Aux / SDO_Aux do U2 flutuando são uma omissão | **Não aceito**, redação do documento uniformizada | DS13510 p.10 / p.48: ambos "ligar a Vdd_IO ou deixar flutuando", com pull-up interno por padrão (PIN_CTRL.OIS_PU_DIS = 0). Deixar flutuando é a prática padrão do manual. Na v2 pode-se deixar uma posição de 0 Ω |
| ③ O `2OE` sempre aberto gera eco; sugere-se desligar a recepção por padrão ao transmitir | O hardware já suporta as duas, o firmware decide | Em 7 de setembro o `2OE` passou a ser a rede `RX_EN` ligada ao PB7; veja o registro abaixo |
| ④ A proteção do ramo lógico não equivale à proteção contra sobretensão da máquina inteira | De acordo, já estava escrito assim | O TVS só trata picos, o PPTC é lento, o LDO só protege o ramo lógico; veja "Revisão comparada com o HAT oficial" |
| ⑤ A corrente de passagem de J1/J2 é um limite de nível de sistema | De acordo, já registrado | Esta placa é o fim da cadeia, conector EH de 3 A |
| ⑥ Os conectores do lado da placa J4/J5 não foram verificados em peça real | **De acordo, deveria ter sido feito antes da primeira placa** | A Feetech só confirma a ponta do cabo AMP2.0-3P; compre alguns conectores e teste com o cabo do servo para ver se o `B3B-PH-K-S` encaixa, o sentido da trava e a ordem dos pinos |
| ⑦ Sugestões de layout (IMU no centro, LDO afastado, pontos de teste, NRST no J3) | Aceitas na v2 | — |
| ⑧ Lista de 8 itens de validação da primeira placa | Aceita, como critério de aceitação | Barramento em alta impedância durante o reset, transmissão/recepção a 1 Mbps de 00/FF/55/AA, estatística de 30 000 comunicações de CRC/timeout/overrun, deriva do zero do IMU com a temperatura, medir a polaridade pino a pino, PC14 não ser acionado com o X1 não montado |

**Autoverificação (2026-09-16)**: conferindo pino a pino pelos segmentos vetoriais do PDF do esquemático, os únicos sem ligação são os quatro que deviam mesmo ficar vazios: U1.3 (PC15), U1.20 (PB3–PB6), U2.10 (OCS_Aux), U2.11 (SDO_Aux). O pino 1 do X1, `Tri-state`, está ligado a 3,3 V (sempre habilitado) (o pequeno círculo vermelho no símbolo é a marca do pino 1 do EasyEDA, não "não conectado"). O "pino 3 (PC15) pode servir de `OSC_EN`" escrito na tabela de ligação **não foi desenhado**: o PC15 está vazio e o X1 não pode ser desligado. Se a primeira placa precisar que o MCU controle o oscilador, ligue um fio PC15 → X1.1 e retire o de 3,3 V, ou não monte o X1 e use o HSI16.

### 2026-09-17 · Teste real da primeira placa que chegou

- 5 placas chegaram; ao energizar, a **saída de 3,3 V está normal** (a cadeia F1 → D1 → U4 está bem).
- **O corpo do J4/J5 (`B3B-PH-K-S`) não combina com o plugue do cabo da Feetech**: o passo de 2,0 mm está certo, mas o formato do plugue não entra no corpo do conector PH. O ponto ⑥ da revisão acertou. Na primeira placa, o corpo do conector foi lixado, mantendo só os pinos, e assim encaixa e funciona. **Na v2 trocar o conector**: ajustar com o cabo Feetech real (perguntar à Feetech o modelo do lado da placa, ou comprar vários conectores horizontais 2,0 mm 3P para testar).
- O relógio X1, NRST, a alimentação do IMU e a continuidade do SWD ainda não foram medidos; o que o firmware precisa fazer e todos os requisitos do controlador principal sobre ele estão em [`software/飞特适配架构.md` §4](../../software/飞特适配架构.md#4-小板固件方案-a--imu_to_dxl).

### 2026-09-22 · O firmware rodou (ST-LINK/V2)

- O firmware está em [`firmware/`](firmware/) (base escrita por um colega, PR #32; a partir da 0.2.0 usa o protocolo Feetech). O ST-LINK/V2 gravou pelos pinos 1/2/3/6 do J3, a verificação de leitura passou, e no IMU, em medição real: WHO_AM_I=0x70, cerca de 121 Hz, aceleração resultante em repouso de 1,007 g, contador de erros 0.
- **Armadilha do chip G0 virgem**: depois da primeira gravação, com reset por software, o programa não roda e para no Bootloader da própria ST (PC=0x1FFF…): o `FLASH_ACR.EMPTY` só é atualizado na energização. **Desligar e religar a energia uma vez** resolve. Apertar o botão de reset (NRST) não reverifica, então não resolve.
- **A v2 deve acrescentar** (faltou na depuração da primeira placa):
  - **Botão de reset + NRST levado ao J3**: fazer junto com o ponto ⑦ da revisão. Quando o programa descontrola e o SWD não conecta, dá para segurar o reset e conectar (under-reset) para recuperar o chip, e também facilita refazer a execução do firmware.
  - **LED de funcionamento**: ligado ao **U1.20** (PB3–PB6, o único GPIO atualmente totalmente livre) + resistor limitador. O firmware acrescenta um heartbeat: piscada lenta quando normal, rápida quando o IMU dá erro, para ver de relance se a placa está rodando mesmo sem depurador.

### 2026-09-07 · Três sugestões de layout/seleção (nenhuma aceita; os motivos ficam registrados)

O revisor olhou os desenhos e fez três sugestões: trocar resistores e capacitores para 0402, colocar o IMU no centro da placa, afastar o LDO do IMU. Cada uma foi medida,
e a **conclusão é manter como está**, mas o motivo de cada uma merece registro, sobretudo o da segunda e o da terceira, que na verdade apontam para a mesma coisa.

**① Trocar resistores e capacitores para 0402: não trocar, e o motivo não é "dá para soldar à mão"**

A outra parte acrescentou "não se dê ao trabalho de soldar você mesmo, junte e mande o LCSC montar, é mais tranquilo". Essa sugestão em si está certa, mas o critério para trocar ou não por 0402
não está aí: a primeira frase do registro de 2026-09-06 neste diretório diz "**o motivo não é que o 0402 não possa ser soldado à mão**"
— a placa tem o LSM6DSV16X em LGA-14 (pad inferior de 3×2,5 mm), que de qualquer modo já exige ar quente.

O motivo real é o **retrabalho na fase de depuração**: o resistor em série `R6` precisa ser testado de 150 Ω a 33 Ω, o pull-up talvez tenha de ser retirado, o desacoplamento talvez precise de reforço.
**Uma primeira versão de placa não validada inevitavelmente será alterada várias vezes**, e isso não muda por mandar o LCSC montar: a montadora só cuida da
primeira vez, e as alterações ainda têm de ser feitas com o seu próprio ferro de solda.

Quanto ao custo: trocar 16 resistores e capacitores por 0402 economiza **12,5 mm², ou seja, 1,3% da área da placa**, e esta placa **usa apenas 36%
da área**, não faltando espaço algum. Os 10k/100nF em 0402 também estão na biblioteca básica do LCSC, e a economia no custo do material vale para os dois lados,
não sendo motivo.

**② IMU no centro da placa · ③ LDO longe do IMU: a direção de ambas está certa, mas a ordem de grandeza é diferente**

Primeiro as medidas:

```
板心 (22.5, 11.0) = 两个 M2 孔的中点（当初就是这么设计的）
U2 (IMU)  离板心 7.47 mm，离两螺钉连线 3.00 mm
U3 (缓冲) 离板心 0.41 mm      ← 最刚的位置被逻辑芯片占了
U2 ↔ U4 (LDO) 中心距 6.37 mm
```

**Ordem de grandeza do calor**: a corrente total da placa é de cerca de 5–10 mA (G031 cerca de 3 mA + LSM6DSV16X cerca de 0,65 mA + buffer cerca de 1 mA
+ pull-ups), a queda no LDO de 8,4 − 3,3 = 5,1 V → **dissipação de cerca de 51 mW** (pior caso com bateria cheia). O SOT-89 sobre pouco cobre tem
θJA de cerca de 100–160 °C/W → elevação na junção de **5–7 °C**; a 6,4 mm de distância, com um plano inteiro de cobre de terra na camada inferior dissipando entre eles, a **diferença de temperatura que chega ao IMU é de cerca de 1–3 °C**. A deriva térmica do zero do giroscópio do LSM6DSV16X é de cerca de 0,05 °/s/°C → **viés de cerca de 0,1 °/s**.
Ao andar, o giroscópio opera na ordem de dezenas de °/s, e dentro do tronco os 15 servos consomem ampères de corrente, com fontes de calor duas a três ordens de grandeza maiores que 51 mW.
**A direção está certa, mas não é o termo dominante.**

**O que realmente vale mexer é o outro lado do ponto ②**: o centro da placa é exatamente o ponto médio dos dois parafusos, a posição menos sujeita a deformação de toda a placa,
e quem está sentado ali agora é o `U3`, que não se importa com a posição. Trocar `U2`/`U3` mata três coelhos de uma cajadada:
o IMU vai de 7,47 → 0,4 mm do centro da placa e, ao mesmo tempo, de 6,37 → **11,3 mm** do LDO (o dobro).

**Por que desta vez ainda não se mexe**: a placa já está com DRC de 0 violações e totalmente roteada; trocar significa reorganizar a região central e rerrotear
SPI×4, 2 interrupções, e o DE/TX/RX/DXL_DATA do buffer. Pela ordem de grandeza estimada acima, o ganho é "mais folga" e não
"resolver um problema conhecido". **Deixar para depois do protótipo, decidindo com dados medidos**: se a medição mostrar o zero do giroscópio derivando claramente com a carga do LDO,
ou ruído de vibração alto, a v2 muda tudo junto.

**Um lembrete mais importante**: a posição e a orientação do IMU **dentro do robô** são fixadas pelo modelo treinado, e importam muito mais do que ele estar 7 mm deslocado na placa.
Isso é determinado por como a placa é instalada no tronco, já verificado e registrado em
[`docs/硬件方案逆向.md`](../../docs/硬件方案逆向.md): `trunk = [+raw_z, +raw_y, −raw_x]`,
rotação de +90° em torno do eixo Y. Mover a posição na placa não afeta isso.

### 2026-09-07 · Reservar uma linha de controle para o `2OE` (R7 + RX_EN)

Alguém sugeriu reservar uma linha até o MCU para "aquele pino EN". Refere-se ao `U3.7 (2OE)`, a habilitação de saída do buffer de recepção,
até então ligada rigidamente a +3V3, com a recepção sempre aberta. **Aceito.**

**Alterações**

| | |
|---|---|
| Desfeita | A ligação direta de `U3.7 (2OE)` a `+3V3` |
| Adicionado | **R7 = 10 kΩ 0603** (`C25804`, a mesma peça de R1/R2/R4/R5, sem novo código de peça) |
| Nova rede | **`RX_EN` = `U1.1 (PB7/PB8)` · `U3.7 (2OE)` · `R7.1`** |
| Mantida | `R7.2 → +3V3` (pull-up) |

**Por que o U1.1**: os pinos livres do U1 são o 1 (PB7/PB8), o 3 (PC15) e o 20 (PB3/PB4/PB5/PB6).
O pino 3 é o PC15, compartilhado com o LSE no G0 e com capacidade de acionamento limitada; o pino 20 agrega quatro portas e é o mais flexível, então fica reservado.
O pino 1 é um GPIO comum, e está na fileira inferior junto com `DE`/`MCU_TX`/`MCU_RX`, passando pelo mesmo canal de roteamento.

**O comportamento padrão não muda em nada**: durante o reset do MCU o PB7 fica em alta impedância, o R7 puxa o `2OE` para alto, e o buffer de recepção continua aberto.
Sem escrever uma linha de firmware, fica idêntico ao que era antes da alteração. É uma **reserva**: para eliminar o eco durante a transmissão, basta configurar o PB7 como saída push-pull em nível baixo,
e ao mesmo tempo ativar o pull-up interno no pino `MCU_RX`, evitando que o RX fique flutuando com o `2Y` em alta impedância
— que é exatamente a preocupação que antes derrubou a opção de "`2OE` ligado ao DE", resolvida com um GPIO independente + pull-up interno.

**Reverificação após a alteração**: componentes 26→27, redes 23→24, `RX_EN` = 3 pinos, em `+3V3` o `U3.7` foi retirado
e o `R7.2` assumiu seu lugar. DRC do LCSC com **0 violações**, menor distância entre redes distintas **0,160 mm** (antes da alteração 0,153),
e os demais itens iguais aos da primeira verificação de 2026-09-07.

**Correção de passagem**: a lista de "pinos flutuantes" do `imu_to_dxl-接线表.md` deste diretório listava por engano os pinos 2, 16 e 17
(que ligam respectivamente a `OSC_IN`, `UART_TX` e `UART_RX`), já corrigida. Agora os únicos realmente livres são o pino 3 e o pino 20.

### 2026-09-07 · Três dúvidas sobre o buffer U3 (duas não aceitas, uma muda a forma de desenhar)

O revisor consultou o manual do SN74LVC2G241 e levantou três pontos sobre a ligação do U3. Cada um foi verificado; a netlist foi
lida diretamente do PCB (a rede do PCB é exportada do esquemático, equivalente às conexões do esquemático):

```
U3 脚1  1OE#  → DE          脚5  2A   → DXL_DATA
U3 脚2  1A    → MCU_TX      脚6  1Y   → DXL_DATA
U3 脚3  2Y    → MCU_RX      脚7  2OE  → 3.3V
U3 脚4  GND   → GND         脚8  VCC  → 3.3V

DXL_DATA  4 脚：R5.2 R6.1 U3.5 U3.6     ← 不含 U3.7
DE        3 脚：R4.2 U1.8 U3.1
MCU_TX    2 脚：U1.9 U3.2
MCU_RX    2 脚：U1.10 U3.3
```

**① "A é entrada e Y é saída, então o pino 3 deveria ir ao DXL_DATA e o pino 5 ao MCU_RX": a premissa está certa, a conclusão é inversa**

O texto original do manual está correto: o dispositivo é um driver de linha de 1 bit com duas habilitações independentes, e a direção dos dados é sempre **A → Y**.
Mas trocar as ligações com base nisso estaria errado:

- `MCU_RX` é a rede **que deve ser acionada pelo buffer** (entrando no pino de recepção do MCU), e precisa estar no **pino de saída Y**
  → agora está no pino 3 (2Y) ✅
- `DXL_DATA` é a rede **que deve ser lida** (o que os servos enviam no barramento), e precisa estar no **pino de entrada A**
  → agora está no pino 5 (2A) ✅

Se fossem trocadas como sugerido, o canal 2 viraria `MCU_RX(2A) → DXL_DATA(2Y)`, equivalente a enviar dados do pino de recepção do MCU
para o barramento: o RX é um pino de entrada e não emite nível; além disso, `1Y` e `2Y` acionariam `DXL_DATA` ao mesmo tempo,
com dois drivers brigando diretamente.

A divisão atual é a ligação half-duplex padrão: **canal 1 = transmitir** (`MCU_TX → barramento`, controlado pelo DE),
**canal 2 = receber** (`barramento → MCU_RX`, sempre aberto). **Não aceito.**

**② "O 2OE deveria ter um resistor de pull-up em série, senão o pull-up é forte demais": eletricamente não se sustenta, mas a forma de desenhar deve mudar**

O `2OE` é uma entrada lógica CMOS, com corrente de entrada da ordem de ±1 µA, e ligá-lo diretamente ao VCC é o uso padrão do manual.
Um resistor em série não traz benefício algum e ainda introduz acoplamento de interferência em uma entrada de alta impedância.

A preocupação com "pull-up forte demais" só se sustentaria se esse nó fosse uma **linha de dados**, e não é:
a rede `DXL_DATA` tem apenas 4 pinos (`R5.2` `R6.1` `U3.5` `U3.6`), **sem incluir o `U3.7`**.
O pull-up do barramento é `R5 = 10 kΩ`, valor também adequado.

**Mas o aviso tem valor**: o mal-entendido surgiu porque, no desenho, o "símbolo de 3.3V do 2OE" está bem em cima
do nó `DXL_DATA`, dando visualmente a impressão de estarem ligados. **Foi listado como item a alterar: afastar esse símbolo de 3.3V e
rotear contornando**, eliminando a ambiguidade. É questão de desenho, sem alterar a conexão elétrica.

**③ "Para ser condutor por padrão, o 1OE# deveria ir a nível baixo": não conduzir por padrão é intencional, não aceito**

O `1OE#` é ativo em baixo, e o `R4` o **puxa para alto**, o que significa que o canal 1 (`MCU_TX → barramento`)
**fica desligado por padrão**. Isso é deliberado:

- Desde a energização até o firmware começar a rodar, e durante travamento / reset do MCU, esta placa **jamais pode acionar o barramento**.
  O barramento é compartilhado pelos 15 servos, e, se for forçado a um nível, toda a comunicação da cadeia é perdida.
- Se o `1OE#` fosse mantido sempre em baixo, o `MCU_TX` despejaria continuamente no barramento, o nível alto de repouso prenderia o barramento,
  os servos jamais conseguiriam responder aos pacotes — o half-duplex não se sustentaria.

Para transmitir, o MCU puxa o `DE` para **baixo**. O DE (Driver Enable) em hardware do USART do STM32 suporta essa polaridade,
com `CR3.DEP = 1` resolvendo em um único bit, sem inversor externo.

> Isto tem a mesma direção do item ① de 2026-09-06 (feedback da comunidade pedindo pull-up no DE):
> a inclusão do R4 daquela vez foi justamente para garantir "não acionar o barramento por padrão", e aqui não se pode inverter.

**Pendência**: no esquemático, afastar o símbolo de 3.3V do `U3.7` para evitar a ligação visual com o nó `DXL_DATA`.

### 2026-09-06 · Feedback da comunidade (três pontos, dois já alterados)

**① O pino DE precisa de pull-up: já alterado, foi uma omissão**

O `U3.1 (1OE#)` é a habilitação de transmissão ativa em baixo. Na energização e durante o reset, o **PA1 do MCU fica em alta impedância** e este pino flutua;
**se flutuar para nível baixo, o buffer de transmissão abre, e esta placa despeja dados no barramento, colidindo com todos os 15 servos.**

Foi adicionado o **R4 10 kΩ, DE → 3,3 V**.

O R1 de pull-up adicionado antes no `SPI_CS` segue exatamente a mesma lógica, **mas não foi estendido ao DE**; foi de fato um descuido.

**② Os dois `DXL_DATA` devem ser ligados e ter pull-up: já alterado**

Eletricamente já eram originalmente a mesma rede (`U3.5 2A` / `U3.6 1Y` / `J1.3` / `J2.3`),
mas no desenho estavam como dois traços curtos, cada um com seu rótulo, **parecendo desligados**, de modo que o desenho era pouco intuitivo. Foi alterado para um nó de conexão explícito.

O pull-up também procede: quando o barramento está ocioso e ninguém aciona, a entrada CMOS `2A` flutuando tem corrente de passagem,
e a UART também pode tomar ruído por bit de início: esta placa deve definir por conta própria o nível de repouso, sem depender do host.
Foi adicionado o **R5 10 kΩ, DXL_DATA → 3,3 V**.

**③ Manter apenas um conector 3P: não aceito, motivos no item 3 acima**

A sugestão vale quando esta placa é com certeza um nó folha; mas se for o primeiro nó da cadeia, são necessários dois conectores,
e a topologia oficial não pode ser confirmada. Mantidos dois, com a sobrecorrente resolvida pela largura do cobre.

Reverificação após a alteração: `DE` passou de 2 para 3 pinos, `DXL_DATA` de 4 para 5 pinos,
sem pinos flutuantes, sem curtos e sem redes de nome duplicado em todo o desenho.

### 2026-09-06 · Revisão comparada com o HAT oficial (duas proteções acrescentadas)

O `dynamixel.kicad_sch` de [`pollen-robotics/elec_RPI_Robot_HAT`](https://github.com/pollen-robotics/elec_RPI_Robot_HAT)
foi lido componente por componente e comparado com esta placa.

**Circuito half-duplex TTL do HAT**

```
                        ┌── R31 10k ── +3V3
   IO_15 (RX) ◄── U6 ───┘                    ┌── R32 10k ── +3V3
              (1G125, OE# 低有效)             │
                        ▲                     ▼
   IO_14 (TX) ──┬── U7 ──────────────────────► ● ── R33 150R ──┬── TH1 100R ── J13.3 / J14.3
                │  (1G126, OE 高有效)                       D4 5V1
                └── R27 10k ── Q1 (PNP) ── Dynamixel_dir        ⏚
```

**Comparação item a item**

| | HAT oficial | Esta placa | |
|---|---|---|---|
| Buffer de transmissão/recepção | Dois: 1G125 + 1G126 | Um: SN74LVC2G241 | Mesma lógica |
| Controle de direção | Derivado automaticamente do TX (Q1 + RC) | **DE em hardware do USART2** | Esta placa é melhor, veja abaixo |
| Pull-up do barramento | R32 10 kΩ | R5 10 kΩ | Igual |
| **Resistor em série da linha de dados** | R33 150 Ω + TH1 100 Ω | ❌ Nenhum → **R6 150 Ω acrescentado** | |
| **Grampo da linha de dados** | D4 5V1 | ❌ Nenhum → **D2 5V1 acrescentado (mesmo código `C151348`)** | |
| Proteção da alimentação | Nenhuma (`+BATT` direto ao conector) | F1 PPTC + D1 TVS | Esta placa é melhor |

**Por que o controle de direção desta placa é melhor**: o HAT deriva a direção do TX e volta ao estado de recepção por temporização RC,
**e se liberar cedo demais trunca o último bit de parada**: é um defeito comum das soluções derivadas do TX.
Esta placa usa o DE em hardware do `USART2`, com `DEAT`/`DEDT` ajustáveis com precisão de 1/16 de bit, sem esse problema.

**Topologia da linha de dados após a complementação** (isomórfica à do HAT):

```
U3.6 (1Y) ──┬─ DXL_DATA ─┬── R5 10k → 3.3V
U3.5 (2A) ──┘            │
                         └── R6 150Ω ──┬── DXL_BUS ── J1.3 / J2.3
                                       │
                                    D2 5V1
                                       ⏚
```

O pull-up fica no **lado do buffer** (R5 antes do R6), na mesma posição do R32 do HAT:
assim, mesmo sem o barramento conectado, a entrada CMOS `2A` tem um nível definido.

Esta placa coloca em série só 150 Ω (o HAT usa 250 Ω), porque o lado do HAT já tem 250 Ω,
e colocar demais no barramento retardaria as bordas.

Verificação: `DXL_DATA` (lado do buffer) 4 pinos, `DXL_BUS` (lado do conector) 4 pinos,
sem pinos flutuantes, sem curtos e sem redes de nome duplicado em todo o desenho.

**Correção de um erro anterior deste documento**: o `TH1 100R termistor` do HAT está em série na **linha de dados**,
não na linha de alimentação. De `+BATT` ao pino 2 do J13/J14 é **trilha grossa direta, sem limitação de corrente**.

### 2026-09-06 · Encapsulamento de resistores e capacitores unificado em 0603

Todos os resistores e capacitores 0402 foram alterados para **0603**, e o D2 de SOD-323 para **SOD-123**.

O motivo não é "o 0402 não pode ser soldado à mão": a placa tem o **LSM6DSV16X em LGA-14** (pad inferior de 3 × 2,5 mm),
que de qualquer modo exige ar quente ou refluxo, e com essas ferramentas o 0402 também não é difícil.

**O motivo real é o retrabalho na fase de depuração**: o resistor em série precisa ser testado de 150 Ω a 33 Ω, algum pull-up precisa ser retirado,
um desacoplamento precisa ser reforçado — em 0603 isso se faz em segundos com um ferro de solda, enquanto no 0402 é preciso montar o ar quente de novo.
Uma placa sem protótipo validado inevitavelmente será alterada várias vezes na primeira versão.

O custo é pequeno: 11 componentes ocupam cerca de 16 mm² a mais, menos de 2% da área da placa de 45 × 22 mm.
Além disso, estes valores em 0603 estão **todos na biblioteca básica do LCSC** (10 kΩ `C25804` · 150 Ω `C22808` ·
33 Ω `C23140` · 100 nF `C14663`), economizando até a taxa de carga de peças em relação à biblioteca estendida do 0402 original.

> O C6 (10 µF/25 V) permanece em 0805; o C5 (4,7 µF) e o C7 (10 µF) já eram 0603.

Ao trocar o encapsulamento, recolocando na mesma coordenada e ângulo, a medição mostrou **coordenadas dos pinos inalteradas e nenhuma ligação rompida**.
Uma armadilha: a ordem dos pinos do símbolo `SOD-123` é oposta à do `SOD-323`, e depois da troca a **polaridade do D2 ficou invertida**
(o cátodo foi parar no lado do terra, e em operação normal grampearia o barramento em 0,7 V), já conferida pelo nome dos pinos e corrigida.
O parâmetro `rotation` de `create()` tem o sinal oposto nesta API — passar 90 grava 270.
