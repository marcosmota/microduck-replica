# Microduck HD1910: treinamento em simulação

> 🌐 Tradução não oficial para pt-BR do [original](https://github.com/fanhao375/microduck-replica). Em caso de divergência, vale o original.

O código de treinamento, os modelos, os parâmetros, os arquivos de lock de dependências e os testes do Feetech HD-1910 são mantidos todos neste diretório.
Baixar ou clonar o `microduck-replica` já dá o projeto completo, sem precisar buscar branch de treinamento nem inicializar submódulos.

Usa os **[parâmetros 1910 BAM M6 publicados pelo LuwuDynamics](src/mjlab_microduck/robot/hd1910/README.md)**;
o framework de treinamento e a geometria do Microduck vêm da [Pollen Robotics](https://github.com/pollen-robotics/microduck_rl),
e o modelo de atuador vem de [Rhoban/BAM](https://github.com/Rhoban/bam). Os parâmetros não são resultado de identificação feita por este projeto.
Este projeto acrescenta a configuração de tarefas do HD1910, a adaptação da inicialização do estado do controlador Feetech e verificações de regressão.

## Início rápido

Use Linux com GPU NVIDIA CUDA, ou WSL2 com GPU já configurada; precisa de Git e [uv](https://docs.astral.sh/uv/).
Execute em um terminal Linux / WSL:

```bash
git clone https://github.com/fanhao375/microduck-replica.git
cd microduck-replica/software/training
uv sync --frozen --python 3.12

# 先用 64 个环境、5 轮迭代检查训练链路
uv run --frozen train Mjlab-Velocity-Flat-MicroDuck-HD1910 \
  --env.scene.num-envs 64 --agent.max-iterations 5 \
  --agent.logger tensorboard
```

Se você já baixou este repositório, basta entrar em `software/training/` para instalar e executar.
O envio opcional de jobs em nuvem HF Jobs precisa de clone via Git, para empacotar o código-fonte segundo as regras de ignore do Git;
baixar o ZIP diretamente serve para o treinamento local acima.
Logs e checkpoints ficam em `logs/rsl_rl/microduck_hd1910_velocity/` neste diretório e não são publicados com o código-fonte.
O treinamento curto de 5 iterações serve para verificar o carregamento, o passo de simulação e o salvamento, e ainda não basta para aprender a andar.

| Entrada | Finalidade |
|---|---|
| `Mjlab-Velocity-Flat-MicroDuck-HD1910` | Tarefa de velocidade em terreno plano do HD1910 |
| `Mjlab-Velocity-Rough-MicroDuck-HD1910` | Tarefa de velocidade em terreno irregular do HD1910 |

Atualmente é a linha de base de integração em simulação, e ainda não há uma política de marcha aprovada em testes no robô real. Massa e inércia ainda seguem os valores
de simulação do Microduck original; depois será preciso corrigi-los com o robô Feetech real e conferir zeros, sentidos, limites, alimentação, IMU e temporização de comunicação.

## Descrição e testes

- [Instruções de uso, registro de validação e explicação do problema de inicialização](docs/hd1910-baseline.md): o que faltava é o histórico de ângulos-alvo do controlador Feetech, e a correção não altera os parâmetros M6 emprestados.
- [Origem, versões fixadas e alcance das licenças](UPSTREAM.md).
- [README do projeto de treinamento original](README.upstream.md): mantém a descrição das tarefas e ferramentas upstream, cuja tarefa padrão é voltada ao XL330; a versão Feetech usa as entradas HD1910 da tabela acima.

Execute neste diretório as verificações de integração e regressão (incluindo o passo de simulação em GPU):

```bash
uv run --frozen --with pytest python -m pytest -q \
  tests/test_hd1910_cfg.py tests/test_hd1910_rollout.py \
  tests/test_ground_pick_cfg.py tests/test_nan_guard.py tests/test_obs_nan_guard.py \
  tests/test_hf_source_snapshot.py
```

A exportação de política continua usando `scripts/export.py`, incluindo a normalização de observações no ONNX. Checkpoints de treinamento curto servem apenas para
validar o fluxo, e não como marcha publicada para o robô real.

## Licença

O código, a configuração, os parâmetros e a documentação deste diretório de treinamento usam [Apache-2.0](LICENSE), mantendo as atribuições de terceiros.
**Os modelos 3D e as malhas seguem a declaração CC BY-NC-SA do upstream** e não passam a Apache-2.0 por terem sido incorporados a este repositório;
veja [UPSTREAM.md](UPSTREAM.md) e o [NOTICE.md](../../NOTICE.md) na raiz.
