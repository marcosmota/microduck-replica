# Atribuição e fontes / Attribution

> 🌐 Tradução não oficial para pt-BR do [original](https://github.com/fanhao375/microduck-replica). Em caso de divergência, vale o original. Licença: CC BY-NC-SA 4.0 (texto alterado: traduzido do chinês/inglês).

Este repositório é uma **pesquisa de réplica do Microduck feita por terceiros**, sem vínculo com a Pollen Robotics e sem o seu endosso.

## Fontes do upstream

| Fonte | Autor | Licença |
|---|---|---|
| [pollen-robotics/microduck_rl](https://github.com/pollen-robotics/microduck_rl) | Pollen Robotics | Código Apache-2.0; **modelos 3D CC BY-NC-SA** (o README do upstream escreve "BY-SA-NC") |
| [pollen-robotics/microduck](https://github.com/pollen-robotics/microduck) | Pollen Robotics | Apache-2.0 |
| Parâmetros 1910 M6 e configuração associada de referência de [LuwuDynamics/xgoduck_rl](https://github.com/LuwuDynamics/xgoduck_rl) | LuwuDynamics | Apache-2.0 |
| Modelo de atuador e biblioteca de identificação de [Rhoban/BAM](https://github.com/Rhoban/bam) (dependência do treinamento) | Rhoban | Apache-2.0 |

O Microduck é um produto comercial da Pollen Robotics, com hardware **parcialmente aberto**:

- **A placa RPI Robot HAT foi totalmente aberta pelo fabricante** ([`elec_RPI_Robot_HAT`](https://github.com/pollen-robotics/elec_RPI_Robot_HAT), Apache-2.0, com projeto KiCad e arquivos de produção)
- **A placa `imu_to_dxl`, o CAD editável das peças mecânicas, a BOM do robô completo e a documentação de montagem** não foram publicados

Toda a informação geométrica deste repositório vem do **modelo de simulação MJCF e das malhas STL** publicados no repositório `microduck_rl` do upstream;
as conclusões sobre a eletrônica vêm do código-fonte, da device tree e dos arquivos de configuração do repositório `microduck`.

> Errata (2026-09-03): este arquivo afirmava antes que «o hardware não é aberto»; a afirmação estava errada — a placa HAT é aberta.

## Conteúdo derivado neste repositório

O conteúdo abaixo foi gerado ou organizado por este projeto a partir do MJCF + STL publicados pelo upstream e é uma **obra derivada sob CC BY-NC-SA**,
portanto é publicado sob a mesma licença:

| Caminho | Conteúdo | Relação com o upstream |
|---|---|---|
| `assembly-drawings/` | Todas as renderizações e vistas explodidas | Geradas por renderização a partir do MJCF + STL do upstream |
| Malhas 3D em `software/training/` | Geometria de simulação usada no treinamento | Mantêm a origem e a licença do modelo do upstream; não são arquivos de impressão física |
| `tools/servo-web/model/` | Modelo de exibição do console web de depuração | Convertido do MJCF / STL do upstream, não é o CAD físico mais recente |
| `software/studio/web/model/` | O pato 3D na página web do Studio | Convertido do modelo de simulação do upstream, mantendo só a forma externa, não é o CAD físico mais recente |
| `docs/hole_analysis.json` | Dados de análise da geometria dos furos | Calculados a partir dos STL do upstream |

> **Migração dos downloads de modelos (2026-09-28)**: `print/` e `cad/` passaram a ser notas de entrada que apontam para o [repositório CAD](https://github.com/fanhao375/microduck-replica-cad); os antigos STL de peças individuais, STL de montagem e tabelas de correspondência foram removidos da versão atual, para evitar que fossem confundidos com os arquivos de impressão mais recentes. Os modelos antigos no histórico do Git continuam sendo obras derivadas, por renomeação, classificação ou transformação de montagem, dos STL do upstream, e seguem a CC BY-NC-SA; remover as cópias para download não altera a autoria nem a licença dos arquivos históricos.

**Conteúdo original deste projeto:**

| Caminho | Conteúdo | Licença |
|---|---|---|
| `scripts/` | Scripts de renderização, exportação, análise de furos etc. | **Apache-2.0** |
| `tools/stl_viewer.html` | Visualizador STL WebGL sem dependências | **Apache-2.0** |
| Código de `software/studio/` | Console web Microduck Studio (sem `web/vendor/`, `deps/`, `web/model/`) | **Apache-2.0** |
| `software/image/` | Scripts de construção da imagem da réplica, de primeiro boot e de configuração de Wi-Fi | **Apache-2.0** |
| `software/updater/` | Construção de pacotes de componentes assinados, adaptação do motor de atualização original, coordenação de atualização pela web e ferramenta de bootstrap inicial | **Apache-2.0**; o updaterd original vem da Pollen Robotics, origem 1fa8438 |
| Código de `software/duck-echo/` | Voz local por R3, gravação e interação com música; não inclui chave privada da máquina de build nem gravações de usuários | **Apache-2.0**; o modelo Vosk externo e as dependências ARM64 seguem suas próprias licenças, e a lista mantém origem e hash |
| `docs/*.md`, `*.md` | Toda a documentação e textos de análise por engenharia reversa | **CC BY-NC-SA 4.0** |
| Fotos em `build-log/` | Fotos da construção do robô real | **CC BY-NC-SA 4.0**, tiradas e licenciadas por participantes deste projeto |
| Materiais originais do projeto em `assets/` | Capa, QR codes do grupo de conversa e imagens de prévia do projeto; não incluem as imagens oficiais de placas abaixo | Idem |

## Imagens oficiais de placas

As duas imagens abaixo são citadas sem alteração, mantendo cada uma o seu autor e a sua licença original, e não são cobertas pela licença CC BY-NC-SA da documentação deste projeto nem dos materiais originais:

| Caminho | Fonte e autor | Licença |
|---|---|---|
| `assets/hw/radxa-zero3w-official.webp` | [Documentação oficial da Radxa ZERO 3](https://docs.radxa.com/en/zero/zero3), Radxa Computer (Shenzhen) Co., Ltd. | [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/) |
| `assets/hw/rpi-robot-hat-official.png` | [Repositório oficial da HAT da Pollen Robotics](https://github.com/pollen-robotics/elec_RPI_Robot_HAT), Pollen Robotics | [Apache-2.0](https://github.com/pollen-robotics/elec_RPI_Robot_HAT/blob/main/LICENSE) |

O endereço das imagens originais, a versão fixada e os valores de verificação estão no [registro de origem das imagens](assets/hw/official-board-images.md).

## Projeto de treinamento embutido

O [`software/training/`](software/training/) reúne o projeto de treinamento completo, para que quem faz a réplica obtenha com um único download
o código, os parâmetros e o modelo de simulação. A versão fixada importada e o escopo das alterações estão em [UPSTREAM.md](software/training/UPSTREAM.md).

- O código de treinamento, as ferramentas e a documentação original da Pollen Robotics mantêm a licença **Apache-2.0**; a tarefa HD1910,
  a adaptação de inicialização, os testes e as instruções acrescentados neste diretório também usam Apache-2.0. O texto completo da licença está em
  [`software/training/LICENSE`](software/training/LICENSE).
- Os **modelos 3D e malhas dentro do diretório de treinamento mantêm a declaração CC BY-NC-SA do upstream** e não são cobertos pela licença Apache do código.
- O `1910_m6.json` da LuwuDynamics é mantido sem alterações; a origem fixada, os valores de verificação e a cópia da licença estão em
  [`software/training/src/mjlab_microduck/robot/hd1910/`](software/training/src/mjlab_microduck/robot/hd1910/).
  Este projeto não identificou esses parâmetros por conta própria; ao treinar políticas com eles no futuro, a referência aos parâmetros continua sendo mantida.
- O Rhoban/BAM é instalado como dependência de versão travada, mantendo a sua origem e licença.

A licença CC BY-NC-SA da documentação deste projeto, na tabela acima, não cobre a documentação Apache dentro do diretório de treinamento.

## Outros softwares de terceiros

**Software de terceiros distribuído sem alterações:**

| Caminho | Conteúdo | Origem | Licença |
|---|---|---|---|
| `tools/飞特/` | Software de depuração Feetech FD 1.9.8.5 + driver CH340 | [gitee.com/ftservo/fddebug](https://gitee.com/ftservo/fddebug), copiado em 2026-09-11 | **MIT** (FTServo 2024, LICENSE no mesmo diretório) |
| `tools/rufus/` | Rufus 4.15 versão portátil Windows x64 e o código-fonte da mesma versão | [pbatard/rufus v4.15](https://github.com/pbatard/rufus/releases/tag/v4.15), baixado sem alterações em 2026-10-25 | **GPL-3.0-or-later**; licença, arquivo do código-fonte, valores de verificação e registro de origem estão nesse diretório |
| `software/studio/web/vendor/` | Bibliotecas de front-end do Studio: Preact, htm 3.1.1, three.js 0.160.0 | Releases oficiais de cada uma | Preact e three.js **MIT**, htm **Apache-2.0** (licenças em `web/vendor/licenses/`) |
| `software/studio/deps/` | Dependências Python do Studio (anyio, click, h11, idna, pyserial, sniffio, starlette, typing_extensions, uvicorn, wsproto) | Wheels originais do pypi.org, versões e valores de verificação em `deps/VERSIONS.txt` | MIT / BSD-3-Clause / PSF-2.0 etc., licença de cada pacote no respectivo `*.dist-info` |

> Os trechos de código-fonte do upstream citados na documentação (comentários, constantes, definições de registradores) vêm de `pollen-robotics/microduck`
> e seguem a sua licença **Apache-2.0**; os pontos de citação indicam o caminho do arquivo.

## Repositório complementar: desenhos 3D em SolidWorks

[**fanhao375/microduck-replica-cad**](https://github.com/fanhao375/microduck-replica-cad)
mantém de forma unificada os arquivos editáveis SolidWorks / STEP, os arquivos de impressão e o manual de montagem de 21 páginas. As versões Feetech e XL330 são fornecidas separadamente, e a lista de arquivos vale conforme a release correspondente desse repositório; o repositório principal não mantém mais uma cópia independente dos modelos de impressão.

| Conteúdo | Autor | Licença |
|---|---|---|
| Desenhos SolidWorks, manual de montagem, diagramas de componentes | **机械行者Robo** (Xiaohongshu 270594280 / Douyin 1852366168) | **CC BY-NC-SA 4.0** |

Esse conjunto de desenhos é um **modelo paramétrico editável reconstruído a partir das malhas STL publicadas pelo upstream**, uma obra derivada sob CC BY-NC-SA,
e por isso é publicado sob a mesma licença. **Os direitos autorais pertencem ao próprio autor**; ao reproduzir ou redistribuir, mantenha a atribuição.
Os desenhos ficam em um repositório independente, e não neste, porque os arquivos-fonte SolidWorks descompactados ocupam cerca de 340 MB,
e incorporá-los faria cada clone deste repositório carregar esse volume.

## Nome da licença

O nome canônico é **CC BY-NC-SA 4.0** (Atribuição - Não Comercial - Compartilhamento pela Mesma Licença).
O README do upstream escreve «BY-SA-NC»; quando este repositório adota essa grafia, refere-se à mesma licença.

> Cabe esclarecer: o arquivo `LICENSE` do repositório `microduck_rl` do upstream contém apenas Apache-2.0,
> e **os termos CC aparecem só em uma linha do seu README**. A adoção, por este repositório, da CC BY-NC-SA, mais restritiva, para os modelos 3D
> é uma postura conservadora — se o upstream esclarecer que é Apache-2.0 puro, este repositório será flexibilizado de acordo.

## Declaração de conformidade

Todas as conclusões deste repositório vêm de:

- **Código-fonte e device tree** publicados pela Pollen Robotics (`pollen-robotics/microduck`, Apache-2.0)
- **Modelo de simulação e malhas** publicados (`pollen-robotics/microduck_rl`)
- **Projeto KiCad** publicado (`pollen-robotics/elec_RPI_Robot_HAT`, Apache-2.0)
- **Parâmetros 1910 M6 e configuração associada** publicados pela LuwuDynamics (`LuwuDynamics/xgoduck_rl`, Apache-2.0)
- Manuais de componentes e páginas de especificação publicados pelos fabricantes

**Nenhum material não público foi usado, nenhum equipamento físico foi desmontado e nenhum arquivo de projeto não publicado foi acessado.**
O firmware da placa `imu_to_dxl` não está em nenhum repositório aberto; este repositório reconstituiu apenas o seu comportamento observável do lado do barramento.

## Declaração de uso não comercial

Os modelos 3D do upstream usam **CC BY-NC-SA**, e as suas obras derivadas **não podem ser usadas para fins comerciais**.
Este repositório destina-se apenas a estudo, pesquisa e réplicas pessoais.

## Como dar atribuição

> Baseado no modelo Microduck da Pollen Robotics (CC BY-NC-SA)
> https://github.com/pollen-robotics/microduck_rl
