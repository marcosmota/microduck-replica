# Andamento do projeto

> 🌐 Tradução não oficial para pt-BR do [original](https://github.com/fanhao375/microduck-replica). Em caso de divergência, vale o original. Licença: CC BY-NC-SA 4.0 (texto alterado: traduzido do chinês/inglês).

**Projeto**: Réplica do Microduck
**Início**: 2026-08-28
**Baseline do andamento histórico**: 2026-09-03 (errata sobre o estado de abertura do hardware: a placa HAT foi aberta pelo fabricante)
**Repositório**: https://github.com/fanhao375/microduck-replica (público)

> **Atualização do ponto de entrada dos modelos (2026-09-28)**: peças estruturais, CAD e projetos de impressão passaram a ser baixados de forma unificada em [microduck-replica-cad](https://github.com/fanhao375/microduck-replica-cad). Os antigos STL de `print/` e `cad/` deste repositório principal foram removidos, e os diretórios mantêm apenas notas de entrada. Os lotes de análise e as quantidades abaixo são registro histórico; o andamento mais recente do robô real está no [diário de construção](构建日志.md), e a conferência de versões, no [registro do CAD](docs/CAD结构件版本核对-20260928.md).

---

## Objetivo

Replicar o **Microduck** da Pollen Robotics (pato robô bípede de 25 cm, $399, vendas a partir do Natal de 2026).
O software oficial é aberto; o hardware é **parcialmente aberto** — **a placa HAT tem projeto KiCad completo e arquivos de produção**
([`elec_RPI_Robot_HAT`](https://github.com/pollen-robotics/elec_RPI_Robot_HAT), Apache-2.0),
mas a placa `imu_to_dxl`, o CAD mecânico editável, a BOM do robô completo e a documentação de montagem não foram publicados.

> **Errata de 2026-09-03**: o registro anterior dizia «hardware não aberto, sem esquemático de PCB», o que era um **erro de avaliação**.
> A causa foi pesquisar só o repositório principal `microduck`, deixando passar os repositórios de hardware com prefixo `elec_` da mesma organização.

Este projeto reconstitui, por engenharia reversa, todas as informações necessárias para a réplica mecânica a partir do **modelo de simulação MJCF + 47 STL** publicados oficialmente.

---

## Estado atual (2026-09-29)

| Etapa | Confirmado | O que falta |
|---|---|---|
| Mecânica Feetech | Robô completo montado, demonstração de levantar/sentar concluída em 09-18; modelos unificados no repositório CAD | Nesta rodada foram testadas 14 juntas; a boca, ID 29, foi excluída a pedido do usuário e não foi validada; ponto zero de montagem e faixa de movimento ainda precisam ser conferidos junta a junta |
| Armbian e HAT | A imagem pessoal com a correção de DHCP já conectou à rede; a HAT lê as juntas por `/dev/ttyS2`; o console de depuração 0.15.4 foi implantado | Validação de longa duração, com comunicação durante o movimento, e da configuração de proteção |
| Meta de leitura de retorno a 50 Hz | 3000 rodadas, 42000 estados de 14 servos, zero erros nesta janela, taxa real de 46,8664 Hz; postura e configurações originais mantidas | Não inclui IMU, inferência nem escrita de ações; não equivale ao laço de controle de 50 Hz do robô completo; em outras janelas ainda há falta ocasional de resposta |
| Câmera | Com o flat correto, o B1 oficial capturou 40 quadros, 1920×1080 | Reteste com a configuração Armbian deste projeto e taxa de quadros sustentada |
| IMU e Rust | A interface Feetech do firmware 0.2.0 tem registros de teste de software/simulação; o fork Rust `feetech` já tem implementação adaptada | Barramento misto real, temporização de respostas e validação do robô completo na placa não foram concluídos |
| Treinamento | O projeto HD1910, os parâmetros M6 e verificações de treinamento curto foram integrados em `software/training` | Calibração local, conferência do modelo e marcha no robô real |
| Imagem pública | A instrução do primeiro login antigo mudou para `root / 1234`, e o duck só é criado depois do assistente; foi adicionada uma candidata com correção de DHCP | A imagem candidata só passou por verificação offline, ainda sem gravação em cartão do novo artefato para teste real; a imagem base não tem o runtime do robô instalado |

Base: [resumo do teste de 3000 rodadas](tools/radxa/50Hz保持姿势读状态-20260928.md), [guia de uso da imagem](tools/radxa/镜像使用说明.md), [registro da câmera](tools/radxa/摄像头调试记录-20260928.md). O próximo passo é validar junta a junta, em pequena faixa e com o respaldo do robô real, e depois concluir a integração de proteção, IMU e runtime. Abaixo mantém-se o processo de análise inicial; os itens concluídos e as pendências ali não substituem o estado atual de validação no robô real.

## Baseline da análise histórica (2026-09-03)

| Etapa | Estado | Descrição |
|---|---|---|
| Geometria das peças | ✅ Concluído | 47 STL |
| Relações de montagem | ✅ Concluído | Precisão de 0,1 mm, vistas explodidas geradas |
| Conjunto CAD | ✅ Concluído | Transformação de mundo aplicada, pode ser importado diretamente |
| Parâmetros das juntas | ✅ Concluído | Eixos e cursos das 14 juntas controladas |
| Massa e inércia | ✅ Concluído | 15 corpos rígidos |
| Lista de fixadores | ✅ Concluído | Reconstruída a partir das feições de furos, sistema M2 |
| Seleção de atuadores | ✅ Concluído | XL330 ×15, parâmetros BAM M6 |
| Especificação dos rolamentos | ✅ Concluído | Ø22×16×4, Ø15×10×3 |
| **Solução de eletrônica** | ✅ Concluído | Totalmente reconstituída a partir do código Rust, veja o «Lote 6» |
| Seleção da placa principal | ✅ Concluído | **Módulo comercial Radxa Zero 3W**, não é placa base customizada |
| **Placa HAT** | ✅ Aberta pelo fabricante | KiCad + Gerber + BOM + coordenadas de pick-and-place, pronta para fabricar (placa de 4 camadas) |
| **Placa `imu_to_dxl`** | ✅ Desenhada por conta própria (não fabricada) | Sem projeto público em lugar nenhum; chip/endereço/protocolo totalmente reconstituídos, este repositório fornece esquemático + PCB |
| **Solução de roteamento de cabos** | ❌ Ausente | O fabricante não tem nenhum material |
| **Software de controle** | ✅ Utilizável | Com a mesma placa principal, o runtime Rust oficial roda direto; só trocar a placa principal exige portar |
| **Retreinamento da política** | ✅ Não necessário | Com hardware igual, os 9 ONNX oficiais funcionam direto; só mudar o corpo/eletrônica exige retreinar |

---

## Concluído

### Lote 1 · Levantamento de materiais

- ~~Confirmar que o **hardware do Microduck não é aberto**~~: o repositório `microduck` é todo de software Rust,
  os 5 subdiretórios de `docs/` são todos de documentação de software, e não se encontra nenhum arquivo `.stl/.step/.f3d/bom` em todo o repositório
  — **esta conclusão estava errada** (corrigida em 2026-09-03). Só se consultou o repositório principal, sem verificar os repositórios de hardware
  com prefixo `elec_` da mesma organização; **a placa HAT está totalmente aberta em [`elec_RPI_Robot_HAT`](https://github.com/pollen-robotics/elec_RPI_Robot_HAT)**
  (Apache-2.0, com KiCad + Gerber + BOM + coordenadas de pick-and-place).
  Lição: para julgar «se um projeto abriu certa parte», é preciso pesquisar **a organização inteira**, não só o repositório principal.
- Confirmar que o **`microduck_rl` contém 47 STL + MJCF completo**, a única fonte de geometria
- Confirmar que a **licença dos modelos 3D é CC BY-NC-SA** (não comercial) e a do código é Apache-2.0
- Descartada a rota do Open Duck Mini v2 (veja «Registro de decisões»)

### Lote 2 · Desenhos de montagem

- Renderização offscreen com MuJoCo, skybox branco, 1600×2100
- 4 vistas normais + 2 vistas explodidas + 1 imagem de comparação por cores
- A vista explodida desloca nível a nível ao longo da cadeia cinemática (48 mm por nível); quanto mais perto do fim da cadeia, mais longe se afasta
- Implementada por conta própria a projeção de coordenadas de mundo para pixels, usada para desenhar linhas de anotação e evitar sobreposições no layout
- Produto: `assembly-drawings/` (7 imagens), script `scripts/render_assembly.py`

### Lote 3 · Conjunto CAD

- Os 47 STL do upstream estão todos no **sistema de coordenadas próprio de cada peça**, e importados diretamente no CAD ficariam todos empilhados na origem
- Obtida a transformação de mundo de cada geom a partir do MJCF, aplicada e exportada agrupando por corpo rígido
- Produto: `cad/` — arquivo único do robô completo (796792 triângulos) + 15 componentes, unidade mm
- Dimensões externas medidas do robô completo: **144 × 141 × 264 mm** (coerente com os 25 cm declarados oficialmente)
- Script `scripts/export_assembly_stl.py`

### Lote 4 · Reconstrução dos fixadores

- Escrito o reconhecimento de feições de furos: soldar vértices → construir o grafo de adjacência de faces → cortar superfícies suaves por ângulo diedro de 35°
  → ajustar um cilindro a cada trecho (eixo = autovetor de menor autovalor da covariância das normais) → projetar o círculo ajustado para obter o diâmetro
  → distinguir furo de ressalto pela orientação da normal
- Varredura das 47 peças em 2,5 segundos
- Produto: `docs/紧固件反推.md`, `docs/hole_analysis.json`, `scripts/analyze_holes.py`

### Lote 10 · Documentação em inglês

- Contexto: ao ser incluído em listas awesome, foi marcado deliberadamente como "In Chinese"; os dados de tráfego mostram apenas 5 visitas vindas do Google
  contra 132 do Bing, e o público de língua inglesa praticamente não chegou
- Prioridade definida pelo número de acessos, traduzindo primeiro `硬件方案逆向.md` (2º mais acessado do site, 84 acessos),
  por ser **conteúdo exclusivo na internet** — a página de dissecação 3D feita pelo funcionário da HF @mishig25 (36 mil visualizações)
  já cobre a camada de «como é o robô completo», os documentos de especificação se sobrepõem a ela, e endereços de chips, protocolo de barramento
  e o layout do bloco de dados de 12 bytes só existem neste repositório
- Produto: `docs/hardware-teardown.en.md` (426 linhas) + `docs/actuator-selection.en.md` (459 linhas),
  ambos com links cruzados para a versão em chinês
- O índice de documentação do README em inglês passou a usar 🇬🇧 para marcar quais têm versão em inglês

### Lote 9 · Análise da seleção de atuadores

- Responde a duas perguntas frequentes: por que usar servos e não motores de passo em malha fechada; se é possível trocar pelo barato STS3215 sem alterar as peças mecânicas
- **Comparação com dados concretos**: para o mesmo pato bípede de 15 corpos rígidos, a versão XL330 pesa **737,2 g** contra **2107,1 g** da versão STS3215,
  uma diferença de 2,86 vezes — o Open Duck Mini v2 é justamente «a resposta da versão STS3215» (42 cm / 2,1 kg)
- **Comparação de parâmetros do atuador**: kp difere 32×, forcerange 3,5×, damping e frictionloss 11× cada,
  armature 15,5× — cada item difere em uma ordem de grandeza
- Descoberta contraintuitiva: o backlash (folga) modelado para o XL330 é, ao contrário, maior (±1,0° contra ±0,5°)
- Conclusão: **não existe caminho intermediário** — manter o XL330 (servos ¥4500) ou trocar para o STS3215
  (o que equivale a fazer o Open Duck Mini v2)
- **Acrescentada comparação horizontal com servos do mesmo nível**: o Feetech STS3032 (20 g / 23,2×12,1×28,5 mm / 0,44 N·m /
  4,8–6 V) é ainda menor que o XL330 e de peso equivalente, mas com cerca de 85% do torque (na mesma base de tensão) e limite de tensão de 6 V;
  o SCS0009 pesa só 11 g, mas tem apenas 0,23 N·m (a própria Pollen o usa no Amazing Hand)
- Conclusão: a lacuna foi definida claramente como «18–20 g / ≥0,9 N·m / 7,4 V / barramento»; por ora só a Robotis atende
- **Avaliação aprofundada do Unitree S288** (leitura completa do manual oficial de 11 páginas): 19,5 g / 34×20×23 mm /
  **relação de redução 288,35:1, idêntica à do XL330** / fixação M2 / half-duplex em fio único — mecanicamente quase um substituto direto
- Duas melhorias substanciais do S288: **controle de impedância** (τ = τ_ff + k_p·Δp + k_d·Δω) e
  **encoder no lado da saída** (OutPos de 13 bits) — este último elimina de vez toda a complexidade da modelagem de backlash
- Três restrições rígidas do S288: tensão de 12,6 V (3S) contra os 7,4 V (2S) do Microduck;
  o espaço de endereços do barramento é de apenas 0~14, sem lugar para um 16º dispositivo (o IMU teria de ir por SPI);
  **a indicação de torque é duvidosa** (nenhuma das duas leituras, lado do rotor ou lado da saída, faz sentido), e foram listadas quatro perguntas a esclarecer com o fabricante
- Produto: reescrita de `docs/执行器选型.md` (106 → 424 linhas)

### Lote 8 · Inteligência da comunidade no X e validação externa ⭐

- Posts públicos do X lidos com os comandos twitter read (search / thread) do opencli
- **Principal ganho: as conclusões da engenharia reversa receberam validação externa independente** — a desmontagem de hardware de @tspy (169 curtidas)
  listou de forma independente Radxa ZERO 3W / RK3566, totalmente consistente com este repositório
- Registrados os dados de popularidade: os três tweets oficiais somam cerca de 8,6 milhões de visualizações; o GitHub cresceu 3,7 vezes em três dias
- Registradas as derivações já feitas pela comunidade (síntese de voz do R2D2, rastreamento de ponteiro laser, cambalhotas, breakdance)
- **Identificado e marcado o ruído**: muitas moedas meme pegando carona na popularidade; no «Microdino», verificou-se que a área de respostas
  está cheia de acusações de golpe e as perguntas sobre CAD são respondidas de forma evasiva; **considerado não confiável e não citado**
- Esclarecida uma premissa: **até 8/31 não havia unidade física no mercado**, todos os teardowns públicos são inferência, não desmontagem física
- Produto: `docs/社区动态.md` (150 linhas)

### Lote 7 · Validação cruzada com as especificações oficiais + novidades da comunidade

- Lançamento oficial em 2026-08-27, com divulgação de parte das especificações; **conferidas item a item** com os resultados da engenharia reversa deste repositório
- Confirmação mútua: RK3566 / NP-F550 / ToF 8×8 / largura de 140 mm, tudo bate
- O oficial acrescenta: RAM de 1 GB + eMMC de 32 GB (**exatamente o SKU da Radxa Zero 3W, o que corrobora por via inversa a avaliação da placa principal**; sem comprovação no código-fonte, a réplica sugeria 2G/16G),
  NPU de 0,8 TOPS, antena NFC dupla
- Exclusivo da engenharia reversa: modelo da placa principal, solução do barramento principal, todos os endereços de chips e protocolos
- **Encontrada uma contradição**: o oficial declara 2 IMUs, mas no código-fonte só 1 está em uso (BMI088 marcado como dormant)
- Registrada a posição oficial sobre código aberto: a Pollen pede expressamente que a imprensa **não** o chame de "hardware aberto" (mas acrescentou "for now")
- Registradas três issues da comunidade sem resposta (#175 STEP / #173 arquivos-fonte de impressão / #174 esquemático da placa de alimentação)

### Lote 6 · Engenharia reversa da solução de hardware ⭐

- Raciocínio: **o runtime que aciona o hardware real precisa fixar em código caminhos de dispositivo, endereços I2C, offsets de registradores,
  baud rate e protocolo — o código é a folha de especificações**
- Extração item a item da solução completa de eletrônica a partir de `duck-control/src/{model,imu,bus}.rs`, `deploy/*.dts`, `deploy/robotd.toml`,
  `tof/`, `mediad/`
- Produto: `docs/硬件方案逆向.md` (394 linhas, dedução completa) + `docs/硬件规格速查.md` (270 linhas, consulta rápida de uma página)

### Lote 5 · Seleção de atuadores

- Organização da configuração BAM M6, das faixas de randomização de três domínios e da modelagem do encoder de backlash
- Extração dos 5 grupos de parâmetros PD nativos calibrados por medição BAM em `joints_properties.xml`
- Produto: `docs/执行器选型.md`

---

## Principais descobertas

1. **O robô completo é um sistema de parafusos M2.**
   Furos passantes Ø2,2 ×77, furos escareados Ø4,4 ×28, furos-guia para rosca Ø1,6 ×20.
   Ø2,2 e Ø4,4 aparecendo em par = a combinação de furo passante + escareado de um parafuso M2 de cabeça cilíndrica com sextavado interno.
   Verificação cruzada: o próprio `xl330.stl` tem Ø2,0×4 + Ø1,6×8, coerente com os furos de fixação M2 do XL330.
   Furos passantes nas peças estruturais: cerca de **146**.

2. **São 15 servos, não 14.**
   A malha `xl330` é referenciada **15 vezes** no MJCF do robô completo. 14 entram no espaço de ações da política
   (perna esquerda 5 + pescoço-cabeça 4 + perna direita 5), e o 15º aciona o bico/mandíbula, via ligações `passive_*`.
   Isto coincide com os "15 servos" do README oficial — contar só a lista de juntas deixa um de fora.

3. **A cabeça responde por um quarto do peso do robô completo.**
   Tronco 199 g, conjunto da cabeça 189 g, robô completo 737 g. O centro de gravidade é muito alto,
   o que explica por que a política de marcha é difícil de treinar.

4. **A modelagem de backlash é o detalhe-chave do sim2real.**
   O encoder magnético do servo real fica no **lado da saída** da folga das engrenagens, e a malha de posição do firmware fecha sobre
   `ângulo da junta principal + ângulo de backlash`; quando o servo gira em vazio na zona morta, a posição medida não muda e o erro do PD também não.

5. **Fazer por conta própria sai mais caro do que comprar.**
   15 XL330, conforme o canal, vão de $359 (site internacional da ROBOTIS) a €629 (MyBotShop),
   ou seja, de um pouco abaixo a muito acima do preço de venda de $399 do robô completo. A estimativa de custo está em BOM.md.

6. **⭐ A placa principal é um módulo comercial, não uma placa base customizada — a avaliação anterior estava errada.**
   (Complemento de 2026-08-31: esta conclusão recebeu validação independente da desmontagem de @tspy no X)
   A device tree fixa em código `compatible = "radxa,zero-3w", "rockchip,rk3566"`,
   que é a **Radxa Zero 3W** à venda no mercado (formato Pi Zero, 65×30 mm, coerente com o STL).
   Isto derruba diretamente a conclusão de que «a eletrônica é um muro intransponível».

7. **O IMU fica no barramento Dynamixel, não em I2C.**
   Placa própria `imu_to_dxl` v2: LSM6DSV16X, ID de barramento 200, registrador 124, bloco de 12 bytes
   (giroscópio 6 bytes + quaternion SFLP em fp16 6 bytes). É lido de volta na **mesma sync_read** que os 15 servos,
   sem nenhum custo extra de barramento, e o host não precisa executar fusão de atitude.

8. **Não há medidor de carga nem ADC.**
   A tensão da bateria é lida diretamente da tensão de alimentação do próprio servo, reportada via Dynamixel.
   Por isso 6,6–8,2 V é a «faixa utilizável sob carga» e não a faixa de tensão da célula. Para a réplica, **não é preciso projetar nenhum circuito de monitoramento de bateria**.

---

## Registro de decisões

| Decisão | Conclusão | Motivo |
|---|---|---|
| Fazer o Microduck ou o Open Duck Mini v2 | **Fazer o Microduck** | O usuário quer expressamente o próprio Microduck. O Open Duck Mini v2, embora totalmente aberto (BOM/CAD/STL/guia de montagem completos), é outro robô |
| Repositório público ou privado | **Público** | Escolha do usuário. A CC BY-NC-SA permite repassar, e a atribuição, o compartilhamento pela mesma licença e a declaração de uso não comercial já estão configurados |
| Empacotar juntos os 6 repositórios do upstream | **Não empacotar** | Os scripts de análise usam `scripts/fetch_upstream.sh` para obter o upstream. Em 2026-09-04 chegou a ser fornecida uma cópia dos STL em `print/`; em 2026-09-28, para evitar impressão por engano, foi removida, e os downloads dos modelos passaram a apontar de forma unificada para o repositório CAD. A origem e a licença do projeto de treinamento estão em `software/training/UPSTREAM.md`. |
| Licença | **Licença dupla** | `scripts/` original → Apache-2.0; `assembly-drawings/` e `cad/` são obras derivadas sob CC BY-SA-NC → pelo ShareAlike, mantêm a mesma licença |
| O que fazer diante do muro das PCBs | **Mecânica copiada + eletrônica própria** | Uma réplica 100% é impossível. O usuário disse que sabe fazer PCB, então este caminho é viável |

---

## Arquivo de pendências antigas (o que segue é estado histórico)

### Prioridade alta
- [x] **Desenhar a placa `imu_to_dxl`** — **esquemático e PCB concluídos** (2026-09-07).
      STM32G031F8P6 + LSM6DSV16X + buffer half-duplex SN74LVC2G241;
      placa de 2 camadas de 45 × 22 mm, camada inferior toda GND, 22 redes todas conectadas, DRC com 0 violações.
      **Não fabricada, sem validação em hardware real** → [projeto e resultados da revisão](hardware/imu_to_dxl/)
- [ ] **Validação da fabricação da `imu_to_dxl`** — antes do pedido, executar a padronização dos componentes e conferir a serigrafia;
      a altura dos conectores (inserção reta de 8 mm contra a cavidade do tronco) precisa ser confirmada com o STEP no conjunto montado
- [x] ~~**Engenharia reversa do contorno da PCB**~~ — **invalidado**: o projeto KiCad oficial e o Gerber já fornecem diretamente o contorno exato da placa,
      os furos de fixação e as aberturas dos conectores, sem necessidade de reconstituir a partir do STL
- [ ] **Decidir se a placa HAT será feita** — se não for preciso gravar áudio nem alto-falante, pode ser omitida por completo;
      o ToF liga direto no i2c3 e a alimentação usa um UBEC pronto

### Prioridade média
- [x] ~~Tradução para inglês~~ **Concluída**: engenharia reversa do hardware / seleção de atuadores / reconstrução dos fixadores, as três em inglês,
      mais o README, 4 documentos bilíngues no total. `社区动态.md` e `PROGRESS.md` não são traduzidos —
      o primeiro é muito sensível ao tempo e expira, o segundo é andamento interno
- [ ] `硬件规格速查.md` por ora não é traduzido — sobreposição alta com o Space de dissecação 3D de @mishig25
- [ ] **Verificação de imprimibilidade dos STL** — verificar se as 47 malhas são estanques e se têm arestas não-manifold,
      para julgar se podem ser fatiadas diretamente
- [ ] **Avaliação do processo de impressão** — as malhas de simulação não têm tolerância de encaixe; é preciso avaliar quais furos precisam de folga
      e quais faces precisam de suporte
- [ ] **Análise de canais de roteamento de cabos** — deduzir caminhos viáveis a partir das cavidades internas do conjunto montado

### Prioridade baixa
- [ ] **Solução do software de controle** — o runtime Rust está vinculado ao RK3566, avaliar o custo de portar
      contra reescrever o laço de controle em Python
- [ ] **Retreinamento da política** — ao trocar a eletrônica, os ONNX oficiais deixam de valer, é preciso retreinar com o `microduck_rl`
      (requer GPU CUDA, ou usar `--hf-jobs` para rodar na HuggingFace)

---

## Riscos e limitações conhecidas

1. **STL de simulação ≠ peça estrutural imprimível.** A simulação só garante forma externa e inércia, e não garante tolerâncias de encaixe,
   furos roscados, alojamentos de porcas de inserção térmica nem espaço para cabos. Imprimir diretamente muito provavelmente não vai montar.
2. **Os fixadores foram reconstruídos, não vêm de um desenho.** O comprimento dos parafusos foi calculado a partir da profundidade dos furos; é uma faixa, não um valor medido.
   A contração da impressão também altera o diâmetro real dos furos (em FDM, Ø2,2 costuma ficar 0,1–0,3 mm menor).
3. **A placa `imu_to_dxl` precisa ser desenhada por conta própria.** Não há projeto público em lugar nenhum; só dá para projetar a partir do protocolo reconstituído.
   (A placa HAT foi aberta pelo fabricante, com Gerber, **e não constitui bloqueio**.)
4. **A política oficial não é portável.** Ao trocar a eletrônica, os 9 ONNX deixam de valer e é obrigatório retreinar.
5. **Restrição de uso não comercial.** Todo o conteúdo geométrico derivado está sujeito à CC BY-SA-NC e não pode ser usado em produtos comerciais.

---

## Índice de arquivos

| Caminho | Conteúdo |
|---|---|
| `README.md` | Visão geral do projeto, estrutura de montagem, parâmetros das juntas, análise de viabilidade |
| `PROGRESS.md` | Este documento |
| `NOTICE.md` | Atribuição e fontes |
| `assembly-drawings/` | 7 desenhos de montagem |
| `cad/`、`print/` | Entrada de download do repositório CAD; os STL antigos e as tabelas de correspondência foram removidos, e os produtos da análise histórica podem ser rastreados pelo histórico do Git |
| `docs/紧固件反推.md` | Sistema de parafusos M2, quantidades de compra, especificação dos rolamentos |
| `docs/执行器选型.md` | Parâmetros do XL330, BAM M6, 5 grupos de PD calibrados, modelagem de backlash |
| `docs/硬件规格速查.md` | Tabela de especificações de hardware em uma página |
| `docs/硬件方案逆向.md` | Dedução completa e evidências da solução de eletrônica |
| `docs/社区动态.md` | Inteligência do X / GitHub, validação externa, ruído e riscos |
| `docs/hole_analysis.json` | Dados brutos da varredura de furos |
| `scripts/` | Buscar o upstream / renderizar / exportar CAD / varrer furos, 4 scripts |
