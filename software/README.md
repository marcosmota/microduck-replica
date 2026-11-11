# software · Software

> 🌐 Tradução não oficial para pt-BR do [original](https://github.com/fanhao375/microduck-replica). Em caso de divergência, vale o original. Licença: CC BY-NC-SA 4.0 (texto alterado: traduzido do chinês/inglês).

> Este diretório mantém os modelos de simulação upstream necessários para o treinamento e **não é a fonte de impressão das peças estruturais físicas mais recentes**. Os downloads de modelos ficam todos em [microduck-replica-cad](https://github.com/fanhao375/microduck-replica-cad); incorporar os parâmetros dos servos Feetech não equivale a ter atualizado a geometria, a massa e a inércia do CAD.

O runtime oficial [`pollen-robotics/microduck`](https://github.com/pollen-robotics/microduck) foi escrito para o Dynamixel XL330, e a linha principal deste repositório usa o Feetech HD-1910, então o programa principal precisou ser modificado.

## Visão geral do software em uso

| Software | Para que serve | Onde está e como usar | Licença |
|---|---|---|---|
| **Imagem da réplica** | Sistema completo que funciona assim que o cartão é gravado: programa principal, modelos, Studio, primeira inicialização e configuração de rede | [`image/`](image/): usuários leiam [Gravação e primeiro uso](image/烧录和第一次使用.md); quem faz a imagem leia [image/README.md](image/README.md). Versões publicadas em [Releases](https://github.com/fanhao375/microduck-replica/releases/latest) | Scripts de construção Apache-2.0; as licenças de cada parte dentro da imagem estão em `/usr/share/doc/microduck/` na própria imagem |
| **Microduck Studio** | Console web instalado no pato: status, liberar torque, zero dos servos, orientação da IMU, gravar no programa principal, controle (gamepad), controle remoto, logs | [`studio/`](studio/). Quem usa a imagem da réplica não precisa instalar à parte | Código Apache-2.0 |
| **Atualização de software pela web** | Pacotes de componentes assinados, serviço de atualização independente, envio offline, instalação e reversão | [`updater/`](updater/), pré-instalado na imagem 20261027; cartões antigos precisam de uma inicialização única via SSH, [instruções](../docs/软件更新与语音使用说明-20261027.md) | Apache-2.0 |
| **Voz local para sentar/levantar** | Clique único no R3 liga/desliga a escuta, comandos em chinês para sentar/levantar, aviso sonoro de grasnado | [`duck-echo/`](duck-echo/); a imagem 20261027 inclui o modelo em chinês e as dependências ARM64; não precisa de chave de nuvem | Código Apache-2.0; modelo e dependências conforme suas próprias licenças |
| **Programa principal robotd / padd / mediad** | Sobre o oficial 0.15.1, adiciona o barramento de servos Feetech HD-1910, módulo de IMU I²C, fusão com giroscópio e controle (gamepad) via UDP | Código-fonte em [fanhao375/Microduck-kissqy](https://github.com/fanhao375/Microduck-kissqy); o branch e o commit usados por cada imagem estão em [image/README.md](image/README.md). Já vem instalado na imagem da réplica | Apache-2.0 |
| **Modelos de marcha e movimentos** | Marcha, sentar / levantar, abaixar a cabeça para pegar, cambalhota para frente, chutes esquerdo e direito, 4 de patinação | Vêm na imagem da réplica. O que cada modelo faz, a qual botão do controle corresponde e quem o treinou estão na [descrição dos modelos](image/overlay/usr/share/doc/microduck/模型说明.txt) | Licença pessoal não comercial do autor, **não é licença open source**: não pode ser distribuído junto de produtos pagos |
| **Projeto de treinamento** | Treinamento em simulação do HD-1910 | [`training/`](training/) | Código Apache-2.0, malhas 3D CC BY-NC-SA |
| **Microduck Training Studio** | Bancada web de treinamento no computador: gestão de experimentos, curvas, avaliação e exportação de modelos | [`training-studio/`](training-studio/README.md), executa separado do Studio do pato | Código Apache-2.0; licenças de dependências e materiais na descrição desse diretório |
| servo-web (antigo) | Console web de depuração de servos de setembro, usado antes do Studio existir | [`tools/servo-web/`](../tools/servo-web/) | Apache-2.0 |

Os pontos de entrada independentes de teste de servos, verificação da IMU e análise de marcha estão no [guia de ferramentas](../tools/README.md); os de placas e firmware estão em [Engenharia de hardware](../hardware/README.md).

## Treinamento em simulação do HD1910

**[`training/`](training/)** contém o projeto Python de treinamento completo: modelos, parâmetros 1910 M6 do LuwuDynamics,
configuração de tarefas do HD1910, adaptação da inicialização do controlador Feetech, arquivos de lock de dependências e testes.
Basta clonar este repositório; não é preciso baixar o branch `microduck_rl` separadamente.

[Início rápido](training/README.md) · [Instruções de execução e explicação dos problemas de inicialização](training/docs/hd1910-baseline.md) ·
[Origem e licenças](training/UPSTREAM.md). Atualmente é a linha de base de integração em simulação; o treinamento de marcha com testes no robô real ainda não foi concluído.

## Depuração da placa IMU avulsa

O firmware STM32G031 + LSM6DSV16X e os materiais de hardware ficam em [`hardware/imu_to_dxl/firmware/`](../hardware/imu_to_dxl/firmware/). A exibição de postura foi incorporada a [`tools/servo-web/`](../tools/servo-web/): use `python server.py --port COM5 --imu-jlink imu-jlink.json`, e o mesmo modelo recebe simultaneamente os ângulos das juntas dos servos e a postura do tronco vinda do J-Link; com apenas a placa avulsa, use `--fake` no lugar do parâmetro de porta serial.

**Escopo: firmware e validação simulada, observação em bancada via SWD.** O código-fonte anexo 0.2.0 já implementa o protocolo Feetech SCS/STS, ID 200, bloco de 15 bytes no endereço 56 e enfileiramento de leitura síncrona; os registros de testes no host e de aceitação em barramento simulado estão em [`VALIDATION.md`](../hardware/imu_to_dxl/firmware/VALIDATION.md). A placa real já tem registros de gravação e observação da IMU, mas o barramento misto real, a temporização de resposta/liberação e a aceitação do robô completo em operação ainda não foram concluídos. A afirmação anterior de que "ainda é a linha de base Dynamixel, contrato não implementado" está desatualizada e não deve ser confundida com "o barramento real ainda não foi aceito".

## Histórico: primeira versão do backend Feetech de setembro (em desuso)

> O programa principal em uso agora é o fork listado na tabela de visão geral acima. O que segue é a primeira versão do backend Feetech, escrita em setembro sobre o oficial 0.14.1, que nunca foi testada na placa e fica como referência.
>
> [`fanhao375/microduck` · branch `feetech`](https://github.com/fanhao375/microduck/tree/feetech): já acompanha o oficial **0.14.1**, com os testes dos três crates passando (duck-control 80 / robotd-params 151 / robotd 93).

| Arquivo | Linhas | O que faz |
|---|---|---|
| [`duck-control/src/bus_feetech.rs`](https://github.com/fanhao375/microduck/blob/feetech/duck-control/src/bus_feetech.rs) | 744 (novo) | **Corpo do `FeetechIo`**: um `sync_read` por tick no endereço 56 com comprimento 15; conversão de posição com tabela de sentidos; escrita do alvo com limitação (clamp); `set_gain` escreve apenas P (o Feetech não tem equivalente de I/D); `reboot` primeiro desliga o torque e depois 0x08; na inicialização verifica ordem de bytes END / nível de resposta / modo de operação / baud rate / fase; `imu_ready()` sempre false. Inclui 11 testes unitários |
| [`duck-control/src/bus_select.rs`](https://github.com/fanhao375/microduck/blob/feetech/duck-control/src/bus_select.rs) | 149 (novo) | Enum `AnyBus`, escolhendo um entre dois backends (`Box<dyn RobotIo>` não compila; o motivo está em [documento de arquitetura](飞特适配架构.md) §3.1) |
| `robotd-params/` | +50 | `[bus] protocol = "dynamixel2" \| "feetech-sts"`, `[bus.feetech] p_scale / directions`, registrados na tabela de parâmetros |
| `robotd/src/main.rs` | +63 | `BusIo` substituído por `AnyBus`, abrindo o barramento com o protocolo; parâmetro inválido gera erro direto, sem nova tentativa |
| `duck-control/src/model.rs` | +8 | Valor esperado do modo de operação 4 de fábrica do HD-1910 |
| `deploy/robotd.toml` | — | Duas chaves novas, com comentários |

Compilação (requer Rust):

```bash
git clone -b feetech https://github.com/fanhao375/microduck.git
cd microduck && cargo test -p duck-control -p robotd-params -p robotd
```

**Por que fica no fork e não neste repositório**: o `FeetechIo` é um arquivo do crate `duck-control` dentro do workspace Rust oficial,
usa o trait `RobotIo` oficial, o genérico `Safety<T>` e o sistema de parâmetros — copiado isoladamente, não compila.
No branch do fork, quando o oficial é atualizado basta um `git rebase` para acompanhar (em 2026-09-20 acompanhou 55 commits, zero conflitos),
e no futuro também dá para abrir PR direto no oficial. O oficial é Apache-2.0; mantendo as alterações no fork, a origem também fica clara.

| Arquivo | Conteúdo |
|---|---|
| [`飞特适配架构.md`](飞特适配架构.md) | Arquitetura atual do runtime oficial (onde fica a junção, o que cada tick faz, sequência de inicialização), diferenças item a item entre os dois servos, plano de adaptação, **arquitetura do firmware da placa pequena `imu_to_dxl` (§4)**, aceitação em bancada, lista de pendências, construção e implantação / relação com a política / modos de falha, registro de revisões |

O contrato de interface do firmware da placa pequena (bloco de 15 bytes, temporização, regras de enfileiramento, aceitação) está em documento próprio: [`hardware/imu_to_dxl/总线协议.md`](../hardware/imu_to_dxl/总线协议.md), com a ferramenta auxiliar [`tools/imu200/`](../tools/imu200/) (placa falsa + vetores de teste + aceitação em placa real).

Como gravar a placa e instalar o runtime oficial está em [Sem HAT](../docs/不打HAT.md) e [`tools/radxa/`](../tools/radxa/).

O texto original do protocolo e da tabela de memória da Feetech (instantâneo da versão de 2026) está em [`docs/飞特资料/`](../docs/飞特资料/).

Antes de mexer no código, primeiro faça os servos responderem: [`tools/servo-web/`](../tools/servo-web/) é um console web de depuração conectado à porta serial — arraste controles deslizantes, veja as leituras de volta, exporte o registro de registradores, e o pato 3D acompanha o movimento; os itens 0–8 da lista de bancada do documento de arquitetura usam todos essa ferramenta.
