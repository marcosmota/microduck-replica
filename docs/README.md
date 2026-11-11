# Navegação da documentação

> 🌐 Tradução não oficial para pt-BR do [original](https://github.com/fanhao375/microduck-replica). Em caso de divergência, vale o original. Licença: CC BY-NC-SA 4.0 (texto alterado: traduzido do chinês/inglês).

[Voltar à página inicial](../README.md) · [English](README.en.md)

Encontre o material de acordo com o que você quer fazer. Para a primeira montagem, comece em "Uso e montagem"; as derivações de pesquisa e o progresso antigo ficam no arquivo mais abaixo.

## Uso e montagem

| Material | Conteúdo |
|---|---|
| [Guia de primeiro uso](上手指南/README.md) | Gravação (flash), configuração de rede, calibração, controle (gamepad) e guia em PDF |
| [CAD e peças impressas](https://github.com/fanhao375/microduck-replica-cad) | Modelos e projetos de impressão para as versões Feetech HD-1910 / XL330 |
| [Lista de compras (BOM)](../BOM/README.md) | Versão Feetech de 2026-10-23, com pré-visualização online e download em Excel |
| [Vídeo-tutorial de montagem](../README.md#-vídeos-de-montagem) | Vídeos do Bilibili divididos por componente |

## Software e treinamento

| Material | Conteúdo |
|---|---|
| [Console web Studio](../software/studio/README.md) | Calibração, controle remoto, estado do dispositivo e gerenciamento de modelos |
| [Construção e publicação da imagem](../software/image/README.md) | Como construir a imagem do sistema e notas de lançamento |
| [Atualização de software e voz local](软件更新与语音使用说明-20261027.md) | Bootstrap único para cartões antigos, atualização/reversão com assinatura pela web, R3 para sentar e levantar |
| [Plataforma de treinamento](../software/training-studio/README.md) | Gerenciamento de experimentos pela web, avaliação e exportação de modelos |
| [Projeto de treinamento](../software/training/README.md) | Ambiente de treinamento e como executar |
| [Checklist de dados pré-treinamento](HD-1910训练前数据清单.md) | Geometria, servos, IMU e itens medidos |
| [Origem do software e nossas alterações](软件来源与我们的改动.md) | Origem no upstream e modificações deste projeto |
| [Acompanhamento de atualizações oficiais](官方更新跟踪.md) | Versão estável, branch de desenvolvimento, PRs pendentes de merge, câmera, ações de emoção e outras atualizações, com sugestões de adaptação para este projeto |

## Ferramentas

[Navegação de ferramentas](../tools/README.md): encontre downloads e instruções de uso por categoria: gravação de cartão, depuração serial, materiais e análise.

- [Rufus portátil e instruções de gravação do cartão](../tools/rufus/README.md) · [Feetech FD, driver e assistente serial](../tools/飞特/README.md) · [Manuais oficiais e SDK da Feetech](飞特资料/README.md)
- [Bancada web de depuração de servos](../tools/servo-web/README.md) · [Ferramenta de teste HD1910M](../tools/FeeTech_HD1910M_Servo/README.md) · [Ferramenta de teste STS3215](../tools/sts3215Servo_testtool/README.md)
- [Verificação do protocolo do IMU](../tools/imu200/imu200.py) · [Análise de marcha](../tools/gait-analysis/README.md) · [Scripts de análise da placa principal](../tools/radxa/onboard-analysis/README.md)

## Parte elétrica e resolução de problemas

| Material | Conteúdo |
|---|---|
| [Projeto da placa de circuito](../hardware/README.md) · [Firmware do IMU](../hardware/imu_to_dxl/README.md) | Arquivos do projeto, ligações e escopo de verificação do firmware |
| [Introdução ao hardware](硬件入门.md) · [Especificações resumidas](硬件规格速查.md) | Função e parâmetros de cada módulo |
| [Registro de armadilhas](../踩坑记录.md) | Sintomas, causas e soluções |
| [Registro de depuração](../调试记录.md) · [Diário de construção](../构建日志.md) | Depuração no robô real e processo de montagem mecânica |

## Pesquisa aprofundada e arquivo

As análises iniciais e os registros de compra mantêm a data, a solução e o escopo de verificação originais; ao montar, consulte primeiro o guia atual e a BOM.

| Material | Conteúdo |
|---|---|
| [Histórico de atualizações](历史更新.md) | Atualizações dia a dia removidas da página inicial |
| [Contribuições da comunidade e projetos dos patos](社区贡献.md) | Créditos dos contribuidores, registros de merge e projetos da comunidade |
| [Arquivo de pesquisa do projeto](项目研究归档.md) | Vista explodida, estrutura de montagem, parâmetros das juntas, viabilidade inicial e comandos de reprodução |
| [Engenharia reversa da solução de hardware](硬件方案逆向.md) | Derivação do sistema eletrônico de controle e origem das evidências |
| [Seleção de atuadores](执行器选型.md) · [Reconstrução dos fixadores](紧固件反推.md) | Comparação de servos e análise dos furos |
| [Análise de compras de eletrônica](电控采购清单.md) · [Análise de compras de mecânica](机械采购清单.md) | Registros iniciais de seleção, compra e custo |
| [Solução sem HAT](不打HAT.md) | Solução alternativa e instruções de ligação |
| [Navegação do ecossistema](生态导航.md) · [Novidades da comunidade](社区动态.md) | Recursos do upstream, modelos e informações da comunidade |
| [Licença e créditos](../NOTICE.md) · [Termos de uso do modelo](../software/image/overlay/usr/share/doc/microduck/模型说明.txt) | Origem e escopo de uso de cada parte |
