# Voz local do pato e escuta de música

> 🌐 Tradução não oficial para pt-BR do [original](https://github.com/fanhao375/microduck-replica). Em caso de divergência, vale o original. Licença: CC BY-NC-SA 4.0 (texto alterado: traduzido do chinês/inglês).

A nova versão, por padrão, ativa a escuta com um clique no botão do analógico direito do controle (gamepad), **R3**, com um breve "quá"; solte e fale, e um novo clique desativa, com um grave "cuu". O interruptor da página web usa o mesmo aviso sonoro, grasnando uma vez apenas quando a troca realmente ocorre; cancelamentos automáticos como Select, desconexão e saída do serviço ficam em silêncio. Por padrão `ECHO_REPLY_MODE=duck`: com o comando de voz ativado, executa comandos explícitos de sentar/levantar, e o modo música responde a perguntas explícitas sobre música; falas comuns ou não reconhecidas ficam em silêncio, sem repetir a voz da pessoa e sem oferecer conversa livre. Desativar cancela o reconhecimento pendente e interrompe a gravação/repetição, e só depois toca o aviso de desativação.

O aviso sonoro usa a duração do WAV da biblioteca de efeitos instalada, mais 100 milissegundos de margem de partida e 350 milissegundos de cauda, período em que a captação fica em pausa; depois do fim do aviso, espere um instante antes de falar. A cada início do serviço, a escuta e o ouvir música continuam desligados por padrão.

A reprodução de gravação é mantida como um comando `record` separado, por ora sem vínculo com novo botão. Se for preciso explicitamente o comportamento antigo de repetição, defina `ECHO_REPLY_MODE=echo` ou passe `--reply-mode echo`; nesse caso o tom e a velocidade padrão sobem para 1,25 vez, e definir 1,0 restaura a voz original.

A voz desta rodada e o painel web de música passaram por instalação em SSH, e o problema de segmentação de frases foi retestado por uma pessoa e recuperado. A acústica da música tocada em alto-falante e o efeito dos movimentos do corpo ainda aguardam aceitação por pessoas, e os testes de software não substituem esses dois itens. Veja [esta validação](../../docs/superpowers/verification/2026-10-25-audio-interaction.md). A aceitação histórica da versão inicial a seguir corresponde à versão de falar segurando o botão.

Este é um serviço independente, instalado por SSH, que reutiliza a interface de movimentos de sentar/ficar em pé do programa principal, sem regravar nem reiniciar o robotd/padd. Ouve o controle físico apenas em leitura, sem ocupar o dispositivo de entrada com exclusividade. O som fica apenas na memória, sem salvar arquivos de gravação de conversa.

**Aceitação histórica da versão inicial: em 2026-10-25 foi implantada por SSH em duck-0de6 (192.168.0.102), configurada para iniciar com o boot. Captação real do microfone, reprodução com mudança de tom e R3 do controle físico segurado/solto funcionaram. 45 testes de software passaram; depois o usuário confirmou claramente que na voz de pessoa real "levantar e sentar deram ambos certo".**

O controle real é um Nintendo Switch Pro Controller, dispositivo de entrada /dev/input/event4. Antes e depois da instalação, o PID 1090 do robotd, o PID 1128 do padd e os horários de início não mudaram, e o laço de controle se manteve a 50 Hz. O serviço foi definido como enabled, e a máquina inteira não foi reiniciada por isso; quando o controle já pareado reconecta no boot, o serviço de repetição o detecta automaticamente.

Hardware aplicável: Linux 64 bits, HAT TLV320AIC3104, um caminho de reprodução de alto-falante já normal, controle físico com suporte a BTN_THUMBR. A captação usa o Mic3R (canal direito) usado pelo script de inicialização do projeto. Não suporta R3 de controle simulado pela página web.

## Ouvir música e perguntas explícitas (instalado no robô, alto-falante e movimentos aguardando aceitação)

A página web pode controlar separadamente a escuta e o ouvir música; ligar o ouvir música liga o microfone ao mesmo tempo, e desligar a escuta desliga também o modo música. O R3 e a página leem o mesmo estado real de escuta. Com o controle físico desligado ainda é possível iniciar a captação pela página; desconexão real do controle, Select, estouro de entrada e saída do serviço cancelam a entrada atual.

O modo música reutiliza este mesmo fluxo de microfone, estimando o ritmo e a sensação aproximada pelo início de energia de curto prazo e pela evidência de repetição do envelope. Animado, calmo e balanço são apenas retorno de ritmo, não representam reconhecimento exato de gênero ou emoção. A música não é enviada para a repetição comum nem para o reconhecimento de comandos de sentar/levantar. Quando se diz perguntas explícitas como "o que você achou" ou "gosta desse tipo de música", um reconhecedor de streaming local separado, reutilizando o Vosk Model existente, aciona os efeitos nativos chirp, coo, greet conforme o ritmo confiável recente; sem ritmo confiável usa inquire. A pergunta completa exige confiança por palavra de pelo menos 0.8; um partial completo estável mantido por 0.5 segundo pode responder enquanto a música continua, com intervalo mínimo de 4 segundos entre respostas. Durante o aviso sonoro e a cauda, a captação do próprio som é suprimida.

A captação contínua solicita período de 20 milissegundos e buffer de 80 milissegundos; o tempo das batidas avança por amostras, evitando a distorção do ritmo causada pela entrega em lotes do driver. A estimativa de repetitividade guarda apenas envelopes de valor numérico finito, calcula no máximo cinco vezes por segundo, e a leitura do estado reutiliza o cache. Música suave costuma precisar de alguns segundos de evidência repetida; sem repetição clara, com o som parado ou com a captação vencida, não se geram batidas. A leitura estável é apenas uma estimativa de período do envelope de volume; tempos fortes e fracos, dobro de velocidade e ritmos mistos ainda podem ser ambíguos; quando evidências conflitam, prevalece a de suporte mais forte, sem garantir um único ponto de batida musical.

Só o modo toggle oferece o `/run/duck-echo/control.sock` privado (diretório 0700, socket 0600). Cada conexão é uma linha JSON UTF-8, no máximo 8192 bytes, suportando apenas `{"method":"status"}` ou `{"method":"set","listening":true,"music":true}`; os booleanos devem ser JSON true/false. Retorna escuta, música, ritmo e uma breve informação de falha, sem oferecer gravação, transcrição nem comandos arbitrários ao robô. Ao reiniciar o serviço, fica desligado por padrão. O hold mantém o antigo comportamento de falar segurando, sem socket de controle de música pela web.

As perguntas e respostas sobre música exigem o modelo de reconhecimento local instalado. Se o modelo não está pronto, o estado mostra falha de reconhecimento de música; a reprodução de gravação separada e o modo echo explícito continuam utilizáveis. Testes de software com ALSA / socket UNIX falsos não equivalem à aceitação de música real em alto-falante, reverberação e reconhecimento de perguntas.

## Modo de comandos de voz em chinês

Modo de comando de voz local: dizer comandos curtos explícitos como "坐下", "坐下来", "站起来", "起立" executa a ação correspondente; falas comuns por padrão não são reproduzidas. "不要坐下", várias ações misturadas numa frase e reconhecimento de baixa confiança não disparam ação. Apertar Select, desconexão do controle ou estouro de entrada cancela comandos antigos ainda não executados e desliga a escuta contínua.

Na CPU do pato já passaram 14 falas de teste sintéticas, cobrindo comandos afirmativos, negação, citação, ações misturadas e falas comuns, com reconhecimento de cerca de 0,26–0,51 segundo e pico de memória de cerca de 144 MB. Usa-se um vocabulário pequeno com fala desconhecida e formas negativas, evitando o atraso de 6–7 segundos do ditado completo. Depois o usuário testou com o microfone real e o R3, e confirmou que levantar e sentar deram ambos certo.

Depois da aceitação, houve um período em que SSH e página web ficaram inacessíveis. Depois, a conectividade foi recuperada por configuração de rede via Bluetooth, fixando este pato em TP-LINK_5G_6494 (5240 MHz), desligando a conexão automática de outras redes salvas e a economia de energia do Wi-Fi. 20 acessos consecutivos à página tiveram sucesso. Os registros Bluetooth com a voz desligada por cerca de 160 segundos e ligada por cerca de 180 segundos não tiveram queda de enlace, e o intervalo máximo dos dados do controle foi em ambos inferior a 49 milissegundos; com a voz ligada, completaram-se repetição por pessoa real, sentar e levantar, obtendo logs com accepted=true em duas ações. O usuário confirmou que a repetição e o analógico estavam normais; em outra vez viu a luz piscar, e a ação também estava normal, sem queda de enlace correspondente nos registros. Os logs anteriores de fato mostravam queda real, cuja causa ainda não foi determinada, e um reteste curto não prova que o problema intermitente de longo prazo tenha desaparecido por completo. Veja docs/network-20261025.md e o local artifacts/duck-network-20261025/final-verification.json.

Usa Vosk small-cn-0.22, com o modelo e as dependências dentro de /opt/duck-echo, sem enviar voz. A gravação completa não é escrita em arquivo, e o log registra apenas a categoria do comando, a confiança e o recibo da ação, sem o texto de conversas comuns. A cada vez primeiro lê o estado atual de sentado/em pé, de modo que repetir "坐下" não faz levantar no sentido contrário; se o programa principal recusa o pedido de postura, não se contorna. A recuperação depois de liberar o torque sentado continua seguindo as exigências do firmware existente.

O modo de clique único processa uma frase depois da pausa na fala, com no máximo 10 segundos; resultados de reconhecimento que passem de 6 segundos não são mais executados. Durante a própria repetição e os avisos de ação, a captação é suprimida, e continua a escutar depois do fim da cauda. A segmentação automática de frases comum é adequada a ambientes silenciosos, e com música tocando continuamente deve-se ligar o modo música.

Para restaurar o antigo falar segurando, defina `ECHO_TRIGGER_MODE=hold` em `/etc/default/duck-echo` e reinicie apenas o serviço `duck-echo`. O modo hold ainda processa depois de soltar o R3; quando a gravação do comando de voz enche 10 segundos, essa frase é cancelada.

```bash
# 先更新运行文件，再准备本地识别依赖和模型。离线安装可提供 ARM64/cp313 wheels。
sudo python3 /home/radxa/duck-echo/install.py
sudo python3 /opt/duck-echo/install_speech.py --mode actions \
  --archive /home/radxa/duck-echo/vosk-model-small-cn-0.22.zip \
  --wheels /home/radxa/duck-echo/arm64-wheels

# 观察识别，暂不执行动作：
sudo python3 /opt/duck-echo/install_speech.py --mode shadow

# 关闭口令识别（需要复读时另设 ECHO_REPLY_MODE=echo）：
sudo python3 /opt/duck-echo/install_speech.py --mode off
```

Fontes: [modelos oficiais do Vosk](https://alphacephei.com/vosk/models), [suporte de instalação ARM64](https://alphacephei.com/vosk/install).

## Instalação no robô via SSH

Envie esta pasta para `/home/radxa/duck-echo` no pato. Execute no pato:

```bash
cd /home/radxa/duck-echo
arecord -l
python3 duck_echo.py devices
sudo python3 install.py prepare-mic
python3 duck_echo.py probe --seconds 3
python3 duck_echo.py record --seconds 4
sudo python3 install.py
journalctl -u duck-echo -n 30 --no-pager
```

Nos 3 segundos do `probe`, fale perto do microfone; a saída inclui peak, rms, dbfs; com silêncio/curto demais/falha de captação, o código de saída é 2. O comando `record` grava em 4 segundos e depois reproduz; só quando o processo de reprodução sai com sucesso é que imprime `playback_ok: true`. O sucesso do processo ainda precisa ser confirmado pelo ouvido quanto à qualidade real do som e ao volume do alto-falante.

A instalação exige sudo; o serviço roda como usuário radxa e entra nos grupos adicionais audio/input/robot. Depende do Python 3 do sistema, do alsa-utils e do systemd, sem precisar baixar pacotes Python.

## Voz original e configuração do dispositivo

```bash
sudo nano /etc/default/duck-echo
# ECHO_PITCH=1.0 为原声，1.25 为较高较快的声音。
# ECHO_CHANNEL=1 为 Mic3R 右声道；仅硬件确实接左路时改成 0。
sudo systemctl restart duck-echo
```

Reinicia apenas o serviço de repetição. Se o nome da placa de som for diferente, modifique ECHO_CAPTURE/ECHO_PLAYBACK; a configuração de microfone deste script de instalação é específica do aic3104, e outras placas de som devem ter o caminho de captação confirmado separadamente conforme seu hardware.

## Diagnóstico e reversão

- Quando existe o novo padd correspondente, o serviço lê `/run/padd/mapping-state.json` e escolhe o nó evdev real do padd, suportando mesmo um controle sem o BTN_THUMBR (318) padrão do Linux. R3／Select personalizados são tratados pelo vínculo de botão bruto ou eixo de hat; se a configuração personalizada omite o R3, não recai no 318. Só em instalações antigas que nunca forneceram esse arquivo de estado se mantém o primeiro controle que suporta o R3 padrão.
- A pausa de calibração cancela a escuta, o reconhecimento, as ações pendentes e a reprodução existentes, e durante ela as teclas apenas atualizam a linha de base; mudança de configuração, dispositivo, processo ou estado de pausa restabelece a linha de base das teclas. Se o R3 já estava pressionado ao iniciar ou retomar, é preciso soltá-lo e apertar de novo. Depois de já ter lido o novo estado, se o estado sumir／estiver corrompido／o processo sair, desativa de forma conservadora até o estado válido voltar.
- Não grava som: veja `arecord -l`, o nível do probe e a soldagem/ligação do Mic3R; o hardware desta máquina ainda não foi validado, e o silêncio não é atribuído a software ou soldagem.
- Aviso de reprodução com Device busy: solte o gatilho que emite som original e tente de novo. Os efeitos nativos e a repetição podem disputar o caminho de reprodução do hardware; esta versão reporta o erro, sem parar o robotd nem adulterar a configuração global de reprodução do ALSA.
- A mudança de tom altera também a velocidade da fala; esta versão suporta, depois de apertar o botão, o reconhecimento de comandos curtos de sentar/levantar, sem incluir ativação contínua, conversa livre nem mudança de voz que preserve a velocidade.

```bash
sudo systemctl stop duck-echo
sudo python3 /opt/duck-echo/install.py uninstall
```

A desinstalação desliga o serviço de repetição e restaura a configuração de captação do microfone de antes da instalação. O programa e os backups ficam em /opt/duck-echo e /var/lib/duck-echo; o volume do alto-falante não é modificado por este script de instalação. Se a instalação ou atualização falhar, tenta restaurar os arquivos modificados desta vez, a configuração de captação e o estado anterior do serviço de repetição; se a restauração falhar, o erro é impresso claramente.

## Validação offline

```bash
python3 -m unittest discover -v
python3 duck_echo.py --help
```

Os testes de integração em Linux executam subprocessos reais com programas arecord/aplay/amixer falsos temporários, cobrindo gravação e reprodução, limite de tempo, desconexão, placa de som ocupada e restauração da configuração do microfone; não representam que o som no robô real tenha sido aceito.
