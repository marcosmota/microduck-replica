# Imagem da réplica Microduck: scripts de construção

> 🌐 Tradução não oficial para pt-BR do [original](https://github.com/fanhao375/microduck-replica). Em caso de divergência, vale o original. Licença: CC BY-NC-SA 4.0 (texto alterado: traduzido do chinês/inglês).

> **Versão de teste 20261104:** esta rodada integra a recalibração da IMU na postura em pé, a verificação do sentido real do ID200, a recuperação do hotspot e as funções do software 31-r1; de fábrica continua sem calibração e, por padrão, compatível com a saída da placa pequena antiga. O quaternion completo exige configuração correspondente explícita e não grava a placa pequena automaticamente. Os limites de aceitação para gravar o cartão novo, trocar de rede e movimentos, e o alcance da atualização de software em cartões antigos, estão nas [notas desta rodada](../../docs/更新说明-20261104.md). Os registros de lançamento abaixo ficam como histórico.

> Usuários leiam: [Gravação e primeiro uso](烧录和第一次使用.md). O que segue é para quem faz a imagem.

> Se o hotspot não aparecer, primeiro confirme se a placa é **Zero 3W ou 3E, sem Wi-Fi integrado**. O hotspot não depende do ID dos servos; esta imagem ainda não foi validada na adaptação para a 3E. Veja [Solução de problemas de hotspot e modelo da placa](../../docs/热点与主板型号排查.md).

> **Lançamento de 2026-10-27:** a [imagem 20261027 Latest](https://github.com/fanhao375/microduck-replica/releases/tag/microduck-replica-image-20261027) e o [pacote de software 2026.10.27](https://github.com/fanhao375/microduck-replica/releases/tag/microduck-software-v2026.10.27) estão públicos, com atualização/reversão assinada pela web, helper correspondente e voz local de sentar/levantar via R3 pré-instalados, incluindo o modelo ASR em chinês e as dependências ARM64. Origem de execução e382f074; o pacote assinado e a imagem usam o mesmo instalador. Tamanhos/hashes de todos os anexos no servidor e o ponto de download público foram verificados; o pato está desligado por ora, sem gravar cartão nem testar no robô, a aceitação física desta versão está pendente; [hashes dos artefatos e limites da validação](../../docs/superpowers/verification/2026-10-27-web-updates.md), [uso de atualização de software e voz](../../docs/软件更新与语音使用说明-20261027.md). As notas de 10-25 abaixo ficam como estado histórico.

> 2026-10-25: o [livro-razão de funções e atualizações de software](../../docs/软件功能与升级总账.md) registra as diferenças entre código-fonte, máquina atual e imagem, e a direção das próximas atualizações dos pacotes de componentes compartilhados. O serviço de voz independente de hoje e a faixa selecionada continuam sem integração à construção da imagem genérica; a atualização online ainda está por implementar.

> Na sequência do mesmo dia: a gravação/reprodução web e o movimento "灵动" foram implementados no c18783c e implantados via SSH no pato atual, e a pré-visualização inline da câmera e a recuperação de entrada pareada também foram sincronizadas com a versão correspondente. A imagem existente não foi reconstruída; pacotes de código congelados e implantação em uma única máquina não podem ser tratados como atualização online genérica. Veja a [aceitação desta rodada](../../docs/superpowers/verification/2026-10-25-web-motion-studio.md).

> Atualização: o aa825fc adicionou a gravação de sentar/levantar, já implantada, e o usuário confirmou que salvar e reproduzir na prática funcionam; a compatibilidade com registros v2 e o alcance da reversão estão na [aceitação de sentar/levantar](../../docs/superpowers/verification/2026-10-25-web-posture-recording.md). Esta versão ainda não teve a imagem refeita.

A imagem da réplica é um cartão SD que funciona logo depois de gravado: o programa principal do nosso fork, os modelos de marcha e movimentos, o Microduck Studio,
a primeira inicialização (dar nome, reconhecer a IMU, criar o hotspot) e a configuração de rede (se não conectar ao Wi-Fi, abre o hotspot) já vêm instalados.
Este diretório contém os scripts que fazem esse cartão: puxam, somente em leitura, o programa principal e os modelos de um pato já ajustado, instalam na imagem base,
limpam a identidade da máquina, os logs e o que outros deixaram, e reduzem e comprimem o resultado.

A imagem é de download gratuito, apenas para réplicas pessoais não comerciais, **não é distribuída junto de nenhum produto pago**: os modelos dentro dela têm licença pessoal não comercial do autor,
veja [`overlay/usr/share/doc/microduck/模型说明.txt`](overlay/usr/share/doc/microduck/模型说明.txt).

## O que há na imagem

| Componente | Local na imagem | De onde vem | Licença |
|---|---|---|---|
| Sistema base | `/` | Armbian 26.2.1 (Debian 13), kernel 6.1.115-vendor-rk35xx, U-Boot da Radxa | Próprias de cada pacote, `/usr/share/doc/<pacote>/copyright` |
| Runtime oficial 0.15.1 | `/opt/robot/daemon/`, `/usr/local/bin/robotctl` | Pollen Robotics, commit 1fa8438, já incluso na imagem base | Apache-2.0 |
| Programa principal robotd, padd, mediad | `/opt/robot/feetech-ft5-r5/` | Nosso fork (oficial 0.15.1 + barramento Feetech, módulo de IMU I²C, fusão com giroscópio, controle (gamepad) via UDP), puxado de um pato ajustado | Apache-2.0 |
| ONNX Runtime 1.28.0 | `/opt/robot/feetech-ft5-r5/onnxruntime/` | Microsoft, sem alterações | MIT |
| 10 modelos de marcha e movimentos | `/opt/robot/feetech-ft5-r5/models/`, `roller/` | Treinados por autores da comunidade, puxados de um pato ajustado, com os arquivos de treinamento removidos | Pessoal não comercial |
| Microduck Studio | A partir de 20261027 `/opt/microduck/software/current/studio/`, serviço `microduck-studio`, porta 8000; nas imagens antigas `/home/radxa/studio-dev/` | [`../studio/`](../studio/), arquivos de execução assinados | Apache-2.0 |
| Voz local de sentar/levantar | `/opt/microduck/software/current/duck-echo/`; recursos em `/opt/microduck/resources/`, por padrão clique único no R3 liga/desliga, confirmação por grasnado | [`../duck-echo/`](../duck-echo/), Vosk small-cn0.22 e dependências ARM64/cp313 | Apache-2.0 e as licenças próprias das dependências |
| Executor de atualização de software | `/usr/local/lib/microduck-updater/`, serviço root independente; árvore de versões `/opt/microduck/software/releases/` | [`../updater/`](../updater/), updaterd0.15.1 ARM original; traz embutida a própria chave pública de lançamento, aplicação automática desativada | Apache-2.0 |
| Primeira inicialização, configuração de rede, pequeno auxiliar root | `/usr/local/sbin/microduck-firstboot`, `microduck-netcheck`, `microduck-helper` | [`overlay/`](overlay/), `deploy/` do Studio | Apache-2.0 |
| Calibração de fábrica | `/etc/robot/feetech-ft5/hd1910-calibration.toml` | [`calib/hd1910-calibration.option-a.toml`](calib/hd1910-calibration.option-a.toml): os 15 servos em 2048 / −1 / sem calibração | — |

O texto completo da licença de cada parte está em `/usr/share/doc/microduck/` na imagem e, na construção, também é copiado para o diretório de saída para ser publicado junto.

Alguns pontos em que a saída de fábrica difere do pato nº 2:

- **O programa principal roda em modo de calibração** (`robotd.service.d/zzzz-zz-microduck-commissioning.conf` adiciona `--commissioning`).
  Um pato sem calibração só consegue ler a IMU, fazer a calibração grosseira da IMU e ver a calibração dos 15 servos no modo de calibração; quando o passo 3 da lista de primeiros passos,
  "gravar no programa principal", tem sucesso, o pequeno auxiliar root apaga esse arquivo e o programa principal volta ao modo de movimento.
- **A câmera usa por padrão uma imagem de teste** (configuração oficial `/etc/robot/robotd.toml`, `[media] source = "test"`),
  para que o mediad não fique travando repetidamente em patos sem câmera.
- **Senha padrão do radxa `microduck`**, root bloqueado, SSH aceita apenas chave.
- Mapeamento de botões do controle e parâmetros de marcha seguem o `robotd.toml` do pato nº 2 (amplitude de movimento 0.7, sem liberar torque automaticamente em queda, sem desligar automaticamente com bateria fraca).

## Preparar as entradas

### 1. Imagem base

`microduck-zero3w-0.15.1-R1-V112.img`: derivada da imagem R1 pública da comunidade, usando o script de conversão R1 em `tools/radxa/`
(parâmetros `--no-tof --overlay i2c4-m0`): troca o U-Boot pelo da Radxa e a árvore de dispositivos B1, e habilita o I²C4 (pinos 27/28)
para o módulo de IMU. A imagem base já contém o runtime oficial 0.15.1. As duas versões 20261020 e 20261021 usaram a mesma base,
sha256 `652dd54e70f49fd3633058d5ed9bbb2f63b435648da2346dfbfa60fcaa4c5bcc`.

A intenção futura é trocar por uma base de Armbian limpo + instalador oficial, para não depender da imagem de terceiros.

### 2. payload (puxado de um pato ajustado)

Esse pato precisa ter o programa principal do nosso fork instalado e já conseguir andar:

- em `/opt/robot/feetech-ft5-r5/` deve haver `robotd`, `padd`, `mediad`, `onnxruntime/`, `models/`, `roller/`;
- `/etc/systemd/system/robotd.service.d/zzzz-r17-0151.conf` (parâmetros de inicialização do programa principal, não pode conter `--commissioning`);
- `/etc/robot/feetech-ft5/robotd.toml` (ao puxar, a ligação local da IMU é removida automaticamente e o pacote de sons volta a apontar para o oficial).

O que é puxado é uma lista de permissão, somente leitura, sem alterar nenhum arquivo do pato:

```sh
# No computador: copiar o script para lá
scp capture-payload.sh radxa@<IP do pato>:/tmp/
# No pato
sudo sh /tmp/capture-payload.sh /tmp/microduck-payload.tgz
# No computador: trazer de volta
scp radxa@<IP do pato>:/tmp/microduck-payload.tgz .
```

A versão 20261020 foi puxada do pato nº 2, com sha256 do payload `9e1e64b8…`. O robotd dentro (sha256 `9b734239…`)
foi compilado em 10-15 na área de trabalho do fork e **não corresponde exatamente** a nenhum commit (antes constava aqui `a844db6`, o que estava errado).

A versão 20261021 usou ainda esse mesmo payload, apenas trocando o robotd por uma versão arm64 compilada do commit `f4bbe53` do branch
`hd1910-v167-imu-autodetect` do fork (sha256 `0a6a86b2…`; o endereço do fork está na seção "Software" da página inicial),
e a entrada do robotd em `payload.json` teve o sha256 alterado de acordo; o sha256 do payload após a troca é `26252fbc…`.
Os robotd futuros são todos compilados a partir de um commit, para corresponder ao código-fonte.

O robotd de 20261023c vem do [branch `hd1910-v167-imu-dxl2` do fork do runtime](https://github.com/fanhao375/Microduck-kissqy/tree/hd1910-v167-imu-dxl2),
commit `0791fcd44f71c705dc8bb0fbcde405593b52d19b`, identificador de construção `4734cbb-feetech-imu-auto-control.44`.
Usa Rust 1.99.0, `cargo zigbuild --locked --release --target aarch64-unknown-linux-gnu.2.31 -p robotd --bin robotd`,
`CARGO_PROFILE_RELEASE_CODEGEN_UNITS=1`; sha256 do robotd
`edf182edb7bf11aeeeedfdab98170ab2ec48a9ec18417f9b68c744b0a60c6d16`.
Apenas o robotd do payload já verificado foi substituído, e em tempo de execução e na configuração salva foi definido explicitamente `[bus] imu_protocol = "auto"`;
os demais daemons, modelos e o ONNX Runtime permanecem como os arquivos originais. O tamanho e o hash de cada arquivo são reverificados no reempacotamento,
`payload.json` guarda o commit de origem, os parâmetros de construção e o ponto de entrada da descrição de origem; o hash completo do payload segue a lista da imagem entregue.
O alcance suportado, a comparação com a falha da versão de teste antiga e os limites da aceitação em placa real estão no [registro desta versão](../../docs/HD1910-20261023-总线IMU自动识别.md).

### 3. Máquina de construção

Ubuntu 22.04 no WSL ou qualquer Linux, com root (dispositivos loop):

```sh
sudo apt install fdisk e2fsprogs xz-utils openssl python3.11
```

O script exige Python 3.11 ou superior (usa `tomllib`). O `python3` que vem no Ubuntu 22.04 é o 3.10;
depois de instalar o python3.11, faça `python3` apontar para ele, por exemplo criando um diretório só com esse link e colocando-o no início do `PATH`.

## Construção

A imagem completa de 20261027 ainda precisa de quatro entradas em conjunto; faltando qualquer uma, o script recusa antes de modificar o rootfs. Primeiro gere o pacote assinado a partir de um commit Git congelado, e use `voice_resources.py` para preparar e verificar o modelo oficial em chinês e os recursos wheel ARM64/cp313; não misture dependências Python da máquina de desenvolvimento nem voz/chave pessoais. A chave privada de lançamento serve apenas para assinar na máquina de construção, não é entrada da imagem.

```sh
export SOFTWARE_UPDATE_PACKAGE=/path/to/2026.10.27.software-update.tar
export SOFTWARE_UPDATE_PUBLIC_KEY=/path/to/software/updater/release.pub
export SOFTWARE_UPDATE_ENGINE=/path/to/verified-arm64-updaterd
export VOICE_RESOURCES_DIR=/path/to/verified-resources
# x86构建机另需qemu-user，并给出已核验ARM64 loader/libs根目录；原生ARM构建机留空即可。
export SOFTWARE_UPDATE_ENGINE_COMMAND_JSON='["qemu-aarch64","-L","/path/to/arm64-libs"]'
```

Os comandos a seguir, executados no shell root de construção, preservam o ambiente acima; ao chamar via sudo, passe explicitamente essas entradas fixas, para que o sudo, ao limpar o ambiente, não volte a uma construção da versão antiga. Sem definir o conjunto completo de componentes, mantém-se apenas o comportamento de construção da imagem antiga, sem o novo pacote de atualização/voz. A imagem final verifica `/usr/share/doc/microduck/software-components.json`, a descrição dos recursos de voz e os arquivos de versão reais.

Quando o software assinado contém um pacote companheiro do controlador, depois que o instalador compartilhado termina, `controller_factory.py` lê da árvore de versões verificada e confere o programa principal congelado, o padd, os dois manifestos nativos e as dependências de voz. Os dois binários no payload da imagem precisam ser idênticos byte a byte para que sejam instalados os quatro drop-ins de serviço declarados, a trava de compatibilidade de voz de propriedade do root e a marca de compatibilidade, todos os arquivos com 0644; manifesto com peça faltando, hash divergente ou link simbólico no destino são recusados. A origem e o hash dos arquivos de suporte ficam registrados em `controller_bundle` dentro de `software-components.json`. Pacotes de software antigos sem pacote companheiro mantêm o comportamento de construção original. Esta etapa opera apenas no ROOT offline: não inicia serviços, não copia estado pessoal, não altera modelos, calibração nem os padrões do protocolo legacy, e não grava MCU.

```sh
sudo STUDIO_VERSION=<número do commit do Studio> IMAGE_VERSION=<número do commit deste diretório> \
    sh build-image.sh /path/to/microduck-zero3w-0.15.1-R1-V112.img \
    /path/to/microduck-payload.tgz ../studio ~/microduck-out
```

- `OUT_DIR` não pode ficar sob `/mnt`: no disco Windows do WSL o loop é muito lento e as permissões estão erradas; use `~/microduck-out`.
- Apenas para o seu cartão de aceitação: `DEV_AUTHORIZED_KEY="$(cat alguma.pub)"` grava essa chave pública no
  `authorized_keys` do radxa, de modo que, se algo der errado nos testes, dá para alterar remotamente sem regravar o cartão; a lista da imagem registra isso.
  **Nunca defina em lançamentos oficiais**: seria deixar uma porta dos fundos no pato de todos os usuários.
- Se não passar os números de versão, o script usa `git rev-parse` por conta própria (com alterações não commitadas adiciona `-dirty`), e se não conseguir escreve "desconhecido".
  No WSL, ler um git worktree no disco Windows muitas vezes falha, então passe-os você mesmo ao lançar.
- O nome do arquivo da imagem é por padrão `microduck-replica-data.img.xz`; ao gerar uma segunda versão no mesmo dia, passe um nome como `IMAGE_TAG=20261021b` (apenas dígitos e letras minúsculas).
- Leva cerca de 10 minutos. Saídas:
  - `microduck-replica-YYYYMMDD.img.xz` e o `.sha256` de mesmo nome;
  - `镜像清单.txt`: sha256 da imagem base e do payload, o que foi removido, o que foi adicionado;
  - `开源许可.txt`, `第三方.txt`, `模型说明.txt`;
  - `work.img`: sempre mantido, apague manualmente depois de confirmar que está tudo certo.

Pontos de entrada de verificação independentes (não precisam de root):

```sh
sh build-image.sh --selftest                 # GPT / MBR 分区行改写
sh capture-payload.sh --clean-toml IN OUT    # robotd.toml 清理函数
sh apply-rootfs.sh ROOT PAYLOAD ../studio ./overlay   # 在假目录树上试装（ROOT 不能是 /）
```

## Testes

```sh
# WSL root 下跑全（有几条要 root 才能验属主和权限）
python3 -I -m unittest discover -s tests -v
sh -n apply-rootfs.sh && sh -n build-image.sh && sh -n capture-payload.sh
```

No Windows também é possível rodar com `py -3.13 -I -m unittest discover -s tests -v` (sh do Git Bash); o teste de permissões é ignorado.

## O que o próprio script bloqueia

Durante a construção, se qualquer um dos itens a seguir não for satisfeito, ele para e diz em que etapa travou e o que fazer:

- o sha256 de algum arquivo do payload não confere com a lista, há arquivos fora da lista ou links simbólicos;
- há quebras de linha do Windows (CR) em overlay, no `deploy/` do Studio ou em `run.py`:
  no Windows, ao clonar com `core.autocrlf=true`, scripts sem extensão viram CRLF e não rodam na inicialização.
  O `.gitattributes` do repositório já define LF para esses diretórios;
- outros arquivos da imagem (incluindo binários) ainda têm a identificação do autor da imagem antiga; a assinatura fixa de `模型说明.txt` e o registro de origem do código em `payload.json` são mantidos;
- há links pendentes em `etc/systemd`, `udev`, `cron`, ou ainda há referências a arquivos já removidos;
- sob `opt/robot/feetech-ft5-r5` ainda há arquivos de treinamento (`export-data.zip`, `training-*.json`, `upstream-params`, `*.pt`);
- ainda há arquivos comuns em "pendente de confirmação manual" (é preciso primeiro confirmar que não contêm dados pessoais e depois liberar com `ALLOW_UNCONFIRMED=1`, o que fica registrado na lista);
- a inicialização oficial da placa de som (`aic3104-init`) está incompleta.

Além disso: toda exclusão é impressa antes de executar; toda escrita passa por `safe_dir` / `safe_file`, e um link simbólico no ROOT apontando para fora faz o script falhar,
sem danificar o host; `apply-rootfs.sh` não toca em dispositivos loop nem faz chroot.

## Verificação antes do lançamento

1. Todos os testes passam (WSL root).
2. Construir com código commitado, `STUDIO_VERSION` e `IMAGE_VERSION` com o número do commit, sem liberação de `ALLOW_UNCONFIRMED` no log.
3. Descompactar de volta o `.img.xz` entregue, montar somente leitura e verificar pelo menos: o sha256 confere; `e2fsck -fn` sem erros;
   o sha256 do programa principal e dos modelos igual ao do payload, e todos os modelos referenciados pelo `robotd.toml` presentes;
   o drop-in do modo de calibração presente; `machine-id` vazio e sem chaves de host SSH; SSH aceita apenas chave;
   a identificação do autor antigo só pode existir na assinatura fixa de `模型说明.txt` e na lista de origem do `payload.json`.
4. Registrar o escopo da aceitação física em que o cartão novo desta versão percorre do início [Gravação e primeiro uso](烧录和第一次使用.md): hotspot → configuração de rede → 6 passos da lista de primeiros passos → levantar e andar. Se não concluído, indicar claramente nas notas de lançamento; não se pode reaproveitar a conclusão de aprovação no robô real da versão anterior. A 20261027 teve autorização do usuário para publicar primeiro o software/imagem já verificados, e a aceitação física desta versão será feita depois de ligar o pato.
5. O commit do fork correspondente ao programa principal da imagem já foi enviado (push) ao fork.
6. Publicação: no GitHub Releases, colocar o `.img.xz`, o `.sha256`, o `镜像清单.txt` e os três txt;
   o primeiro parágrafo das notas do Release diz "gratuita, apenas para réplicas pessoais não comerciais, não distribuída junto de produtos pagos".
