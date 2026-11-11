# Microduck Réplica

> 🌐 Tradução não oficial para pt-BR do [original](https://github.com/fanhao375/microduck-replica). Em caso de divergência, vale o original. Licença: CC BY-NC-SA 4.0 (texto alterado: traduzido do chinês/inglês).

**Português (BR)** · [English](README.en.md)

Projeto de réplica do robô completo, de terceiros, baseado no [Microduck da Pollen Robotics](https://pollen-robotics.com/microduck/), com materiais que vão da compra de peças, impressão e montagem até a calibração, o controle e o treinamento.

**Imagem oficial 20261104**. Integra a recalibração do IMU na postura em pé (HOME), a verificação independente de orientação do ID 200, a recuperação de hotspot e funcionalidades de software anteriores, como o mapeamento do controle (gamepad). Boot com cartão novo, hotspot após troca real de rede Wi-Fi, marcha e levantar após queda ainda aguardam validação no robô real. Predefinições de orientação de montagem fixa ficam para o futuro. [Baixar imagem](https://github.com/fanhao375/microduck-replica/releases/tag/microduck-replica-image-20261104) · [Software de teste correspondente](https://github.com/fanhao375/microduck-replica/releases/tag/microduck-software-v2026.11.4) · [Lista completa de alterações](docs/更新说明-20261104.md)

**A versão Feetech HD-1910 concluiu a validação do robô completo no hardware real: levantar, marcha e levantar depois da cambalhota para frente funcionam.** A linha principal usa **15 servos Feetech HD-1910** e oferece uma **imagem da réplica** pronta para gravar e o **console web Microduck Studio**, que permite configurar o Wi-Fi, calibrar e controlar remotamente pelo navegador do celular ou do computador.

**O fluxo de primeiro uso também foi concluído no robô real:** gravar a imagem → configurar Wi-Fi pelo hotspot → calibrar servos e IMU → parear o controle (gamepad) e testar a parada de emergência → levantar e andar. [Ver registros de validação no robô real](docs/历史更新.md#atualizações-recentes)

## Como o pato está hoje

**2026-10-24 · Movimento validado no robô real**: os dois vídeos reais abaixo mostram a cambalhota para frente seguida de levantar e a marcha sobre a mesa.

<table>
<tr>
<th align="center">Cambalhota para frente e levantar</th>
<th align="center">Marcha sobre a mesa</th>
</tr>
<tr>
<td align="center" valign="middle"><a href="assets/demos/2026-10-24-roll-and-rise.mp4"><img src="assets/demos/2026-10-24-roll-and-rise.gif" alt="O robô real faz uma cambalhota para frente e volta a ficar de pé" width="260"></a></td>
<td align="center" valign="middle"><a href="assets/demos/2026-10-24-walking.mp4"><img src="assets/demos/2026-10-24-walking.gif" alt="Pato versão Feetech caminhando sobre a mesa" width="400"></a></td>
</tr>
<tr>
<td align="center"><a href="assets/demos/2026-10-24-roll-and-rise.mp4">MP4 completo · cerca de 8 segundos</a></td>
<td align="center"><a href="assets/demos/2026-10-24-walking.mp4">MP4 completo · cerca de 6 segundos</a></td>
</tr>
</table>

Os GIFs são prévias sem som; os MP4 mantêm o vídeo completo e o áudio.

**[Baixar imagem](https://github.com/fanhao375/microduck-replica/releases/latest)** · **[Guia de primeiro uso (PDF / web)](docs/上手指南/README.md)** · **[CAD e peças impressas](https://github.com/fanhao375/microduck-replica-cad)** · **[Tutoriais de montagem](#-vídeos-de-montagem)** · **[Lista de compras (BOM)](BOM/README.md)** · **[Projetos de hardware](hardware/README.md)**

---

### Versões compatíveis

- **Feetech HD-1910 + Radxa Zero 3W**: use a imagem da réplica deste repositório e o console web Studio.
- **Dynamixel XL330**: há CAD e materiais de montagem correspondentes; é preciso usar um software de execução compatível.

[Escolher arquivos de impressão e lista de compras](#arquivos-de-impressão-e-lista-de-compras) · [Ver comparação detalhada dos servos](docs/执行器选型.md)

---

## Grupo de conversa

Há quem esteja fazendo a mesma coisa, então montamos um grupo no WeChat para discutir o andamento da réplica, os problemas encontrados e a compra de componentes.

Abaixo está o QR code do **Grupo 15 da Réplica do Pato** (um grupo do WeChat com 200 membros não pode mais ser acessado por QR code).

<div align="center">
  <img src="assets/wechat-group-15.png" alt="QR code do WeChat do Grupo 15 da Réplica do Pato" width="280">
  <br>
  <sub><b>Grupo 15 da Réplica do Pato · QR code válido até 2026-11-12</b> (QR codes de grupo do WeChat expiram automaticamente em 7 dias)<br>
  Se tiver expirado, abra uma <a href="https://github.com/fanhao375/microduck-replica/issues">issue</a> avisando e eu coloco um novo</sub>
</div>

---

## 📺 Vídeos de montagem

**[Série do Bilibili "MicroDuck 从零安装教程" (montagem do zero)](https://space.bilibili.com/312747174/lists/9279084?type=season)** — gravada por **机械行者Robo**, autor dos desenhos CAD e da BOM de montagem. Começa pela lista de peças e monta um subconjunto por episódio. São 8 episódios no total; 6 foram publicados até 2026-10-12:

À direita da lista de episódios ficam as contas do autor, nesta ordem: **Bilibili, Xiaohongshu (RED) e Douyin**. Escaneie o código ou clique na imagem para abrir a conta.

<table>
<thead>
<tr><th>Episódio</th><th>Conteúdo</th><th nowrap>Duração</th><th>Bilibili</th><th>Xiaohongshu</th><th>Douyin</th></tr>
</thead>
<tbody>
<tr>
<td nowrap><a href="https://www.bilibili.com/video/BV1RoHX68EFV/">Ep. 1</a></td><td width="260"><a href="https://www.bilibili.com/video/BV1RoHX68EFV/">Lista de peças e acessórios</a></td><td nowrap>1:32</td>
<td rowspan="6" align="center" valign="middle" width="210"><a href="https://space.bilibili.com/312747174"><img src="assets/social/bilibili-qr.jpg" alt="Bilibili: 机械行者Robo" width="200"></a><br><sub><b>机械行者Robo</b><br>UID 312747174</sub></td>
<td rowspan="6" align="center" valign="middle" width="210"><a href="https://www.xiaohongshu.com/user/profile/5ad950114eacab046016f038"><img src="assets/social/xiaohongshu-qr.jpg" alt="Xiaohongshu: 机械行者" width="200"></a><br><sub><b>机械行者</b><br>ID do Xiaohongshu 270594280</sub></td>
<td rowspan="6" align="center" valign="middle" width="210"><a href="https://www.douyin.com/user/MS4wLjABAAAArLa_6yE3GE-AoRURoQnIHdbbI7yM-GHhzlgLZ-XbeYE"><img src="assets/social/douyin-qr.jpg" alt="Douyin: 机械行者Robo" width="200"></a><br><sub><b>机械行者Robo</b><br>ID do Douyin 1852366168</sub></td>
</tr>
<tr><td nowrap><a href="https://www.bilibili.com/video/BV1Xppb6tECr/">Ep. 2</a></td><td><a href="https://www.bilibili.com/video/BV1Xppb6tECr/">Montagem das canelas</a></td><td nowrap>2:21</td></tr>
<tr><td nowrap><a href="https://www.bilibili.com/video/BV1ycpb6yEwB/">Ep. 3</a></td><td><a href="https://www.bilibili.com/video/BV1ycpb6yEwB/">Montagem das coxas</a></td><td nowrap>5:13</td></tr>
<tr><td nowrap><a href="https://www.bilibili.com/video/BV1Kwpg6wE9G/">Ep. 4</a></td><td><a href="https://www.bilibili.com/video/BV1Kwpg6wE9G/">Montagem do conjunto do tronco</a></td><td nowrap>2:29</td></tr>
<tr><td nowrap><a href="https://www.bilibili.com/video/BV1EEpg6nEkD/">Ep. 5</a></td><td><a href="https://www.bilibili.com/video/BV1EEpg6nEkD/">Montagem do pescoço</a></td><td nowrap>0:40</td></tr>
<tr><td nowrap><a href="https://www.bilibili.com/video/BV1aKpE6hEzj/">Ep. 6</a></td><td><a href="https://www.bilibili.com/video/BV1aKpE6hEzj/">Montagem da cabeça</a></td><td nowrap>5:50</td></tr>
</tbody>
</table>

### Arquivos de impressão e lista de compras

Escolha primeiro os arquivos de impressão conforme a versão do servo e depois prepare as peças compradas correspondentes.

| Material | Versão do servo | Onde encontrar |
|---|---|---|
| **Arquivos de impressão e CAD** | **Feetech HD-1910** | [Seção da versão Feetech](https://github.com/fanhao375/microduck-replica-cad#飞特-hd-1910-打印资料): projeto de impressão, fontes CAD e notas de atualização |
| **Arquivos de impressão e CAD** | **Dynamixel XL330** | [Seção da versão XL330](https://github.com/fanhao375/microduck-replica-cad#dynamixel-xl330-打印资料): fontes CAD e instruções de impressão da versão correspondente |
| **Lista de compras de montagem** | **Feetech HD-1910** | **[Prévia online](BOM/README.md)** · **[Baixar Excel](BOM/装配BOM-机械行者Robo-2026-10-23-带采购链接.xlsx)**: organizada por 机械行者Robo, versão de 2026-10-23, com quantidades, materiais e links de compra |

As peças de encaixe das duas versões de servo não são intercambiáveis. A versão e a data de atualização dos arquivos de impressão são mantidas na seção correspondente do repositório CAD. Esta lista de compras segue a solução Feetech; os servos XL330 e a eletrônica associada precisam ser conferidos à parte.

---

## Onde estamos

Até **2026-10-25**, **a réplica da versão Feetech, o primeiro uso e a cadeia de movimento no robô real estão funcionando**. A seguir estão os resultados de cada parte e os projetos de extensão; o escopo dos testes e o histórico estão no [histórico de atualizações](docs/历史更新.md).

---

## 💻 Software · imagem da réplica e Microduck Studio

**Revisão de 2026-10-31 · 2026.10.3101: correção das ilustrações de boca 34 / guinada 32**: corrige a inversão das carcaças de dois servos no modelo 3D e o cache de modelo antigo; seleção, destaque e linhas de numeração ficam consistentes, sem trocar os IDs dos servos físicos. Quem já tem a entrada de atualização da versão 27 pode instalar direto, e a versão 31 também pode ser atualizada; o programa nativo correspondente continua o da versão 31, sem regravar a imagem. [Pacote de revisão de software](https://github.com/fanhao375/microduck-replica/releases/tag/microduck-software-v2026.10.3101) · [Passos de atualização e materiais do grupo 14](docs/软件修订-20261031-r1.md)

**2026-10-31 · Atualização do pacote de software: novo mapeamento do controle (gamepad), zoom da câmera no lugar e orientações de SSH**: dispositivos que já têm a entrada de atualização da versão 27 só precisam instalar o pacote de software, sem regravar a imagem. O controle aprende botões, direcionais analógicos e gatilhos e salva a configuração; o driver precisa ser confirmado e instalado separadamente conforme as instruções da página, não basta atualizar a página web. O atualizador original da versão 27, o mapeamento no Linux e a verificação de permissões do serviço passaram, mas a instalação física do novo driver do controle e os vários modelos de controle ainda aguardam validação. [Pacote de software e anexo de bootstrap inicial](https://github.com/fanhao375/microduck-replica/releases/tag/microduck-software-v2026.10.31) · [Passos de operação e escopo da validação](docs/功能软件升级-20261030.md)

**2026-10-30 · Atualização do pacote de software: diagnóstico independente conectando só o IMU**: o IMU de barramento deixa de ser bloqueado pela verificação de inicialização dos servos. Sem servos ou com apenas parte deles conectados, as páginas de visão geral e de IMU continuam mostrando resposta da placa, identificação e estado das amostras; resultados vencidos, desconexões e troca de configuração não reaproveitam resultados antigos. O diagnóstico não libera movimento nem escrita de calibração. Atualize primeiro a página web e depois confirme explicitamente a atualização do programa principal para `.47` / API 42. [Pacote de software e anexo de bootstrap](https://github.com/fanhao375/microduck-replica/releases/tag/microduck-software-v2026.10.30) · [Passos de atualização e instruções de uso](docs/仅接IMU的独立诊断与软件更新-20261030.md)

**2026-10-29 · Atualização do pacote de software: estado do IMU e dos servos exibido separadamente**: enquanto o programa principal espera pelo IMU, as páginas de visão geral e de servos continuam mostrando as respostas já recebidas dos servos, posição bruta, tensão e temperatura, distinguindo dados ausentes, desconhecidos e vencidos. Atualize primeiro a página web e depois confirme explicitamente a atualização do programa principal para `.46`; cartões existentes instalam pela atualização de software. [Pacote de software e anexo de bootstrap](https://github.com/fanhao375/microduck-replica/releases/tag/microduck-software-v2026.10.29) · [Passos de atualização e escopo da validação](docs/软件更新与独立设备状态说明-20261029.md)

**2026-10-28 · Atualização do pacote de software (sem regravar o cartão SD)**: quem já tem a entrada de atualização na página de configurações deve instalar primeiro o pacote web / voz `2026.10.28`; depois de reconectar, pode apoiar o robô para aliviar a carga dos servos, confirmar explicitamente a atualização do programa principal e então, na página de IMU, escolher `Qinmo · Feetech` e verificar a orientação. A atualização automática do programa principal só é compatível com a linha de base conhecida 20261027; as placas existentes continuam com identificação automática por padrão. O Qinmo no hardware real e a atualização do robô completo nesta versão ainda aguardam validação; o acompanhamento de música e o diálogo livre por voz com IA continuam sem solução. [Pacote de software e anexo de bootstrap](https://github.com/fanhao375/microduck-replica/releases/tag/microduck-software-v2026.10.28) · [Passos de atualização para usuários existentes](docs/软件更新与Qinmo使用说明-20261028.md) · [Uso do Qinmo pela web](docs/Qinmo-IMU-网页使用.md)

**Imagem da réplica 20261027: atualização de software pela web e voz local para sentar/levantar.** A página de configurações pode verificar, baixar ou enviar pacotes de software assinados, com reversão se a instalação falhar; um toque em R3 liga/desliga a escuta, que reconhece "sentar" e "levantar" (em mandarim), com modelo em chinês incluído e sem necessidade de conta na nuvem. Mantêm-se a gravação e reprodução de movimentos pela web, o modo animado e o gerenciamento de modelos. Cartões existentes instalam uma vez a entrada de atualização via SSH; depois disso, as atualizações rotineiras de software dispensam regravação.

Esta versão mantém o programa principal, os modelos de marcha e a identificação automática do IMU no ID 200 do barramento da 20261023c. Software, transações assinadas, navegador e verificação somente leitura da imagem passaram; o primeiro uso com cartão novo, a atualização/reversão de cartões antigos e a acústica com voz humana real ainda aguardam validação física. O acompanhamento de cabeça e corpo com a música continua pendente, e o diálogo livre por voz com IA ainda não está integrado. [Imagem e checksums](https://github.com/fanhao375/microduck-replica/releases/tag/microduck-replica-image-20261027) · [Pacote de atualização de software](https://github.com/fanhao375/microduck-replica/releases/tag/microduck-software-v2026.10.27) · [Uso de atualização e voz](docs/软件更新与语音使用说明-20261027.md) · [Escopo da validação](docs/superpowers/verification/2026-10-27-web-updates.md)

<p align="center"><a href="software/studio/README.md"><img src="assets/studio-overview-20261025-demo.jpg" alt="Demonstração da interface atual do Microduck Studio; as leituras do dispositivo são dados simulados" width="920"></a><br><sub>Demonstração da interface do Studio; as leituras do dispositivo são dados simulados</sub></p>

**[Baixar imagem](https://github.com/fanhao375/microduck-replica/releases/latest)** · **[Guia de primeiro uso](docs/上手指南/README.md)** · **[Documentação do Studio](software/studio/README.md)**

A imagem é gratuita e destinada apenas a réplicas pessoais não comerciais; as licenças de cada componente estão na página da release. A câmera usa por padrão uma fonte de teste; o escopo de validação da câmera real e do ToF está nas notas desta versão.

---

## 🔧 Mecânica · impressão e montagem em um só lugar

**机械行者Robo mantém dois conjuntos de materiais: Feetech HD-1910 e Dynamixel XL330.** Modelos, projetos de impressão e notas de atualização ficam no repositório CAD; escolha conforme a versão do seu servo.

| O que você precisa | Onde encontrar |
|---|---|
| **Impressão em um só lugar** | **[Bambu MakerWorld (versão Feetech)](https://makerworld.com.cn/zh/models/2963569-microduck#profileId-3478428)** · [Projeto 3MF do Bambu Studio](https://github.com/fanhao375/microduck-replica-cad#打印) |
| **Modelos e materiais de impressão** | **[Seção Feetech HD-1910](https://github.com/fanhao375/microduck-replica-cad#飞特-hd-1910-打印资料)** · **[Seção XL330](https://github.com/fanhao375/microduck-replica-cad#dynamixel-xl330-打印资料)** |
| **Tutoriais de montagem** | **[Manual de montagem em PDF](https://github.com/fanhao375/microduck-replica-cad/blob/master/安装说明书/microduck装配安装说明书.pdf)** · **[Vídeos por episódio](#-vídeos-de-montagem)**; as duas versões usam as peças estruturais correspondentes |
| **Lista de compras** | **[Prévia online da BOM](BOM/README.md)** · **[Baixar Excel](BOM/装配BOM-机械行者Robo-2026-10-23-带采购链接.xlsx)**: versão Feetech de 2026-10-23; os servos XL330 e a eletrônica associada precisam ser conferidos à parte |

[Repositório CAD e releases mais recentes](https://github.com/fanhao375/microduck-replica-cad/releases) · [Diário de construção](构建日志.md)

---

## 🔌 Elétrica · cinco placas de circuito

Este repositório fornece os projetos da placa IMU de barramento, do HUB de barramento dos servos e da placa de interface da bateria; a placa principal é a **Radxa ZERO 3W**, e a placa de expansão é a **RPI Robot HAT**, de código aberto oficial da Pollen Robotics.

<table>
<tr>
<th>Placa IMU de barramento</th>
<th>HUB de barramento dos servos</th>
<th>Placa de interface da bateria</th>
<th>Placa principal Radxa<br>Radxa ZERO 3W</th>
<th>HAT oficial<br>RPI Robot HAT</th>
</tr>
<tr>
<td align="center" valign="middle" width="170"><a href="assets/hw/imu-to-dxl-1v0a-3d.png"><img src="assets/hw/imu-to-dxl-1v0a-3d.png" alt="Prévia do projeto 3D da placa IMU de barramento" width="160"></a></td>
<td align="center" valign="middle" width="170"><a href="assets/hw/wire-hub-1v0a-3d.png"><img src="assets/hw/wire-hub-1v0a-3d.png" alt="Prévia do projeto 3D do HUB de barramento dos servos" width="160"></a></td>
<td align="center" valign="middle" width="170"><a href="assets/hw/pogo-pin-1v0a-3d.png"><img src="assets/hw/pogo-pin-1v0a-3d.png" alt="Prévia do projeto 3D da placa de interface da bateria" width="160"></a></td>
<td align="center" valign="middle" width="170"><a href="assets/hw/radxa-zero3w-official.webp"><img src="assets/hw/radxa-zero3w-official.webp" alt="Imagem oficial da placa Radxa ZERO 3W" width="160"></a></td>
<td align="center" valign="middle" width="170"><a href="assets/hw/rpi-robot-hat-official.png"><img src="assets/hw/rpi-robot-hat-official.png" alt="Renderização oficial da RPI Robot HAT da Pollen Robotics" width="160"></a></td>
</tr>
<tr>
<td align="center" valign="top"><a href="assets/hw/imu-to-dxl-1v0a-pcb.png">Prévia do PCB</a><br><a href="hardware/新版/imu_to_dxl_1V0A/">Arquivos do projeto</a></td>
<td align="center" valign="top"><a href="assets/hw/wire-hub-1v0a-pcb.png">Prévia do PCB</a><br><a href="hardware/新版/wire_hub_1V0A/">Arquivos do projeto</a></td>
<td align="center" valign="top"><a href="assets/hw/pogo-pin-1v0a-pcb.png">Prévia do PCB</a><br><a href="hardware/新版/POGO_PIN_1V0A/">Arquivos do projeto</a></td>
<td align="center" valign="top"><a href="https://item.taobao.com/item.htm?id=763812025816">Comprar (Taobao)</a><br><a href="https://docs.radxa.com/en/zero/zero3">Documentação oficial</a> · <a href="https://docs.radxa.com/en/zero/zero3/download#hardware-design">Arquivos de projeto</a></td>
<td align="center" valign="top"><a href="https://github.com/pollen-robotics/elec_RPI_Robot_HAT">Projeto KiCad</a><br><a href="https://github.com/pollen-robotics/elec_RPI_Robot_HAT/tree/main/production">Arquivos de produção</a></td>
</tr>
</table>

As três primeiras imagens são prévias de projeto 3D das placas próprias; as imagens da Radxa e da HAT vêm de materiais oficiais. Clique na imagem para ver o original; veja os [créditos das imagens](assets/hw/official-board-images.md). Os materiais de projeto da Radxa incluem esquemáticos, dimensões 2D e modelos 3D; os projetos no EasyEDA (嘉立创), os STEP e o andamento das medições reais das três placas próprias estão na descrição de cada projeto.

[Todos os projetos de hardware](hardware/README.md) · [Notas sobre as três placas](hardware/新版/README.md) · [Firmware do IMU de barramento](hardware/imu_to_dxl/README.md)

---

## 🧠 Algoritmos · BAM oficial da Feetech

O [**BAM oficial do HD-1910 da Feetech**](https://gitee.com/ftservo/ftservo-bam) fornece o modelo dinâmico do servo e os parâmetros identificados, como referência para o treinamento em simulação. Este repositório oferece um [projeto de treinamento](software/training/README.md) de código aberto e uma [bancada web de treinamento](software/training-studio/README.md); a configuração do modelo e a origem dos parâmetros usados aqui estão nas [notas de parâmetros](docs/HD-1910-M6参数来源.md).

---

## 🖥️ Plataforma de treinamento de código aberto · Microduck Training Studio

Também criamos o **Microduck Training Studio**: no computador, pelo navegador, é possível criar experimentos de treinamento, ver curvas, pausar e retomar, avaliar checkpoints e exportar modelos ONNX. A bancada de treinamento e o Studio do pato são dois serviços independentes.

<p align="center"><a href="software/training-studio/README.md"><img src="assets/training-studio-20261025-demo.jpg" alt="Interface real do Microduck Training Studio: modelo 3D, controles de treinamento e curvas sintéticas de demonstração" width="920"></a><br><sub>Interface de demonstração da plataforma de treinamento: o 3D é uma prévia do modelo e as curvas são dados sintéticos de demonstração</sub></p>

Foram validadas a cadeia de teste curto em GPU real, reprodução de avaliação e exportação de modelo; o treinamento longo e a validação no robô real de políticas treinadas por nós ainda estão pendentes.

**[Plataforma de treinamento e instruções de uso](software/training-studio/README.md)** · **[Guia do Action Lab](software/training-studio/web/custom-actions.md)** · **[Projeto de treinamento](software/training/README.md)** · [Registro de validação](software/training-studio/docs/verification.md)

---

## 🧰 Ferramentas · gravação do cartão, depuração e materiais

Os softwares e ferramentas de desenvolvimento mais usados estão reunidos no [índice de ferramentas](tools/README.md); escolha conforme a tarefa.

| O que fazer | Ferramentas e links |
|---|---|
| Gravar a imagem | **[Rufus portátil (EXE)](tools/rufus/rufus-4.15p.exe)** · [Instruções de gravação do cartão](tools/rufus/README.md) |
| Depuração de porta serial e servos | **[FD da Feetech e assistente de porta serial](tools/飞特/README.md)** · [Console web 3D de depuração](tools/servo-web/README.md) |
| Consultar materiais da Feetech, integrar o SDK | **[Manuais oficiais, especificações dos modelos e SDK](docs/飞特资料/README.md)**, indicando a versão do site oficial e a data da cópia offline |
| Testar servos, analisar problemas | [Ferramenta de teste HD1910M](tools/FeeTech_HD1910M_Servo/README.md) · [Ferramenta de teste STS3215](tools/sts3215Servo_testtool/README.md) · [Análise de marcha](tools/gait-analysis/README.md) |

---

## 🦆 Contribuições do bando

Este repositório não foi escrito por uma pessoa só. O código incorporado e as conclusões verificadas em bancada estão registrados aqui.

| Quem | O que contribuiu |
|---|---|
| [@yoyojacky](https://github.com/yoyojacky) | **Ferramenta de teste em Rust para servos Feetech** ([PR #27](https://github.com/fanhao375/microduck-replica/pull/27), já incorporado). Implementação própria e mínima do protocolo STS/SCS (montagem de quadros, checksum, ping, leitura e escrita de registradores, interpretação de estado); a linha de comando varre o barramento (com barra de progresso), lê o estado, move para uma posição indicada, faz teste de um servo ou em lote, informa erro de posição, corrente de pico, carga de pico, tensão e temperatura e dá o veredito PASS/FAIL. A validação em bancada gera uma tabela-resumo com um único comando → [`tools/sts3215Servo_testtool/`](tools/sts3215Servo_testtool/) |
| [@Abo1ish](https://github.com/Abo1ish) | **Firmware da placa IMU + exibição de atitude no console de depuração** ([PR #32](https://github.com/fanhao375/microduck-replica/pull/32), já incorporado). Escreveu a base de firmware STM32G031 para a placa imu_to_dxl, que a Pollen nunca abriu, validada em placa real, com testes no host e um [`VALIDATION.md`](hardware/imu_to_dxl/firmware/VALIDATION.md) que deixa claros os limites → [`hardware/imu_to_dxl/firmware/`](hardware/imu_to_dxl/firmware/); além disso, lê a atitude do IMU em tempo real via J-Link e a integra ao console web de depuração, de modo que o tronco do pato 3D gira junto com a placa real (`--imu-jlink`; sem placa, dá para ver o efeito com `--imu-demo`) → [`tools/servo-web/`](tools/servo-web/) |
| Um membro do grupo do WeChat | **Receita de boot da Radxa Zero 3W V1.12J.** Nesse lote de placas (Wi-Fi trocado por AIC8800DS2), só trocar o bootloader pelo da Radxa ainda não faz iniciar; ele descobriu por testes que a device tree também precisa ser a B1 da Radxa: **bootloader B1 + DTB B1 + kernel Armbian 6.1.115 + sistema Trixie**, e conseguiu fazer funcionar → registrado no [registro de armadilhas](踩坑记录.md#software) e no [guia de uso da imagem](tools/radxa/镜像使用说明.md#6-se-a-sua-placa-não-for-deste-lote) |

---

## Projetos do bando

Alguns membros criam seus projetos em repositórios próprios, fora deste, mas diretamente relacionados ao pato e bem feitos; ficam aqui para divulgação:

| Projeto | Autor | O que faz |
|---|---|---|
| [microduck-color-studio](https://github.com/LathamZ/microduck-color-studio) ([teste online](https://lathamz.github.io/microduck-color-studio/)) | [@LathamZ](https://github.com/LathamZ) | **Estúdio de cores 3D do pato: defina as cores no navegador antes de imprimir.** Pinte cada uma das 70 peças, escolha o material (PLA / fosco / PETG / metálico / fibra de carbono / TPU) e a iluminação para ver o resultado; informe os filamentos que você tem e ele recomenda paletas conforme o estoque (só o que você já tem / acrescentar uma cor / detalhes em acrílico). Também importa o seu próprio 3MF (por exemplo, o da versão Feetech) e exporta um **3MF multiplacas colorido** e STLs separados por peça, com predefinições para Bambu P1S / A1 mini / H2D. Interface em chinês e inglês, funciona também no celular |
| [MICDUCK_FTHD1901_REBUILD](https://github.com/fengj4780-sudo/MICDUCK_FTHD1901_REBUILD) | [@fengj4780-sudo](https://github.com/fengj4780-sudo) | **Melhoria estrutural da versão Feetech + reforços CNC opcionais.** Continua o trabalho a partir dos nossos [desenhos SolidWorks editáveis](https://github.com/fanhao375/microduck-replica-cad): adapta ao servo Feetech (escrito como HD1901 no repositório dele; os arquivos das peças usam HD-1910-C001), reforça as peças fracas, corrige interferências de montagem e conserta alguns modelos danificados. **5 peças das pernas podem ser trocadas por peças CNC** para uma distribuição de esforços mais razoável (ainda em protótipo e teste; ele estima cerca de 45 yuans pelas 5 peças com frete); imprimir em vez de usinar também funciona, e a resistência também é melhor que a do original. Acompanha um 3MF para impressão em um clique (Bambu H2D, 0,4 mm, 5 placas) e os desenhos SolidWorks revisados |
| [Microduck-kissqy](https://github.com/kissqy/Microduck-kissqy) | [@kissqy](https://github.com/kissqy) | **Referência de fluxo de treinamento e de implementação de controle para o Feetech HD1910.** Oferece console de treinamento, instruções de preparação do ambiente e gerenciamento de tarefas, exportação de modelo ONNX, além de console do robô, firmware e vídeos de demonstração; recomendado como referência para o fluxo de treinamento concreto na parte de algoritmos |

Fez algo relacionado ao pato (ferramenta, tutorial, modificação, política...)? Abra uma issue com o link e nós adicionamos.

Quer participar: abra diretamente uma [issue](https://github.com/fanhao375/microduck-replica/issues) ou envie um PR.
Hardware, firmware, algoritmos, documentação e dados medidos, tudo conta — **dados medidos são especialmente bem-vindos**; a regra deste repositório é registrar as conclusões com honestidade, sejam positivas ou negativas.

---

## Navegação da documentação

Os caminhos são relativos à raiz deste repositório; o CAD mecânico é mantido em um repositório independente.

| Item | Local no repositório | Descrição e links |
|---|---|---|
| Uso e montagem | [Repositório CAD](https://github.com/fanhao375/microduck-replica-cad) · [BOM/](BOM/README.md) | [Guia de primeiro uso](docs/上手指南/README.md) · [Lista de compras](BOM/README.md) · [Vídeos de montagem](#-vídeos-de-montagem) |
| Software e imagem | [software/](software/README.md) | [Console Studio](software/studio/README.md) · [Construção da imagem](software/image/README.md) |
| Plataforma e projeto de treinamento | [software/training-studio/](software/training-studio/README.md)<br>[software/training/](software/training/README.md) | Bancada web de treinamento, ambiente de simulação e exportação de modelos |
| Elétrica e firmware | [hardware/](hardware/README.md) | [Projetos das placas](hardware/README.md) · [Firmware do IMU](hardware/imu_to_dxl/README.md) |
| Ferramentas de depuração e análise | [tools/](tools/README.md) | [Índice de ferramentas](tools/README.md) · Gravação do cartão, depuração serial, materiais da Feetech e scripts de análise |
| Documentos e registros | [docs/](docs/README.md) | [Navegação completa da documentação](docs/README.md) · [Armadilhas](踩坑记录.md) · [Depuração](调试记录.md) · [Construção](构建日志.md) |
| Histórico e comunidade | [docs/](docs/README.md#pesquisa-aprofundada-e-arquivo) | [Acompanhamento de atualizações oficiais](docs/官方更新跟踪.md) · [Histórico de atualizações](docs/历史更新.md) · [Contribuições da comunidade](docs/社区贡献.md) |

---

## Licença e fontes

Código, materiais de simulação e modelos incluídos na imagem estão sujeitos a licenças próprias; veja [licenças e atribuição](NOTICE.md) e os [termos de uso dos modelos](software/image/overlay/usr/share/doc/microduck/模型说明.txt). **A imagem da réplica é destinada apenas a réplicas pessoais não comerciais e não pode ser distribuída junto com produtos pagos.**

Este projeto não tem vínculo com a Pollen Robotics nem conta com o seu endosso.

### ❤️ Apoie o autor

Da montagem e depuração ao software e à documentação, este projeto foi construído com muitas tentativas. Se estes materiais e ferramentas ajudaram você, fique à vontade para dar uma Star ⭐, relatar problemas ou compartilhar sua experiência de montagem; também é possível fazer uma contribuição voluntária, pagando um café ao autor para apoiar a manutenção do projeto. Obrigado a todos os que apoiam o pato.

<p align="center"><a href="assets/alipay-support.jpg"><img src="assets/alipay-support.jpg" alt="QR code do Alipay para contribuição; clique para ver o original" width="240"></a><br><sub>Apoio opcional por QR code do Alipay · Clique na imagem para ver o original</sub></p>
