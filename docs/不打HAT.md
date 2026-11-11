# Sem HAT · substituindo por módulos de ¥70 com fios soldados

> 🌐 Tradução não oficial para pt-BR do [original](https://github.com/fanhao375/microduck-replica). Em caso de divergência, vale o original. Licença: CC BY-NC-SA 4.0 (texto alterado: traduzido do chinês/inglês).

**Português (pt-BR)**

> O HAT oficial é uma placa de 4 camadas, com montagem nos dois lados e 110 componentes: fabricar e montar custa mais de mil yuans de uma vez, e **muita gente acha que não compensa**.
> Esta é a alternativa: um módulo step-down + uma placa adaptadora half-duplex, e o resto com fios soldados. **Em funções, perde-se apenas o áudio integrado e o sinal de detecção de desligamento.**

Leia primeiro a [Introdução ao hardware](硬件入门.md) para saber o que o HAT faz originalmente, e depois esta página para saber como dispensá-lo.

[![Diagrama de ligação sem HAT](../assets/hw/不打HAT-接线图.png)](不打HAT-接线图.html)

> **O arquivo-fonte da imagem é o [`不打HAT-接线图.html`](不打HAT-接线图.html)**: SVG embutido; depois de baixado e aberto no navegador, dá para dar zoom, selecionar texto e imprimir em A3.
> Para trocar módulos ou pinos, altere-o diretamente. O PNG acima é apenas uma pré-visualização no GitHub.

**Objetivo**: o controlador toma energia da bateria, comunica-se com o barramento dos servos por meio da placa adaptadora e o ToF é ligado ao I²C: **sem fabricar a placa HAT**, com módulos de ¥70 no lugar de mais de mil yuans de fabricação.

> ⚠️ **Este capítulo é um plano, ainda não foi executado.** A parte de ligação vem das folhas de especificação e do esquemático oficial; a parte de software vem do código-fonte do `setup-board.sh` oficial.
> Ao executar, se algum passo estiver errado, corrija aqui. Depois de executar, limpe a seção "Não feito, a validar".

## O que comprar

Os links do Taobao expiram, **mas os termos de busca e os números a conferir continuam valendo**. Em cada linha, busque pelo "termo de busca" e escolha pelo "Conferir antes de comprar". Quando encontrar um adequado, cole o link na [电控采购清单](电控采购清单.md).

| Item | Termo de busca | Conferir antes de comprar | Preço aprox. |
|---|---|---|---|
| **Módulo step-down** | 「UBEC 5V 3A 2S」 | **Limite inferior de entrada ≤ 6 V** (muitos anunciados como 2–6S na verdade partem de 7 V, e com a 2S quase descarregada ele sai da regulação primeiro) · saída ≥ 2 A contínuos · step-down síncrono, **não compre LM2596** | ¥10–15 |
| **Placa adaptadora half-duplex** | 「串口总线舵机驱动板 ST SC」 | Versão TTL ou USB, tanto faz. **A faixa de alimentação lógica inclui 3.3 V** (algumas só aceitam 5 V, e então não podem ser ligadas ao controlador) · a alimentação do lado dos servos precisa aguentar 8.4 V. **Se você já tem uma FE-URT-2, ela é a versão USB e dá para não comprar por enquanto** (veja o passo 4) | ¥22 |
| Barra de pinos 2×20 | 「树莓派 彩色排针 2x20」 | 2.54 mm · vertical · coloridos facilitam contar os pinos | ¥1 |
| microSD de alta durabilidade ×2 | 「SanDisk 高耐久 64G」ou 「海康 PLUS 64G」 | **Vendido e entregue pela JD** · marcado High Endurance / para vigilância · **não** os cartões comuns tipo Ultra / EVO | ¥90×2 |
| Leitor de cartão | 「USB3.0 读卡器 TF」 | USB 3.0 | ¥15 |
| Fio de silicone | 「18AWG 硅胶线 红黑」 | 18 ou 20 AWG · isolamento de silicone (macio, resistente à temperatura) · 1 m de vermelho e 1 m de preto | ¥10 |
| Porta-fusível + fusível | 「5x20 保险丝座 带线」+「5A 保险丝 5x20」 | Porta-fusível com fios economiza solda · fusível rápido de 5 A | ¥5 |
| Interruptor de energia | 「船型开关 KCD1」 | Nominal ≥ 6 A · unipolar de uma via basta | ¥3 |
| Cabos com terminal PH2.0 3P ×5 | 「PH2.0 3P 端子线 单头」 | Passo de 2.0 mm · 3 pinos · terminal em uma ponta com cabo (a outra ponta com fio solto) | ¥5 |
| Fios dupont | 「杜邦线 母对母 20cm」 | Fêmea-fêmea · 20 cm · uma fileira de 40 fios | ¥5 |
| Eletrolítico de 470 µF | 「470uF 16V 电解电容」 | 16 V ou mais · 105 ℃ | ¥1 |
| *(Opcional) Placa de som USB* | 「USB 声卡 免驱 3.5mm」 | **Sem driver** (o Linux reconhece o padrão UAC) · com entrada de microfone | ¥15 |
| *(Opcional) Alto-falante pequeno* | 「3W 4欧 小喇叭」 | 4 Ω 3 W · para ligar à saída de 3.5 mm da placa de som é preciso passar por um amplificador; ou compre direto uma caixinha USB com amplificador | ¥5 |

Sem o HAT, perde-se: o **sinal de detecção de desligamento** (LM5050-1 IO_06), o BMI088 integrado (o software oficial já não o usa) e o **áudio integrado**; o áudio pode ser recuperado com uma placa de som USB, veja o passo 8.
Sem sinal de desligamento + microSD = corte brusco de energia que danifica o cartão, **desligue primeiro por software e só depois corte a energia**.

## Passo 1 · Ligar o controlador, sem conectar mais nada

> ⚠️ **A Zero 3W comprada depois de 2024-11 é a versão V1.12 (módulo WiFi AIC8800), e a imagem oficial do Armbian gravada diretamente não liga**:
> os LEDs piscam duas vezes, nunca entra no sistema e não aparece na rede local. A causa é que o U-Boot mainline que o Armbian usa para esta placa
> (firmware de inicialização da memória é o v1.21 de 2024-01) não consegue inicializar a placa nova; basta trocar pelo bootloader da própria Radxa. Se o sistema oficial da Radxa liga e o Armbian não, é isso.
> Levamos dois dias e nove cartões gravados para descobrir, veja o [registro de armadilhas](../踩坑记录.md). O script de criação de cartão abaixo já trata esse passo junto com as demais armadilhas.

**Imagem**: Armbian **26.2.1 · Debian 13 (trixie) · minimal · kernel vendor 6.1.115**, a versão indicada na documentação oficial da Pollen.
Os espelhos na China agora só têm a 26.8.1; a 26.2.1 está aqui (376 MB):

```
https://fi.mirror.armbian.de/archive/radxa-zero3/archive/Armbian_26.2.1_Radxa-zero3_trixie_vendor_6.1.115_minimal.img.xz
```

Não escolha `current_6.18` (kernel mainline, que o runtime oficial não reconhece) e não escolha a versão desktop.

**1a · Criar o cartão** (WiFi, senha, bootloader e porta serial embutidos na imagem; depois de gravar, basta inserir e dar ssh)

Copie os três arquivos de [`tools/radxa/`](../tools/radxa/) para qualquer diretório local; o computador precisa ter WSL2 (se não tiver, execute `wsl --install` no PowerShell como administrador e reinicie uma vez):

1. Abra o `card.conf` no Bloco de Notas e altere o nome e a senha do WiFi. O usuário e a senha padrão são `duck` / `duck1234`. Se precisar instalar o software oficial dentro da China, preencha `HTTP_PROXY=` com o endereço do proxy do computador (por exemplo, com o Clash em "permitir LAN", preencha `http://IP do computador:7890`).
2. Clique com o botão direito em `1-做卡.ps1` → "Executar com o PowerShell" e escolha o `.img.xz` baixado. Ele descompacta no WSL, troca o bootloader, grava a configuração e em um minuto gera um `xxx-鸭子卡.img` (2.2 GB).
3. Solde antes a barra de pinos: primeiro dois pinos em diagonal, vire para ver se ficou reta, e depois solde os outros 38.

**1b · Gravar o cartão** (Rufus portátil, 2 MB, sem instalação)

1. Baixe `rufus-x.xxp.exe` em [rufus.ie](https://rufus.ie/zh/). Conecte o leitor de cartão ao computador, **conecte só esse dispositivo do tipo pendrive** e dê duplo clique no Rufus. Quando perguntar se quer verificar atualizações pela internet, clique em **Não**:

   <img src="../assets/flash/烧卡-1-Rufus问更新-点否.png" width="420">

2. Em "Dispositivo", o leitor é selecionado automaticamente. Clique em "**Selecionar**" e escolha o `xxx-鸭子卡.img` gerado acima:

   <img src="../assets/flash/烧卡-2-设备自动选中-点选择.png" width="380">

3. Clique em "**Iniciar**"; quando aparecer "os dados do cartão serão apagados", confirme; a gravação leva cerca de 2 minutos:

   <img src="../assets/flash/烧卡-3-正在写入.png" width="380">

4. Quando a barra de status voltar a "Pronto", terminou; feche o Rufus e retire o cartão. **Durante o processo o Windows vai perguntar várias vezes "deseja formatar o disco"; clique em cancelar todas as vezes**, ele não reconhece partições Linux, é normal.

   <img src="../assets/flash/烧卡-4-完成-准备就绪.png" width="380">

**1c · Ligar, achar o IP, ssh**

Insira o cartão na Radxa e ligue um carregador de 5V (um de 20W da Apple serve, não use carregador rápido da Huawei) na **porta `5V IN / USB_OTG`** (serigrafia no verso da placa, do lado do slot de cartão TF).
Espere de 2 a 3 minutos e ela se conecta sozinha ao WiFi. Observe o LED: **pisca duas vezes por alguns segundos → pisca duas vezes por dois minutos → fica aceso** significa que subiu; se ficar piscando duas vezes sem mudar, não subiu.
No painel do roteador, procure um dispositivo chamado `radxa-zero3` para obter o IP; o Windows já vem com ssh:

```
ssh duck@那个IP        # 密码就是 card.conf 里那个
```

**Uma vantagem a mais: este cabo USB também é uma porta serial.** Ligue a porta `5V IN` da placa a uma porta USB do computador (o computador a alimenta); no Gerenciador de Dispositivos aparece um "Dispositivo serial USB (COMx)",
e abrindo um terminal serial a 115200 aparece o prompt de login; a senha do root está em `card.conf`. Se o WiFi não conecta ou o ssh não entra, é por aqui que se entra para ver.

Só prossiga depois de ligar e conseguir dar ssh. **Não misture problemas de "não liga" com problemas de ligação.**

## Passo 2 · Fazer o `/dev/ttyS2` aparecer

O script de criação de cartão já deixou tudo configurado: o overlay `uart2-m0` foi carregado no extlinux, o console do kernel foi para o tty1, e `serial-getty@ttyS2` e `ttyFIQ0` foram mascarados. Entre por ssh e confirme:

```bash
ls -l /dev/ttyS2            # 有
sudo fuser -v /dev/ttyS2    # 没人占
cat /proc/cmdline           # 里面没有 console=ttyS2 / ttyFIQ0
```

O `setup-board.sh` oficial altera `overlay_prefix` / `overlays` / `console` em `/boot/armbianEnv.txt`, o que não tem efeito neste cartão (o U-Boot da Radxa usa extlinux e não lê boot.scr),
mas as verificações dele passam do mesmo jeito, então pode rodar normalmente. **Se um `apt upgrade` atualizar o pacote do kernel, o nome de arquivo no extlinux.conf precisa ser ajustado**; antes de atualizar, veja `/boot/extlinux/extlinux.conf`.

Da mesma forma, para adicionar overlays no futuro (por exemplo o da câmera `rk3568-radxa-zero3-rpi-camera-v2.dtbo`, que o `setup-board.sh` espelha com esse nome), acrescente o caminho no fim da linha `fdtoverlays` do extlinux.conf e reinicie. Sem câmera conectada, o `mediad` reinicia continuamente, e os demais itens do `robotctl health` não são afetados; é normal.

## Passo 3 · Alimentação, meça a cada trecho ligado

```
NP-F550 ──► interruptor ──► fusível 5A ──┬──► módulo step-down ──► 5 V ──► Radxa pin 2 (ou 4)
                                 │        │
                                 │        └── 470 µF em paralelo na entrada
                                 │
                                 └──► VDD do barramento dos servos (tensão direta, 18 AWG)
Terra comum: negativo da bateria · terra de saída do step-down · Radxa pin 6 · GND do barramento dos servos
```

```
1  Ligar interruptor e fusível, medir: com o interruptor fechado deve haver continuidade nas duas pontas do fusível
2  Ligar o módulo step-down, ainda sem a Radxa, medir a saída: 5.0 V ±0.2, polaridade correta
3  ⚠️ Confirmar a polaridade antes de ligar a Radxa — os pin 2/4 não têm proteção, invertendo queima a placa
4  Tirar o USB-C da Radxa, acendê-la pelos 5 V do pin 2, o ssh deve entrar
5  Só então ligar o VDD do barramento dos servos
```

## Passo 4 · Ligar o barramento dos servos ao controlador

```
Radxa 40 pinos                  Placa adaptadora, lado MCU
pin 1   3.3 V    ──────────►   VCC       ⚠️ só 3.3 V, com 5 V o TX dela danifica o RX do controlador
pin 8   UART2_TX ──────────►   RX        ← cruzado
pin 10  UART2_RX ◄──────────   TX        ← cruzado
pin 6   GND      ──────────►   GND

Placa adaptadora, lado dos servos
DATA  ──► S do barramento dos servos
V     ◄── VDD do barramento dos servos (tensão direta da bateria, ligada no passo 3)
G     ──► terra comum
```

**Placa adaptadora versão USB**: não é preciso ligar os pin 1/8/10; conecte à porta USB da Radxa, a serial passa a ser `/dev/ttyUSB0` ou `/dev/ttyACM0`,
e depois altere `port` no `robotd.toml` para esse nome. É prático, mas o overlay uart2-m0 do passo 2 deixa de ser usado (o agetty continua mascarado, sem problema).

> 💡 **A placa de depuração Feetech FE-URT-2 é uma placa adaptadora versão USB**: depois de configurar os IDs, basta mudá-la para o robô: USB-C na porta host da Radxa, porta TTL com S no barramento e G no terra do barramento.
> **A tensão direta da bateria vai para o barramento dos servos, sem passar pelo terminal V1 da URT-2**: o V1 pode estar ligado direto ao 5 V do USB, e injetar 7.4 V ali retornaria para a Radxa.
> O DIP switch da placa é o nível do sinal (3.3 / 5 V), coloque em 5 V. A placa adaptadora pode deixar de ser comprada por enquanto.
> Primeiro verifique se a Radxa reconhece o CH343: veja em `dmesg | tail` se existe `ttyACM0`; se não, o site da WCH tem o código-fonte do driver para Linux.
> Ela é a única placa de depuração, e depois de instalada no robô não dá para configurar servos no computador: **configure os IDs dos 15 servos antes de mudá-la**.

**Como o barramento se ramifica**: cabeça, perna esquerda e perna direita se dividem em três ramais no tronco. O barramento de fio único half-duplex é **paralelo, não encadeado**; estrela e árvore funcionam, e a cadeia em margarida é só uma conveniência de roteamento:

```
TTL da placa adaptadora ──┬──► cabeça/pescoço   30 → 31 → 32 → 33 → 34
                          ├──► perna esquerda   20 → 21 → 22 → 23 → 24
                          └──► perna direita    10 → 11 → 12 → 13 → 14
                          (a imu_to_dxl 200 fica em qualquer um dos ramais dentro do tronco)
```

Como dividir: a URT-2 já traz duas portas TTL, cada uma com um ramal, e o terceiro sai de um servo; ou compre uma "placa divisora de barramento de servos 3P" de uma entrada e quatro saídas (¥5); ou solde três fios em paralelo.
**V e G devem ir em 18 AWG, separadamente, até os três ramais**, o S pode ser roteado como quiser. ⚠️ **Não passe a corrente dos três ramais pela placa `imu_to_dxl`**: embora os quatro soquetes dela sejam paralelos, o cobre passante da placa de 2 camadas não foi projetado para vários ampères e queima. Ela fica na ponta ou no meio de um dos ramais.

## Passo 5 · Validar a comunicação com um servo

Conecte apenas **um** servo com ID já configurado (por exemplo o joelho esquerdo, 23), e no controlador:

```bash
# 官方运行时带的总线基准工具（第 7 步装完才有），或先用 Python 快速 ping：
pip install feetech-servo-sdk    # 或 scservo_sdk
# 1 Mbps，ping ID 23 —— 回了就通了
```

Se não comunicar, verifique na ordem: se o `/dev/ttyS2` existe → se o `fuser` mostra alguém ocupando → se TX/RX estão cruzados → se o VCC da placa adaptadora é 3.3 V → contato do fio S.

## Passo 6 · ToF

```
VL53L8CX          Radxa 40 pinos
SDA      ◄──►    pin 3
SCL      ◄──►    pin 5
3V3      ◄───    pin 17     (o pin 1 foi para a placa adaptadora, usar separados)
GND      ────►   pin 6
```

Os pin 3/5 já têm pull-up na placa, não é preciso acrescentar.

⚠️ **O I²C ainda precisa de um overlay.** No oficial, o `i2c3` é remapeado (mux) para M0 (os pin 3/5), ao custo de perder a negociação USB-C PD.
O overlay oficial está em `deploy/audio/i2c3-pihat.dts` e foi escrito para o HAT; **se pode ser usado diretamente sem o HAT, ainda precisa ser validado**.

## Passo 7 · Rodar o runtime oficial

```bash
curl -fsSL https://raw.githubusercontent.com/pollen-robotics/microduck/main/scripts/provision.sh -o /tmp/provision.sh
sudo sh /tmp/provision.sh
# = setup-board.sh → migrate-network.sh → 重启 → install.sh，全自动
robotctl health
```

O `robotd.toml` fica em `/etc/robot/`, com a serial padrão `/dev/ttyS2`; para a placa adaptadora USB, altere para `/dev/ttyUSB0`.

## Passo 8 · Se quiser som (opcional)

Sem o HAT não há codec, e os latidos oficiais, o reconhecimento de carinho na cabeça, o teclado eletrônico com ToF e o coral de vários patos não funcionam: **mas a marcha do robô não é afetada**; o comentário oficial diz textualmente "uma placa sem codec anda exatamente igual, só não emite som".

Para recuperá-lo, conecte uma **placa de som USB** (do tipo sem driver, que o Linux reconhece direto):

```bash
aplay -l                          # 看到 USB Audio 那张卡的名字，比如 card 1: Device
sudo nano /etc/robot/robotd.toml
```

```toml
[audio]
enabled = true
device = "plughw:CARD=Device"     # 换成 aplay -l 看到的名字，官方默认是 plughw:aic3104
```

```bash
sudo systemctl restart robotd
```

O alto-falante vai na saída de 3.5 mm da placa de som (alto-falante pequeno precisa de amplificador; o mais prático é comprar direto uma caixinha USB com amplificador), e o microfone na entrada mic da placa de som.

⚠️ **Não validado**: os `sounds` e o `pet_detect` oficiais foram escritos para a taxa de amostragem e o número de canais do aic3104, e uma placa de som USB pode não coincidir. Se funcionar, corrija este item.

## Não feito, a validar

| Item | Dúvida | Como validar |
|---|---|---|
| Placa adaptadora de terceiros @ 1 Mbps | O circuito de direção do HAT oficial é `1G08+1G125+1G126`; a placa de ¥22 troca a direção rápido o bastante? | Passo 5 funcionando, contador de erros do `bench_dynamixel_bus` igual a 0 |
| O `i2c3-pihat.dts` funciona sem HAT | Esse overlay foi escrito para o HAT | Passo 6: ver 0x29 em `i2cdetect -y 3` |
| O módulo step-down aguenta a queda de tensão na partida dos servos | Com 15 servos movendo ao mesmo tempo a bateria cai | O controlador não reinicia durante a marcha; se reiniciar, aumente o capacitor de entrada |
| Qual imagem exata do Armbian | O oficial diz 26.2.x, sem dizer se é minimal ou desktop | Basta conseguir entrar por ssh no passo 1 |
| A placa de som USB pode substituir o aic3104 | O código de áudio oficial foi escrito para a taxa de amostragem do codec | Passo 8: a linha de áudio do `robotctl health` sem erro e latido ao ligar |

O diagrama de ligação está no início do documento; o arquivo-fonte é `不打HAT-接线图.html`, no mesmo diretório.

