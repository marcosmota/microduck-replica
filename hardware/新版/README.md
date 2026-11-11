# Hardware da nova versão (2026-10)

> 🌐 Tradução não oficial para pt-BR do [original](https://github.com/fanhao375/microduck-replica). Em caso de divergência, vale o original. Licença: CC BY-NC-SA 4.0 (texto alterado: traduzido do chinês/inglês).

Três placas, todas projetos do EasyEDA (LCSC) edição Pro (`.eprj2`, com esquemático e PCB no mesmo arquivo). Uma pasta por placa:

| Pasta | O que é | Estado (2026-10-20) |
|---|---|---|
| [`imu_to_dxl_1V0A/`](imu_to_dxl_1V0A/) | Placa IMU de barramento v2. A versão antiga e o firmware estão em [`../imu_to_dxl/`](../imu_to_dxl/) | Roteamento concluído; o DRC só acusa 12 definições duplicadas de furos de fixação (não afeta a fabricação); ainda não chegou para teste |
| [`wire_hub_1V0A/`](wire_hub_1V0A/) | HUB do barramento de servos: entrada da bateria + derivação do barramento | Protótipo ainda não feito; altura do CN1 precisa ser trocada |
| [`POGO_PIN_1V0A/`](POGO_PIN_1V0A/) | Placa de conexão da bateria: dois bornes banana + pads de solda de fio | Furos, pads e trilhas ajustados conforme a placa de referência; protótipo ainda não feito |

Em cada pasta:

- `<nome da placa>.eprj2`: o projeto
- `<nome da placa>-PCB.step`: modelo 3D da placa inteira, para a equipe de mecânica verificar a estrutura
- `<nome da placa>-原理图.pdf`: por enquanto só existe para a placa IMU

Gerber, BOM e tabela de ligação ainda não foram gerados; para ter uma ideia, veja a versão antiga em [`../imu_to_dxl/`](../imu_to_dxl/).

## imu_to_dxl_1V0A: placa IMU de barramento v2

A função é a mesma da versão antiga: um LSM6DSV16X ligado a um STM32G031, e o MCU se apresenta como um escravo no barramento, usando o mesmo barramento half-duplex de fio único dos servos. Protocolo e firmware em [`../imu_to_dxl/`](../imu_to_dxl/). O circuito segue o da versão antiga, com novo layout e conectores trocados.

| Posição | Modelo | Função |
|---|---|---|
| U1 | STM32G031F8P6 | MCU |
| U2 | LSM6DSV16XTR | IMU |
| U3 | SN74LVC2G241DCUR | Transceptor half-duplex do barramento |
| U4 | HT7533-1 | LDO de 3,3 V |
| F1 / D1 | MF-NSMF020X-2 / SMF12A | Fusível rearmável / TVS |
| D2 | BZT52C5V1 | Grampo de 5,1 V na linha de dados do barramento |
| R6 | 0 Ω (C21189) | Resistor em série do barramento; na primeira placa era 150 Ω |
| J1 | A2007WR-3P | Barramento de servos, 2,0 mm 3P, ângulo reto com pinos. 1 = DXL_BUS, 2 = VDD_BUS, 3 = GND |
| J2 | BM03B-ACHSS-GAN-ETF | ACH de 1,2 mm 3P, montagem horizontal SMD. 1 = GND, 2 = VDD_BUS, 3 = DXL_BUS |
| J3 | PZ254V-11-06P | Porta de depuração 2,54 mm 6P: 1 = GND, 2 = SWDCLK, 3 = SWDIO, 4 = UART_RX, 5 = UART_TX, 6 = 3.3V |

Há ainda alguns resistores e capacitores 0402/0603, totalizando 24 componentes.

Comparação com a versão antiga:

- O conector de servo da versão antiga, `B3B-PH-K-S`, não aceitava o plugue do cabo da Feetech (registrado no README da versão antiga); esta versão usa o A2007WR-3P
- O cristal externo X1 foi removido (o firmware já usava o HSI16 interno), então o firmware 0.2.0 pode ser gravado sem alterações
- R6 passou de 150 Ω para 0 Ω: cada servo no barramento tem pull-up, e, combinados em paralelo, um resistor em série elevaria o nível baixo. Ao receber a placa, meça primeiro o nível baixo do barramento durante a resposta
- U2 foi girado 180° e o formato da placa mudou; depois de instalada no pato, o mapeamento dos eixos do IMU precisa ser redefinido. O firmware emite a atitude bruta do chip sem rotação, então só a rotação de montagem no lado do computador host precisa mudar, com aceitação em três posturas estáticas
- Dos itens pendentes da v2 registrados no README da versão antiga, **o botão de reset e o LED de funcionamento não existem nesta versão**
- O J3 não tem modelo 3D no PCB, então o STEP exportado está sem esse conector de pinos

## wire_hub_1V0A: HUB do barramento

Não há componentes ativos, apenas conectores. Todos os conectores usam somente as três redes `Battery`, `DXL_BUS` e `GND`.

| Posição | Modelo | Especificação |
|---|---|---|
| J1 | HC-1.2-3PWT | 1,2 mm 3P horizontal SMD (compatível com Pico-EZmate), 1,5 A |
| J2 | BM03B-ACHSS-GAN-ETF | ACH de 1,2 mm 3P horizontal SMD, 2 A |
| J3, J4 | A2007WR-3P | 2,0 mm 3P, ângulo reto com pinos, 4 A |
| J5, J6 | A2007WV-3P | 2,0 mm 3P, vertical com pinos, 4 A |
| CN1 | DB125-2.54-2P-GN-S | Entrada da bateria, borne de parafuso verde, 7 A, **altura de 8,4 mm sobre a placa** |

A ordem dos pinos coincide com a da placa IMU: J1 e J2 do HUB e o J2 da placa IMU têm 1 = GND, 2 = alimentação, 3 = DXL_BUS, e ligam-se com cabos de mesmo sentido; J3–J6 do HUB e o J1 da placa IMU têm 1 = DXL_BUS, 2 = alimentação, 3 = GND.

Pendências:

- O CN1 é alto demais e interfere com a estrutura; naquela posição só cabe algo com cerca de 5 mm de altura. Candidatos: um borne de pressão SMD de passo 4 mm (do tipo WAGO 2060, com 4,5 mm de altura) ou não colocar borne e soldar o fio direto

## POGO_PIN_1V0A: placa de conexão da bateria

O contorno e os furos foram adaptados da placa de referência banana_pcb (projeto KiCad, só com Gerber). A placa mede 39,19 × 6,79 mm.

| Posição | Rede | Dimensões |
|---|---|---|
| U2 (esquerda) | GND | Furo do borne banana Ø1,7 mm metalizado, pad Ø3,2 mm |
| U1 (direita) | Battery | Igual; distância entre centros dos dois furos de 33,8 mm |
| Dois pads centrais | Esquerdo GND, direito Battery | Pads SMD na camada superior de 2,4 × 4,4 mm, espaçamento de 4,0 mm, sem furo, para solda de fio |

A posição dos dois pads também segue a placa de referência: o pad de GND fica a 10,98 mm do furo esquerdo e o de Battery a 14,98 mm do furo esquerdo, ambos 0,59 mm abaixo da linha que liga os dois furos. O espaçamento de 4,0 mm também permite colar diretamente um borne de pressão de passo 4 mm.

O projeto se chama POGO_PIN, mas a placa já não tem pogo pins; são apenas pads de solda de fio.
