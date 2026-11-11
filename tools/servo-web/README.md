# servo-web · Console web de depuração dos servos Feetech

> 🌐 Tradução não oficial para pt-BR do [original](https://github.com/fanhao375/microduck-replica). Em caso de divergência, vale o original. Licença: CC BY-NC-SA 4.0 (texto alterado: traduzido do chinês/inglês).

> O modelo 3D da página serve para exibição durante a depuração e continua baseado na geometria de simulação do upstream, **não podendo ser exportado e usado como as peças impressas Feetech mais recentes**. As peças estruturais e os modelos de montagem devem ser obtidos, de forma unificada, em [microduck-replica-cad](https://github.com/fanhao375/microduck-replica-cad).

Conecte a porta serial, arraste os controles deslizantes no navegador para mover os servos, e o pato 3D gira junto. Serve para montagem do robô, calibração do zero e aceitação em bancada, e também é a ferramenta dos itens 0–8 da lista de bancada da [arquitetura de adaptação Feetech](../../software/飞特适配架构.md).

![Interface](../../assets/servo-web.png)

> Observação: a própria interface da página permanece em chinês; os nomes de botões e campos entre aspas neste texto são traduzidos, e a forma original aparece entre parênteses nas primeiras menções importantes.

- 15 controles deslizantes agrupados em perna esquerda / cabeça e pescoço / perna direita; a linha amarela é a posição real do servo, e cada um tem uma chave de torque. **Antes de ligar o torque, alinhe o alvo com a posição atual**: com o torque desligado, se a junta foi movida à mão, uma pose foi enviada ou houve calibração, o registrador de alvo ainda guarda o valor antigo, e, sem alinhamento, ao ligar o torque a junta dispara para lá; o servo cuja posição não puder ser lida não liga
- Leitura em tempo real: posição, tensão, temperatura, carga, corrente, bits de estado (sobrecorrente / superaquecimento / sobrecarga ficam em vermelho)
- O modelo 3D acompanha a posição real (usa o modelo MuJoCo do [microduck_rl](https://github.com/apirrone/microduck_rl), Apache-2.0); o ± ao lado de cada junta é o sentido da junta, guardado em `directions.json` e compartilhado por todas as páginas; 3D, posição em pé, sequências e "灵动" (movimentos expressivos da cabeça) fazem a conversão por ele, sem mover os servos; "copiar tabela de direções do robotd" (复制 robotd 方向表) gera a linha exigida pelo `robotd.toml`. Conferido servo a servo no robô real: os 15 estão invertidos (o sentido em que a leitura do HD-1910 aumenta é oposto ao do XL330, e a montagem foi copiada do XL330, portanto tudo ficou invertido uma vez)
- Poses: salva a posição real atual como uma pose e a reenvia com um clique; um JSON com `steps` é uma sequência (já traz uma de três passos "levantar"). Todas as poses / posições em pé / zeros são interpoladas linearmente da posição atual até o alvo no "tempo de transição", e dá para clicar em "parar" (停) a qualquer momento
- Calibração com um clique (só para Feetech, o XL330 não tem esta função): posicione o pato na forma de "referência" e, com um clique, a leitura da "posição atual" dos servos selecionados é reescrita para o valor de referência. Há três referências:
  - **Zero 2048**: com o torque ligado, ajuste com os controles deslizantes até a perna ficar esticada, a sola do pé vertical, a cabeça olhando de frente e o bico fechado; segure e clique. As poses salvas são convertidas automaticamente para o novo zero
  - **Dobra oficial (cabeça e pescoço alinhados)** (recomendado): o pato é dobrado e apoiado, sem precisar segurá-lo no ar. O joelho dobra até o tornozelo encostar exatamente no servo da coxa (a peça serve de batente, sempre igual), o pitch do quadril é posicionado com um esquadro perpendicular ao tronco, o yaw/roll do quadril e o tornozelo são alinhados, o pescoço fica vertical, a cabeça olha de frente e o bico fecha. Leituras-alvo: as pernas são calculadas pelos ângulos FOLD oficiais e pelos ± (com todos os ± em −1, 22/23 = 1025 e 12/13 = 3071), os demais 2048; depois de calibrado o zero é o 2048 oficial, as poses salvas são convertidas automaticamente, e os 15 servos são marcados de uma vez. Cabeça e pescoço não seguem os 57° do FOLD oficial: aquilo não é um ponto de contato, e não dá para posicionar com precisão a olho (veja [`tools/fold_contact`](../fold_contact/)). O alvo do joelho é calculado pelos 1,57 rad (90,0°) oficiais; na geometria oficial, o tornozelo encosta no servo da coxa a 89,6°, de modo que o zero fica com cerca de 0,36° de diferença (4 passos). Com o torque desligado basta posicionar à mão, e depois da calibração o torque continua desligado. Ao escolher esta referência, o 3D sobrepõe automaticamente a "dobra para calibração" (校准用折叠), para posicionar seguindo-a
  - **Pose salva** (por exemplo "encolher"): o próprio pato fica encolhido e estável, posicionado por contato mecânico, sem precisar segurar. O zero continua sendo o 2048 oficial, de modo que o código e a política oficiais não precisam de offset; as demais poses usam o mesmo conjunto de leituras da referência, sem deslocamento

  Ao clicar no botão, aparece primeiro uma caixa de confirmação, listando servo a servo "agora → será escrito, diferença em graus" (calculado no servidor, a mesma conta da calibração real). **Diferenças de 30° ou mais pedem uma segunda confirmação, mas não são bloqueadas**: ao montar o horn, girar meia volta dá uma diferença de 180°, e a calibração justamente corrige isso; um sentido ± configurado ao contrário também dá uma diferença em torno de 180° (juntas de ±90° na dobra), e calibrar assim fica errado, por isso se alerta para conferir primeiro o ± (com o torque desligado, mova um pouco à mão e veja se o 3D gira para o mesmo lado). Depois da calibração, o log lista à parte os que tiveram maior diferença.

  Usa o comando de calibração de posição 0x0B (o HD-1910 não aceita escrever 128 na chave de torque, veja abaixo). Servo a servo: destravar → ler a posição → desligar o torque → 0x0B → reler → só alterar o alvo se coincidir → restaurar o torque → travar; o torque fica desligado só alguns milissegundos, a junta não é derrubada pela gravidade nem dá salto; se não coincidir, fica parada no lugar e reporta falha; mesmo com erro no meio, a trava é recolocada. O deslocamento original é guardado em `calib/` e pode ser desfeito (só se deslocam de volta as poses deslocadas naquela vez). No final, só se liberam a velocidade e a aceleração (41, 46), sem escrever junto o alvo 42
- Operações por servo: lista suspensa para escolher o servo (com o nome da junta, os offline são marcados), exportar todos os registradores 0–86 (JSON / tabela Markdown, para servir de **livro-razão de registradores**), reinício 0x08, calibração do ponto médio, trocar o ID (o ID antigo lista só os online, o novo lista 15 posições padrão, marcando as ocupadas), escrever registrador (a lista traz o nome do registrador, a área EEPROM marca automaticamente o destravamento, e relê para conferir após escrever)
- Todo o tráfego bruto da serial vai para o arquivo de log (hexadecimal + nome do registrador + resposta do servo); é por ele que se investigam aquelas operações da Feetech que "respondem sucesso mas não executam"
- O log é dividido em Sistema / Barramento / Movimento / Calibração / Registrador / Pose / Direção / IMU / Página, com avisos e erros coloridos e contados, filtráveis por categoria, mostrando só avisos e erros, e copiáveis com um clique. Cada entrada também é escrita em `logs/servo-web-日期.log`, e as com erro trazem o traceback; a calibração tem uma linha por servo: leitura antes e depois, deslocamento antes e depois, resposta de cada passo. Se houver problema, basta enviar esse arquivo para investigar
- Teste de temporização do `sync_read`: lê 15 servos de uma vez 200 vezes e reporta min / avg / p99 / max e o número de falhas
- Check-up dos servos (linha de comando, **somente leitura**): `python servo_report.py --port COM5` lê, de cada um, as chaves de proteção (condição de descarga 19), bits de estado, temperatura, tensão, corrente, deslocamento, limites de ângulo, classifica em vermelho/amarelo conforme o passo 5 do *registro de depuração*, e salva em `logs/体检-时间.md` e no `.json` de mesmo nome (valores brutos 0–86). Para ligar só um servo, use `--ids 21`; se não souber o ID, use `--scan-all`. A porta serial não pode estar ocupada ao mesmo tempo pelo console de depuração ou pelo FD
- Diagnóstico remoto: [`远程排查交接.md`](远程排查交接.md) é um procedimento escrito para o Claude Code em outro computador (check-up → diagnóstico servo a servo → ligar as proteções → calibração na dobra); lá, diga "primeiro leia tools/servo-web/远程排查交接.md e me conduza pelo procedimento que está lá"
- Autoteste: `python selftest.py`, sem ligar servos. O [`sim_bus.py`](sim_bus.py) é um simulador do HD-1910 em nível de protocolo (envia e recebe pacotes Feetech byte a byte, simulando trava, torque, 0x0B, o HD-1910 não aceitar 128, deslocamento, queda de linha), deixando o `feetech.py` + `server.py` reais executarem calibração, desfazer, troca de ID, escrita de registrador e exportação. Depois de alterar código que lida com servos, rode isso antes de ir para o robô real
- O back-end não depende do SDK da Feetech: o [`feetech.py`](feetech.py) envia pacotes diretamente conforme o manual de protocolo versão 2026 ([`docs/飞特资料/`](../../docs/飞特资料/)), em pouco mais de cem linhas

## Como executar

```bash
pip install -r requirements.txt
python server.py --port COM5                 # Windows + URT-2，串口号看设备管理器
python server.py --port /dev/ttyUSB0         # 板子上，URT-2 插板子的 USB 口
python server.py --port /dev/ttyS2           # 板子上走 40 脚的 UART2（要先接半双工转接，见下）
python server.py --fake                      # 没舵机：模拟模式，舵机是虚拟的
```

**Modo de simulação** (sem porta serial conectada): os servos virtuais seguem uma curva trapezoidal conforme os registradores de velocidade e aceleração dos servos reais; poses, controles deslizantes, sequências, "灵动" e poses oficiais se movem no 3D, com a mesma velocidade do robô real; o torque vem ligado por padrão, e não há gravidade. O campo "Simulação" (模拟) no canto superior direito permite escolher o IMU: sem IMU / com placa pequena virtual (ID 200 no barramento falso, o mesmo caminho do `--imu-bus`) / placa pequena virtual fora de linha (para ver como o console tira o 200 da lista, faz PING isolado e o reintegra). Ao conectar o USB e escolher a porta serial na barra superior e clicar em "Conectar" (连接), passa-se aos servos reais, esse campo some automaticamente e a placa pequena virtual também é retirada.

Na simulação também se pode ensaiar a calibração com um clique e o desfazer: só altera os servos virtuais, com o backup na pasta temporária do sistema, **sem tocar em `calib/` nem `poses/`** (se o deslocamento dos servos virtuais fosse misturado em `calib/`, ao conectar os servos reais e clicar em "Desfazer" (撤销) seria escrito nos servos reais). Salvar pose e alterar ± são operações explícitas, e gravam arquivos normalmente.

Se a porta serial não abrir, não há falha: a página abre do mesmo jeito (barramento falso), a barra superior mostra uma lista suspensa de portas seriais, e, conectado o USB, basta escolher uma e clicar em "Conectar" para assumir, sem reiniciar.

Abra o navegador em `http://127.0.0.1:8080`; se estiver rodando na placa, use o IP da placa, e até o celular consegue abrir. Os IDs gerenciados por padrão são `20-24,30-34,10-14`; para outros, use `--ids`. Os arquivos do modelo 3D estão em `model/` e já foram gerados; para gerar de novo:

```bash
python build_model.py --src <microduck_rl>/src/mjlab_microduck/robot/microduck
```

### Placa principal + HAT, depuração via Wi-Fi

Primeiro confirme que a placa principal já obteve endereço Wi-Fi e que o login por SSH funciona, configurando conforme a [descrição da imagem da placa principal](../radxa/镜像使用说明.md#4-conectar-ao-wi-fi). Coloque este diretório e as dependências na placa principal e, depois de confirmar que `robotd` e outros programas não estão ocupando o `/dev/ttyS2`, execute:

```bash
python server.py --port /dev/ttyS2 --ids 20-24,30-33,10-14 --host 0.0.0.0 --http-port 8080
```

Conecte o computador/celular à mesma rede local com comunicação entre si e abra no navegador `http://IP-da-placa:8080`. Este exemplo exclui o bico; para incluí-lo, ajuste conforme o ID real. A página web pode operar servos físicos, use-a apenas em uma rede local confiável. A partir da 0.15.2, ao iniciar/conectar a porta serial só se faz varredura e leitura de fase, sem liberar automaticamente os limites de velocidade e aceleração; os botões de movimento, calibração etc. da página ainda enviam de fato comandos de controle.

A partir da 0.15.4, a frequência-alvo de leitura do estado dos servos reais e de envio para a página web é de **50 Hz** (antes 10 Hz); o modo de simulação continua em 30 Hz. O "Atualização" (刷新) da barra superior é a frequência com que a página realmente recebe o estado, afetada pelo tempo gasto na serial, timeout, escalonamento do sistema e rede. Aumentar a frequência de leitura não altera a pose, o estado de habilitação nem o ritmo dos comandos de movimento, nem equivale a ter validado o laço de controle completo a 50 Hz incluindo IMU, inferência da política e escrita de comandos.

Os controles deslizantes só escrevem a posição-alvo, mantendo a configuração de velocidade/aceleração original do servo; se essa fase definir "velocidade 0 = parar" e a velocidade atual for 0, arrastar o controle pode não mover nada. Quando for preciso movimento, confirme a proteção, o apoio e a faixa mecânica, e use a função de transição suave com velocidade explicitamente definida.

Para comparar com a "conexão direta USB-serial do computador", primeiro pare o programa de depuração da placa principal, depois desligue normalmente, corte a energia e troque o cabo. Depois de religar a energia, no computador execute `python server.py --port COM-número-real --ids 23` e compare usando o mesmo servo; não deixe o controlador principal do HAT e o USB-serial enviarem comandos ao mesmo barramento ao mesmo tempo. O relatório somente leitura também pode ser feito com `python servo_report.py --port COM-número-real --ids 23`.

## Poses oficiais: ir para a pose / comparação com sobreposição 3D

O campo "Poses oficiais" (官方姿势) tem os quatro keyframes do `scene.xml` do microduck_rl: **zero** (tudo em 2048), **em pé** (STAND), **sentado** (SIT), **dobrado** (FOLD).

- "Ir" (去): converte para leituras conforme os ângulos oficiais, o ± de cada um e o zero atual, e vai até lá no "tempo de transição". **Ao enviar o dobrado, o joelho só dobra 88°**: calculado pela grade exata, na geometria oficial, com o quadril a 90° e o tornozelo a 0°, quando o joelho dobra a 89,6° o tornozelo bate no servo da coxa, e os 90° oficiais fariam o servo empurrar a peça com força (veja [`tools/fold_contact`](../fold_contact/))
- "Sobreposição 3D" (3D 叠加): desenha no 3D um segundo pato azul semitransparente na pose oficial, e abaixo lista junta a junta quanto "real − oficial" difere em graus (>2° amarelo, >5° vermelho). Quando o pato é posto naquela pose, a diferença indica o quanto o zero está desviado

## Opcional: atitude do tronco pelo IMU (barramento dos servos / ST-Link / DAPLink / J-Link)

O console de depuração pode combinar, no **mesmo modelo**, duas leituras: os ângulos das juntas dos servos continuam acionando as juntas originais, e o quaternion do IMU aciona a atitude do tronco. Reutiliza este serviço, o `/ws`, os arquivos do modelo e a CDN, sem iniciar um segundo serviço web. Sem parâmetros de IMU, essa área fica oculta e as funções seriais originais seguem como antes.

Aplicável à **mesma placa STM32G031F8P6 + LSM6DSV16X** já verificada; o modo J-Link exige Windows x64, Python 3.11+, o driver USB SEGGER J-Link instalado por você e o `JLink_x64.dll`. O firmware da placa nua e os passos de gravação estão em [`hardware/imu_to_dxl/firmware/`](../../hardware/imu_to_dxl/firmware/), e o esquemático e a ligação no [diretório de hardware](../../hardware/imu_to_dxl/). J3.1 vai a GND, J3.2 a SWCLK, J3.3 a SWDIO, **J3.6 a VTref**; J3.4/5 são a serial e não podem ser ligados como referência de alimentação.

1. Copie `imu-jlink.example.json` para `imu-jlink.json` e preencha o seu próprio `dll` e `probe_serial`. O arquivo já é ignorado pelo Git.
2. Feche o Keil, o J-Link Commander etc., saindo da depuração, para liberar a sonda; a placa já deve estar executando o HEX correspondente.
3. Em `tools/servo-web/`, execute os comandos abaixo, abrindo o navegador ainda em `http://127.0.0.1:8080`:

```bash
python server.py --port COM5 --imu-jlink imu-jlink.json  # 舵机串口 + 真 IMU
python server.py --fake --imu-jlink imu-jlink.json       # 只有 IMU 裸板，关节为演示值
python server.py --fake --imu-demo                       # 两者都是演示，不接硬件
```

Os caminhos no JSON são relativos ao arquivo de configuração; a DLL pode usar um caminho completo como `E:/SEGGER/JLink/JLink_x64.dll`, mas deve ser trocado pelo local real da sua instalação. O `firmware` pode ser omitido, usando por padrão o `hardware/imu_to_dxl/firmware/Build/imu_to_dxl.hex` do repositório; `hz` vai de 1–20; `resume` é `false` por padrão. Só defina `resume: true` quando precisar explicitamente retomar uma CPU pausada, e isso também deve passar antes pela verificação do firmware. O número de série não tem valor padrão, e a sonda não é escolhida automaticamente.

Ao receber a primeira atitude válida, o zero de exibição é definido automaticamente, e também se pode clicar em "Zerar" (归零) com a placa na horizontal. A direção de montagem padrão é +90° em torno de Y (`DEFAULT_MOUNT`); ajuste conforme a direção real de montagem e zere de novo. Essas operações só afetam a página web, sem escrever no IMU, nos servos nem na Flash. Depois de zerar, a tela mostra a atitude relativa; a "atitude bruta" também é apenas relativa à referência de inicialização do IMU.

### O mais prático: `--imu-bus`, lendo direto do barramento dos servos

A placa pequena fica no barramento dos servos (a partir do firmware 0.2.0 usa o protocolo Feetech); **sem nenhum depurador**, com um fio do URT-2 se leem servos e IMU juntos:

```bash
python server.py --port COM5 --imu-bus      # 真总线：15 颗舵机 + ID 200
python server.py --fake --imu-bus           # 没硬件：假总线上挂一块假小板，看效果
```

- Envia o **mesmo** sync_read que o controlador principal envia a cada tick: endereço 56, comprimento 15, **ID 200 em primeiro**, seguido dos servos. O estado dos servos é decodificado como antes, e o bloco do 200 é decodificado conforme os critérios do controlador principal (quaternion todo zero = fusão sem dados; norm² > 1,02 rejeitado; 12 bytes sem mudar por 25 quadros seguidos = frozen; o byte reservado deve ser 0; os bits de estado BIT1/BIT2 são falhas reportadas pela própria placa), o mesmo conjunto do `tools/imu200`.
- A especificação diz: quando a placa não responde, os dados dos servos seguintes no mesmo sync_read também se perdem. Por isso o 200 **sai da lista após 5 quadros seguidos sem resposta**, os servos são lidos isoladamente, a cada 2 segundos faz-se um PING isolado ao 200 e, se responder, ele volta para a lista —— assim o console funciona normalmente mesmo sem a placa pequena ligada, e a categoria IMU do log informa onde travou.
- Compatível com a placa `imu_to_dxl` do protocolo DXL 2.0 original: a página permite trocar o protocolo, ou use `--imu-bus-protocol dxl2`; o padrão continua sendo o protocolo Feetech. O DXL2 lê o ID 200, endereço 124, comprimento 12.
- Só faz polling quando a página web está conectada (no mesmo laço do estado dos servos); pausa durante a calibração. Na bancada, com só a placa pequena ligada e a varredura sem achar servos, o 200 também é lido normalmente.
- Com o barramento real lido a 50 Hz-alvo, o firmware de 120 Hz geralmente atualiza cerca de 2–3 vezes entre duas leituras; quando o período real se alonga, o BIT3 ("lendo devagar demais") ainda pode aparecer. Até a 0.15.3, lia-se a 10 Hz, com cerca de 12 atualizações entre duas. O console continua sem usar o BIT3 sozinho como alarme, e os bits de estado brutos ficam no quadro (`status_bits`).
- Uma placa com firmware 0.1.0 (Dynamixel 2.0) não responde no modo Feetech padrão: mantendo o firmware original, basta trocar para o modo de compatibilidade DXL2; também dá para gravar a 0.2.0 conforme [`firmware/`](../../hardware/imu_to_dxl/firmware/) e usar o modo Feetech padrão. O modo DXL2 separa o IMU e os servos Feetech em duas transações seriais, enquanto o modo Feetech continua usando a leitura síncrona acima.

### Sem J-Link: `--imu-swd` (ST-Link / DAPLink, via pyOCD)

A leitura é como a acima: primeiro consulta a tensão do alvo, depois compara byte a byte a Flash da placa e então lê o mesmo snapshot de RAM, sem gravar nem escrever na memória. A diferença é que passa pelo [pyOCD](https://pyocd.io/), em que **tanto o ST-Link quanto o CMSIS-DAP (DAPLink) funcionam**, sem arquivo de configuração. **Para J-Link, use o `--imu-jlink` acima**: conectar um J-Link via pyOCD, por padrão, fornece 5 V pelo pino 19 e pode atualizar silenciosamente o firmware da sonda, contornando as proteções daquele caminho, por isso o `--imu-swd` recusa diretamente o J-Link.

```bash
pip install pyocd
python server.py --fake --imu-swd                           # 只插一个调试器：自动选
python server.py --fake --imu-swd 301612179216303030303032  # 插了多个：用 UID 指定（pyocd list 查）
python server.py --port COM5 --imu-swd                      # 舵机串口 + 真 IMU
```

- Ligação como acima: J3.1 GND, J3.2 SWCLK, J3.3 SWDIO, J3.6 ao **VTref / TVCC** do depurador (só mede a tensão, não alimenta). A placa se alimenta sozinha pela porta do barramento; se o depurador tiver um pino de saída de 3,3 V (alguns ST-LINK/V2 têm no pino 19), **não ligue ao mesmo tempo que a alimentação do barramento**.
- O conector de 20 pinos do ST-LINK/V2 tem versões diferentes: em algumas, os pinos 4/6/8/10 são GND, em outras são UART, BOOT0, SWIM. **Use como GND só os pinos 12/18/20**.
- Conecta em modo `attach`: sem reset, sem pausa, e ao desconectar não envia Go; não carrega o `pyocd.yaml` / scripts de usuário do diretório atual. Ainda escreve em registradores de depuração (DHCSR.C_DEBUGEN etc.), e após desconectar o C_DEBUGEN permanece ativado, portanto não se pode dizer que não afeta o alvo. Em teste real (2026-09-22, ST-LINK/V2 V2J43S7, 100 kHz): 3 conexões seguidas, o relógio do chip bateu com o relógio de parede, **a placa não reinicia**; atualização de cerca de 20 Hz, idade dos dados de 6–11 ms.
- Antes de conectar ao chip, consulta a tensão do alvo: só o ST-Link consegue medi-la, e se não estiver em 3,0–3,5 V recusa (perto de 0 V avisa que o VTref/TVCC não está ligado ou a placa não está energizada); o DAPLink não lê a tensão, a página mostra "—" e não bloqueia. Quando há erro, distingue entre "o depurador não abre" (ocupado pelo Keil/CubeProgrammer, firmware do ST-Link muito antigo) e "não conecta ao chip" (ligação, alimentação).
- Para voltar ao Keil ou ao CubeProgrammer, primeiro pare este serviço, liberando o depurador.

Conexão, anomalias, mudanças de estado e a calibração de exibição são registradas no mesmo `logs/servo-web-日期.log`, com a categoria **IMU**; os dados não são registrados quadro a quadro. Parar o serviço libera o depurador (J-Link / ST-Link / DAPLink); para voltar à depuração no Keil, pare primeiro este serviço, pois fechar só a aba do navegador não libera a sonda. Atualmente não há operação de reconexão ou reset da sonda na página web; depois de tratada a anomalia de conexão, reinicie o serviço.

### Verificação do firmware e escopo da validação

- Apenas um `.hex` é enviado ao repositório, sem necessidade de baixar `.bin` nem `.map`. O back-end SWD / J-Link valida os registros Intel HEX e a imagem de Flash contínua, que decodificada tem **10724 bytes** (firmware 0.2.0), com SHA-256 `66bc7c532f5a1ad87e3dfff71a4b51f8c4f20840a8f62436874fa5b7c665938e`, layout de RAM `sample=0x2000006C`, `tick=0x20000A34`. Só essa imagem usa o layout de RAM já revisado, e depois de conectar ainda se compara byte a byte a Flash real; firmwares diferentes têm a leitura da atitude ou a retomada da CPU recusadas.
- Depois de alterar o firmware, confira primeiro a estrutura, o endereço e o tamanho no novo MAP, e só então atualize o hash verificado e o layout correspondentes. Não basta cancelar a verificação do hash. Esta ferramenta não executa gravação.
- Antes, com J-Link 9.78 e SWD a 100 kHz, a leitura na placa real foi de cerca de 20 Hz; o estabelecimento da conexão pelo SDK já causou reinício do programa da placa, de modo que não pode ser visto como observação sem perturbação. A versão integrada atual fez à parte um autoteste sem hardware e **ainda não foi validada com servos reais e IMU real funcionando juntos**, sendo preciso conferir no robô os eixos e a estabilidade de longa duração.
- **O firmware 0.2.0 que o acompanha já usa o protocolo Feetech** (ID 200, bloco de 15 bytes no endereço 56), e o código real do protocolo passou no `imu200.py check` sobre o barramento simulado; mas **o barramento real ainda não foi testado**, e uma tela do SWD normal também não significa que o barramento real e a aceitação do `FeetechIo` tenham passado. Antes de ir para o robô completo, execute `imu200.py check --port`.
- O rumo de um IMU de seis eixos deriva, e a página não mede translação nem julga o contato real da sola do pé. O centro de gravidade e a região de apoio continuam sendo estimativas da geometria do modelo, e não permitem concluir que o robô completo consegue ficar de pé ou andar com estabilidade; com a interrupção do fluxo, mantém-se a última atitude e marca-se como inválida.

### Verificação de regressão sem hardware

```bash
pip install -r requirements-test.txt
python selftest.py
python -m pytest test_feetech_sync.py test_imu_bridge.py test_imu_server.py test_imu_swd.py test_imu_bus.py test_dxl2.py test_sim.py test_servo_report.py
node --test test_imu_attitude.cjs
```

O Node.js só é usado nos testes matemáticos do front-end; para usar o console de depuração normalmente não é necessário. Os testes de host do firmware e o histórico de registros em placa real estão em [VALIDATION.md](../../hardware/imu_to_dxl/firmware/VALIDATION.md).

## Atenção

- **Primeiro conecte o URT-2 ao USB, depois energize os servos.** Na ordem inversa o nível do barramento fica preso em 2,1 V, a varredura não acha nenhum servo e só o broadcast recebe alguns 0, veja o [registro de armadilhas](../../踩坑记录.md#ferramentas).
- **Na placa, antes de usar o ttyS2, execute `sudo systemctl stop robotd`**. O daemon oficial abre essa porta a cada segundo procurando servos, e se os dois lados enviarem ao mesmo tempo os pacotes colidem. Usando o URT-2 via ttyUSB0 não há esse problema.
- **Ligar o ttyS2 direto aos servos com fios não recebe resposta**. A UART da Radxa é full-duplex com dois fios de 3,3 V, e o barramento Feetech é half-duplex de fio único, sendo preciso no meio um buffer tri-state (como no HAT oficial) ou desmontar uma solução de chip de um URT-2. Depois de pronta a placa adaptadora, use primeiro o teste de temporização da página para ver se há eco: o eco aparece como tudo "sem resposta".
- A calibração do ponto médio e a troca de ID escrevem na EEPROM, e a página pede confirmação. Ao trocar o ID, só deixe ligado ao barramento aquele servo.
- **O HD-1910 (firmware 3.46) não aceita escrever 128 na chave de torque**. O mapa de memória diz que permite calibração do ponto médio, mas em teste real, 4 rodadas com 15 servos: destravar, com o torque desligado escrever 128, o servo responde "sucesso", e nenhum deslocamento mudou. Na tabela 4.9 do manual do protocolo, HLS ≥3.43 também consta como sem suporte. A calibração sempre passa pelo 0x0B.
- "Tudo de volta a 2048" é executado imediatamente em servos com o torque ligado; os montados nas pernas, desligue o torque primeiro ou faça um a um.
- A página carrega o three.js do jsdelivr, então o computador que abre a página precisa ter acesso à internet; o back-end em si não precisa.
- A boca (ID 34) não tem essa junta no modelo; o controle deslizante move o servo, mas o 3D não exibe.

## Versões

O número da versão aparece ao lado do título da página. Quando as versões da página e do serviço diferem, fica em vermelho e avisa para primeiro reiniciar o `server.py` e dar Ctrl+F5; isso geralmente é o navegador ainda usando a página antiga. Ao alterar código, aumente juntos o `VERSION` do `server.py` e o `PAGE_VERSION` do `index.html`.

| Versão | Data | O que mudou |
|---|---|---|
| 0.15.6 | 2026-10-23 | Corrigido o polling normal do IMU DXL 2.0 que escrevia continuamente no log bruto; reutiliza a configuração `trace_skip` do barramento, mantendo as requisições que falharam e os logs de diagnóstico manual |
| 0.15.5 | 2026-10-17 | Novo modo de compatibilidade com a `imu_to_dxl` DXL 2.0 original e troca de protocolo na página; o padrão Feetech não muda |
| 0.15.4 | 2026-09-28 | A frequência-alvo de leitura do estado dos servos reais e de envio para a página subiu de 10 Hz para 50 Hz; o modo de simulação continua em 30 Hz, sem alterar a lógica de movimento e habilitação |
| 0.15.3 | 2026-09-28 | A leitura síncrona passa a ser arquivada pelo ID real do pacote de resposta, corrigindo o problema de, faltando um pacote, as respostas normais seguintes serem julgadas como falha em cadeia; a perda real de leitura continua visível, sem esconder com mais tentativas; acrescentados testes de regressão do fluxo de bytes do protocolo |
| 0.15.2 | 2026-09-28 | Ao iniciar/conectar a porta serial só se lê a fase, sem escrever automaticamente nos registradores de velocidade e aceleração; acrescentados a depuração via Wi-Fi da placa principal + HAT e o método de comparação com USB-serial |
| 0.15.1 | 2026-09-22 | Check-up dos servos `servo_report.py` (somente leitura, verificado no simulador de protocolo que não envia nenhum comando de escrita); `远程排查交接.md`; na caixa de confirmação da calibração a frase sobre o tornozelo passou a "a sola do pé forma ângulo reto com a canela" (o antigo "alinhar" era fácil de entender ao contrário) |
| 0.15.0 | 2026-09-22 | Correções conforme revisão (revisada por subagent): antes de ligar o torque, alinha-se o alvo com a posição atual (calibrando com o torque desligado e depois abrindo à mão, ao ligar o torque a junta voltava de supetão para a dobra); o final da calibração não reescreve mais o alvo 42 com o 67 relido; na simulação a calibração / desfazer não toca em `calib/` nem `poses/`; o deslocamento do barramento falso é codificado em sinal-magnitude; ao conectar a porta serial real, o IMU virtual simulado é retirado, e a conexão / desconexão é difundida a todas as páginas. A caixa de confirmação da calibração lista servo a servo "agora → será escrito, diferença em graus", com segunda confirmação para diferenças de 30° ou mais, sem bloquear; o 3D pode sobrepor a "dobra para calibração"; aviso para não apertar "dobrar" antes de calibrar o zero |
| 0.14.0 | 2026-09-22 | Modo de simulação: os servos virtuais seguem uma curva trapezoidal conforme os registradores de velocidade / aceleração (antes só havia seguimento de primeira ordem com o torque ligado), torque ligado por padrão, estado enviado a 30 Hz; o campo "Simulação" permite conectar / desconectar a placa IMU virtual a qualquer momento. O 3D passou a desenhar só quando há mudança (mais de 400 mil triângulos desenhados sempre a 60 quadros, e a caixa de confirmação demorava a aparecer ao clicar no botão). A calibração com um clique ganhou a referência "dobra oficial": o alvo é calculado pelos ângulos FOLD oficiais × ±, depois de calibrado o zero é o 2048 oficial, as poses salvas são convertidas junto; pernas na dobra oficial, cabeça e pescoço alinhados, os 15 servos marcados de uma vez |
| 0.13.0 | 2026-09-22 | Campo "Poses oficiais": quatro botões zero / em pé / sentado / dobrado (keyframes oficiais convertidos por ±, e o joelho da dobra dobra 2° a menos para não empurrar a peça); o 3D sobrepõe a pose oficial semitransparente, listando junta a junta o ângulo "real − oficial", para ver o quanto o zero está desviado |
| 0.12.0 | 2026-09-22 | Opcional `--imu-bus`: lê a placa IMU pequena direto do barramento dos servos, no mesmo sync_read do controlador principal (endereço 56, comprimento 15, 200 em primeiro), decodificando o bloco pelos critérios do controlador principal (não pronto, rejeitado, frozen, byte reservado, bit de falha da placa); o 200 sai da lista após 5 quadros seguidos sem resposta e recebe PING isolado a cada 2 segundos, e sem a placa pequena os servos continuam sendo lidos; `--fake --imu-bus` tem uma placa pequena falsa para ver o efeito. Acompanha o firmware 0.2.0 (protocolo Feetech), e o hash do HEX e os endereços de RAM do observador SWD/J-Link foram atualizados em conjunto |
| 0.11.0 | 2026-09-22 | Opcional `--imu-swd`: lê a atitude do IMU via pyOCD com ST-Link / DAPLink, sem a DLL do J-Link nem arquivo de configuração (o J-Link continua usando `--imu-jlink`, e o `--imu-swd` o recusa). O mesmo conjunto de proteções do caminho J-Link: antes de conectar ao chip consulta a tensão, compara a Flash da placa, só lê a RAM, a CPU pausada não é retomada automaticamente; conecta em modo attach, e em teste real não reinicia a placa. Com um só depurador ligado, escolhe automaticamente; com vários, é preciso indicar o UID. O ST-Link mostra de passagem a tensão do alvo; os avisos do pyOCD também entram no log do IMU. A aba da página mostra o nome real do depurador |
| 0.10.0 | 2026-09-21 | Opcional `--imu-jlink`: o mesmo WebSocket, log e modelo 3D combinam os ângulos das juntas dos servos e a atitude do tronco pelo IMU; mantém a direção de montagem/zeramento de exibição, suporta `--imu-demo` sem hardware, e com o IMU desativado a interface original não muda. O firmware e os materiais de hardware que acompanham ficam juntos, e o repositório só guarda o HEX |
| 0.9.1 | 2026-09-20 | O tráfego bruto da serial vai para o arquivo de log: o que foi enviado em cada pacote e o que o servo respondeu, hexadecimal + nome do registrador (`→ #32 位置校准 ff ff 20 02 0b f2` / `← #32 状态0x00 …`). Operações que a Feetech não suporta respondem "sucesso" do mesmo jeito, e só olhando os bytes brutos se distingue "realmente fez" de "fingiu fazer". O polling de estado a 10 Hz não é registrado, senão o log só teria isso |
| 0.9.0 | 2026-09-20 | Calibração reescrita conforme revisão de código (apontada por subagent, todas do tipo "arrasta a junta"): o critério passou de "a leitura é igual ao alvo" para "o registrador de deslocamento mudou", porque operações que a Feetech não suporta respondem sucesso do mesmo jeito; **compensação da queda por gravidade** —— nos milissegundos com o torque desligado a junta cai (medido 0,62 passo/ms no pescoço, enquanto uma ida e volta da USB-serial leva 1~16 ms), e a queda é calculada de "leitura antes da calibração, alvo, mudança do deslocamento" e compensada de volta, reduzindo o erro de zero de 99 passos para 0 com um barramento lento de 16 ms; releitura de confirmação depois de desligar o torque (se esse pacote se perdesse, o 0x0B seria executado com o torque ligado, e ao mudar o deslocamento a junta seria puxada a toda velocidade); o alvo escrito é sempre a leitura atual, nunca o valor do alvo; mesmo com erro, escreve o alvo, restaura o torque e trava no finally; durante a calibração, usa o barramento com exclusividade e reduz aceleração e velocidade; o sentido do sinal do deslocamento é sondado uma vez na inicialização (±1 passo), sem tentativa e erro a cada servo; os registradores 31/42/46 passam a usar codificação sinal-magnitude (antes, números negativos escritos em complemento de dois eram entendidos como -32763 e batiam no limite); a faixa de deslocamento foi reduzida a ±2047; ao desfazer, as poses salvas depois da calibração também são deslocadas de volta; backup escrito atomicamente, primeiro gravado em disco e só depois deslocam-se as poses. O simulador ganhou queda por gravidade e atraso de barramento, e agora esse tipo de bug é detectado |
| 0.8.1 | 2026-09-20 | Mesmo que a porta serial não abra, a página sobe (USB não conectado, número da porta mudou, porta ocupada, nada mais trava), recua para o barramento falso e lista no log as portas seriais existentes; a barra superior ganhou uma lista suspensa de portas e o botão "Conectar", e basta conectar o USB e clicar para assumir, sem reiniciar o serviço |
| 0.8.0 | 2026-09-20 | A calibração deixou de escrever 128 e passou ao comando 0x0B (o HD-1910 em teste real não aceita 128, e o "sucesso" da 0.7.x era na verdade a leitura já próxima de 2048); o sucesso exige que a leitura realmente vire o alvo; é possível escolher uma pose salva como referência (calibrar "encolhido"); o torque fica desligado só alguns milissegundos (na 0.7.x ficava desligado 30–130 ms, e o pescoço caía 80 passos); desfazer só desloca as poses deslocadas naquela vez (na 0.7.x desfazer deslocou ao contrário as "em pé" e "encolhido" salvas depois, já restauradas pelo log); o nome do arquivo de backup tem milissegundos; a calibração de um servo isolado segue a referência |
| 0.7.3 | 2026-09-20 | Corrigida a falha da calibração em todos os 15 da 0.7.1 (`'_thread.lock' object is not callable`: o mutex da serial se chamava `self.lock` e encobria o método `lock()` que trava a EEPROM); a exportação lê até o 86 (antes lia até o 90, e o robô real respondia "short read"); mesmo com erro no meio da calibração, trava, e com erro não restaura o torque; categorias de log + gravação em arquivo + filtro e cópia; operações por servo viraram lista suspensa; simulador em nível de protocolo e autoteste |
| 0.7.2 | 2026-09-20 | As poses podem ser excluídas (botão ×, movidas para `poses/.trash/` de onde dá para recuperar); botões de poses com o mesmo nome trazem a data |
| 0.7.1 | 2026-09-20 | Corrigida a calibração do ponto médio: a 0.7.0 escrevia 128 com o torque ligado e travado, o servo respondia "sucesso" mas não calibrava de fato, e em seguida escrevia o alvo como 2048, o que arrastava a junta de volta ao zero antigo. Mudou para, servo a servo, destravar → desligar o torque → escrever 128 → reler para confirmar → alvo → restaurar o torque → travar; se a calibração não tiver sucesso, fica parado no lugar |
| 0.7.0 | 2026-09-20 | Calibração do ponto médio com um clique (todos ou IDs indicados) + desfazer; número de servos online, taxa de atualização e tensão média na barra superior; cada servo mostra o ângulo da junta (inspirado no console de depuração de um membro do grupo); direção dos 15 totalmente invertida |
| 0.6.0 | 2026-09-20 | O sentido das juntas é guardado em `directions.json`, compartilhado por todas as páginas; conferidos no robô real 23, 13 e 30–34 invertidos; "copiar tabela de direções do robotd"; a página proíbe o cache do navegador; número de versão |
| 0.5.0 | 2026-09-19 | Movimentos de cabeça "灵动"; o movimento suave passou a não bloquear o back-end; corrigido o caso de fase com BIT3=0 em que escrever 0 no registrador de velocidade equivale a parar |
| 0.4.0 | 2026-09-18 | Projeção do centro de gravidade e região de apoio no 3D; movimento suave (o próprio servo percorre a curva trapezoidal); releitura da posição-alvo para conferir, reenvio dos que não chegaram |
| 0.3.0 | 2026-09-18 | Salvar/carregar poses, botões "encolher" e "em pé", sequências levantar / encolher, tempo de transição |
| 0.2.0 | 2026-09-17 | Dobradiça da boca, aviso de torque desligado, botão de posição em pé oficial, porta padrão alterada para 8080 |
| 0.1.0 | 2026-09-17 | Primeira versão: controles deslizantes, leitura em tempo real, 3D acompanhando, exportação de registradores, teste de temporização, troca de ID, calibração do ponto médio |

## Arquivos

| Arquivo | Para que serve |
|---|---|
| `server.py` | Serviço HTTP + WebSocket, starlette + uvicorn |
| `feetech.py` | Protocolo Feetech: PING / READ / WRITE / SYNC_READ / SYNC_WRITE / REBOOT, tabela de nomes de registradores |
| `sim_bus.py` / `selftest.py` | Simulador de servos em nível de protocolo / autoteste do back-end |
| `servo_report.py` / `test_servo_report.py` | Check-up dos servos (somente leitura) / testes contra o `sim_bus` |
| `远程排查交接.md` | Procedimento de diagnóstico e calibração para a IA em outro computador |
| `logs/` `calib/` | Logs de execução, backups do deslocamento antes da calibração, locais desta máquina, não vão para o repositório |
| `index.html` | Front-end de arquivo único, o three.js vem da CDN |
| `imu_bridge.py` / `test_imu_bridge.py` | Leitura J-Link / pyOCD, verificação do firmware, decodificação de amostras e testes sem hardware |
| `test_imu_swd.py` | Testes do leitor `--imu-swd` (pyOCD): usa um módulo pyocd falso, sem ligar a sonda |
| `imu_bus.py` / `dxl2.py` | `--imu-bus`: leitura do IMU pelo barramento e adaptação de compatibilidade DXL2 |
| `imu_attitude.js` / `test_imu_attitude.cjs` | Conversão do quaternion do IMU, direção de montagem e referência de exibição, e testes de regressão |
| `imu-jlink.example.json` / `test_imu_server.py` | Modelo de configuração pessoal / testes de integração do IMU com o serviço existente |
| `build_model.py` | MuJoCo XML + STL → `model/model.json` + `model/meshes.bin` (vértices quantizados em int16, 21 MB viram 2,9 MB) |
| `model/` | Modelo já gerado, 14 juntas |
