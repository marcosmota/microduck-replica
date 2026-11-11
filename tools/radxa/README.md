# Radxa Zero 3W · Fazer um cartão Armbian que inicializa direto

> 🌐 Tradução não oficial para pt-BR do [original](https://github.com/fanhao375/microduck-replica). Em caso de divergência, vale o original. Licença: CC BY-NC-SA 4.0 (texto alterado: traduzido do chinês/inglês).

> **2026-10-21: as duas imagens-base de setembro (20260920, 20260929) foram retiradas dos Releases**, por estarem muito antigas. Para um cartão pronto, use a [imagem da réplica](https://github.com/fanhao375/microduck-replica/releases/latest): é só gravar e usar, com programa principal, modelo de marcha e Studio. Os scripts de preparação do cartão deste diretório foram mantidos e continuam servindo para fazer o seu próprio cartão; as menções abaixo àqueles dois Releases são apenas histórico.

Use junto com [Sem HAT · passo 1](../../docs/不打HAT.md#passo-1--ligar-o-controlador-sem-conectar-mais-nada).
Anteriormente, a placa V1.12 desta máquina (Wi-Fi AIC8800) não inicializava com a imagem Armbian original testada; o problema foi resolvido após comparar e substituir o bootloader; para outros lotes, veja a descrição da imagem. **O pisca duplo em si não é código de falha**: em 2026-09-28 o B1 oficial também usou o `heartbeat` com pisca duplo durante conexão de rede e coleta normais.
A investigação anterior apontou incompatibilidade entre o U-Boot mainline / firmware de inicialização de memória usado pelo Armbian e esta placa; usando o bootloader da Radxa, ela passou a inicializar.
Este diretório embute na imagem a troca de bootloader, a gravação do Wi-Fi, a configuração do DHCP, a habilitação da porta serial e do console USB, para conexão à rede na primeira inicialização.

Em 2026-09-28 foi corrigida a omissão do DHCP de `wlan0` nos scripts de preparação do cartão; a imagem e os checksums do Release de 20 de setembro continuam sendo os arquivos antigos, sem essa correção. Quem já gravou a imagem antiga só precisa complementar a configuração conforme os [passos de rede](镜像使用说明.md#4-conectar-ao-wi-fi), sem precisar regravar apenas por isso; as instruções antigas de rede dentro do cartão também devem seguir este texto. A imagem pessoal corrigida desta máquina obteve endereço DHCP e login por SSH em teste real; isso não significa que todas as placas e roteadores foram validados, veja o [registro de testes](联网与HAT调试记录-20260928.md).

**Não quer fazer o cartão você mesmo?** Use a [imagem da réplica](https://github.com/fanhao375/microduck-replica/releases/latest) (é só gravar e usar, com programa principal e Studio). O trecho a seguir é a descrição histórica da imagem-base de fim de setembro (sem Wi-Fi; conecte o USB como porta serial para entrar),
e a configuração inicial e as diferenças entre lotes de placas estão na [descrição da imagem](镜像使用说明.md). Trocar de computador não afeta o sistema; trocar de Wi-Fi exige preencher novamente os dados de conexão, sem precisar regravar.

**Errata do primeiro login (2026-09-29): `root / 1234`.** Só depois de concluir o assistente do Armbian é que `duck / duck1234` é criado e a senha do root é alterada; o usuário e a senha em `card.conf` são predefinições do assistente, não credenciais da primeira autenticação. Troque a senha padrão logo em seguida. A nova imagem candidata de 20260929 já traz DHCP predefinido, instruções atualizadas dentro do cartão e sem chaves de host SSH pré-geradas; foi verificada offline e ainda não foi gravada em cartão para teste real. Cartões antigos não precisam ser regravados: basta corrigir o DHCP e recriar as chaves de host SSH conforme o guia.

| Arquivo | Para que serve |
|---|---|
| `card.conf` | Altera nome / senha do Wi-Fi; no primeiro login como `root / 1234`, o assistente cria por padrão `duck` / `duck1234`. Na China, pode-se preencher um proxy HTTP para instalar o software oficial. **Duas linhas de Wi-Fi vazias = modo de imagem pública**: não grava credenciais de Wi-Fi, mas ainda predefine o DHCP |
| `镜像使用说明.md` | Para a imagem do release: gravar o cartão, login por serial USB, conectar ao Wi-Fi, trocar o bootloader para salvar placas de outros lotes |
| [摄像头调试记录-20260928.md](摄像头调试记录-20260928.md) | Diagrama de referência do cabo flat, resultados da comparação com o B1 oficial, reconhecimento do IMX219 e comando de captura limitada de quadros; o reteste com a ligação correta no Armbian original ainda está pendente |
| [联网与HAT调试记录-20260928.md](联网与HAT调试记录-20260928.md) | Login na placa principal após a correção do DHCP, resultado da comunicação somente leitura com os servos via HAT e itens pendentes de investigação |
| `1-做卡.ps1` | No Windows, clique com o botão direito em "Executar com o PowerShell", escolha o `.img.xz` baixado, gere um `xxx-鸭子卡.img` e grave-o com o Rufus |
| `build-armbian-card.sh` | O script que realmente faz o trabalho; no Linux / WSL: `sudo bash build-armbian-card.sh xxx.img.xz card.conf` |
| `kissqy-r1-v112.sh` | Troca para o bootloader da Radxa e escreve o extlinux na "imagem de sistema em um clique R1" do kissqy, para que a placa V1.12 inicialize; o firmware, o modelo e a primeira configuração de rede dele são mantidos como estão. Opcionais: `--no-camera`, `--no-tof`, `--overlay i2c4-m0`. `sudo bash kissqy-r1-v112.sh microduck-zero3w-0.15.1-R1.img.xz [opções]`. Por que é preciso alterar e o resultado da verificação offline estão na [análise da imagem R1 do kissqy](../../docs/HD1910-20261012-kissqy镜像R1分析.md); **ainda não testado no robô real** |

Os dois scripts procuram primeiro, na mesma pasta, o pacote de bootloader da Radxa `u-boot-rk2410_2017.09-64-39cd993_arm64.deb`, e só baixam de radxa-repo.github.io se ele não existir. No WSL, o curl às vezes tem o SSL interrompido ao baixar desse site; nesse caso, baixe antes no Windows e coloque na mesma pasta do script (o script confere o checksum). Apague depois de usar e não envie esse .deb ao repositório.

A imagem usa **Armbian 26.2.1 trixie vendor 6.1.115 minimal** (especificado oficialmente pela Pollen; os espelhos na China só têm mais a 26.8.1;
a 26.2.1 está em `https://fi.mirror.armbian.de/archive/radxa-zero3/archive/`). O bootloader da Radxa é baixado automaticamente do pacote `u-boot-rk2410` em
`radxa-repo.github.io`, com versão e sha256 fixados no script.

O script faz cinco coisas:

1. Grava `idbloader.img` no setor 64 e `u-boot.itb` no setor 16384 (Radxa u-boot-rk2410 2017.09-64, DDR v1.25, BL31 v1.46)
2. `/boot/extlinux/extlinux.conf`: o U-Boot da Radxa reconhece extlinux, não o boot.scr do Armbian; aplica dois overlays, `uart2-m0` (porta serial dos servos `/dev/ttyS2`) e `dwc3-peripheral` (porta OTG como serial USB), e coloca o console do kernel em tty1
3. Predefine o assistente do primeiro login em `/root/.not_logged_in_yet` (usuário, senha, fuso horário, locale), aplicado só após a autenticação; `wpa_supplicant-wlan0.conf` configura a autenticação Wi-Fi e `/etc/systemd/network/25-wlan0.network` configura o DHCP sem fio; remove as chaves de host SSH pré-geradas da imagem-base, mantendo o serviço de geração na primeira inicialização do sistema original
4. `g_serial` + `serial-getty@ttyGS0`: ao conectar a placa ao computador surge uma porta COM extra, 115200, com login root
5. Faz mask de `serial-getty@ttyS2` / `ttyFIQ0`, para que o console de login não ocupe a porta serial dos servos

Depois de concluir o assistente do primeiro login e a configuração de rede, procure `radxa-zero3` no roteador e entre com `ssh duck@IP`. A imagem básica não inclui o ambiente completo de controle do robô, então não se deve concluir que o controle web ou dos servos já funciona apenas por ter rede.

Para as instalações seguintes, pode-se consultar o [install-dev.md](https://github.com/pollen-robotics/microduck/blob/main/docs/robot/install-dev.md) oficial da Pollen, mas **esta imagem não pode seguir diretamente os passos `migrate-network.sh` → reinicialização**. O script de migração do upstream verificado importa credenciais sem fio do netplan, não lê o `wpa_supplicant-wlan0.conf` desta imagem e também não sai automaticamente das configurações independentes `wpa_supplicant@wlan0` / `25-wlan0.network`. Deve-se manter primeiro o método de rede atual e só migrar depois de adaptar a importação das credenciais, a desativação dos serviços antigos e a recuperação em caso de perda de conexão, para evitar perder o acesso após reiniciar.

As alterações do script de instalação em `armbianEnv.txt` também precisam ser conferidas no `extlinux.conf` realmente usado na inicialização; não se deve julgar o hardware como configurado apenas pelo resultado da verificação do script.

## Scripts de análise no robô

[`onboard-analysis/`](onboard-analysis/): scripts usados na noite de 2026-10-12 para investigar o "disparo assim que o modelo assume o controle", todos somente leitura. Incluem gravação a 50 Hz (serviço de inicialização automática), registro de tensão e temperatura, verificação de autoconsistência do IMU, verificação das quatro direções, **calibração do ponto zero do IMU** (em pé na HOME → a simulação calcula a inclinação esperada → novo quaternion de montagem), ponto de virada da perda de controle, estatística de passos ao empurrar o joystick e análise de desconexão do IMU.
