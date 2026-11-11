# Microduck Studio

> 🌐 Tradução não oficial para pt-BR do [original](https://github.com/fanhao375/microduck-replica). Em caso de divergência, vale o original. Licença: CC BY-NC-SA 4.0 (texto alterado: traduzido do chinês/inglês).

Console web instalado no pato. Abra `http://<IP do pato>:8000/` no navegador do computador ou do celular para ver o estado atual do pato, liberar o torque com um clique, e também ficar em pé/andar, parear controle (gamepad) Bluetooth (pelo `robotctl pad pair` oficial), calibrar os servos, calibrar a orientação da IMU, gravar essas duas coisas e a orientação no programa principal de uma vez, e fazer a verificação de montagem pela "lista de primeiros passos". Quando o pato abre o hotspot esperando configuração de rede, também dá para usar o celular para escolher o Wi-Fi de casa e digitar a senha.

## O que faz e o que não faz

- No dia a dia conversa com o programa principal do pato, o robotd (`/run/robotd.sock`, interface v40); para mexer diretamente nos servos na página de servos, primeiro faz o robotd entregar o barramento e, depois do uso, devolve.
- Todo estado na página vem da leitura de volta do robotd: por exemplo, "torque liberado ✓" só aparece quando o busStatus confirma o torque desligado, não porque "o botão foi apertado".
- Parar o Studio não afeta o pato; parar o robotd **não** faz os servos liberarem o torque (os servos mantêm o último comando), e a página avisa.
- **Sair do Studio não libera o torque**: se o Studio sair (reinício, atualização) com a sessão de barramento da página de servos aberta, apenas fecha a porta serial, e os servos que estavam segurando continuam segurando.
  Ao abrir o barramento grava `servo-session.json` no diretório state (porta serial, hora de início, quem abriu), e só o apaga quando "devolver ao programa principal" tem sucesso; quando o Studio sobe e o vê,
  e o robotd não está rodando, reassume automaticamente o barramento (aberto somente leitura, sem alterar o torque), e a barra de aviso no topo de cada página continua como antes; se não conseguir reassumir, a barra diz
  "da última vez que saiu, ainda podia haver servos segurando; só depois de assumir na página de servos dá para liberar o torque", e o botão de liberar torque no topo leva à página de servos. Se o robotd está rodando: ao subir ele já soltou tudo, e a marca é apagada.
- Durante a sessão de servos, a cada 5 segundos olha o socket do robotd: se ele foi iniciado manualmente (ele solta todos os servos e disputa a porta serial com o Studio), registra erro e o Studio fecha o barramento (sem escrever nos servos),
  e a barra de aviso fica vermelha. Para voltar ao programa principal, clique em "Devolver ao programa principal"; não use `systemctl start robotd` manualmente.

## Diretório

```
run.py              ponto de entrada (python3 -I run.py …): coloca deps/ no caminho de busca e depois entra no studio
run_tests.py        roda todos os testes
studio/
  log.py            log: categorias, níveis, tempo em milissegundos, deduplicação, mascaramento, rotação diária; comando registra "enviado → recibo · duração"
  auth.py           senha de administrador (scrypt), sessões de login, recuo por falhas, verificação de Host/Origin
  rpc.py            conexão JSON-RPC com robotd/configd (uma conexão, um pedido por vez)
  robotd.py         conexões: sondagem / fluxo de estado em tempo real / comandos / configd, uma permanente de cada; liberar torque abre sempre uma nova (após reinício do robotd não bate numa conexão morta)
  estop.py          máquina de estados de liberar torque: ready → pending → confirmed / unavailable; só considera leituras "enviadas depois do recibo de liberar torque"
  state.py          resume os dados brutos no estado enviado à página (postura, bateria, servos, IMU, controle, pendências, configuração de rede)
  models.py         biblioteca de modelos, restauração do padrão, leitura de volta da troca efetiva, remapeamento de botões do controle
  model_package.py  análise do pacote de implantação de modelo e pré-verificação do robotd sem hardware
  sensors.py        configuração da origem da câmera e snapshot sob demanda do ToF (independentes do controle de movimento)
  net.py            configuração de rede: lê o net.json do netcheck, net.status / scan / connect / forget do configd, liga/desliga hotspot
  app.py            serviço web: arquivos estáticos, login, logs, pacote de diagnóstico, WebSocket
web/                páginas (Preact + htm, three.js desenha o pato 3D; tudo arquivos locais, sem CDN; wifi.js é o cartão e as configurações de conexão Wi-Fi;
                    probe.svg é a imagem pequena usada para achar o novo endereço do pato depois de conectar ao Wi-Fi)
deps/               dependências Python com versão fixada (starlette, uvicorn etc., veja deps/VERSIONS.txt)
dev/
  record_robotd.py  grava, somente leitura, no pato, uma resposta real do robotd (para o robotd falso e testes)
  fake_robotd.py    robotd falso + configd falso: reproduz os dados gravados, roda no computador sem pato conectado
tests/              testes unitários + fixtures com dados gravados no robô real fixtures/now-20261018.jsonl
```

## Rodar no computador (sem pato)

Requer Python 3.13 (igual ao da placa; no Windows é `py -3.13`).

```bash
py -3.13 -I dev/fake_robotd.py --fixture tests/fixtures/now-20261018.jsonl \
    --net-mode hotspot --net-state-file ./state-dev/net.json   # 假 robotd + 假热点配网，听 17001/17002
py -3.13 -I run.py --listen 127.0.0.1:18080 --robotd tcp:127.0.0.1:17001 --configd tcp:127.0.0.1:17002 \
    --net-state ./state-dev/net.json --dev-autologin --state-dir ./state-dev --log-dir ./logs-dev
```

Abra `http://127.0.0.1:18080/` no navegador. O `--dev-autologin` só aceita requisições locais e só pode ser configurado com `127.0.0.1`, para facilitar capturas de tela e testes; não use no pato.
O robotd falso aceita `--mode commissioning|motion` para escolher o modo do programa principal (o cenário de fábrica é `--mode commissioning --calib factory`), e `--posture limp|policy|commissioning|starting` para trocar a postura; `--net-mode hotspot|wifi|none` escreve um net.json falso.
O modo de calibração segue o comportamento real do fork: ao subir, primeiro responde o relatório de inicialização (`phase=configuring_bus`), e só responde o snapshot em tempo real depois da primeira leitura bem-sucedida; `--commissioning-stuck-status` simula o fork antigo (o relatório de inicialização nunca some, e o Studio vê "iniciando" o tempo todo).
Sem `--net-state`, lê `/run/microduck/net.json`; se o arquivo não existir (no computador, imagens antigas) não mostra o Wi-Fi. Sem `--duck-label`, a interface usa o nome do host (no pato é `duck-xxxx`).

O Python do Windows não tem socket Unix, então o endereço é escrito como `tcp:host:porta`; no pato é `unix:/run/robotd.sock` (padrão).

## Câmera e ToF

À esquerda, depois de "Controle (gamepad)", há duas páginas independentes, "Câmera" e "ToF", e a barra inferior do celular também tem a entrada.

A barra de estado das páginas de visão geral / controle mostra, em ordem, bateria, servos, IMU, câmera, ToF, controle e placa. A câmera só usa o `hello` somente leitura do socket local do mediad para julgar "serviço online", e não trata arquivo de configuração nem imagem de teste como câmera real conectada; só ao abrir a página é possível ver o vídeo real. O ToF aparece online conforme novos quadros, e fake/sim é marcado claramente como dados simulados. A barra de estado verifica a cada 10 segundos, cancela ao trocar de página ou ir para segundo plano, e limpa o estado vencido; a verificação não pede quadros de vídeo, não estabelece WebRTC e não inicia serviços do sistema.

Descendo a visão geral até a área do controle, sob o pato 3D à esquerda há a pré-visualização da câmera, e à direita ficam o controle e o controle remoto. Na área de trabalho a coluna esquerda inteira fica fixa no topo, distribui postura e vídeo conforme a altura disponível e, se faltar espaço, rola junto com a página; no celular a ordem é desenho do controle, 3D, câmera, controle remoto. Só quando o cartão de pré-visualização está na tela, a visão geral está visível, e o relatório de verificação somente leitura indica que está configurada a câmera e o serviço de vídeo está online, é que se conecta automaticamente ao vídeo WebRTC existente. Fonte de teste, fonte desconhecida e serviço offline mostram um espaço reservado, verificam de novo a cada 10 segundos, e não reproduzem automaticamente a imagem de teste. Trocar de página, rolar o cartão para fora da tela ou ir para segundo plano cancela a leitura e libera a conexão de vídeo; a página independente da câmera ainda pode reproduzir ativamente a fonte de teste. Esse layout reutiliza o mesmo cliente de vídeo, não inicia serviços do sistema, e também não equivale a ter implementado a configuração automática por conexão a quente de hardware de câmera.

- **Câmera**: conecta ao vídeo WebRTC de rede local do mediad existente (porta de sinalização padrão 8443), podendo iniciar, parar e reconectar. A página só mostra "reproduzindo" quando a imagem real começa a atualizar; ao trocar de página ou ir para segundo plano, libera a conexão. Lê apenas o vídeo e os metadados da imagem, não abre a câmera do computador/celular e não envia comandos de movimento. A imagem é exibida conforme o ângulo de instalação informado pelo lado de execução.
- **Marca de origem**: `[media] source="test"` da imagem indica imagem de teste; mesmo que reproduza, não significa que uma câmera real esteja conectada. Quando configurado como camera, também mostra apenas "Configuração: câmera"; o arquivo de configuração não prova o estado de execução atual; quando o serviço de vídeo é um dispositivo simulado, isso é indicado à parte. A versão atual ignora a antiga chave `media.camera`, e o Studio não deduz a origem real por ela.
- **ToF**: obtém as zonas de distância 8×8 via `tof.stream` de `/run/tofd/tof.sock`, mostrando o número de células válidas, os valores mínimo/máximo válidos em milímetros e o estado bruto de cada célula. Só entram na estatística o estado 5/9 com distância positiva; sem alvo (255) e falha de medição são exibidos separadamente. Serviço desconectado, sensor não ligado, aguardando o primeiro quadro, formato inválido e dados vencidos têm cada um explicação própria. Dados fake/sim são marcados como "dados simulados".
- A leitura dos sensores é independente da conexão de movimento do robotd e só ocorre quando a página relacionada está visível; não há desvio automático de obstáculos, navegação nem gravação de vídeo em segundo plano, e não altera serviços do sistema nem configuração.

Em instalações avançadas, `--tof unix:/run/tofd/tof.sock`, `--camera-socket unix:/run/mediad/media.sock` e `--camera-signaling-port 8443` indicam os serviços locais; `--tof none` ou `--camera-socket none` desativam explicitamente a sondagem somente leitura correspondente. Em testes de desenvolvimento, os sockets podem apontar para `tcp:127.0.0.1:porta` controlado. A página não aceita socket nem endereço de servidor de vídeo arbitrários. A sinalização da câmera usa o nome de host com que o Studio foi aberto, e um túnel SSH que apenas encaminha a porta 8000 não encaminha automaticamente o canal de vídeo.

O `tests/test_sensors_e2e.py` local usa um ToF falso claramente marcado e uma implementação de teste de RTC do navegador, verificando reprodução real na página, navegação, erros e liberação de recursos; esses testes de software não substituem a aceitação do hardware de câmera, codificador e ToF.

## Gravação, reprodução e "灵动" de movimentos pela web

A área do controle remoto ganha "Gravação e reprodução de movimentos". Primeiro, pelo fluxo original, deixe o pato firme em pé e pronto para andar, desligue o controle físico, clique em "Iniciar gravação" e opere o analógico da página, cabeça/corpo/boca, vozes, ou clique em "Sentar／Levantar" (o direcional para baixo da página também serve). Gravar apenas "sentar→levantar" também pode ser salvo como um trecho; durante sentar/levantar, espere a confirmação do aviso de estado antes de operar o próximo passo. Também é possível ligar ao mesmo tempo o "灵动" (movimento vivo), que gera pequenos movimentos aleatórios de cabeça e boca, e ao começar a sentar/levantar o 灵动 é parado antes. Pausar a gravação não conta no tempo de espera; ao terminar, digite um nome para salvar.

Escolha um trecho na lista e reproduza 1, 3 ou 5 vezes; há pausar, continuar, parar, renomear, excluir, e o conteúdo salvo permanece após atualizar a página ou reiniciar. O 灵动 também pode ser ligado sozinho. O movimento continua sendo executado pelo robotd; a página não assume o barramento dos servos, e ligar o 灵动 não faz o pato levantar nem aplicar torque automaticamente.

Registra os comandos de controle finalmente autorizados a ser enviados e o ritmo, sem garantir repetir a mesma trajetória espacial; não grava vídeo da câmera, áudio do microfone, aplicar/liberar torque, iniciar/parar a política de andar nem alterações de configuração. Sentar e levantar são salvos como passos de objetivo explícitos; na reprodução lê-se a postura atual, sem apertar às cegas o botão de alternância; só depois que uma fase de política nova e leituras contínuas de junta real/alvo confirmam é que se executa o passo seguinte, e durante a espera a linha do tempo de reprodução fica em pausa. Registros com sentar/levantar são salvos como v2, os registros v1 antigos continuam legíveis; voltar a um programa que só suporta v1 ignora os v2, e ao restaurar o programa novo eles voltam a ser lidos; não é preciso apagar os registros originais.

Um registro que termina sentado pode ser reproduzido uma vez; ao escolher várias repetições, é preciso gravar de volta a postura em pé no fim, senão a página explica o motivo e não acrescenta um passo de levantar automaticamente. Pausa/parada normal ou sair da página só cancela os passos seguintes e limpa o controle contínuo; o sentar/levantar já entregue ao programa principal ainda será concluído; se precisar liberar o torque imediatamente, use a parada de emergência existente. Se a fase/leitura de sentar/levantar for incerta, esgotar a espera de 15 segundos ou perder o recibo, para os passos seguintes e explica o motivo, sem tentar a alternância de novo automaticamente.

Habilidades por modelo como chutar, rolar e pegar continuam sem ser gravadas, e as habilidades do controle remoto comum não são afetadas. Logo depois de executar essas habilidades por modelo, é preciso parar a política manualmente, começar a andar de novo e esperar a leitura nova confirmar prontidão, antes de iniciar a gravação/灵动; não se pode julgar o fim da habilidade por uma duração estimada.

Só há uma sessão de página por vez. Tomada manual por tecla/analógico primeiro para a reprodução automática; conexão do controle físico, página sair da área do controle remoto, segundo plano, perda de foco, desconexão, parada de emergência ou estado anormal param toda saída automática, e reconectar não retoma a reprodução. Pausar limpa a velocidade e os alvos de cabeça/corpo/boca, e ao continuar restaura a postura travada do ponto de pausa, com a velocidade esperando por quadros de ação novos; após mudança de configuração de modelo/controle, trechos antigos não são mais reproduzidos. No fim de cada ciclo, centraliza, sai do travamento do corpo e fecha a boca, e alvos contínuos atrasados são mesclados para não reenviar uma série de ações vencidas.

Cada trecho tem no máximo 60 segundos, 6.000 eventos e 1 MiB; no máximo 20 trechos, com nomes de até 32 caracteres. Os arquivos JSON versionados ficam em `motion-recordings/` no diretório state do Studio, com escrita atômica; registros corrompidos são ignorados e avisados na página, sem afetar outros controles nem registros utilizáveis. Ao desinstalar/atualizar, é preciso preservar esse diretório de dados do usuário.

A implementação se divide em `studio/motion_store.py` (validação/salvamento de arquivos), `studio/motion_studio.py` (sessão/linha do tempo/灵动), `studio/motion_posture.py` (alvos de sentar/levantar e julgamento de leitura), `web/motion-studio.js` (interface) e `web/motion-heartbeat.js` (página viva). A integração é por `op=motion` do `pad.op` existente, sem renomear o estado `pad.motion` original. O registro atual de desenvolvimento e aceitação está no [plano de implementação de movimentos web](../../docs/superpowers/plans/2026-10-25-web-motion-studio.md) e no [plano de extensão de sentar/levantar](../../docs/superpowers/plans/2026-10-25-web-posture-recording.md); a aprovação em simulação local e a aceitação no pato real são registradas separadamente.

2026-10-25: o c18783c foi implantado no endereço de controle original do pato atual, reiniciando apenas o Studio; 351 verificações de software e a leitura de volta de recursos estáticos/dados do usuário/identidade do serviço de controle passaram. Esse dispositivo tem proteção de login na web, e não se contornou o login para conferir o snapshot; o efeito real do movimento ainda aguarda confirmação do usuário. O pacote de código congelado, o backup e o alcance da reversão estão na [aceitação da implantação](../../docs/superpowers/verification/2026-10-25-web-motion-studio.md). A imagem e a atualização online genérica não foram atualizadas.

No mesmo dia, depois que o usuário confirmou a entrada por captura de tela, descobriu-se que a barreira de recuperação de habilidade após parar a política esperava, por engano, o interruptor de configuração estática ser desligado. O db970dd corrigiu conforme o estado de execução em tempo real de `bus.policy_enabled`, e avisa para clicar em "começar a andar" depois de confirmar que parou; 205 itens de backend e o fluxo completo de recuperação/gravação-reprodução no navegador passaram; atualizado no pato original, reiniciando apenas o Studio. O efeito real dos movimentos de cabeça e boca ainda aguarda confirmação do usuário, veja as correções posteriores no mesmo registro de aceitação.

Na sequência do mesmo dia, o aa825fc incluiu sentar/levantar na gravação, com 389 verificações de software e revisão independente aprovadas e implantação feita. O usuário confirmou "salva, e reproduzir sentar/levantar também funciona"; o alcance no robô real, a espera de estado e o limite de reversão v2 estão na [aceitação de sentar/levantar](../../docs/superpowers/verification/2026-10-25-web-posture-recording.md).

## Modo de bancada: controlador principal não gravado, ajustar os servos no computador com a placa de depuração Feetech

Conecte ao computador a placa de depuração Feetech (ou um USB-TTL), ligue os servos e a alimentação, e então:

```bash
py -3.13 -I run.py --bench
```

Ele não se conecta ao programa principal e abre automaticamente o navegador na página "Servos"; no menu "Porta serial" do cabeçalho escolha a porta COM e clique em "Conectar" para ajustar (igual ao servo-web).
Se quiser só dar uma olhada sem hardware: escolha em porta serial "Demonstração (sem servos conectados)", e o Studio finge por software que há 15 servos.

## Instalar no pato

1. Copiar: copie o diretório `software/studio` inteiro para `/home/radxa/studio-dev` no pato (`scp -r`, ou descompactar de um pendrive, tanto faz).
2. Autoverificação: no pato, execute `python3 -I run.py --selfcheck` (somente leitura: imprime a versão e pergunta uma vez a cada um entre robotd / configd / padd).
3. Instalar o pequeno auxiliar root: `cd deploy && sudo sh install.sh radxa`. O auxiliar 0.7.2 cuida da passagem do barramento de servos, do pareamento do controle, de ligar/desligar o hotspot, de `calib-read` / `calib-commit` (gravar a calibração no programa principal, passar ao modo de movimento), de `ssh-status` / `ssh-password` (a chave de SSH da página "Mais") e de `imu-recalibrate` (revalidar a orientação de instalação da IMU já salva); antes de desligar o hotspot, escreve `/run/microduck/studio-connecting.json`, para o netcheck não interferir durante a conexão. Ao devolver pela página de servos, o snapshot em tempo real do modo de calibração também conta como programa principal de pé (o 0.5.1 só reconhecia o `control` do modo de movimento, e no modo de calibração esperava sempre os 30 segundos completos e julgava "não sobe"). Passos, desinstalação e conferência no robô real estão em `deploy/INSTALL.md`.
4. Serviço de inicialização: instale conforme o comando no cabeçalho do arquivo `deploy/microduck-studio.service`:

   ```bash
   sudo install -m 644 microduck-studio.service /etc/systemd/system/ && sudo systemctl daemon-reload && sudo systemctl enable --now microduck-studio
   ```

   Para desfazer:

   ```bash
   sudo systemctl disable --now microduck-studio && sudo rm /etc/systemd/system/microduck-studio.service && sudo systemctl daemon-reload
   ```

5. Abra `http://<IP do pato>:8000/` e siga a "lista de primeiros passos" da página.

O caminho principal do pareamento do controle é o auxiliar root chamando o `/usr/local/bin/robotctl pad pair --json` oficial: na placa `/var/lib/robot/weird-ble` ele primeiro para o `btd`, desliga e liga a alimentação do adaptador Bluetooth, e ao terminar o pareamento sobe o `btd` de volta. Sem o auxiliar instalado ou com versão antiga, recorre ao configd → bluetoothctl; nesta placa isso pode não conseguir parear, e o assistente avisa para reinstalar o auxiliar, ou executar `sudo robotctl pad pair` no pato.

Resultado do pareamento não equivale a botões prontos. Depois que o helper/configd retorna "vinculado", o assistente, ao esperar a conexão e a entrada do padd, faz no máximo duas solicitações de conexão adicionais ao controle alvo, cada uma de no máximo 5 segundos e sem exceder a janela de espera atual; não limpa o pareamento nem reinicia o Bluetooth. Só um relatório de entrada do padd com o mesmo MAC confirma a prontidão; outros controles conectados não contam. Quando o serviço de estado atualiza devagar, ou a entrada chega bem na borda do tempo-limite, vale a entrada real; se o mesmo controle voltar pouco depois, também limpa o aviso antigo do assistente "não conectado／não lê os botões". Se o usuário cancelou, o padd não está rodando ou começou a andar, não continua tentando reconectar. A explicação de degradação da versão antiga sem desvio de entrada continua marcada claramente como "não dá para conferir de novo".

Essa reconexão é para o fluxo de pareamento pela web; quando o controle já foi pareado e apenas está em repouso, primeiro aperte brevemente a tecla Home / Xbox para acordá-lo, sem precisar esquecer e parear de novo a cada vez. Esta correção não oferece reconexão infinita em segundo plano, nem significa que o comportamento de reconexão depois de desligar a energia tenha sido aceito modelo a modelo.

A imagem de fábrica faz o robotd rodar em modo de calibração: em `/etc/systemd/system/robotd.service.d/zzzz-zz-microduck-commissioning.conf` o `ExecStart` leva `--commissioning`. Assim `calibration-status` e `busStatus.last_sample` podem ser lidos imediatamente, e a calibração grosseira na página da IMU e o ponto médio na página de servos funcionam. Quando o passo 3 da lista de primeiros passos é gravado com sucesso, o auxiliar apaga esse arquivo, faz `daemon-reload` e reinicia o robotd, e o programa principal volta ao modo de movimento. Se `systemctl show robotd -p ExecStart` tem `--commissioning`, mas esse arquivo não existe, o auxiliar recusa a troca automática (`foreign_commissioning`); vá à página de logs exportar o pacote de diagnóstico.

A "Orientação de instalação da IMU" da página da IMU sempre mantém a entrada: mesmo um quaternion de instalação já salvo e validado pode ser reaferido com "Recalibrar orientação". Primeiro segure o pato e libere o torque; depois da segunda confirmação, o modo de movimento volta temporariamente ao modo de calibração pelo novo auxiliar root; quem já está no modo de calibração recoleta diretamente. Mantém o procedimento original: parado 3 segundos, o tronco inteiro inclinado para a frente 15–30° coletando 3 segundos, cálculo da orientação e validação pelo programa principal. **Antes de clicar em "Entregar ao programa principal para validar", realinhe o pato real e deixe-o parado**, e só então incline para a frente conforme o aviso. A sessão de recuperação da recalibração do auxiliar mantém o torque liberado; o modo de calibração de fábrica também suporta a coleta na postura em pé descrita a seguir, e o fluxo de refinamento em pé não muda.

A IMU do barramento com ID 200 é reconhecida automaticamente pelo programa principal na inicialização, com suporte a FT6, IMU-to-DXL (Feetech); SFLP 124/12 (Dynamixel 2.0). As páginas da IMU nos modos de calibração e movimento mostram o protocolo e a placa realmente em uso. O nome da placa indica o layout de registradores reconhecido, não prova o fabricante. Quando for preciso especificar manualmente, mude na configuração do programa principal `bus.imu_protocol = "auto" / "feetech" / "dxl2"`; a página não tem botão para trocar o protocolo durante a execução. O fluxo do BNO085 e do LSM6DSV16X via I²C não muda.

A seleção web única do Qinmo, a reconexão controlada e a validação de orientação de instalação obrigatória estão no [guia de uso](../../docs/Qinmo-IMU-网页使用.md). O tipo configurado e a conexão real são exibidos separadamente; o Qinmo é marcado claramente como configurado, e o fabricante não pode ser certificado automaticamente por ID200 ou layout. O novo helper 0.7.2 também mantém `imu_bus_profile` (padrão legacy), e uma mudança de tipo, mesmo ambos Feetech, também invalida a orientação antiga e o resultado pendente de gravação. A verificação local de software não equivale a aceitação no robô real, e a imagem publicada não foi implantada nem atualizada.

O `imu_bus_protocol` da orientação de instalação no barramento é gravado depois que o programa principal valida com sucesso; calibrações antigas sem esse campo são compatíveis apenas com Feetech. Quando o protocolo detectado não confere, a página pede recalibração, e pular o passo 2 também não pode tratar a orientação antiga como validada. O programa principal mantém a entrada de refazer pelo estado desabilitado do modo de movimento. O passo 3 e o helper 0.7.1 preservam e leem de volta esse campo; ao concluir o refazer pode-se adotar o novo vínculo, e cancelar restaura o arquivo original. Mudança de protocolo, do layout da placa ou dos bytes de identidade bruta invalida a coleta atual, o cálculo em andamento e o resultado antigo da calibração em pé. Placas DXL2 não fornecem número de sequência de amostra, bits de estado nem idade da leitura do sensor; o programa principal usa a atualidade de recepção do host e a verificação de congelamento dos bytes brutos, e a página não inventa esses valores de diagnóstico.

Ao fazer pela primeira vez a calibração de orientação de instalação no modo de calibração de fábrica, também se pode clicar manualmente antes em "Habilitar postura em pé": espere o HOME segurar firme (erro ≤5°), segure o tronco inteiro com as duas mãos coletando parado e inclinado, e depois realinhe e entregue ao programa principal para validar. São permitidos os dois estados, torque liberado ou claramente firme em pé; levantando, torque desconhecido, em movimento e liberação de torque pendente de confirmação continuam recusados. A sessão de recuperação de recalibração que entra do modo de movimento pelo auxiliar continua exigindo torque liberado o tempo todo, para evitar que reiniciar o programa principal ao restaurar o modo afete a postura em pé. A coleta não aplica torque nem inicia movimento automaticamente.

A sessão de recalibração preserva a calibração original e a configuração de inicialização, pode continuar depois de atualizar a página ou reiniciar o Studio, e também permite "Cancelar e restaurar parâmetros originais". Depois que a validação passa, lê de volta item a item o chip atual e a nova orientação, restaura apenas a permissão de movimento e o modo de execução originais; o zero dos servos (inclusive zeros diferentes de 2048), sentido, limites, outras IMUs e configurações desconhecidas são todos preservados; restaurar com sucesso ainda mantém o torque liberado. Se falhar, mantém a entrada de restauração e não declara como concluída uma orientação que não foi lida de volta. Esse fluxo exige atualizar juntos o Studio, o auxiliar root e o sudoers; não chama o passo 3 da lista de primeiros passos para reescrever a calibração dos servos.

**O estado do modo de calibração exige o fork novo**: assim que sobe, o fork do robotd escreve um relatório de inicialização (`{"mode":"commissioning","phase":"configuring_bus",...}`), e `robot.busStatus` continua devolvendo-o enquanto a phase ainda for de inicialização; só depois da primeira leitura bem-sucedida responde o snapshot em tempo real do modo de calibração (phase `ready` / `moving` / `read_error` / `stop_unconfirmed` / `power_pending`, com `last_sample`). O modo de calibração do fork antigo nunca limpa o relatório de inicialização, e o Studio ficaria para sempre vendo "iniciando" sem obter `last_sample` (a aceitação no robô real de 10-20 foi assim); o fork já foi alterado para limpá-lo depois da primeira leitura bem-sucedida, use uma imagem posterior a 20261020 (ou reinstale esse robotd). Do lado do Studio: estas phases do snapshot em tempo real contam todas como "programa principal de pé" (`read_error` é de pé, apenas com erro ao ler os servos, e repete-se a mensagem original do programa principal); se reportar fase de inicialização por mais de 30 segundos seguidos (contados desde esta conexão), o cabeçalho da visão geral muda para "o programa principal reporta 'iniciando' o tempo todo" com "ir à página de logs", e a barra superior, as pendências e o topo da lista de primeiros passos também deixam de dizer "normalmente 10 segundos".

Quando a bateria é alimentada por fonte de bancada, em "Configurações → Bateria" escolha o modelo "Fonte de bancada": mostra apenas a tensão, sem calcular nem reportar carga.
O Studio por padrão não tem senha, e qualquer um na mesma rede pode abri-lo. Em "Configurações → Senha de login da web" dá para defini-la diretamente; depois de definida, é preciso logar antes de alterar, e também pode ser removida. O comando `python3 -I run.py set-password` continua utilizável, mas deve ser executado com o usuário radxa, sem `sudo` (o Studio não consegue ler um arquivo de senha escrito pelo root).

### Conectar o assistente de IA do computador via SSH

Abaixo da faixa do painel da visão geral e no topo da página de configurações há a entrada "Deixar a IA conectar ao pato (SSH)"; ao clicar vai a `#/settings/ssh` e posiciona nas instruções de três passos. Não é preciso monitor; o computador precisa antes conseguir acessar o endereço web／da placa do pato.

1. **Senha de login da web**: para a página de controle no navegador, com pelo menos 6 caracteres. Se já foi definida, mostra "Definida" e só expande quando for preciso alterar.
2. **Senha de login do sistema SSH**: para o terminal do computador ou o assistente de programação com IA fazerem login na conta de sistema da placa (normalmente `radxa`), 8–64 caracteres. Primeiro defina a senha de login da web, depois ative conforme necessário; para o controle web do dia a dia pode ser pulada. Se já está ativada ou já há chave pública, não é preciso redefinir a senha, e o registro／remoção de chave pública permanece na área expansível.
3. **Instruções de conexão para copiar à IA**: mostra o endereço da placa, o nome de usuário, a porta SSH padrão 22 e o comando de login. Só aparece ao ler que o serviço SSH está rodando e que já há senha／chave pública como forma de login. O conteúdo copiado inclui a forma de conexão, não a senha; a senha do sistema é digitada pela própria pessoa quando o terminal conectado pedir. Se o navegador não permitir a cópia automática, mostra o texto completo selecionável para cópia manual.

As duas senhas são salvas separadamente e alteradas de forma independente; "Confirmar senha" é digitar de novo a mesma senha que se está definindo, para evitar erro de digitação, e o login normal pede apenas uma vez. Depois de salvar a senha web pela primeira vez, o front-end atualiza a sessão do navegador atual e o CSRF e só então entra nas configurações de SSH. O estado SSH mostrar configurado não significa que o computador já conectou com sucesso; deve-se primeiro fazer uma verificação somente leitura da conexão. Com estado desconhecido ou serviço parado, a página oferece atualizar e a indicação do motivo.

Esta parte é a entrada de depuração remota para o assistente de programação com IA no computador. Os testes de navegador de desenvolvimento usam um fixture de helper SSH isolado, sem alterar de fato a senha do sistema do computador, o serviço SSH nem as chaves do usuário; não substitui a aceitação ponta a ponta no robô real.

## Lista de primeiros passos: Fazer, Concluído, Pular

Cada linha tem sempre três botões (usuário 10-20: "pode ser que alguém já tenha feito e queira passar de novo" "permitir que as pessoas pulem direto"):

- **Fazer**: vai à página daquele passo; linhas em que o teste encontrou problema dizem "Refazer".
- **Concluído**: eu já fiz antes (por exemplo, o zero do pato nº 2 já foi calibrado há tempo na página de servos do Studio), a linha fica ✓ e diz "você marcou como concluído (hora)". O antigo "já foi calibrado, conferir" é isto.
- **Pular**: não fazer por ora, a linha fica ↷.

As linhas marcadas oferecem "Refazer" e "Cancelar" (as puladas, "Cancelar pulo"). A marca fica presa à impressão digital daquele passo (zero dos servos, orientação da IMU, calibração, controle...); se a impressão mudar, deixa de valer e volta a "a fazer"; se a impressão não pôde ser lida na hora da marca, ela é presa quando for lida.
Situações que antes ficavam bloqueadas — o programa principal diz que não terminou, não consegue ler (?), o teste encontrou problema, testar parada de emergência, ajustar servos — agora podem todas ser clicadas, mas "Concluído" e "Pular" exigem dois cliques, o segundo botão mostra a consequência (algo como "Confirma pular? O programa principal diz que a calibração não terminou: se pular, o pato também não consegue levantar"), e o backend também recusa se não receber a confirmação.
Se o passo 1 foi marcado como concluído ou pulado, na prontidão do passo 3 "Gravar no programa principal" o "zero dos servos" conta como satisfeito (os 15 servos gravam o zero como 2048; o zero só está correto se foi calibrado pela página de servos do Studio).
Se o passo 2 foi marcado como concluído ou pulado, a orientação da IMU em uso no programa principal não foi confirmada e a página da IMU também não fez calibração em pé (não há `imu-pending.json`), o passo 3 **grava essa IMU conforme a instalação padrão e a marca como confirmada** (aceitação no robô real de 10-21: depois de pular tudo, o fork só entra no modo de movimento com a orientação da IMU confirmada, e o pato ficou o tempo todo no modo de calibração):

- Placa no barramento (`bus`): o `imu_mount_quat` já existente no arquivo (de fábrica é a instalação oficial `[0.7071067811865476, 0, 0.7071067811865476, 0]`);
- BNO085 (`bno08x`): `[0.5, 0.5, 0.5, 0.5]`, o `BNO08X_DEFAULT_MOUNT` de `duck-control/src/trunk_imu.rs` do fork;
- LSM6DSV16X (`lsm6dsv16x`): `[0.7071067811865476, 0, 0.7071067811865476, 0]`, a instalação recomendada da réplica (módulo com x para baixo, z para a frente, medido no robô real do pato nº 2 em 10-17). O padrão do próprio fork é o quaternion unitário ("a placa pequena não tem uma direção para cima consensual"), que não é usado.

A prontidão escreve "Você marcou o passo 2 como concluído: IMU gravada conforme a instalação padrão (chip: qual instalação). Se a instalação for diferente, ao levantar o pato vai tombar para um lado — depois de gravar, calibre uma vez em pé na página da IMU e grave o programa principal de novo"; o segundo clique em "Gravar no programa principal" diz "Confirma? IMU gravada conforme a instalação padrão". Se a página da IMU fez calibração em pé (existe `imu-pending.json`), usa-se isso como antes, e se a orientação já está confirmada no programa principal usa-se a do programa principal como antes; se o que a página da IMU registrou é de outro chip, ou o programa principal ainda não informou qual está em uso, não se adivinha e continua bloqueando como antes. O `calib-commit` do auxiliar continua aceitando apenas `imu: {source, quat}`. Assim, **pular os passos 1 e 2 → passo 3 gravar no programa principal → modo de movimento** funciona de ponta a ponta.

## Como escolher o endereço para abrir o Studio

Um computador Windows com o proxy de sistema do Clash ligado não abre `duck-xxxx.local` (passa pelo proxy, 502); `duck-xxxx` sem ponto abre (o Windows pergunta ao pato por LLMNR, e o proxy contorna por `<local>`). Por isso a página escreve por dispositivo:

- Computador Windows: `http://duck-xxxx:8000/` (abre mesmo com proxy ligado);
- iPhone, Mac: `http://duck-xxxx.local:8000/`;
- Celular Android, ou se nenhum abrir: procure `duck-xxxx` na lista de dispositivos do roteador e use o IP dele;
- Computador com o modo TUN do Clash ligado ainda não abre: no Clash configure a rede local (192.168.0.0/16) como conexão direta.

A página "Mais" tem um bloco "Endereços deste pato", listando os IPs utilizáveis agora (`ip4` do `net.status` do configd; no modo hotspot é 10.42.0.1), `duck-xxxx` e `duck-xxxx.local`, cada um copiável; na barra superior da visão geral, passar o mouse sobre o nome do pato também mostra o IP. Na lista de Wi-Fi para conectar, o hotspot do próprio pato é filtrado (conforme `hotspot_ssid` do net.json; se não houver, filtra os "começando com `Microduck-` e já salvos").

**Depois de conectar ao Wi-Fi, pula automaticamente para o novo endereço do pato** (usuário 10-21: "conectar ao Wi-Fi deu três voltas"): ao clicar em conectar, a página anota os `http://duck-xxxx:8000/` e `http://duck-xxxx.local:8000/` a procurar (o nome do host vem do net.json, e a porta acompanha o `--listen`).
Depois que a conexão da página com o pato cai (o WebSocket caiu, ou 5 segundos sem mensagem), o cartão muda para "O pato está conectando a 'Wi-Fi de casa'... conecte este dispositivo de volta ao 'Wi-Fi de casa' (computador com cabo de rede não precisa), e quando achar o pato abre automaticamente", listando esses dois endereços (clicáveis) e "Se não achar: procure o IP de duck-xxxx na lista de dispositivos do roteador".
A cada 2 segundos carrega com `new Image()` o `<endereço>probe.svg?t=<hora>` (`web/probe.svg`, que como os demais arquivos estáticos não exige login nem usa cache; carregar imagem entre origens não precisa de CORS), e o primeiro que carregar com sucesso recebe o salto; uma sondagem espera no máximo 6 segundos e não repete o mesmo endereço enquanto não houver resposta.
Procura por no máximo 3 minutos, e se não achar volta ao texto original de abertura por dispositivo. Quando a página reconecta ao endereço original (o hotspot voltou, significa que não conectou), a sondagem para e continua mostrando o motivo da falha.

## Na imagem vs instalação manual

O cartão "Conectar ao Wi-Fi" depende de várias coisas da imagem da réplica, que não vêm com o próprio Studio:

- o estado de configuração de rede `/run/microduck/net.json` escrito pelo netcheck;
- a configuração do hotspot `Microduck-Hotspot` no NetworkManager;
- o configd rodando com `--allow-user btd,radxa`, para o Studio ter permissão de chamar `net.scan` / `net.connect` / `net.forget`;
- a configuração de mDNS, para o celular achar o pato por `duck-xxxx.local`.

Em patos em que o Studio e o auxiliar root foram instalados manualmente conforme `deploy/INSTALL.md`, mas sem o overlay da imagem da réplica, essas coisas não existem, e a página não mostra o cartão "Conectar ao Wi-Fi"; visão geral, controle, servos, IMU e logs continuam utilizáveis normalmente.

## Convenção entre a página e o Studio (WebSocket `/ws`)

No handshake verifica Host e Origin, e se não conferirem recusa o handshake (o navegador vê apenas 1006, e o motivo vai para o log); quem não está logado é aceito primeiro e depois fechado com 4401, e ao ver 4401 a página vai para a tela de login.

| Direção | Mensagem | Descrição |
|---|---|---|
| Studio → página | `{"t":"hello","version","duck"}` | Enviada uma vez ao conectar |
| | `{"t":"state", …}` | 10 Hz com a página visível; campos em `summarize` de `studio/state.py` |
| | `{"t":"hb"}` | 1 Hz com a página em segundo plano (nesse caso não assina o estado em tempo real do robotd, poupando CPU) |
| | `{"t":"log","entry":{…}}` | Novo log |
| | `{"t":"result","id","ok","reason","data"}` | Recibo de comando; dados como a lista de redes da varredura de Wi-Fi ficam em `data` |
| | `{"t":"pad", …}` | Snapshot do controle: 10 Hz quando o bloco do controle está na tela, 1 Hz quando não está (a célula do controle na visão geral precisa); quando listas longas não mudaram, leva `lists_same` e não leva a lista |
| Página → Studio | `{"t":"hb","seq","visible","route","pad_visible"}` | Uma vez por segundo; ao entrar e sair da área do controle, envia de novo imediatamente. `pad_visible` = o bloco do controle está na tela (#/pad é só uma âncora na visão geral) |
| | `{"t":"cmd","id","name","args"}` | relax, servo.*, imu.op, calib.op (`preview` / `commit`: passo 3 da lista de primeiros passos), pad.op, onboard.op (da lista de primeiros passos: `done` concluído, `skip` pular, `undone` / `unskip` cancelar, `{step, confirmed}`: só o segundo clique, dos dois, leva `confirmed: true`; o antigo `confirm_servo` ainda é reconhecido), net.op (`scan` / `connect` / `forget`) |

Campos da sessão de servos em `state` (usados pela barra de aviso no topo de cada página): `servo_session` (existe só quando aberta: `holding` quantos estão segurando, `unknown` quantos não têm torque legível,
`restored` se foi reassumida depois do reinício do Studio), `servo_handback_s` (presente depois de devolver, até o programa principal conectar), `servo_orphan` (estava aberta na última saída e desta vez não foi reassumida),
`servo_conflict_s` (durante a sessão o robotd foi iniciado manualmente, dentro de 10 minutos). `servo.close {if_idle:true}` (sair da página de servos) só devolve se o torque dos 15 foi lido na hora e todos são 0,
nenhuma outra página está na página de servos e não há calibração / movimento em curso; `servo.close {}` (clicar em "Devolver ao programa principal") primeiro espera terminar as operações de servo em trânsito, depois libera o torque e devolve.

`state.starting_s` / `state.starting_long`: quantos segundos o programa principal vem reportando fase de inicialização desde esta conexão, e se passou de 30 segundos (usado pelo cabeçalho da visão geral, barra superior e pendências para mudar o texto para "reporta iniciando o tempo todo").

`state.net` é o estado da configuração de rede: `known=false` significa que não há net.json (todo o bloco de Wi-Fi fica oculto); caso contrário traz `mode` (wifi / hotspot / none), o `ssid` / `signal` / `ip4` atuais, `hostname`,
`url_local` (`duck-xxxx.local`, a porta acompanha o `--listen`), `url_host` (`duck-xxxx` sem ponto), `ip_now` (o IP utilizável agora: no modo hotspot é o endereço do hotspot),
`urls` (as linhas de "Endereços deste pato": `{kind: ip / host / local, url, note}`), o hotspot do pato `hotspot {ssid, ip, psk}`, o último resultado `last_try`, `busy`. `net.op scan` filtra o hotspot do próprio pato. `net.op connect` primeiro responde "começando a trocar" (a página usa `url_host` / `url_local` como os endereços a procurar depois), e desligar de fato o hotspot e chamar o configd é feito em segundo plano;
o resultado é escrito em `net-last-try.json` no diretório state (sem a senha). A senha do Wi-Fi de casa passa da página ao configd uma única vez, e não entra em log, state nem pacote de diagnóstico.

Redes criptografadas já salvas também exigem digitar a senha a cada vez: o configd oficial, diante de rede criptografada sem senha, responde `unsupported`. Senha errada primeiro faz `net.forget` da configuração estragada desta vez e logo reabre o hotspot;
`timeout`, `other` e o próprio Studio esgotando o tempo do configd também reabrem o hotspot imediatamente. Quando o Studio espera 60 segundos sem resposta, registra à parte "o programa de configuração de rede do pato ficou 60 segundos sem responder", sem misturar com o `timeout` do configd.
Antes de desligar o hotspot o auxiliar escreve `/run/microduck/studio-connecting.json` (o netcheck não interfere por 90 segundos), e o apaga depois de reabrir o hotspot com sucesso.

A página julga "sem conexão" apenas pelo intervalo entre mensagens recebidas (cinza em 1 segundo, alarme em 3 segundos), sem olhar os relógios dos dois lados (o relógio do pato pode estar em 1970).

## Logs

`logs/studio-YYYY-MM-DD.log`, mantidos por 14 dias, no total não mais que 50 MB. Uma linha por evento:

```
2026-10-18T00:41:56.194+09:00 [命令] → 卸力（电脑 127.0.0.1） · 步骤：robot.relax
2026-10-18T00:41:56.197+09:00 [命令] ← 卸力 成功 · 3 ms · 已发出，等回读确认 · 步骤：robot.relax · 原文：{"accepted": true}
```

Em caso de erro, escreve "travou em: qual etapa · texto original: … · sugestão: … · contexto: {…}". Senhas e tokens são mascarados automaticamente. A mesma frase repetida em 1 minuto é registrada uma só vez, indicando quantas vezes foi mesclada.
A página "Logs" mostra as últimas 500 entradas e também exporta o pacote de diagnóstico (logs + estado da hora + respostas brutas do robotd + versão + log de sistema do robotd), para enviar ao grupo ao pedir ajuda.

Todo comando no log registra envio e recibo, com duração e contexto; o código é escrito por nós mesmos.

## Testes

```bash
py -3.13 -I run_tests.py               # 电脑
python3 -I run_tests.py                # 鸭子
```

Regressão direcionada da integração da IMU de barramento (inclui páginas com Edge headless; sem Edge / Node >=22, os itens de navegador são ignorados):

```bash
py -3.13 -I -c "import sys;sys.path[:0]=['deps','.','dev'];import unittest;unittest.main(module=None,argv=['x','tests.test_imu_backend','tests.test_commit','tests.test_helper_calib','tests.test_helper_imu_redo','tests.test_imu_bus_ui','tests.test_imu_wizard_e2e'])"
```

Cobertura:
- Login: sessão, recuo após 5 senhas erradas, sair exige CSRF; Host 421, matriz de Origin; WebSocket sem login recebe 4401;
- Log: formato, mascaramento, deduplicação, limpeza diária (limpa mesmo com o relógio parado em 1970);
- JSON-RPC: meia linha, linha longa demais, lixo e JSON que não é objeto, reconexão por tempo-limite;
- Liberar torque: não é bloqueado por consulta lenta, confirmação por leitura de volta, flag hd1910 não limpa não conta como confirmação, modo de calibração, sucesso já na primeira vez após reiniciar o robotd, depois da confirmação, outro aplicar torque não faz reenvio automático;
- Sondagem: um item com erro não bloqueia os demais; só assina quando há alguém olhando, sem heartbeat é tratado como ninguém olhando; robotd cai e volta;
- Julgamento de postura (com dados gravados no robô real); ida e volta completa de liberar torque via WebSocket;
- Pareamento do controle: validação de parâmetros de `pad-pair` / `pad-forget` do auxiliar, interpretação de sucesso / falha / tempo-limite do robotctl; o caminho principal do assistente, o retorno ao auxiliar antigo, nova tentativa quando o registro do agente do bluetoothctl falha, espera de 30 segundos do padd, reconexão limitada após o pareamento, entrada real antes do serviço de estado, borda do tempo-limite e cancelamento, entrada tardia do mesmo controle restaura o estado.
- Configuração de rede: verbos de hotspot do auxiliar, arquivo da janela de conexão; net.json ausente / corrompido / cada modo; cache de varredura; ordem de conexão, motivos de falha (bad_key faz forget e depois abre o hotspot), recuperação do hotspot, conecta apenas um por vez; a senha não entra em log / state / last-try / pacote de diagnóstico.
- Senha de login: primeira definição pela web, alterar só depois de logado, remover; alterar a senha preserva a sessão atual do navegador; `set-password` executado como root é recusado; aviso do cartão no modo hotspot sem senha.
- Estado do modo de calibração: o robotd falso primeiro responde o relatório de inicialização (quatro chaves), depois da leitura bem-sucedida responde o snapshot em tempo real com campos do fork, `--commissioning-stuck-status` responde sempre o relatório de inicialização; o snapshot em tempo real conta como programa principal de pé (visão geral, prontidão e devolução do auxiliar), com o fork antigo em `configuring_bus` o tempo todo a devolução continua julgando que não subiu, dizendo a verdade; reportar fase de inicialização por 30 segundos seguidos muda o texto para "reporta iniciando o tempo todo".
- Os três botões da lista de primeiros passos: cada situação antes bloqueada pode "Concluído" / "Pular" (confirmação em dois passos, texto do segundo correto, backend recusa sem confirmação), registro e invalidação de "Concluído", impressão digital ilegível registra primeiro e prende depois, passo 1 marcado → zero dos servos do passo 3 satisfeito, passo 2 marcado mas orientação não confirmada → grava a IMU na instalação padrão (uma para placa de barramento / BNO085 / LSM6DSV16X: prontidão satisfeita, quaternion entregue ao auxiliar correto, leitura de volta passa na validação do modo de movimento), com `imu-pending.json` usa-o, com o programa principal já confirmado usa o do programa principal, chip que não confere continua bloqueando como antes.
- "Habilitar / Desabilitar postura em pé" da página da IMU: no modo de calibração, ao clicar mostra confirmação e envia `imu.op hold`, o programa principal recebe `hold-home` de 15 direções + 3000 ms, depois de firme em pé escreve "modo de calibração (em pé)", desabilitar vai por `relax` e volta a mole; no modo de movimento o comando enviado é idêntico ao "Levantar" do cartão de controle remoto e ao "Liberar torque" do topo (ponta a ponta no navegador).
- Endereços: lista de endereços nos modos hotspot / Wi-Fi, o resultado da varredura filtra o hotspot do próprio pato, três tipos de endereço no aviso de troca.
- Achar o pato depois de conectar ao Wi-Fi: `probe.svg` sem login, os dois nomes de host entregues, sem cache; lógica de sondagem (Node roda o código real de `web/wifi.js`, com relógio falso) sonda dois endereços a cada 2 segundos, salta para o primeiro que responder, desiste em 3 minutos, para quando o hotspot volta, não repete sondagem que ficou pendurada; ponta a ponta no navegador vê o cartão depois da desconexão, para quando o hotspot volta, salta quando acha.
- Gravar no programa principal: `calib-read` do auxiliar só leitura / sem pegar trava, `calib-commit` pega a trava primeiro e depois verifica o flag, no modo de calibração apaga o drop-in e depois passa ao modo de movimento, só conta como sucesso quando `mode=motion` + `phase=control` + `calibration-status` leem de volta tudo consistente, erro de calibração / erro de barramento de `waiting_for_bus` reportados separadamente, leitura inconsistente / falha ao escrever `.saved.toml` / falha ao reiniciar fazem todas rollback (arquivo e modo reportados separadamente, se após o rollback não sobe, diz a verdade), recusa se a política está rodando, recusa se `ExecStart` tem `--commissioning` mas o drop-in não existe, backups mantidos em 10 grupos por número de sequência; prontidão do Studio nos dois modos, fallback de `calib-read`, tempo-limite lê `last-result.json`, mudança de pending / centered faz o passo 3 voltar a "a fazer", desfazer a calibração do ponto médio apaga o registro; os dois TOMLs do grupo geram a mesma saída em código.

## Estado atual (primeira versão)

O que funciona:
- **Visão geral**: estado em tempo real, pato 3D, bateria, temperatura da placa, liberar torque com um clique (a barra de espaço também serve).
- **Página de servos**: varredura, calibração do ponto médio de um ou de todos (gravada direto nos servos), sentido, postura, P/D/limite de força.
- **Página da IMU**: calibração grosseira no modo de calibração de fábrica (segurar alinhado + inclinar a cabeça para a frente, entregue ao `verify-imu` do programa principal para validar), e depois que consegue ficar em pé, refinamento em pé.
  Acima de "calibração em pé" há "Habilitar postura em pé" e "Desabilitar (liberar torque)" (pelo mesmo caminho do "Liberar torque" do topo: `relax`), e ao lado uma linha diz se está agora em pé / mole / levantando.
  No modo de movimento, "Habilitar postura em pé" usa o mesmo caminho do "Levantar" do cartão de controle remoto (`pad.op stand`, e o backend julga de novo conforme a postura);
  **no modo de calibração também dá para ficar em pé** (usuário 10-21: "não seria bom a gente poder ficar em pé também na calibração? Precisa mesmo mudar para o modo de movimento?"): envia `imu.op hold` → `hold-home` do modo de calibração do fork,
  os 15 servos, da posição atual, giram juntos em 3 segundos até o HOME e travam, a IMU lê normalmente; a orientação é a mesma que o passo 3 grava no programa principal, e o zero usa o do arquivo do programa principal (de fábrica 2048).
  No modo de calibração o Studio lê o `busStatus` a cada 0,1 segundo, e `last_sample.imu` serve de leitura em tempo real: a inclinação do bloco "agora" e a calibração em pé funcionam, e mesmo com a orientação não confirmada dá para calibrar em pé (calcula uma orientação nova inteira, e o passo 3 a grava).
  Enquanto está em pé, o passo 3 fica bloqueado ("primeiro clique em Desabilitar"), porque gravar no programa principal exige reiniciar o programa principal e o pato desabaria na hora. Com o pato mole, ao iniciar a calibração em pé, avisa para primeiro clicar em "Habilitar postura em pé".
- **Login remoto por SSH** (página "Mais", usuário 10-21: "se algo der errado eles consertam por SSH sozinhos, sem regravar o cartão"): a imagem de fábrica só aceita chave, não traz a chave de ninguém e não tem senha padrão para login remoto. O usuário primeiro define a senha de login da web (sem ela o backend recusa: senão qualquer um conectado ao hotspot abriria a página e de quebra ligaria o SSH), depois define a própria senha SSH (8–64 caracteres, não pode ser algo que todos adivinham como microduck), `POST /api/ssh {action: on}` → auxiliar `ssh-password`: `chpasswd` define a senha do radxa pelo stdin, escreve `/etc/ssh/sshd_config.d/05-microduck-ssh-password.conf` (ordenado antes do `10-microduck.conf` da imagem), `sshd -t`, `sshd -T` lê de volta que o login por senha realmente foi aberto e só então `reload`, e se qualquer passo falha, apaga o arquivo e volta atrás. Desativar = apagar esse arquivo e dar reload. **A senha só passa pelo stdin do auxiliar, e não entra na linha de comando, log nem arquivo de resultado** (os testes conferem item a item). O cartão mostra o comando de login `ssh radxa@<este endereço agora>`.
  "Alternativa 2: registrar chave" (`studio/sshkeys.py`): o Studio já roda como radxa, e `~/.ssh/authorized_keys` é arquivo dele mesmo, sem passar pelo auxiliar. Só aceita uma linha inteira de chave pública (ed25519 / rsa / ecdsa / sk-*), o tipo dentro do base64 deve coincidir com o escrito no início, e não pode haver opções como `command=` na frente; colar uma chave privada é reconhecido e barrado. A mesma chave (pela impressão digital SHA256, como `ssh-keygen -l`) não é gravada duas vezes; excluir é por impressão digital. Diretório 700, arquivo 600, e recusa se `~/.ssh` ou o arquivo for link simbólico. Como ativar o login por senha, exige primeiro definir a senha de login da web.
- **Gravar no programa principal**: o passo 3 da lista de primeiros passos grava de uma vez o zero dos servos, a orientação e a orientação da IMU na calibração do robotd. A imagem de fábrica está em modo de calibração; ao gravar com sucesso, o auxiliar apaga o drop-in commissioning e reinicia o programa principal para o modo de movimento. Sucesso não é só reiniciar: também precisa `mode=motion`, `phase=control`, e `calibration-status` lido de volta consistente item a item; se não, o arquivo e o modo são revertidos. Depois de reiniciar, o Studio lê `last-result.json` e põe o último resultado de volta no cartão do passo 3.
- **Controle (gamepad)**: pareamento (pelo `robotctl pad pair` oficial), controle da página, testar parada de emergência, e o remapeamento／calibração do controle físico abaixo.
- **Controle remoto**: levantar / começar a andar / sentar, habilidades, movimentos e movimentos combinados.
- **Ajuste de sensação**: apenas visualização. **Gestão de modelos**: importar pacote de implantação, renomear, escolher modelo, restaurar o embutido, definir botões de ação e excluir modelos importados; o formato específico está na [descrição da importação de modelos](MODEL_IMPORT.md).
- **Logs**: página de logs e pacote de diagnóstico; na barra inferior do celular, a partir de "Mais" também dá para entrar nos logs e exportar o pacote de diagnóstico.
- **Conectar ao Wi-Fi**: com o pato em hotspot, usar o celular para escolher o Wi-Fi de casa e digitar a senha; depois de trocar, a página procura sozinha o novo endereço do pato e pula quando acha; nas configurações dá para ver a rede atual, trocar de rede e esquecer redes salvas.
- **Senha de login**: definir, alterar ou remover em "Mais → Senha de login"; quando está em modo hotspot e sem senha, o cartão de conectar ao Wi-Fi da visão geral avisa para primeiro definir a senha.

O que ainda não funciona (próxima versão):
- **Alterar** na página os parâmetros de ajuste de sensação (amplitude da marcha, suavização, P/D/limite de força).

## Remapeamento／calibração do controle físico

Ao lado do título "Controle (gamepad)", clique em "Remapear／Calibrar". Primeiro pare a política e fique firme em pé, encerre o controle remoto web, o pareamento, a gravação／reprodução, os servos e as sessões de calibração; só depois que o driver confirma a pausa o aprendizado começa. Durante o aprendizado e o teste a saída de movimento é proibida, e o liberar torque da web no topo continua utilizável.

A figura usa posições padrão: A embaixo, B à direita, X à esquerda, Y em cima, que podem diferir da serigrafia dos botões físicos. Clique na tecla alvo e depois acione uma vez a entrada física; dá para aprender em ordem e também escolher qualquer item a qualquer momento para regravar. As quatro direções aceitam botões ou eixo de hat, e LT／RT aceitam botão digital ou eixo analógico. Select, Start e os quatro eixos dos analógicos são obrigatórios; os demais itens podem ser pulados ou limpos, e teclas não definidas não disparam a ação correspondente.

Os analógicos primeiro são empurrados até o fim na direção positiva indicada (direita／baixo), depois até a outra extremidade, e por fim voltam ao centro para confirmar; os gatilhos são apertados até o fim e soltos por completo para confirmar. Cada eixo pode ser invertido e ter a zona morta ajustada separadamente. O teste em tempo real na figura vem dos valores de pré-visualização lidos de volta do padd. Depois de clicar em "Salvar", é preciso obter a leitura de volta de salvamento do driver para contar como aplicado com sucesso; o controle só é retomado depois que todas as teclas são soltas e os analógicos voltam ao centro estável. "Cancelar" mantém a configuração original; "Restaurar padrão" apaga a configuração personalizada do dispositivo atual e usa o reconhecimento padrão original. Os modos USB／Bluetooth são salvos separadamente e carregados automaticamente ao reconectar.

Para trocar duas entradas obrigatórias já ocupadas, dá para primeiro limpar os rascunhos desses dois itens e depois regravar cada um. Limpar não altera a configuração salva; sem completar os itens obrigatórios não é possível salvar, e cancelar mantém o mapeamento original.

Só a página que iniciou o aprendizado é dona da sessão naquele momento, e as outras abas só podem visualizar. Trocar de página, ir para segundo plano ou fechar a página cancela, e depois de uma queda de rede a concessão de cinco segundos do driver também expira automaticamente; a página não retoma o aprendizado sozinha após reconectar. Interrupção da entrada ou troca de controle exigem recomeçar. Se o recibo de início se perder, a página pausa de forma conservadora até que um estado novo do driver confirme.

Este recurso exige o novo padd e o duck-echo correspondentes; um padd antigo avisa para atualizar, e a página não faz um salvamento falso. O padd oferece a interface de controle `/run/padd/mapping.sock` v1, e o `/run/padd/pad.sock` original continua somente leitura; a configuração é salva atomicamente em `/var/lib/padd/mappings.json`, e o serviço precisa de `StateDirectory=padd`. O duck-echo lê o `/run/padd/mapping-state.json` sem token de sessão, trata R3／Select pelo mesmo vínculo bruto e respeita o estado de pausa; a configuração de serviço da instalação independente e do bootstrap do conjunto de software precisa do grupo adicional `robot` para ler o arquivo de estado 0640. O protocolo específico e a aceitação estão no [design](../../docs/superpowers/specs/2026-10-30-pad-remap-design.md) e no [registro de verificação](../../docs/superpowers/verification/2026-10-30-pad-remap-studio.md). A implementação local não foi implantada na placa física.

Verificação de software (executar a partir do diretório `software/studio`):

```powershell
py -3.13 -X utf8 -I -c "import sys;sys.path[:0]=['deps','.'];import unittest;unittest.main(module=None,argv=['x','tests.test_pad_mapping','tests.test_pad_mapping_integration','tests.test_pad_mapping_e2e'])"
```

O navegador usa o Studio／autenticação／WebSocket reais e fixtures de entrada bruta isolados, e as capturas de tela cobrem 1440, 390 e 320 pixels. No Linux, `tests/pad_mapping_native_contract.py --fixture <executável de teste Rust> --dir <novo diretório temporário>` usa o handler de socket Unix de produção e verifica o aprendiz Python com o mapeamento／armazenamento／concessão reais em Rust; nenhum deles equivale a ter aceitado controles físicos de modelos diferentes.

## Referências

Este projeto se baseia no Microduck open source oficial, e no desenvolvimento se consultou a abordagem de vários projetos open source da comunidade; o código foi escrito por nós mesmos, e no desenvolvimento se usou programação assistida por IA.

## Licença

Apache-2.0 (igual ao repositório). Preact, htm e three.js em `web/vendor/` são todos MIT, e as licenças estão em `web/vendor/licenses/`; a origem do modelo 3D está em `web/model/NOTICE.md`; as licenças dos pacotes em `deps/` estão em cada um dos seus dist-info.
