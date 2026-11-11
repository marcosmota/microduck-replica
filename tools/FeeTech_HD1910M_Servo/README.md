# Ferramenta de teste de servo HD1910M (feetech-HD1910M-tester)

> 🌐 Tradução não oficial para pt-BR do [original](https://github.com/fanhao375/microduck-replica). Em caso de divergência, vale o original. Licença: CC BY-NC-SA 4.0 (texto alterado: traduzido do chinês/inglês).

Ferramenta de teste, implementada em Rust, para o servo serial Feetech HD1910M (protocolo FT-SCS).
Cobre todos os registradores legíveis e graváveis do mapa de memória oficial: configuração de ID/baud rate, PID das três malhas (posição/velocidade/corrente), torque e controle de força, modo de operação, parâmetros de proteção, limites de ângulo e calibração do ponto zero (0x0B), trava da EPROM, monitoramento em tempo real, entre outros.

## Início rápido

```bash
cargo build
./target/debug/hd1910-tester scan      # 扫描舵机（带进度条）
./target/debug/hd1910-tester dump      # 读整张内存表
./target/debug/hd1910-tester monitor   # 实时监控反馈
./target/debug/hd1910-tester --help    # 全部命令
```

## Documentação

O manual didático completo em chinês (explicação do protocolo + conceitos de Rust + tutorial de experimentos) está em **[TUTORIAL.md](TUTORIAL.md)**.

## Duas especificações da Feetech incluídas

| Arquivo | O que é | Relação com o pato |
|---|---|---|
| [`HD-1910-C001串型规格书20260907.pdf`](HD-1910-C001串型规格书20260907.pdf) | **O servo usado neste projeto**. 4–8,4 V, redução de 1:320, torque de travamento de 15 kg·cm @7,4 V. Limiar de proteção contra sobretensão de 10 V e subtensão de 4 V, portanto os 8,4 V da bateria 2S carregada ficam dentro da faixa | ✅ Os 15 servos do pato completo são este modelo |
| [`HL-2915-C002串型规格书20260227.pdf`](HL-2915-C002串型规格书20260227.pdf) | Outro servo da Feetech. **9–14 V (bateria 3S)**, 22,5 g, redução de 1:320 | ⚠️ **Não é usado neste projeto, apenas para referência**. A tensão é incompatível com a bateria 2S do pato; serve para comparação caso se queira migrar para uma solução 3S |

Ambas são as especificações originais emitidas pela Feetech, com o conteúdo técnico inalterado. **Nomes de contatos e números de celular/WeChat foram apagados**
(o original traz os contatos pessoais do vendedor, que não é adequado deixar em um repositório público); a especificação do HD-1910 usa a versão oficial, cujo contato é o telefone fixo da empresa.
Para falar com a Feetech, use o site oficial [www.feetech.cn](https://www.feetech.cn) ou o telefone da empresa 0755-89335266.

Os materiais públicos do site oficial da Feetech, como manual do protocolo e mapas de memória, estão em [`docs/飞特资料/`](../../docs/飞特资料/) neste repositório.

## Testes

```bash
cargo test    # 协议层/内存表层单元测试（无需硬件）
```
