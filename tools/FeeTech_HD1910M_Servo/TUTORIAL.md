# Ferramenta de teste de servo HD1910M —— Manual didático prático de Rust

> 🌐 Tradução não oficial para pt-BR do [original](https://github.com/fanhao375/microduck-replica). Em caso de divergência, vale o original. Licença: CC BY-NC-SA 4.0 (texto alterado: traduzido do chinês/inglês).

> Este manual acompanha a leitura do código-fonte. Objetivo: por meio de um projeto que "realmente controla hardware",
> aprender os conceitos centrais de Rust e, ao mesmo tempo, dominar o protocolo serial de servos FT-SCS da Feetech.
>
> Hardware: servo HD1910M (6V 12kg·cm, encoder magnético de 360°, serial TTL half-duplex, com controle de força)
> Software: Rust 1.98+ / Raspberry Pi / adaptador USB-serial CH340

---

## Sumário

1. [Conexão de hardware](#1-conexão-de-hardware)
2. [Primeiros cinco minutos](#2-primeiros-cinco-minutos)
3. [O protocolo de comunicação FT-SCS em detalhe](#3-o-protocolo-de-comunicação-ft-scs-em-detalhe)
4. [A tabela de memória em detalhe](#4-a-tabela-de-memória-em-detalhe)
5. [Estrutura do projeto e design em camadas](#5-estrutura-do-projeto-e-design-em-camadas)
6. [Conceitos de Rust em detalhe](#6-conceitos-de-rust-em-detalhe)
7. [Referência completa dos comandos](#7-referência-completa-dos-comandos)
8. [Tutorial de experimentos (mão na massa)](#8-tutorial-de-experimentos-mão-na-massa)
9. [Precauções de segurança](#9-precauções-de-segurança)
10. [Solução de problemas comuns](#10-solução-de-problemas-comuns)

---

## 1. Conexão de hardware

Cabo de saída do servo (conector AMP2.0-3P):

| Pino | Cor do fio | Função | Liga a quê |
|------|------|------|----------|
| 1 | Preto | Signal (sinal serial TTL) | TX e RX do CH340 (half-duplex, veja abaixo) |
| 2 | Preto | Vcc (4~8,4V, típico 6V/7,4V) | **Positivo da fonte externa** (não dá para alimentar pelo USB!) |
| 3 | Preto | GND | Negativo da fonte externa, **deve ter terra comum com o GND do CH340** |

Pontos-chave:

- **Half-duplex**: o servo tem uma única linha de sinal, e tanto o envio quanto a recepção passam por ela. O CH340 é full-duplex (TX/RX separados);
  a ligação simples é curto-circuitar o TX e o RX do CH340 e ligá-los juntos à linha de sinal do servo —— o custo é que os dados que
  você mesmo envia voltam para você ("eco"), e então é preciso acrescentar o parâmetro `--echo` ao executar o programa para descartar o eco.
  O jeito mais correto é usar a placa de conversão half-duplex oficial da Feetech (URT-1 etc.), e então o `--echo` não é necessário.
- **Terra comum**: o GND do servo, o GND da fonte e o GND do CH340 devem estar ligados entre si, senão a comunicação não é confiável.
- **Alimentação**: a corrente de travamento pode chegar a 1,6A (a 6V), o USB não aguenta, use sempre uma fonte independente.
- No Raspberry Pi, o CH340 normalmente é reconhecido como `/dev/ttyUSB0` (o padrão desta ferramenta).

---

## 2. Primeiros cinco minutos

```bash
# 构建（第一次会下载依赖并编译，产物在 target/debug/）
cargo build

# 以后每次这样运行（cargo run 会先编译再执行，-- 后面是给我们程序的参数）
cargo run -- scan

# 也可以直接运行编译好的二进制
./target/debug/feetech-HD1910M-tester scan

# 1. 扫描总线上的舵机（带进度条）
./target/debug/feetech-HD1910M-tester scan

# 2. 读取整张内存表
./target/debug/feetech-HD1910M-tester dump

# 3. 实时监控（位置/速度/电压/温度/电流，Ctrl+C 停止）
./target/debug/feetech-HD1910M-tester monitor

# 4. 转到 90° 位置（1024 ≈ 90°，中位是 2048）
./target/debug/feetech-HD1910M-tester goto 1024

# 5. 查看所有命令
./target/debug/feetech-HD1910M-tester --help
./target/debug/feetech-HD1910M-tester pid --help   # 查看某个子命令的帮助
```

Parâmetros globais (podem ficar antes ou depois do subcomando):

```
-p / --port     串口路径，默认 /dev/ttyUSB0
-b / --baud     波特率，默认 1000000
-i / --id       舵机 ID，默认 1
-t / --timeout  超时毫秒数，默认 50（扫描时可调小到 20 加速）
-v / --verbose  打印收发的原始报文（学习协议神器！）
    --echo      TX/RX 短接接法时开启，丢弃回显
```

---

## 3. O protocolo de comunicação FT-SCS em detalhe

Código-fonte correspondente: `src/protocol.rs`

A configuração serial de fábrica do servo: **1Mbps, 8 bits de dados, sem paridade, 1 bit de parada** (seção 7-2 da especificação).
O protocolo é o clássico "mestre-escravo de pergunta e resposta": o host (Raspberry Pi) envia um pacote de instrução e o servo responde com um pacote de resposta.

### 3.1 Pacote enviado pelo host

```
 字节:    0     1     2      3         4         5..N+4     N+5
 内容:  0xFF  0xFF   ID   Length  Instruction   参数...   Checksum
```

- **Cabeçalho** `0xFF 0xFF`: dois bytes fixos, pelos quais o receptor alinha o início de um quadro;
- **ID**: número da estação do servo de destino, 0~253; `0xFE` é o endereço de broadcast (todos os servos executam, mas nenhum responde);
- **Length** = número de parâmetros N + 2 (1 byte da instrução + 1 byte do checksum);
- **Checksum** = `~(ID + Length + Instruction + todos os parâmetros)`, tomando os 8 bits baixos.
  O receptor recalcula; se não coincidir, houve erro de transmissão e o pacote inteiro é descartado.

### 3.2 Pacote de resposta do servo

O formato é quase idêntico ao do pacote enviado, só que o 4º byte passa de "instrução" para "**palavra de estado de erro**":

```
 0xFF  0xFF   ID   Length   Error   参数...   Checksum
```

Error igual a 0 significa normal; se diferente de 0, interpreta-se bit a bit (BIT0 tensão / BIT1 encoder magnético / BIT2 temperatura / BIT3 corrente).

### 3.3 Conjunto de instruções

| Instrução | Valor | Finalidade |
|------|-----|------|
| PING | 0x01 | Handshake, verificar se o servo está online |
| READ | 0x02 | Ler a tabela de memória, parâmetros = [endereço inicial, nº de bytes] |
| WRITE | 0x03 | Escrever na tabela de memória, parâmetros = [endereço inicial, dados...], efeito imediato |
| REG WRITE | 0x04 | Escrita assíncrona: guarda em buffer primeiro, sem executar |
| ACTION | 0x05 | Dispara todas as escritas assíncronas em buffer (a chave para ação sincronizada de vários servos) |
| RESET | 0x06 | Restaurar as configurações de fábrica |
| SYNC WRITE | 0x83 | Uma mensagem escreve o mesmo endereço em vários servos |

### 3.4 Um exemplo real

Com o parâmetro `-v` é possível ver as mensagens realmente trocadas, por exemplo `goto 911 --speed 500`:

```
>> 发送: [FF, FF, 01, 05, 03, 2E, F4, 01, D3]   # WRITE 地址46(速度) = 0x01F4 = 500
<< 接收: [FF, FF, 01, 02, 00, FC]               # 应答：Error=0 正常
>> 发送: [FF, FF, 01, 04, 03, 28, 01, CE]       # WRITE 地址40(扭矩开关) = 1
>> 发送: [FF, FF, 01, 05, 03, 2A, 8F, 03, 3A]   # WRITE 地址42(目标位置) = 0x038F = 911
```

Verifique o checksum à mão: `0x01+0x05 = 0x06，+0x03 = 0x09，+0x2E = 0x37，+0xF4 = 0x12B，+0x01 = 0x12C`;
mantendo só os 8 bits baixos obtém-se `0x2C`, e invertendo bit a bit `!0x2C = 0xD3` —— exatamente igual ao último byte da mensagem ✔

> **Exercício**: abra os testes unitários no final de `src/protocol.rs` (`cargo test`);
> neles há a asserção completa do pacote PING; tente verificá-la calculando à mão.

---

## 4. A tabela de memória em detalhe

Código-fonte correspondente: `src/registers.rs`

A tabela de memória é o "registro de parâmetros" interno do servo, com cada endereço guardando um parâmetro. **Dados de dois bytes têm o byte baixo primeiro (little-endian)**.

### 4.1 Cinco regiões

| Endereço | Região | Permissão | Descrição |
|------|------|------|------|
| 0~4 | Informações de versão | Somente leitura | Número de versão do firmware/servo, flag de ordem de bytes |
| 5~39 | Configuração EPROM | Leitura e escrita | ID, baud rate, PID, parâmetros de proteção…… **Guardados ao desligar** |
| 40~55 | Controle SRAM | Leitura e escrita | Chave de torque, posição/corrente/velocidade-alvo……Efeito em tempo real, perdidos ao desligar |
| 56~75 | Realimentação SRAM | Somente leitura | Posição/velocidade/carga/tensão/temperatura/corrente atuais |
| 77~86 | Parâmetros de fábrica | Somente leitura | Valores de calibração de fábrica, não mexer |

### 4.2 Flag de trava (endereço 55) —— o mecanismo de persistência da EPROM

Este é o mecanismo em que mais se tropeça:

- Trava = **1 (padrão de fábrica)**: escrever na região EPROM (endereços 5~39) só altera o valor na memória, que **volta ao original ao desligar**;
- Trava = **0**: o valor escrito na região EPROM é realmente gravado na EPROM, **guardado ao desligar**.

Por isso o fluxo padrão para "persistir uma configuração" é: **destravar → escrever → travar de novo**.
O `set <registrador> <valor> --save`, o `set-id` e o `set-baud` desta ferramenta seguem automaticamente esse fluxo
(veja a função `save_reg` em `src/servo.rs`).

> Por que não se pode deixar sempre destravado? A EPROM tem vida útil de escrita/apagamento (cerca de 100 mil vezes),
> e travar evita que um bug do programa cause escritas frenéticas que danifiquem o servo.

### 4.3 Codificação sinal-magnitude —— números negativos não são complemento de dois!

As grandezas com sinal no protocolo Feetech (posição, velocidade, corrente, carga, deslocamento de posição) **não** representam números negativos
com o complemento de dois binário usual dos computadores, e sim com "**bit de sinal + valor absoluto**":

- A maioria das grandezas: **o BIT15 é o bit de direção**, e os 15 bits baixos são o valor absoluto. Por exemplo, `-100` é codificado como `0x8000 | 100 = 0x8064`;
- A carga atual (endereço 60) é especial: **o BIT10 é o bit de direção**, e os 10 bits baixos são o valor absoluto.

Se decodificar `0x8064` como complemento de dois, obtém-se -32668 em vez de -100 —— completamente errado.
Esta ferramenta trata isso de forma unificada em `decode_value`/`encode_value` de `src/registers.rs`,
e o campo `sign_bit` de cada registrador declara qual bit ele usa como bit de sinal.

### 4.4 Modo de operação (endereço 33)

| Valor | Modo | Descrição |
|----|------|------|
| 0 | Modo servo de posição | Malha fechada de posição + limite de força por corrente-alvo (permite controle de força) |
| 1 | Modo de velocidade constante do motor | Malha fechada de velocidade, a velocidade não cai com o aumento da carga (para acionamento de rodas) |
| 2 | Modo de corrente constante do motor | Malha fechada de corrente, saída de torque constante |
| 3 | Regulagem de velocidade PWM em malha aberta | Tensão por razão cíclica, sem malha fechada, a rotação oscila com a carga |
| 4 | Modo PD de posição pura | Malha fechada de posição proporcional-derivativa (Sim2Real, **padrão de fábrica desta unidade**) |

### 4.5 Dois conjuntos de endereços dos parâmetros PID

Este é o ponto mais engenhoso do projeto da tabela de memória:

- **Região de configuração EPROM** (21/22/23 P/D/I da malha de posição, 37/39 P/I da malha de velocidade, 34/35 P/I da malha de corrente):
  guarda os "valores padrão de fábrica", copiados para a região de execução **na energização**;
- **Região de execução SRAM** (50 Kp / 51 Kd / 52 Ki): os valores que **realmente valem em operação**, e alterá-los surte efeito imediato, sendo perdidos ao desligar.

A forma correta de ajustar: primeiro experimente repetidamente na região de execução com `pid position --kp 40 --ram`,
e quando estiver satisfeito use `pid position --kp 40 --save` para gravar permanentemente na EPROM.

Atenção: o "significado muda conforme o modo" nos endereços 50/51/52 da região de execução: no modo servo de posição é o PID da malha de posição;
no modo de velocidade constante (modo 1) vira o PI da malha de velocidade (o Kd é inválido).

### 4.6 Registradores de proteção

- Endereço 13, limite máximo de temperatura (padrão 80°C, acima disso o torque desliga);
- Endereços 14/15, tensão de entrada máxima/mínima (unidade de 0,1V);
- Endereço 28, corrente de proteção (unidade de 6,5mA) + endereço 38, tempo de proteção contra sobrecorrente (unidade de 10ms):
  quando a corrente excede o limite por esse tempo, entra-se na proteção de sobrecorrente (reenviar um comando de posição a limpa);
- Endereço 19, condição de descarga / endereço 20, condição de alarme do LED: **máscaras de bits**,
  BIT0(1)=tensão BIT1(2)=encoder magnético BIT2(4)=superaquecimento BIT3(8)=sobrecorrente, 1 liga.
  Por exemplo, ligar as proteções de tensão e superaquecimento ao mesmo tempo é `1+4=5`.

---

## 5. Estrutura do projeto e design em camadas

```
FT-HD1901M_Servo/
├── Cargo.toml          # 项目清单：包名、版本、第三方依赖
├── src/
│   ├── main.rs         # 第 4 层：命令行定义与分发（clap）
│   ├── servo.rs        # 第 3 层：串口驱动（打开串口、收发、超时处理）
│   ├── protocol.rs     # 第 2 层：协议组包/解包（纯数据，不碰硬件）
│   ├── registers.rs    # 第 2 层：内存表定义（纯数据，不碰硬件）
│   └── error.rs        # 贯穿各层：统一错误类型
└── TUTORIAL.md         # 本手册
```

**O sentido da separação em camadas**: as duas camadas de baixo (protocol, registers) são pura conversão de dados, sem nenhuma operação serial,
e portanto podem receber testes unitários sem hardware (`cargo test`). Se todo o código fosse misturado,
para testar a lógica do protocolo seria preciso ligar um servo de verdade —— isto é a "separação de responsabilidades".

As dependências fluem em um só sentido: `main.rs → servo.rs → protocol.rs / registers.rs`,
e o `error.rs` é usado por todas as camadas. Jamais uma camada inferior referencia uma superior.

---

## 6. Conceitos de Rust em detalhe

Explicados na ordem em que você os encontra no código-fonte. Cada conceito indica a posição correspondente no código.

### 6.1 Cargo e o sistema de módulos —— `Cargo.toml`, início do `src/main.rs`

- O `Cargo.toml` declara as dependências, que são baixadas automaticamente do crates.io na compilação e travadas em versão no `Cargo.lock`;
- `mod error;` declara um módulo, e `use crate::error::Result;` traz para o escopo um nome de dentro do módulo;
- `crate::` significa "procurar a partir da raiz do projeto", semelhante a um caminho absoluto.

### 6.2 Tratamento de erros: Result e `?` —— `src/error.rs`

Rust não tem exceções. Funções que podem falhar retornam `Result<T, E>`:

```rust
// 传统写法（啰嗦）
let port = match serialport::new(path, baud).open() {
    Ok(p) => p,
    Err(e) => return Err(ServoError::Serial(e)),
};

// 用 ? 运算符（等价，一行）
let port = serialport::new(path, baud).open()?;
```

A semântica do `?`: **em caso de falha, converte o tipo de erro e retorna antecipadamente; em caso de sucesso, extrai o valor e continua**.
A anotação `#[from]` (fornecida pelo thiserror) faz o `serialport::Error` ser convertido automaticamente no nosso próprio
`ServoError`, então um único `?` basta.

**Por que definir um enum de erro próprio?** Assim as camadas superiores lidam com um único tipo de erro, e com `match` se distingue
"timeout" (pode ser só que esse ID não existe, normal durante uma varredura) de "erro de checksum" (há problema na linha).
Veja o `ping()` em `servo.rs`.

### 6.3 Enums: o "trunfo" do Rust —— `src/error.rs`, `src/protocol.rs`

```rust
pub enum ServoError {
    Timeout,                        // 不带数据
    Checksum,
    Malformed(&'static str),        // 带一个字符串
    ServoStatus(u8, String),        // 带两个数据
}
```

Cada variante pode carregar dados de tipos e quantidades diferentes. Combinado com a **verificação de exaustividade** do `match`
(esquecer de tratar uma variante faz a compilação falhar), a refatoração fica muito segura:

```rust
match self.transact(id, Instruction::Ping, &[], true) {
    Ok(_) => Ok(true),
    Err(ServoError::Timeout) => Ok(false),  // 超时 = 没有这个 ID，不算失败
    Err(e) => Err(e),                        // 其他错误继续上抛
}
```

### 6.4 Option: acabando com o ponteiro nulo —— `find_register` em `src/registers.rs`

"Uma busca que pode falhar" retorna em Rust `Option<&Register>` (`Some(x)` ou `None`),
e não null. O chamador deve tratar explicitamente o caso None, e o compilador garante isso à força:

```rust
let reg = find_register(name).ok_or_else(|| {
    ServoError::InvalidParam(format!("未知寄存器 \"{name}\""))
})?;  // 找不到就把 None 转成 Err 并返回
```

Os "parâmetros opcionais" da linha de comando (como `goto --speed`) também são `Option<i64>`:
se fornecido, é `Some(valor)`; se não, `None`, tratado com `if let Some(s) = speed { ... }`.

### 6.5 Propriedade e empréstimo: a garantia, em nível de linguagem, do acesso exclusivo à serial —— `src/servo.rs`

As regras centrais do Rust:

1. Cada valor tem um e somente um **dono** (owner), e quando o dono sai do escopo o valor é liberado automaticamente;
2. Pode haver qualquer número de **empréstimos imutáveis** (`&T`), ou **um** empréstimo mutável (`&mut T`), um ou outro;
3. Um empréstimo não pode durar mais que o tempo de vida do dono.

O handle da serial é **possuído** pela struct `Servo`; todos os métodos de operação exigem `&mut self`.
Por isso o compilador garante que, a cada instante, só um trecho de código pode operar a serial.
**Bugs comuns em C como "várias threads escrevendo na serial ao mesmo tempo e intercalando mensagens" simplesmente não compilam em Rust.**

### 6.6 Structs e impl —— `src/servo.rs`

```rust
pub struct Servo {          // 数据
    port: Box<dyn SerialPort>,
    discard_echo: bool,     // 私有字段（没有 pub），外部不能乱改
    verbose: bool,
}

impl Servo {                // 行为（方法）
    pub fn connect(...) -> Result<Self> { ... }  // 关联函数（类似静态方法/构造函数）
    pub fn ping(&mut self, id: u8) -> Result<bool> { ... }  // &mut self = 可变借用自己
}
```

Os três tipos de receptor, `&self` (empréstimo somente leitura) / `&mut self` (empréstimo mutável) / `self` (toma a propriedade),
colocam na assinatura da função "este método modifica ou não o objeto", visível de relance pelo chamador.

### 6.7 trait e objetos trait —— `Box<dyn SerialPort>`

Um trait é semelhante a uma interface de Java/Go. Na biblioteca `serialport`, a serial é de tipos diferentes em Linux/Windows/macOS,
mas todos implementam o trait `SerialPort`. `Box<dyn SerialPort>` significa
"na heap, um valor que implementa esse trait", e só o usamos pelos métodos definidos pelo trait.
Isso torna o código multiplataforma e independente de tipos concretos.

### 6.8 Iteradores e closures —— espalhados por toda parte

```rust
// protocol.rs：折叠求和（闭包 |acc, &b| ... 捕获环境）
let sum = bytes.iter().fold(0u8, |acc, &b| acc.wrapping_add(b));

// main.rs：map + collect 把 Vec<u8> 变成 Vec<String> 再 join
let list: Vec<String> = found.iter().map(|id| id.to_string()).collect();

// main.rs：逗号分隔字符串解析成 Vec<u8>
let ids: Result<Vec<u8>, _> = ids.split(',').map(|s| s.trim().parse::<u8>()).collect();
```

Iteradores são uma **abstração de custo zero**: depois de compilados equivalem a um laço for escrito à mão, sem custo em tempo de execução.

### 6.9 Macros derive e clap —— `src/main.rs`

```rust
#[derive(Parser)]                    // 让 clap 自动实现参数解析
struct Cli {
    /// 这一行文档注释会自动变成 --help 里的帮助文字！
    #[arg(short, long, default_value_t = 1)]
    id: u8,
}
```

`#[derive(...)]` é uma "macro derivada": em tempo de compilação a macro gera código automaticamente.
Neste projeto, `Debug` (implementa automaticamente a impressão de depuração), `Clone/Copy` (implementa automaticamente a cópia),
`Error` (o thiserror implementa automaticamente o trait de erro) e `Parser/Subcommand/ValueEnum` (clap) são todos assim.

### 6.10 Testes unitários —— `#[cfg(test)] mod tests` no final de cada arquivo

```rust
#[cfg(test)]           // 只在 cargo test 时编译，发布版本不含
mod tests {
    use super::*;      // super = 父模块
    #[test]
    fn test_build_ping_packet() {
        assert_eq!(build_packet(1, Instruction::Ping, &[]),
                   vec![0xFF, 0xFF, 0x01, 0x02, 0x01, 0xFB]);
    }
}
```

Basta executar `cargo test`. Os testes das camadas de lógica pura (protocolo, tabela de memória) não precisam de hardware.

### 6.11 Referência rápida de pequenos truques

| Escrita | Significado | Onde aparece |
|------|------|------|
| `wrapping_add` | Soma com retorno em caso de estouro (o checksum precisa de módulo 256) | protocol.rs |
| `u16::from_le_bytes([a, b])` | Junta dois bytes em little-endian em um u16 | servo.rs |
| `to_le_bytes()` | Divide um u16 em um array de bytes little-endian | servo.rs |
| `Vec::with_capacity(n)` | Pré-aloca capacidade, evitando expansões repetidas | protocol.rs |
| `format!` / `println!` com variável inline `{value}` | Interpolação de strings do Rust 2021 | vários |
| `.expect("...")` | Desembrulha quando se tem certeza de que não falha; se falhar, dá panic com mensagem | servo.rs |
| `matches!(x, Err(...))` | Verifica se o valor casa com um padrão | testes de protocol.rs |
| `&[u8]` vs `Vec<u8>` | Slice emprestado vs array que possui a propriedade | projeto dos parâmetros de função |

---

## 7. Referência completa dos comandos

> Todas as operações de escrita, por padrão, **só alteram a memória (voltam ao original ao desligar)**; para registradores da região EPROM, só com `--save` há persistência ao desligar
> (executa automaticamente destravar→escrever→travar). Esse é um projeto para proteger a vida útil da EPROM.

### Consultas

```bash
feetech-HD1910M-tester scan                          # 扫描 ID 0~253（带进度条）
feetech-HD1910M-tester scan --from 1 --to 10         # 只扫一部分
feetech-HD1910M-tester dump                          # 打印整张内存表（分 5 区，含单位与说明）
feetech-HD1910M-tester get pos-p                     # 读单个寄存器（支持英文名）
feetech-HD1910M-tester get 21                        # 也支持十进制地址
feetech-HD1910M-tester get 0x15                      # 和十六进制地址
feetech-HD1910M-tester get status                    # 状态字自动翻译成中文
feetech-HD1910M-tester monitor                       # 实时监控（Ctrl+C 停止）
feetech-HD1910M-tester monitor --interval 50 --count 100
```

### Escrita genérica (cobre todos os registradores graváveis da tabela de memória)

```bash
feetech-HD1910M-tester set <寄存器> <值>             # 写内存（立即生效，掉电恢复）
feetech-HD1910M-tester set <寄存器> <值> --save      # 写 EPROM 并掉电保存
feetech-HD1910M-tester set <寄存器> <值> --defer     # 异步写（缓存）
feetech-HD1910M-tester action                        # 广播触发所有缓存的异步写
feetech-HD1910M-tester swrite 1,2,3 goal-pos 2048    # 同步写：多个舵机同一寄存器
```

Exemplos: `set deadband-cw 3`, `set integral-limit 10 --save`, `set min-start-force 20`.

### ID / baud rate / trava / reset de fábrica

```bash
feetech-HD1910M-tester set-id 2              # 把 ID 从当前值改为 2（自动保存，改完用 -i 2 连接）
feetech-HD1910M-tester set-baud 4            # 波特率索引 0~7 → 此处改为 115200（保存，重启生效）
feetech-HD1910M-tester lock on               # 上锁：EPROM 写入掉电不保存（默认状态）
feetech-HD1910M-tester lock off              # 解锁：EPROM 写入掉电保存
feetech-HD1910M-tester factory-reset         # 恢复出厂设置（慎用！ID 回 1，波特率回 1M）
```

### Controle de movimento

```bash
feetech-HD1910M-tester mode 0                        # 位置伺服模式（0~4，含义见 4.4 节）
feetech-HD1910M-tester goto 2048                     # 转到中位（自动开扭矩）
feetech-HD1910M-tester goto 1024 --speed 500 --acc 50
feetech-HD1910M-tester torque on / off / damp        # 扭矩开关 / 松轴 / 阻尼（示教用）
feetech-HD1910M-tester speed 1000                    # 运行速度（恒速模式下 BIT15 决定方向）
```

### Torque / controle de força

```bash
feetech-HD1910M-tester torque-limit 500      # 转矩限制 50%（0~1000 = 0~100%）
feetech-HD1910M-tester goal-current 100      # 目标电流 ≈ 650mA（单位 6.5mA）
feetech-HD1910M-tester set max-torque 800 --save     # 最大扭矩 80%（EPROM，上电拷贝给48号）
feetech-HD1910M-tester set protect-current 300 --save # 保护电流 ≈ 1.95A（EPROM）
feetech-HD1910M-tester set min-start-force 20 --save  # 最小启动力 2%
```

### Ajuste de PID (três malhas)

```bash
feetech-HD1910M-tester pid position --kp 40 --kd 8            # 位置环，写 EPROM 配置（本次有效）
feetech-HD1910M-tester pid position --kp 40 --kd 8 --ram      # 写运行区，立即生效（调参推荐）
feetech-HD1910M-tester pid position --kp 40 --kd 8 --save     # 写 EPROM 并掉电保存（调好后）
feetech-HD1910M-tester pid velocity --kp 60 --ki 20           # 速度环（恒速模式用）
feetech-HD1910M-tester pid current --kp 200 --ki 50           # 电流环（力控核心）
```

### Proteção e limites

```bash
feetech-HD1910M-tester protect --max-temp 80                          # 过温阈值 80°C
feetech-HD1910M-tester protect --min-voltage 40 --max-voltage 100     # 电压窗口 4.0~10.0V
feetech-HD1910M-tester protect --overcur-time 200                     # 过流持续 2 秒触发保护
feetech-HD1910M-tester protect --unload 12 --led 12                   # 过热+过流时卸载并闪灯
feetech-HD1910M-tester limits 0 4095 --save                           # 角度限制（多圈模式设 0 0）
feetech-HD1910M-tester limits 90 270 --deg --save                     # 也可以按角度输入（0~360°）
feetech-HD1910M-tester offset -100 --save                             # 位置偏移（手动校正零位）
feetech-HD1910M-tester zero                                           # 零位校准：当前位置标定为中位 2048
feetech-HD1910M-tester zero --to 1024                                 # 或标定为指定读数（自动回读确认）
```

---

## 8. Tutorial de experimentos (mão na massa)

> Faça em ordem; cada experimento tem "pontos de observação". Recomenda-se acrescentar `-v` o tempo todo para ver as mensagens.

### Experimento 1: handshake de comunicação e observação do protocolo

```bash
feetech-HD1910M-tester -v get pos
```

Pontos de observação: no pacote enviado `FF FF 01 04 02 38 02 ...`, o `02` é a instrução READ, o `38` é o endereço inicial 56,
e o `02` é a leitura de 2 bytes; no pacote de resposta é possível encontrar os bytes little-endian da posição atual. Verifique o checksum calculando à mão.

### Experimento 2: entender a tabela de memória

```bash
feetech-HD1910M-tester dump
feetech-HD1910M-tester get mode        # 看当前运行模式（出厂 = 4 纯位置PD）
feetech-HD1910M-tester get status      # 状态字的中文解析
```

### Experimento 3: chave de torque e ensino manual

```bash
feetech-HD1910M-tester torque off      # 松轴，用手转动舵机
feetech-HD1910M-tester monitor         # 另开一个终端观察：位置随手转动变化，负载≈0
feetech-HD1910M-tester torque damp     # 阻尼模式：再用手转，有明显阻力
feetech-HD1910M-tester torque on       # 恢复扭矩
```

### Experimento 3.5: calibração do zero (definir a posição mecânica atual como ponto médio)

Depois da montagem mecânica, o zero muitas vezes não coincide com o ponto médio 2048 do servo; nesse caso usa-se a calibração do zero (equivalente ao "definir zero" do software FD da Feetech):

```bash
feetech-HD1910M-tester torque off      # 1. 松轴
# 2. 用手把关节摆到你要的零位（中间不要等待，重力会把关节压偏）
feetech-HD1910M-tester zero            # 3. 立即校准：当前位置被标定为 2048，掉电保存
feetech-HD1910M-tester torque on       # 4. 恢复扭矩（校准后当前读数已是 2048，不会跳变）
```

Pontos de observação: o comando relê automaticamente "posição atual + deslocamento de posição do endereço 31" para confirmar que a calibração surtiu efeito.
Atenção: o HD-1910 (firmware da série HLS) **não suporta** o método antigo de calibração "escrever 128 na chave de torque" (escrever também responde sucesso mas não surte efeito);
é preciso usar o comando de calibração de posição 0x0B correspondente ao comando `zero`. Não calibre no modo multivoltas.

### Experimento 4: movimento de posição e realimentação

```bash
feetech-HD1910M-tester monitor --count 100 &          # 后台监视（或另开终端）
feetech-HD1910M-tester goto 1024 --speed 300
feetech-HD1910M-tester goto 3072 --speed 1000
```

Pontos de observação: a flag `moving` vira 1 durante o movimento; a velocidade primeiro sobe e depois cai (planejamento trapezoidal);
ao chegar, a posição se estabiliza no alvo ±1 (efeito da zona morta).

### Experimento 5: ajuste de PID (região de execução, seguro e reversível)

```bash
# Kp 调小 → 动作变软、到位慢
feetech-HD1910M-tester pid position --kp 8 --ram
feetech-HD1910M-tester goto 1024

# Kp 调大 → 响应快但可能抖动/嗡嗡叫
feetech-HD1910M-tester pid position --kp 80 --ram
feetech-HD1910M-tester goto 3072

# 恢复出厂值（dump 里看到的 pos-p=32），或重新上电自动恢复
feetech-HD1910M-tester pid position --kp 32 --kd 40 --ram
```

### Experimento 6: controle de força (limite de corrente-alvo)

```bash
feetech-HD1910M-tester mode 0                 # 位置伺服模式（位置+限流）
feetech-HD1910M-tester goal-current 30        # 目标电流 ≈ 195mA，力气变得很小
feetech-HD1910M-tester goto 2048
# 用手轻轻阻挡舵机：它推不过你，电流被钳在设定值（monitor 里看电流）
feetech-HD1910M-tester goal-current 500       # 恢复（≈3.25A 上限）
```

### Experimento 7: funções de proteção

```bash
feetech-HD1910M-tester protect --max-temp 40  # 温度阈值调低到 40°C
# 用手捂住舵机或等它升温……超过 40°C 后扭矩关闭，get status 显示过热
feetech-HD1910M-tester protect --max-temp 80  # 玩完记得调回来
```

### Experimento 8: verificação do mecanismo de trava da EPROM

```bash
feetech-HD1910M-tester set resp-level 1       # 不加 --save：只改内存
feetech-HD1910M-tester get lock               # 看当前锁状态（默认 1）
feetech-HD1910M-tester set min-angle 10       # 改个值
# 给舵机断电重启，再 dump：min-angle 恢复成 0 —— 因为锁=1 没保存
feetech-HD1910M-tester set min-angle 10 --save
# 再断电重启：这次值留住了
feetech-HD1910M-tester set min-angle 0 --save # 实验完恢复
```

### Experimento 9: trocar o ID (o primeiro passo para montar uma rede de vários servos)

```bash
feetech-HD1910M-tester set-id 2               # 当前舵机变成 ID 2
feetech-HD1910M-tester -i 2 dump              # 用新 ID 连接
feetech-HD1910M-tester -i 2 set-id 1          # 改回来（单舵机练习时）
```

### Experimento 10: escrita assíncrona e escrita síncrona (sincronização de vários servos, exige 2 ou mais)

```bash
feetech-HD1910M-tester -i 1 set goal-pos 1024 --defer   # 缓存，不动
feetech-HD1910M-tester -i 2 set goal-pos 3072 --defer   # 缓存，不动
feetech-HD1910M-tester action                            # 广播：两台同时启动！
```

---

## 9. Precauções de segurança

1. **O travamento esquenta**: em travamento prolongado a corrente pode chegar a 1,6A, e a temperatura sobe muito rápido. Em experimentos de controle de força,
   limite a saída com `torque-limit` ou `goal-current` e fique de olho na temperatura no monitor.
2. **Vida útil da EPROM**: cerca de 100 mil ciclos de escrita/apagamento. Na fase de ajuste, use sempre `--ram` ou não use `--save`,
   e só salve depois de definido. Nunca use `--save` dentro de um laço!
3. **Registrador de fase (endereço 18)**: controla configurações de baixo nível como direção do motor/direção do encoder;
   mexer sem cuidado faz o servo perder o controle e girar descontrolado. A menos que você entenda completamente a seção 3.1 da documentação, não toque.
4. **Antes de alterar o ID / baud rate**: confirme que consegue se reconectar com os novos parâmetros (anote!).
   Se esquecer, use `factory-reset` ou faça uma varredura baud rate por baud rate para recuperar.
5. **Instalação mecânica**: durante os experimentos, não deixe o servo com carga balançando no ar, para evitar bater na mão ou arremessar peças.

---

## 10. Solução de problemas comuns

| Sintoma | Investigação |
|------|------|
| `响应超时` | ① O servo está alimentado? ② Há terra comum? ③ O baud rate está certo (padrão 1M, se foi alterado use `-b`)? ④ O ID está certo (procure com `scan`)? ⑤ Na ligação com TX/RX curto-circuitados é preciso `--echo` |
| `串口错误: Permission denied` | Adicione o usuário ao grupo dialout: `sudo usermod -aG dialout $USER`, e faça login de novo |
| `校验和错误` | Interferência na linha / mau contato, verifique os fios dupont; ou baud rate incompatível |
| O scan não acha mas se tem certeza de que está online | Use `-t 30` para alongar o timeout; teste baud rate por baud rate: `-b 115200 scan` etc. |
| O servo acusa "sobrecorrente" | Reenviar um comando de posição limpa; solução de longo prazo: reduzir protect-current ou torque-limit |
| A configuração alterada sumiu após reiniciar | A escrita na EPROM exige `--save` (mecanismo de trava, veja a seção 4.2) |
| A leitura de posição é um estranho número negativo grande | Não acontece mais —— mas saiba o motivo: a codificação sinal-magnitude foi confundida com complemento de dois (veja a seção 4.3) |

---

## Apêndice: trilha de estudo recomendada

1. Primeiro faça os experimentos 1~4 funcionarem, criando intuição sobre o protocolo e a tabela de memória;
2. Leia `src/protocol.rs` por inteiro (o mais curto, 200 linhas), entendendo a montagem/desmontagem de pacotes;
3. Leia `src/registers.rs`, entendendo a ideia de "transformar a tabela do documento em código";
4. Leia `src/servo.rs`, com foco em entender o encadeamento de `&mut self`, Result e `?`;
5. Leia `src/main.rs`, vendo a CLI declarativa do clap e o despacho por match;
6. Altere o código para praticar: acrescente ao `dump` a função "mostrar só os registradores diferentes do valor de fábrica";
   ou acrescente um subcomando `wave` que faça o servo oscilar senoidalmente, observando com o monitor.

Bom estudo!
