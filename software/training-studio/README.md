# Microduck Training Studio

> 🌐 Tradução não oficial para pt-BR do [original](https://github.com/fanhao375/microduck-replica). Em caso de divergência, vale o original. Licença: CC BY-NC-SA 4.0 (texto alterado: traduzido do chinês/inglês).

Bancada visual de treinamento do patinho. Usa por padrão os parâmetros recomendados, com configurações avançadas expansíveis; oferece curvas de treinamento, modelo tridimensional, reprodução de avaliação de checkpoints, pausar/continuar, comparação de experimentos e exportação de resultados.

A interface segue a paleta cinza claro do Studio de controle do robô, o emblema do pato, os botões pretos e a navegação inferior para celular.

O laboratório de movimentos oferece as abas "Receitas de treinamento" e "Modelos pré-treinados". O catálogo de modelos prontos inclui fixamente os 10 modelos XL330 da coleção V8 oficial e os 10 modelos HD1910 do V167 do kissqy, mostrando origem, hardware, versão, licença e o endereço de obtenção upstream. Aqui não se instala, avalia nem retoma automaticamente ONNX externos, nem se os registra como experimentos locais; novos experimentos de receita continuam treinando do zero. Alguns pesos da comunidade só podem ser obtidos no pacote de lançamento do firmware completo do autor.

Os detalhes do treinamento incluem, atualizados a cada cerca de 3 segundos, os recursos do lado de execução: modelo/utilização/temperatura/memória da GPU, modelo/utilização/temperatura da CPU, memória. Linux/WSL usam `/proc`, sensores de temperatura de CPU disponíveis e `nvidia-smi` com tempo-limite; o Windows usa contadores do sistema e ferramentas do driver; não é preciso instalar dependências de monitoramento. Campos ausentes ficam vazios, com explicação do motivo. Os recursos são a leitura geral do lado de execução atual; a CPU/memória do WSL é o alcance visível ao subsistema, e não pode ser tomada como uso exclusivo ou histórico de um único experimento.

O motor de treinamento fica no adjacente [`../training`](../training/README.md), e o controle do robô fica em [`../studio`](../studio/README.md). Este diretório é um serviço independente, que usa o projeto de treinamento existente, sem precisar atualizar a imagem do robô.

## Experimentar primeiro na própria máquina

Requer Python 3.12. O serviço usa apenas a biblioteca padrão, a página não tem etapa de build nem carrega dependências de CDN. Mantenha o `studio/web` adjacente, que fornece as bibliotecas web e o modelo tridimensional já distribuídos com o repositório.

```console
cd software/training-studio
python run.py --demo --open
```

No Windows também dá para dar duplo clique em `start-demo.cmd`; no Linux/macOS execute `sh start-demo.sh`. Endereço padrão: <http://127.0.0.1:8093>.

O modo demonstração inicia um subprocesso leve independente, gerando **métricas sintéticas, checkpoints de texto e uma reprodução esquemática em SVG claramente identificada**, para experimentar o fluxo de operação. Ele não executa treinamento de política em GPU. O pacote de exportação da demonstração só tem registros, sem pesos ONNX/PT implantáveis.

## Treinamento real

Em um computador Linux com GPU NVIDIA já preparado ou no Windows WSL2, use o projeto de treinamento deste repositório e suas dependências travadas. O modo real verifica o backend gráfico do MuJoCo, o mjlab 1.3.0, o CUDA e o registro da tarefa HD1910; não instala pacotes automaticamente, não solicita computador em nuvem nem cuida de recargas. As operações locais estão em [Teste curto local](docs/local-smoke.md).

```sh
cd software/training
uv sync --frozen --python 3.12
cd ../training-studio
../training/.venv/bin/python run.py
```

Na máquina local, abra outro terminal para encaminhar a porta (usuário e endereço conforme o seu computador em nuvem):

```sh
ssh -N -L 8093:127.0.0.1:8093 用户名@云电脑地址
```

Depois abra <http://127.0.0.1:8093>. Dá para usar `--training-root /caminho/software/training` para indicar a localização do motor, e `--state-dir /diretório-persistente/training-studio` para indicar o diretório de registros. Fechar a página não interrompe o treinamento; **fechar o serviço da bancada de treinamento interrompe os subprocessos que ele possui**. Para execução prolongada, mantenha o `run.py` rodando por meio de um gerenciador de sessão de terminal ou gerenciador de serviços no computador em nuvem.

O serviço por padrão só escuta na máquina local. Se for realmente necessário usá-lo por outra entrada de rede, defina `STUDIO_ACCESS_TOKEN` e inicie com `--host 0.0.0.0`; a página pedirá o token. Para acessar o computador em nuvem a partir da máquina local, recomenda-se o encaminhamento SSH acima; a conexão direta pela rede local exige o endereço de escuta da máquina do serviço. O token fica apenas na sessão atual do navegador, e a API e o download de artefatos exigem `X-Studio-Token`; a verificação de Host/Origin permanece ligada.

## Um exercício completo

O "Laboratório de movimentos" na navegação é uma página independente da biblioteca de receitas (`#/actions`), e a página inicial mantém a entrada. Dá para buscar, filtrar por categoria, ver a pré-visualização do modelo ou a descrição em texto, e então usar a receita para iniciar o treinamento. Caminhada em terreno plano, terreno irregular e equilíbrio em um pé existentes são cada um marcados com seu estado de validação; o modo de inclusão de contribuições da comunidade está em [Guia de contribuição de receitas de movimento](docs/recipe-contribution.md).

Entre eles, o **equilíbrio em um pé do HD1910**, experimental, permite escolher o pé de apoio, a altura de elevação do pé (0.5–2 cm) e o tempo de manutenção (1–5 segundos), sem alterar código. O ciclo completo inclui preparação, elevação do pé, manutenção e recuperação ao apoiar. A ilustração tridimensional do alvo não representa equilíbrio físico nem resultado de treinamento. Saltos, giros precisos e imitação de vídeo ainda não foram integrados.

O passo a passo para iniciantes, a interpretação de resultados e os passos de desenvolvimento para novos movimentos estão no [Guia de início do Laboratório de movimentos](web/custom-actions.md), que também pode ser aberto pela barra lateral ou pela janela de novo treinamento.

1. Em "Ambiente de computação" confirme que a detecção passou; depois de instalar as dependências, pode-se detectar de novo.
2. Crie um experimento, escolhendo primeiro **Caminhada em terreno plano do HD1910**. Na primeira execução, faz-se um teste curto fixo de **64 ambientes, 5 iterações**, para verificar o carregamento, o passo de simulação e a cadeia de salvamento. O teste curto não é um resultado de treinamento.
3. Depois do teste bem-sucedido, entra-se no treinamento oficial. É possível expandir o número de ambientes paralelos, iterações de treinamento, intervalo de salvamento, semente e taxa de aprendizado; recolhido, ainda mostra o resumo das alterações. Depois de modificar o alvo do movimento, a taxa de aprendizado, a semente aleatória ou o perfil importado, é preciso refazer o teste curto com as configurações correspondentes; o número de ambientes e de iterações podem ser ampliados para o treinamento oficial.
4. Veja as métricas brutas de treinamento, as iterações e o log. A porcentagem indica o progresso das iterações planejadas, não o grau de domínio do movimento. O modelo tridimensional pode ser girado; **o modelo CAD e os movimentos de demonstração não podem ser usados para julgar o desempenho da política**.
5. Pause, continue ou pare a tarefa. Depois de escolher um checkpoint, faz-se a avaliação: tarefas de caminhada geram uma reprodução de 250 passos; tarefas de equilíbrio fazem 8 tentativas de ciclo completo com sementes registradas, e mesmo falhando cedo preservam o vídeo e o motivo da falha. A página mostra o número de sucessos e a situação de manutenção e apoio. A reprodução corresponde ao checkpoint escolhido, não é uma imagem de treinamento contínuo em tempo real.
6. Compare experimentos oficiais da mesma tarefa e do mesmo modo; depois de confirmar os checkpoints necessários, exporte o pacote do modelo e siga o fluxo existente de validação em simulação e no robô real.

| Operação | Comportamento |
|---|---|
| Pausar | No Linux usa SIGSTOP no grupo de processos, preservando política, otimizador e memória de vídeo; o subprocesso de demonstração só atualiza a interface depois de confirmar a pausa |
| Continuar | No estado pausado, continua com o processo original; nos estados parado/concluído, cria um novo experimento a partir do checkpoint escolhido e registra a origem |
| Parar | Encerra o processo de treinamento que possui, preservando os checkpoints com escrita concluída; não garante salvar iterações ainda não escritas |
| Avaliar | Depois que o treinamento para, executa inferência com limite de passos no checkpoint escolhido e gera o vídeo |
| Exportar | Chama o `scripts/export.py` do motor, preservando a normalização de observações embutida e empacotando ONNX, checkpoint, configuração e registro de origem |

Pausar ainda ocupa memória de vídeo e **não desliga o computador em nuvem nem interrompe a cobrança da plataforma automaticamente**. O mesmo serviço executa uma operação de treinamento, avaliação ou exportação por vez. Reiniciar o serviço marca as tarefas antigas não encerradas como interrompidas, sem operar outros processos com base no PID dos registros.

## Perfis e resultados

Suporta importar JSON `microduck-studio-duck-profile/v1` (no máximo 64 KiB). O perfil é salvo junto com o experimento para conferência e **não sobrescreve automaticamente a massa, a inércia, os zeros das juntas ou a configuração de servos usadas no treinamento**. A receita atual do HD1910 é uma linha de base em simulação, e ainda precisa de validação independente com o seu robô real.

Os registros do serviço ficam por padrão em `state/runs/<id>/`, e os logs brutos do motor e os checkpoints PT continuam em `../training/logs/rsl_rl/`. Ao fazer backup ou migrar, preserve esses dois locais; os registros da interface referenciam o caminho original dos checkpoints do motor. O ZIP exportado pode ser guardado separadamente, mas esta versão não tem entrada de "enviar ZIP para retomar o treinamento".

A tarefa de equilíbrio usa o diretório de experimento independente `microduck_hd1910_balance` e usa o `skill.json` no treinamento, retomada, avaliação e exportação. O pacote exportado inclui a convenção de comandos. Embora a política continue sendo de 61 dimensões de observação, 14 de ação, 50 Hz, o significado dos slots de comando mudou, e o lado de implantação de caminhada existente precisa se adaptar ao escalonamento por fases; o equilíbrio em um pé ainda não foi validado no robô real.

O equilíbrio em um pé já completou dois testes curtos reais em GPU (esquerda e direita), 8 avaliações e verificação de consistência do ONNX. A política atual de 5 iterações tem taxa de sucesso 0/8, o que só prova que a cadeia funciona. Veja [Aceitação do Laboratório de movimentos](docs/action-lab-verification.md). O posicionamento e o fluxo de contribuição da biblioteca comunitária de receitas de movimento estão na [descrição do design](docs/action-lab-design.md#后续定位社区动作配方库); compartilhamento e instalação automática ainda não foram implementados.

A tarefa em terreno plano do HD1910 já completou, em WSL2 / RTX 5060 Ti, treinamento real de 64 ambientes e 5 iterações, salvamento de PT, avaliação em vídeo de 250 passos e exportação para ONNX, e a consistência de inferência PT/ONNX foi conferida de forma independente. Veja o [registro de aceitação](docs/verification.md). Terreno irregular, treinamentos longos, o computador em nuvem alvo e a implantação no robô real ainda precisam de aceitação separada; o teste curto não prova que o movimento esteja treinado de forma madura.

## Validação de desenvolvimento

```console
python -m unittest discover -s tests -v
node --test tests/state.test.mjs
```

A aceitação no navegador usa Playwright; o script inicia por conta própria um serviço de demonstração isolado, define um token de teste, chama a interface de verdade e salva capturas de tela. Defina `PLAYWRIGHT_MODULE_ROOT` com o caminho de um `node_modules` existente para reaproveitar um Playwright já instalado; também é possível instalar o Playwright no ambiente de desenvolvimento. `BROWSER_EXECUTABLE` pode indicar o executável do Chrome/Edge, e `PYTHON` pode indicar o Python 3.12. Os testes unitários acima rodam sem instalar dependências de navegador.

```console
node dev/browser-check.mjs
```

A saída da aceitação é gravada por padrão no diretório temporário do sistema, e `BROWSER_CHECK_OUTPUT` pode indicar o diretório. O teste não se conecta a plataforma em nuvem, GPU nem robô. O design e as convenções da API estão em [design.md](docs/design.md), a lista de aceitação da implementação em [implementation-plan.md](docs/implementation-plan.md), e os resultados desta rodada de testes e revisão em [verification.md](docs/verification.md).

O guia de movimentos tem como arquivo-fonte `web/custom-actions.md`. Depois de modificá-lo, em um ambiente de edição de documentos com Python-Markdown instalado, execute `python dev/build-guide.py` e faça o commit também do `web/guide.html` gerado. A página usa diretamente o arquivo já gerado; executar a bancada de treinamento não exige instalar Markdown nem executar build.

## Atribuição

O código original deste diretório adota o Apache-2.0 do repositório; a documentação mantém a licença de documentos correspondente do repositório. Os modelos compartilhados, as bibliotecas web e o motor de treinamento seguem suas próprias licenças, veja [NOTICE.md](NOTICE.md).

Para a comparação com projetos semelhantes e as prioridades seguintes, veja a [comparação com o controle de treinamento do kissqy](docs/reference-comparison-2026-10-23.md).
