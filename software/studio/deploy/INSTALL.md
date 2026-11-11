# Auxiliar root do Studio: passagem do barramento de servos, gravar calibração, pareamento de controle (instalação, desinstalação, conferência no robô real)

> 🌐 Tradução não oficial para pt-BR do [original](https://github.com/fanhao375/microduck-replica). Em caso de divergência, vale o original. Licença: CC BY-NC-SA 4.0 (texto alterado: traduzido do chinês/inglês).

Escrito para humanos e também para a IA que assumir depois: cada passo traz o porquê.

## Refazer a calibração inicial da IMU (imu-recalibrate)

O 0.7.2 acrescenta ao mesmo verbo fixo `enter` o parâmetro opcional `profile="legacy"|"qinmo"`, para a página da IMU salvar após uma única escolha. O parâmetro não aceita comandos, caminhos nem serviços. Exige evidência explícita de capacidade `configured_imu_profile` do novo robotd; não se corta de uma vez pelo número de versão da API. O Qinmo é marcado como configurado, suporta Feetech, velocidade angular raw/16.4 °/s; as placas originais usam reconhecimento automático por padrão, raw×0.0175 °/s. O fluxo do usuário está no [guia de uso web](../../../docs/Qinmo-IMU-网页使用.md).

A transação de tipo mantém ao mesmo tempo o helper e `/run/microduck-service-management.lock`, recusa disputa com atualização / barramento / redo antigo, pausa o controle (gamepad) e o serviço de voz fixo duck-echo, e depois de pousar com segurança restaura o estado original dos serviços; a página também recusa ações de controle durante a sessão, mantendo liberar torque e as operações de calibração necessárias. Só opera sobre o `--params /etc/robot/feetech-ft5/robotd.toml` e `--socket /run/robotd.sock` fixos do ExecStart direto do robotd realmente em uso, com sobreposições conhecidas, e o arquivo saved dessa configuração e os arquivos de calibração conhecidos. Recusa, nível a nível, links simbólicos, proprietário não root ou diretório / arquivo gravável por outros usuários; recusa read-only, sobreposição externa, origem I²C, protocolos e caminhos de calibração não suportados. A configuração principal / saved altera apenas `bus.imu_profile` e o `bus.imu_protocol` compatível, comparando antes e depois da análise para garantir que os demais valores se mantêm, incluindo chaves entre aspas da imagem e arrays de política inline. Os bytes originais, proprietário / permissões, resumo e modo são primeiro salvos em backup persistente; a calibração apenas limpa o verified da orientação de instalação do barramento, evitando reaproveitar orientações históricas do mesmo tipo.

O controle normal usa os limiares originais: duas leituras novas de desabilitado / sem política / ganhos restaurados. A recuperação de inicialização pré-controle suportada de `configuring_bus` / `waiting_for_bus` baseia-se no ciclo de vida atual do daemon e na capacidade da configuração, sem fingir que se sabe que o torque físico está desligado, e sempre entra em commissioning. O commissioning original de fábrica pode escolher o tipo, e depois da verificação preserva o fluxo de fábrica. O cancelamento da recuperação de inicialização primeiro instala o modo de calibração de fábrica conhecido, depois remove o arquivo de modo redo próprio, restaura a configuração / calibração originais e encerra a sessão; continua em modo de calibração e pode escolher de novo, evitando deixar uma sessão de encerramento permanentemente bloqueada. O cancelamento do controle normal restaura exatamente a configuração / calibração e o modo originais.

A conclusão exige que a configuração de tipo principal / saved em disco ainda corresponda ao resumo escrito, que a configuração em execução e o tipo realmente descoberto coincidam, e que o protocolo + perfil e o verified da nova orientação sejam lidos de volta de forma consistente. Falha restaura os arquivos originais; se não for possível restaurar, mantém `recovery_required` e o backup original, preservando a cena recuperável. O commit normal, o redo e o cancelamento mantêm o `imu_bus_profile` opcional, cujo padrão continua legacy; mudança de tipo invalida o que estava aguardando amostragem / cálculo / gravação. O Qinmo sem validação do programa principal não pode ser promovido a verified por "pular" nem por gravação normal. Atualmente não há aceitação no robô real; esta alteração não atualiza a imagem pública nem implanta automaticamente o robotd.

O novo auxiliar acrescenta o verbo fixo `imu-recalibrate`; o stdin só aceita as quatro actions `enter/status/finish/cancel`, e não pode especificar comando, caminho nem serviço. No modo de movimento, mesmo com `verified=true` e quaternion salvo já existentes, é possível revalidar: primeiro segurar firme e liberar torque, `enter` lê o estado novo duas vezes, exigindo `motion/control`, `policy_enabled=false`, sem levantar, torque nativo explicitamente false, sem liberação de torque pendente de confirmação, ganho temporário HD1910 restaurado. O serviço de controle (gamepad) é parado primeiro, e em seguida os limiares são verificados de novo; qualquer estado incerto é recusado.

- Entrar: `{"by":"Studio","action":"enter","confirmed":true}`. Preserva os parâmetros de inicialização diretos do robotd realmente em vigor, acrescentando apenas `--commissioning`; comando embrulhado ou ExecStart ambíguo é recusado. Não usa nem apaga o `zzzz-zz-microduck-commissioning.conf` de fábrica.
- Estado: `{"by":"Studio","action":"status"}`. A sessão tem `active/phase/session_id/source`; a página lê o manifest público somente leitura, sem chamar sudo a cada envio de estado.
- Concluir: `{"by":"Studio","action":"finish","session_id":"…","source":"bus","quat":[1,0,0,0]}`. Só é chamado depois que o `verify-imu` original passou e a origem atual, o verified, o quaternion normalizado e o arquivo de calibração foram lidos de volta item a item de forma consistente. O verifier original põe `motion_enabled` em false; o auxiliar só restaura esse valor de antes da entrada, e todos os demais campos, exceto a orientação de instalação / verified do chip atual, devem permanecer idênticos. Não reconstrói a calibração inteira, nem sobrescreve com padrões zeros diferentes de 2048, sentidos, limites, outra IMU ou configurações desconhecidas. Se o programa principal não preservou esses campos, recusa concluir e mantém a sessão para cancelar e restaurar.
- Cancelar: `{"by":"Studio","action":"cancel","confirmed":true,"session_id":"…"}`. A calibração principal e a saved restauram cada uma o backup original, restaurando o modo e o estado do serviço de controle originais; não conta como nova calibração bem-sucedida.

A sessão é primeiro persistida em `/var/lib/microduck-helper/imu-recalibrate/manifest.json` (root:root 0644), cada diretório de backup `<session_id>/` independente com 0700, e a calibração principal, a saved e a calibração original analisada são salvas à parte e verificadas por SHA256. Diretório pai 0755, o registro de operações continua 0600. A sessão usa seu próprio arquivo de modo persistente `/etc/systemd/system/robotd.service.d/zzzzzz-studio-imu-recalibrate.conf`, com ID de sessão e resumo do conteúdo, só permitindo apagar o arquivo desta sessão exatamente como foi; reiniciar o Studio / atualizar a página restaura a sessão, e depois de queda de energia o arquivo de modo persistente permanece. Falha na fase de início restaura o estado original; falha na fase de conclusão restaura o modo de calibração seguro já validado, podendo refazer a validação original ou cancelar. Só se restaura o serviço de controle (gamepad) com modo de movimento, calibração completa lida de volta de forma consistente e já desabilitado.

O `imu-recalibrate` e as operações de barramento / calibração compartilham o `helper.lock`. Com a sessão ativa, `bus-take` e o `calib-commit` normal são recusados; liberar torque pela página continua disponível. A desinstalação (inclusive `--purge`) recusa quando há sessão não encerrada ou ilegível, devendo primeiro concluir ou cancelar, para não apagar os backups de recuperação. Esse fluxo mantém a calibração grosseira original de 3 segundos parado / 3 segundos inclinado e a validação em runtime de inclinar a cabeça depois de ficar ereto; o modo de calibração de fábrica usa diretamente o fluxo original.

Origem do plano: scratchpad `console/handover-plan.md` (versão final v2) §2.2, §4.2, §5; regras conforme `console/studio-ops-plan.md` §7.

## 1. Para que serve

A página de servos do Studio precisa abrir ela mesma o barramento de servos `/dev/ttyS2`, então o programa principal robotd precisa parar primeiro e ser devolvido depois do uso.

- Parar e iniciar o robotd exige root, e o Studio não é root.
- Por isso ao Studio só se dá "uma chave que abre apenas esta porta": o sudoers libera somente o arquivo `/usr/local/sbin/microduck-helper`.
- Ele só reconhece doze verbos fixos, valida os parâmetros um a um, e o estado do robotd ele mesmo lê, sem confiar no que a página envia (desde o 0.5.0: novo `calib-read`, e `calib-commit` reconhece modo de calibração / movimento conforme o modo de inicialização real do fork; 0.5.2: `bus-give-back` também conta o snapshot em tempo real do modo de calibração como "programa principal de pé"; 0.6.0: novos `ssh-status` / `ssh-password`, a chave de SSH da página "Mais"; 0.7.0: novo `imu-recalibrate`, que salva de forma persistente a sessão de refazer a calibração inicial da IMU).

**O busStatus do modo de calibração** (`robot.busStatus` em `robotd/src/main.rs` do fork, `Status` em `feetech_commissioning.rs`): assim que sobe, o robotd escreve um relatório de inicialização `{"build","mode":"commissioning","phase":"configuring_bus","port"}`, e enquanto a phase ainda for de inicialização continua devolvendo-o; só depois da primeira leitura bem-sucedida devolve o snapshot em tempo real (phase `ready` / `moving` / `read_error` / `stop_unconfirmed` / `power_pending`, com `last_sample`). O fork antigo nunca limpa o relatório de inicialização — ao devolver pela página de servos, o auxiliar espera os 30 segundos completos e julga `soft_failed` (`detail.still_starting = true`, o texto diz "reporta iniciando o tempo todo", sem dizer que "não sobe"). Exige o fork novo (imagem posterior a 20261020).

**A imagem de fábrica roda em modo de calibração**: a imagem coloca `/etc/systemd/system/robotd.service.d/zzzz-zz-microduck-commissioning.conf`, com o `ExecStart` levando `--commissioning` (fork `main.rs:1300-1315`: o mode do busStatus só depende dos parâmetros de inicialização, independentemente do que o arquivo de calibração diga). No modo de calibração, `calibration-status` pode ser lido imediatamente, o busStatus traz `last_sample` (15 servos + `imu`), e o `verify-imu` grava a orientação no arquivo de calibração; no modo de movimento, com calibração inadequada / barramento de servos que não abre, ele fica em `waiting_for_bus`, e `calibration-status` responde `calibration not available yet`. Quando o passo 3 é gravado com sucesso, o auxiliar apaga esse arquivo, faz `daemon-reload` e reinicia o robotd, e o programa principal volta ao modo de movimento. Se `ExecStart` tem `--commissioning` mas o arquivo não existe, o auxiliar recusa a troca automática (`foreign_commissioning`), para a pessoa ir à página de logs exportar o pacote de diagnóstico.

## Na imagem vs instalação manual

O cartão "Conectar ao Wi-Fi" não é algo que o Studio sozinho forneça; depende de quatro coisas da imagem da réplica:

- `/run/microduck/net.json` escrito pelo netcheck;
- a configuração do hotspot `Microduck-Hotspot` no NetworkManager;
- configd rodando com `--allow-user btd,radxa`;
- a configuração de mDNS (`duck-xxxx.local`).

Em patos em que o Studio e o auxiliar root foram instalados manualmente conforme este texto, mas sem o overlay da imagem da réplica, sem essas coisas o Studio não mostra o cartão "Conectar ao Wi-Fi"; visão geral, controle (gamepad), servos, IMU, logs e a definição de senha pela web continuam normais.
A definição de senha pela web fica em "Mais → Senha de login"; o comando `python3 -I run.py set-password` deve ser executado com o usuário radxa, sem usar `sudo`, senão o Studio não consegue ler o arquivo de senha.

| Verbo | stdin (um objeto JSON) | O que faz |
|---|---|---|
| `bus-take` | `{"by": "quem·qual computador", "allow_seated": false}` | Assume, **sem liberar torque**: primeiro para o serviço do controle (gamepad) → a cada 200 ms lê duas vezes o estado do programa principal (política rodando é recusada; postura sentada só passa com `allow_seated` e parada; voltando à postura em pé espera no máximo 3 segundos) → levanta a flag → `systemctl stop --no-block robotd`, e se em 10 segundos não parou faz KILL → confirma que ninguém ocupa a porta serial |
| `bus-give-back` | `{"by": …, "soft": true}` | Devolução suave: o Studio já liberou o torque de tudo no barramento. Baixa a flag → inicia o robotd → espera os 4 passos de inicialização todos com sucesso e o programa principal de pé (modo de movimento `phase=control`, ou o snapshot em tempo real do modo de calibração `ready` / `moving` / `read_error` / `stop_unconfirmed` / `power_pending`; `read_error` também conta como de pé, e o texto original vai no reason) → traz de volta o serviço do controle (gamepad) → libera a trava. Se em 30 segundos ainda reporta fase de inicialização: `soft_failed`, `detail.still_starting = true` |
| | `{"by": …, "marker": {…}}` | Devolução sem liberar torque: escreve a marca de passagem → baixa a flag (**sem soltar a trava**) → inicia o robotd → espera o robotd assumir (assume segurando, sem abrir modelo). Se não assumiu, retoma o barramento e entrega ao Studio para continuar segurando. O modo de calibração não reconhece a marca (ao abrir o barramento todos liberam o torque): assim que chega o snapshot em tempo real responde `limp`, sem esperar 15 segundos para retomar o barramento |
| `bus-abort` | `{"by": …}` | Aborta a passagem (liberar torque pela página durante a contagem regressiva): levanta a flag → KILL no robotd → apaga a marca → confirma que a porta serial ficou livre. Meta de 1 segundo, depois o próprio Studio abre a porta serial e libera o torque. **Recusa quando não há sessão (saída 3)**: se a flag, a trava de gestão e a marca de passagem não existem, não toca em nada |
| `pad-pair` | `{"by": …, "mac": opcional, "timeout": 10–120 (padrão 60)}` | Executa como root o `/usr/local/bin/robotctl pad pair --json --timeout N [MAC]` oficial. O MAC é convertido para maiúsculas antes da validação. Tempo-limite total do subprocesso = `timeout + 30` segundos (estourou, é morto). Na placa `/var/lib/robot/weird-ble`, o robotctl primeiro para o btd, desliga e liga o adaptador, e por fim sobe o btd de volta. **Não toca** na flag do barramento, na unidade hold nem no robotd, e também não pega a trava de gestão de serviço; só pega o `helper.lock`, rodando um pareamento por vez |
| `pad-forget` | `{"by": …, "mac": …}` | Executa como root o `/usr/local/bin/robotctl pad forget MAC` oficial (MAC obrigatório, validação como acima). Também não toca no barramento, na unidade hold nem no robotd |
| `wifi-hotspot-down` | `{"by": …, "ssid": opcional}` | **Antes** de desligar o hotspot escreve `/run/microduck/studio-connecting.json` (root 0644, `until` = agora + 90 segundos, com o SSID alvo; primeiro escreve arquivo temporário e depois rename, diretório 0755), e nesse período o netcheck não interfere. Depois `nmcli connection down Microduck-Hotspot` (comando com tempo-limite de 15 segundos). Hotspot que já estava desligado conta como sucesso (nmcli dizendo "não é uma conexão ativa" também conta como sucesso); espera no máximo 10 segundos, até a wlan0 deixar de ser essa conexão. **Não toca** na flag do barramento, na unidade hold nem no robotd, e também não pega o `helper.lock`, usando o `wifi.lock` |
| `wifi-hotspot-up` | `{"by": …}` | `nmcli connection up Microduck-Hotspot`. Já estar ligado conta como sucesso; a partir do up espera no máximo 30 segundos, até a wlan0 realmente pendurar essa conexão. Depois de subir, apaga `/run/microduck/studio-connecting.json` (o arquivo não existir não é erro). Também não toca no barramento de servos, usando o `wifi.lock` |
| `calib-read` | `{"by": …}` (pode omitir) | Só lê o arquivo de calibração atual: responde `calibration` já analisado, em que modo está agora (`commissioning` / `motion` / `foreign`), se o drop-in de commissioning existe e se o `ExecStart` tem `--commissioning`. **Não pega o `helper.lock`, não escreve nenhum arquivo**, servindo de reserva para o Studio quando o modo de movimento fica preso em `waiting_for_bus` e não lê o `calibration-status`. |
| `calib-commit` | `{"by": …, "joints": [{"id", "direction"} × 15], "imu": {"source": "bus\|bno08x\|lsm6dsv16x", "quat": [w,x,y,z]}, "dry_run": false}` | Grava o zero dos servos (2048), o sentido e a orientação de instalação da IMU no `hd1910-calibration.toml` do robotd. Ordem: **primeiro pega o `helper.lock`, depois verifica a flag bus-held**; o modo de calibração pula o limiar de postura, e o modo de movimento lê o busStatus duas vezes a 200 ms (política rodando / em pé é recusada, saída 3); reconhecimento do modo: o `zzzz-zz-microduck-commissioning.conf` da imagem presente = modo de calibração (depois de escrever apaga e faz `daemon-reload` para passar ao modo de movimento), ausente e `ExecStart` sem `--commissioning` = modo de movimento (troca só a calibração), `ExecStart` com mas arquivo ausente = `foreign_commissioning` (não ousa trocar automaticamente, saída 3). Lê o arquivo atual, monta o conteúdo novo pelos limites padrão, valida item a item (igual ao `Calibration::validate` do fork). A cada gravação escreve um grupo `calib-backups/<número de 4 dígitos>-<UTC>/` (arquivo principal, `.saved.toml`, arquivo commissioning, `manifest.json`, com número = maior existente +1, mantendo 10 grupos); `.previous`, `.toml`, `.saved.toml` primeiro escrevem arquivo temporário, fsync, rename, preservando proprietário e permissões originais; depois de escrever, lê de volta e valida item a item. Desde a primeira gravação, qualquer exceção primeiro faz rollback e depois reporta o erro. Depois de reiniciar o robotd espera no máximo 40 segundos pelo hello; depois lê `robot.busStatus` a cada 0,5 segundo, por no máximo 30 segundos: **sucesso = `mode=="motion"` e `phase=="control"`, e os 15 servos lidos de volta por `calibration-status` (name / id / zero_raw / direction / calibrated / min_rad / max_rad), `motion_enabled`, `imu_mount_quat` / `imu_mount_verified` da placa de barramento, e quat / verified de cada célula de `imu_mounts` coincidem com o que foi escrito (ponto flutuante 1e-9)**. Em falha, responde conforme a causa `calib_rejected` (a calibração não passou na verificação do programa principal) ou `bus_unavailable` (o programa principal não abre o barramento de servos, primeiro veja a alimentação dos servos), e faz rollback. O trecho inteiro, quando não é dry_run, roda numa unidade independente por `systemd-run`, e o resultado é escrito em `last-result.json`; `dry_run: true` para ao terminar a validação, só responde o texto completo em `detail.toml`, sem escrever arquivo nem reiniciar. |
| `ssh-status` | `{"by": …}` (pode omitir) | Somente leitura: se o login SSH por senha está ligado (olha se `05-microduck-ssh-password.conf` existe), se o serviço `ssh` está rodando, quantas chaves estão registradas no `authorized_keys` do radxa. Não pega trava, não escreve nenhum arquivo (também não sobrescreve `last-result.json`). |
| `ssh-password` | `{"by": …, "on": true, "password": "…"}` / `{"by": …, "on": false}` | Ligar: senha de 8–64 caracteres visíveis, que não pode ser algo que todos adivinham como `microduck` (se inadequada responde `bad_password`, saída 2); pega o `ssh.lock`, `chpasswd` define a senha do radxa pelo stdin, escreve `/etc/ssh/sshd_config.d/05-microduck-ssh-password.conf` (`PasswordAuthentication yes`), só depois de `sshd -t` passar e `sshd -T` ler de volta `passwordauthentication yes` é que faz `systemctl reload-or-restart ssh`; se algum passo falhar, apaga esse arquivo e volta atrás (`sshd_config_bad` / `not_effective` / `reload_failed`). Desligar: apaga o arquivo, reload, sem mexer na senha do radxa (o sudo ainda a usa). **A senha fica só no stdin, não entra na linha de comando, journal, `last-result.json` nem oplog.** |

**Saída**: um JSON por linha: `{"step", "level", "ms", "msg", "detail"}`, a última linha com `step="done"`, e em `detail` há `exit` e `result`. O mesmo resultado é escrito em `/run/microduck-helper/last-result.json` (0644), que o Studio lê depois de reiniciar.

O `detail` de `pad-pair` ainda tem: `pair` (stdout do robotctl, analisado como `PadPairResult`: `{"outcome":"paired","pad":{…}}` ou `{"outcome":"failed","reason":"not_found|ambiguous|timeout|no_adapter|rejected|other","detail":…}`), `stderr` (algumas linhas de aviso do robotctl, como `paused btd …`, `warning: …`) e `stdout`. Pareou, sai com 0; o robotctl diz falha, sai com 5, e `detail.reason` preserva o motivo oficial. O `detail` de `pad-forget` tem `removed` (true / false / null), `stdout`, `stderr`.

**Códigos de saída**

| Código | Significado |
|---|---|
| 0 | Sucesso |
| 2 | Erro de parâmetro (verbo desconhecido, stdin não é objeto JSON, chave desconhecida, validação da marca não passou) |
| 3 | Estado não permite (proteção não instalada, política rodando, pato em pé / andando, sem sessão, não é root; `calib-commit` ainda pode ser bus-held, `foreign_commissioning`) |
| 4 | Ocupado ou abortado (trava de gestão ocupada por script da placa, o auxiliar está rodando outra operação de barramento / outro pareamento ou esquecimento, outra operação de hotspot Wi-Fi, devolução abortada) |
| 5 | Falha de execução (não consegue parar, porta serial ocupada, o robotd não reconheceu a marca e já liberou tudo, devolução suave não sobe; ou `pad-pair` / `pad-forget` não acham o robotctl, o robotctl diz falha de pareamento, tempo-limite e morto; desligar / ligar hotspot não deu certo, erro do nmcli, tempo-limite de espera; `calib-commit` falha de escrita / falha ao reiniciar / programa principal não entrou no modo de movimento / leitura de volta inconsistente, já com rollback) |
| 6 | A passagem não deu certo, o barramento foi retomado (flag levantada de novo, robotd parado, trava ainda lá), o Studio reabre a porta serial e continua segurando |

Valores de `result`:

- bus-take: `taken`, `refused`, `no_guard`, `busy`, `stop_failed`, `port_busy`
- bus-give-back: `adopted`, `retake`, `retake_failed`, `limp` (o modo de movimento não reconheceu a marca, ou modo de calibração), `aborted`, `soft`, `soft_failed` (quando o programa principal reporta fase de inicialização o tempo todo, `detail.still_starting = true` e `detail.msg` é a frase inteira), `refused`, `no_session`, `detach_failed`
- bus-abort: `aborted`, `abort_incomplete`, `no_session` (recusa quando não há sessão, saída 3)
- pad-pair: `paired`, `pair_failed` (saída 5, `detail.reason` é o motivo oficial), `timeout`, `no_robotctl`, `bad_output`
- pad-forget: `forgotten`, `forget_failed`, `timeout`, `no_robotctl`
- wifi-hotspot-down: `hotspot_down`, `already_down`, `down_failed`, `timeout`, `nmcli_failed`
- wifi-hotspot-up: `hotspot_up`, `already_up`, `up_failed`, `timeout`, `nmcli_failed`
- calib-read: `read`. `detail` tem `calibration`, `mode` (`commissioning` / `motion` / `foreign`), `commissioning_present`, `exec_has_commissioning`, `mode_text`; não pega trava, não escreve arquivo
- calib-commit: `preview` (dry_run), `committed` (`mode=motion` e `phase=control`, e `calibration-status` lido de volta consistente item a item), `not_motion` (em 30 segundos ainda não entrou no modo de movimento), `calib_rejected` (o programa principal diz que a calibração não passou na verificação), `bus_unavailable` (o programa principal não abre o barramento de servos), `readback_mismatch` (entrou no modo de movimento mas a leitura de volta é inconsistente), `bus_held`, `no_calibration`, `bad_calibration`, `joint_mismatch`, `invalid`, `bad_toml`, `restart_failed`, `write_failed`, `backup_failed`, `read_failed`, `foreign_commissioning`, `refused` (política rodando / em pé). No resultado de falha, `detail.rolled_back` / `files_restored` / `mode_restored` / `robotd_up_after_rollback` são reportados separadamente; `readback_ok`, `mismatches`, `bus_status` (o último mode / phase / error) e `backup` também estão lá. Se o próprio rollback não foi totalmente bem-sucedido, o `msg` diz "vá à página de logs exportar o pacote de diagnóstico"
- Qualquer um pode ter: `bad_args`, `bad_marker`, `bad_verb`, `not_root`, `busy`, `error`

## 2. Arquivos e caminhos

| Item | Local | Por quê |
|---|---|---|
| Auxiliar | `/usr/local/sbin/microduck-helper` (root:root 0755, `#!/usr/bin/python3 -IS`) | `-I` não lê o ambiente nem os diretórios do usuário, `-S` não carrega o site. O Linux trata o que vem depois do shebang como um único argumento, então escreve-se `-IS`, e não pode ser `-I -S` |
| drop-in do robotd | `/etc/systemd/system/robotd.service.d/zzzz-studio-bus.conf` | `ConditionPathExists=!/run/microduck-helper/bus-held`: com a flag presente, o robotd não sobe. **Não usa `systemctl mask --runtime`**: o corpo do robotd está em /etc, e o mask em /run não o cobre |
| sudoers | `/etc/sudoers.d/microduck-studio` (0440) | Libera um único caminho completo. Antes de instalar, `visudo -cf` |
| Flag | `/run/microduck-helper/bus-held` (diretório 0755, arquivo 0644) | Fica em disco de memória: queda de energia ou reinício a apagam, e o robotd inicia normalmente com o boot. Não fica no diretório do Studio porque, se a unidade do Studio vier a usar `RuntimeDirectory`, o systemd apagaria esse diretório quando o Studio reiniciasse. **A unidade do Studio não pode usar `RuntimeDirectory=microduck-helper`** |
| Pedido de aborto | `/run/microduck-helper/abort` | bus-abort escreve, e o bus-give-back em execução, ao ver, para |
| Trava do próprio auxiliar | `/run/microduck-helper/helper.lock` (flock) | bus-take, bus-give-back, calib-commit, imu-recalibrate, pad-pair, pad-forget rodam um por vez (ao gravar a calibração / reiniciar o programa principal, a página de servos não pode assumir o barramento, e só se permite um pareamento por vez). **O bus-abort não pega**: liberar torque não pode ser bloqueado por uma devolução em andamento, e seus passos podem todos ser repetidos |
| Trava do hotspot | `/run/microduck-helper/wifi.lock` (flock) | `wifi-hotspot-down` / `wifi-hotspot-up` rodam um por vez; separada do `helper.lock` do barramento de servos, sem se bloquear |
| robotctl oficial | `/usr/local/bin/robotctl` (fixo em `deploy/microduck-helper`) | Só `pad-pair` / `pad-forget` o chamam. Se não existe responde `no_robotctl` (saída 5), sem procurar em outros caminhos |
| Trava de gestão | `/run/microduck-service-management.lock`, mantida o tempo todo pela unidade temporária `microduck-bus-hold` (`flock -n … sleep infinity`) | O `service.sh` da placa, não conseguindo esta trava, não mexe no robotd |
| Marca de passagem | `/run/microduck-handover/handover.json` (diretório 0700, arquivo 0600, arquivo temporário no mesmo diretório + fsync + rename) | Só root lê e escreve; o robotd a apaga depois de ler |
| Registro de operações | `/var/lib/microduck-helper/oplog.jsonl` (diretório pai 0755, arquivo de log 0600; subdiretório privado de backup 0700) | Uma linha por vez: quem, quando, resultado. O diretório pai permite ao Studio ler o estado público da sessão da IMU, enquanto log e backups continuam legíveis só pelo root |
| Backup de calibração | `/var/lib/microduck-helper/calib-backups/<número de 4 dígitos>-<UTC>/` (diretório 0700, arquivo 0600, mantém só os 10 grupos mais recentes) | Cada grupo tem o `.toml` de antes da escrita, o `.saved.toml` (cada um com seu backup), o drop-in commissioning (se houver) e o `manifest.json` (proprietário original, permissões, sha256). O número é o maior dos diretórios existentes +1, sem ordenar pelo relógio de parede; o rollback restaura cada um conforme o manifest |

**Formato da marca** (o robotd do fork lê assim):

```
{"schema": "microduck-handover/1", "requested_by": str, "boot_id": str, "mono_ms": int,
 "ttl_ms": int, "port": "/dev/ttyS2", "arm_registers": {"limit", "p", "d"}（可选）,
 "joints": [{"id", "torque", "goal", "present", "offset", "load"?, "temp"?, "status"?} × 15]}
```

- `boot_id` e `mono_ms` (milissegundos de CLOCK_MONOTONIC) são preenchidos **pelo próprio auxiliar**, sobrescrevendo mesmo que o Studio os forneça.
- **ttl_ms acima de 30000 é recusado direto (saída 2), sem truncar silenciosamente**, e abaixo de 1000 também é recusado; padrão 20000. O robotd também só reconhece no máximo 30 segundos, e os dois lados concordam.
- Os ids devem ser exatamente 10–14, 20–24, 30–34, cada um uma vez; torque deve ser 1 (o programa principal só assume com os 15 servos todos ligados); goal, present, offset são inteiros de −32767..32767.

**Vários tempos**:

| Tempo | Valor |
|---|---|
| Intervalo entre as duas leituras do gate | 0.2 s |
| Espera pela volta à postura em pé | 3 s |
| De stop a KILL | 10 s |
| Espera depois do KILL | 3 s |
| robotd lê a marca | 15 s |
| Depois de assumir, completar o 4º passo da inicialização | 20 s |
| Devolução suave | 30 s |
| Abortar a passagem | espera 0.5 s depois do KILL |
| Esperar o robotd depois de gravar a calibração | 40 s (unit active + hello de `/run/robotd.sock`) |
| Esperar o modo de movimento | 30 s (lê o busStatus a cada 0.5 s; para ter sucesso ainda é preciso o calibration-status lido de volta consistente item a item) |
| Studio espera o auxiliar | 180 s (no tempo-limite também vai ler o `last-result.json`, sem reportar diretamente TimeoutExpired) |

**Algumas variáveis de ambiente**: `MICRODUCK_HELPER_DETACHED`, `MICRODUCK_HELPER_REQUEST`, `MICRODUCK_HELPER_REQUESTED_MS` (usadas por `bus-give-back` e `calib-commit`).

- Finalidade: na primeira chamada de bus-give-back / calib-commit sem dry_run, ele se coloca numa unidade temporária independente com `systemd-run --wait --pipe --collect --quiet --unit=microduck-handover-<hora>-<pid>` e executa de novo, encaminhando o stdin tal e qual e devolvendo a saída tal e qual. Assim, se o Studio morrer, a devolução não fica cortada no meio do caminho.
- Estas variáveis **apenas** decidem "não encaminhar de novo" e como casar o pedido de aborto e o resultado, **sem dar nenhuma permissão**. O sudo por padrão faz env_reset, e quem chama também não consegue passá-las.
- **Nenhuma variável de ambiente pode mudar o diretório raiz**: o teste importa o módulo e troca por um Ctx falso, sem depender de variáveis de ambiente.

**Em todo lugar que julga se o robotd subiu**, olha-se `systemctl show robotd -p ActiveState,SubState,Result,ConditionResult`. Com a flag presente, `systemctl start robotd` retorna 0, mas a unidade fica inactive, `ConditionResult=no`, e não se pode tomar isso por subiu. Antes de puxar o robotd sempre se faz `reset-failed`, evitando bater em start-limit-hit.

**Fronteira de confiança**: o usuário que pode usar sudo no auxiliar (hoje radxa) já poderia abrir o ttyS2 diretamente; e o robotd ainda relê no barramento para conferir a marca. Então forjar a marca não dá permissão a mais; no pior caso é "segurar os servos que já estavam sendo segurados".

## 3. Instalação (uma vez, SSH)

```sh
# 电脑上：把 deploy 目录拷到鸭子上
scp -r deploy radxa@鸭子:/tmp/studio-deploy
# 鸭子上
cd /tmp/studio-deploy && sudo sh install.sh radxa
```

O `install.sh` faz quatro coisas:

1. Instala o auxiliar em `/usr/local/sbin` (0755); se `/usr`, `/usr/local`, `/usr/local/sbin` tiverem proprietário que não é root, ou forem graváveis pelo grupo / outros, avisa;
2. Instala o drop-in e depois `systemctl daemon-reload`. **Não reinicia o robotd**;
3. Gera a linha do sudoers, e só instala (0440) depois de `visudo -cf` passar;
4. Imprime as informações de conferência.

**O passo 1 só instala o trecho `[Unit]` do drop-in.** O `[Service] Environment=MICRODUCK_HANDOVER_MARKER=…` está comentado: depois que o robotd do fork estiver instalado e o passo 3 (o Studio devolve sem liberar torque) for aceito, então se abrem manualmente essas duas linhas e `daemon-reload`. Depois de abertas, o robotd só olha a marca de passagem na próxima inicialização; sem marca, fica exatamente igual ao V167. O robotd de versão antiga não reconhece essa variável, e abri-la não tem efeito.

Conferir logo depois de instalar:

```sh
systemctl show robotd -p DropInPaths            # 要有 zzzz-studio-bus.conf
echo '{}' | sudo -n /usr/local/sbin/microduck-helper bus-nope; echo $?   # 用 radxa：应该是 2
echo '{}' | sudo -n /usr/local/sbin/microduck-helper bus-take;  echo $?  # 应该是 2：缺 by（什么都没动）
echo '{}' | sudo -n /usr/local/sbin/microduck-helper pad-pair;  echo $?  # 应该是 2：缺 by（不跑 robotctl）
echo '{}' | sudo -n /usr/local/sbin/microduck-helper wifi-hotspot-down; echo $?  # 应该是 2：缺 by（不碰 nmcli）
echo '{}' | sudo -n /usr/local/sbin/microduck-helper wifi-hotspot-up;   echo $?  # 应该是 2：缺 by（不碰 nmcli）
echo '{}' | sudo -n /usr/local/sbin/microduck-helper calib-read;       echo $?  # 应该是 0：读文件 + 模式（不拿锁、不写）
echo '{}' | sudo -n /usr/local/sbin/microduck-helper calib-commit;   echo $?  # 应该是 2：缺 by（不读写标定）
ls -l /usr/local/bin/robotctl                                        # 官方 0.15.1 或更新
stat -c '%U:%G %a %n' /usr /usr/local /usr/local/sbin /usr/local/sbin/microduck-helper
sudo -l -U radxa | grep microduck-helper
```

## 4. Desinstalação

```sh
sudo sh uninstall.sh            # 旗在（Studio 正占着总线）时拒绝
sudo sh uninstall.sh --purge    # 连 /var/lib/microduck-helper 一起删
```

A desinstalação apaga a linha do sudoers, o drop-in (e depois daemon-reload), o auxiliar, para o `microduck-bus-hold`, e limpa `/run/microduck-helper` e `/run/microduck-handover`. O robotd não é reiniciado.

## 5. O que olhar quando algo dá errado

- `sudo ls -l /run/microduck-helper/`: `bus-held` presente = o programa principal está bloqueado pelo Studio, e não subir é de propósito. Para liberar: devolva no Studio, ou reinicie.
- `cat /run/microduck-helper/last-result.json`: o que foi feito da última vez e em que passo travou (`steps` são as últimas 30 linhas de progresso).
- `journalctl -t microduck-helper -b`: cada linha de progresso é escrita aqui.
- `systemctl status microduck-bus-hold`: se a trava de gestão está nas mãos do Studio.
- `sudo tail /var/lib/microduck-helper/oplog.jsonl`: registro de operações.

## 6. Lista de conferência no robô real (pato nº 2, com o usuário segurando; ele diz que ao começar roda em sequência, sem perguntar a cada vez)

**Primeiro conferência somente leitura** (sem mexer em nada):

- `systemctl --version` (257), `sudo -V | grep -i -E 'env_reset|secure_path'`, `ls -l /etc/sudoers.d`.
- **Alguns comportamentos do systemd-run**:
  - `systemd-run --wait --pipe --collect --quiet -p … /bin/sh -c 'exit 6'; echo $?`: se o `--wait` traz de volta o código de saída do serviço. O auxiliar não depende disso: primeiro lê o request_id do last-result para casar, e só se não casar usa o código de saída do systemd-run;
  - com `--pipe`, matar o chamador faz o processo dentro do serviço, ao escrever no stdout, receber EPIPE (o auxiliar ignora SIGPIPE), em vez de ser morto junto;
  - a unidade temporária por padrão tem `KillMode=control-group`, então `systemctl stop microduck-bus-hold` para também o flock e o sleep, e a trava é solta junto. Confira com `flock -n /run/microduck-service-management.lock true; echo $?`.

**Testes no robô real do passo 1 (os relacionados ao auxiliar)**:

| Nº | O que fazer | O que olhar |
|---|---|---|
| U1 | `bus-take` com torque liberado | Saída 0; flag presente; robotd inactive; o Studio abre a porta serial, lê 40 todos 0, posição inalterada |
| U2 | `bus-take` com postura em pé segurada (depois do init) | 0; o Studio lê 40 todos 1, diferença de posição <1°, o pato não afunda |
| U3 | `bus-take` com a política ligada (Start do controle entrando em hold) | 3, o motivo é "primeiro aperte o Start do controle…"; o padd voltou; o robotd não foi parado. Depois de apertar o Start por 2 segundos voltando à postura em pé, take de novo → 0, sem soft |
| U4 | `sudo systemctl start robotd` com a flag presente | Saída 0, mas `systemctl show robotd -p ActiveState,ConditionResult` é inactive / no; o `service.sh status` da placa (o caminho vale o real da placa) é recusado pela trava: "já há uma operação de gestão de serviço em execução" |
| U4b | Reinício automático depois do KILL com a flag presente | **Não faça com o robotd enquanto o usuário está com o barramento.** Primeiro valide com uma unidade falsa: crie um serviço temporário com a mesma Condition, suba, levante a flag, `systemctl kill -s KILL`, e veja se 2 segundos depois ele foi puxado de novo. Quando o usuário estiver livre, faça mais uma vez no robotd |
| U4c | `systemctl start robotd` 6 vezes seguidas com a flag presente | Veja se o `Result` vira start-limit-hit. Depois baixe a flag (`bus-give-back {"soft":true}` faz reset-failed primeiro) e confirme que sobe |
| U5 | `systemctl restart studio-trial` durante a sessão | O pato não amolece; a flag e o `microduck-bus-hold` continuam; depois que o Studio sobe, `bus-take` retorna 0, com `session_was_open=true` no detail |
| U6 | Devolução suave | Segure; `bus-give-back {"soft":true}` retorna 0; os quatro passos do robotd com sucesso; o padd volta; o `microduck-bus-hold` para |
| U7 | Sair da página de servos | Se há servo segurando, não devolve (parte de outro subagente, validar junto) |
| V0 (parte do auxiliar) | Medir quanto tempo levam `bus-take` e `bus-abort` | Olhe o `ms` do last-result; meta do abort ≤1 segundo |
| U8 | Parear um controle uma vez no Studio (Thunder G50 Pro, modo Switch Pro, nome `Pro Controller`) | No assistente, "o pareamento usa" diz auxiliar; `cat /run/microduck-helper/last-result.json` com `verb=pad-pair`, `result=paired`, `detail.stderr` com `paused btd …`; em 30 segundos o padd o reconhece e entra no teste de botões. Sem o novo auxiliar, o assistente avisa para reinstalar primeiro |
| U9 | `wifi-hotspot-down {"by":"核对","ssid":"Home"}` com o hotspot ligado | Saída 0, `result=hotspot_down`; antes de desligar já existe `/run/microduck/studio-connecting.json` (root 0644, `until` cerca de 90 segundos depois, `ssid=Home`); `nmcli connection show --active` não tem `Microduck-Hotspot`; a conexão do celular ao hotspot cai. Chamar de novo com o hotspot já desligado → 0, `already_down`, e sem escrever a janela de conexão |
| U10 | `wifi-hotspot-up {"by":"核对"}` com o hotspot desligado | Saída 0, `result=hotspot_up`; `nmcli connection show --active` tem `Microduck-Hotspot` (wlan0); o celular consegue buscar o hotspot; o `studio-connecting.json` foi apagado. Chamar de novo → 0, `already_up`, e arquivo ausente também não dá erro |
| U11 | Primeiro só pré-visualizar: `calib-commit {…, "dry_run": true}` (joints / imu com o mesmo conteúdo da pré-visualização do Studio) | Saída 0, `result=preview`; no `detail.toml` os 15 `calibrated = true`, `motion_enabled = true`, limites e quaternion presentes; `detail.mode` é `commissioning` (de fábrica) ou `motion`; o sha256 e o mtime do `hd1910-calibration.toml` não mudaram, o robotd não reiniciou |
| U12 | Gravar de verdade (segure o pato): o mesmo com `dry_run: false` | Saída 0, `result=committed`; `detail.mode_before=commissioning`, `detail.bus_status.mode=motion`, `phase=control`, `readback_ok=true`, `mismatches=[]`; o `zzzz-zz-microduck-commissioning.conf` sumiu, `daemon-reload` rodou; o `.saved.toml` igual ao arquivo principal, o `.previous.toml` é o de antes da escrita; `calib-backups/` ganhou um grupo (no manifest os três arquivos estão); `systemctl show robotd -p ActiveState` é active; o passo 3 da lista do Studio vira concluído, a barra superior mostra modo de movimento |
| U12b | Rollback (calibração recusada): deixar a IMU em uso sem verified (ou com origem que não confere) e rodar o U12 de novo | Saída 5, `result=calib_rejected`; o `reason` é "a calibração não passou na verificação do programa principal: motion requires all 15 calibrated joints…"; `detail.bus_status` é `mode=motion`, `phase=waiting_for_bus`; `detail.files_restored` / `mode_restored` / `robotd_up_after_rollback` todos true, `detail.rolled_back` também true; arquivo principal e `.saved.toml` iguais aos de antes do U12, arquivo commissioning de volta, robotd subiu mais uma vez (voltando ao modo de calibração) |
| U12c | Rollback (servos sem alimentação): desligar a alimentação dos servos e rodar o U12 de novo | Saída 5, `result=bus_unavailable`; o `reason` é "o programa principal não abre o barramento de servos: torque readback…. A alimentação dos servos está ligada?"; `detail.bus_status.phase=waiting_for_bus`; arquivos e modo revertidos |
| U12d | Rollback (leitura inconsistente): alterar temporariamente o calibration-status do programa principal falso / stub de teste para um `zero_raw` de um servo e rodar o U12 de novo | Saída 5, `result=readback_mismatch`, `detail.readback_ok=false`, `detail.mismatches` diz qual item; arquivos e modo revertidos |
| U13 | Rollback: fazer temporariamente o robotd não subir (criar um `robotd.service.d/zzzz-e2e-fail.conf` com `[Unit]` + `ConditionPathExists=/nonexistent-e2e`, `daemon-reload`) e rodar o U12 de novo | Saída 5, `result=restart_failed`, `detail.files_restored=true`, `detail.robotd_up_after_rollback=false` (só conta como true se realmente esperou o hello); arquivo principal e `.saved.toml` iguais aos de antes do U12; `detail.journal` tem as últimas 30 linhas do robotd; o `reason` traz "vá à página de logs exportar o pacote de diagnóstico". Depois da conferência apague esse drop-in, `daemon-reload`, `systemctl start robotd` |

**Ainda não coberto**: a devolução com marca (`{"marker": …}`) só pode ser testada na placa depois que o robotd do fork estiver instalado e o Environment do drop-in estiver aberto; isso é V1–V12 do passo 4.

## 7. Limitações conhecidas

- No bus-take, entre "terminar de julgar o estado" e "levantar a flag e parar o robotd" há uma janela de cerca de 0,1 segundo, durante a qual o App Bluetooth ainda pode ligar a política. O padd e o gamesir já foram parados antes, mas o btd não.
- O `system.reboot` do configd e um `poweroff` direto por SSH não são barrados (plano §2.5). Se reiniciar, quando o robotd sobe libera o torque de tudo como de costume.
- Durante a contagem regressiva da passagem o Select do controle não funciona (o padd só inicia depois que os quatro passos do robotd têm todos sucesso); nesses segundos use liberar torque pela página (que vai por bus-abort) ou desligue a alimentação.
- O auxiliar de versão antiga (que não reconhece `pad-pair`) responde `bad_verb`; o Studio o trata como sem auxiliar novo e volta ao configd → bluetoothctl. Nesta placa (`/var/lib/robot/weird-ble`) o caminho de reserva pode não parear, e o assistente avisa para reinstalar o auxiliar ou executar `sudo robotctl pad pair` no pato.
- O auxiliar de versão antiga (0.2.0, que não reconhece `wifi-hotspot-down` / `wifi-hotspot-up`) também responde `bad_verb`; na configuração de rede o Studio avisa "o auxiliar é muito antigo, reinstale com deploy/install.sh".
