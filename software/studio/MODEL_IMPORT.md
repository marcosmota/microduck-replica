# Importar um modelo treinado

> 🌐 Tradução não oficial para pt-BR do [original](https://github.com/fanhao375/microduck-replica). Em caso de divergência, vale o original. Licença: CC BY-NC-SA 4.0 (texto alterado: traduzido do chinês/inglês).

Abra a "Gestão de modelos" do Studio, sem precisar regravar a imagem do sistema.

1. No computador de treinamento, localize o diretório do modelo exportado da mesma implantação.
2. Empacote `policy.onnx`, `manifest.json` e `deployment-contract.json` em ZIP, tar.gz ou tgz. Se na exportação houver `deployment-execution.json`, `deployment-runtime.json` e `model-info.json`, coloque-os também no mesmo diretório. Não misture logs de treinamento, scripts nem vários modelos.
3. Escolha o pacote de implantação, opcionalmente informe um nome de exibição e clique em "Importar modelo". O limite do pacote de implantação é 64 MiB; depois de expandido, 256 MiB.
4. Importação bem-sucedida significa apenas que foi instalado. Primeiro libere o torque, depois escolha a finalidade no cartão do modelo e clique em "Usar este modelo". A página só mostra sucesso depois que o programa principal confirma o caminho real do modelo.
5. Quando precisar de botão de ação, clique em "Definir botão" do modelo e aperte uma vez o botão de ação do controle (gamepad) físico, ou escolha na lista suspensa "Botão do controle"; um botão ocupado exige confirmação para substituição. A caminhada continua usando o controle original de ficar em pé e andar.

Um arquivo ONNX isolado, sem finalidade de ação, ordem das juntas, postura em pé e parâmetros de execução, não pode ser tomado diretamente como pacote de implantação compatível. O Studio verifica a identidade do arquivo e o contrato, e depois chama o `robotd validate-policy` local para pré-verificar o ONNX. Sem um verificador prévio utilizável, a importação é recusada, e não se trata a descompactação bem-sucedida como compatibilidade.

A postura em pé base e o parâmetro P dos servos do programa em execução atual são determinados na inicialização. Modelos diferentes da postura em pé base, do parâmetro P ou da ordem das juntas existentes não podem ser trocados a quente diretamente, e a página dá o motivo; é preciso antes concluir a adaptação de implantação correspondente.

## Relação entre modelo, ação e botão

- "Instalado": o arquivo do modelo está na biblioteca; "Selecionado atualmente": essa finalidade já escolheu este modelo; "Em execução": o programa principal está executando-o.
- O botão é vinculado à ação, e a ação usa o modelo atualmente selecionado. Os botões configuráveis são A, X, LB, RB, direcional para baixo. Caminhar e ficar em pé isolado não são vinculados como botões de ação comuns.
- A captura de botão físico exige confirmar que o torque está liberado, que a configuração dos cinco botões está completa e é de ação comum, e esperar dois segundos de configuração estável; se houver vínculo de ação especial como levantar direto, use a lista suspensa para atribuir, evitando que a tecla capturada dispare também a ação original.
- "Desvincular" só limpa esse botão; "Restaurar botões padrão" restaura as ações padrão dos cinco botões; nenhum dos dois exclui modelos.
- "Restaurar embutido" volta uma finalidade ao modelo padrão registrado inicialmente, e o modelo importado continua na biblioteca.
- O padrão embutido não pode ser excluído. Um modelo importado ainda referenciado por uma finalidade atual não pode ser excluído; primeiro troque para outro modelo ou restaure o embutido.

Os arquivos importados e os nomes de exibição ficam no diretório state do Studio; no pato normalmente é `/var/lib/microduck-studio`. Ao atualizar o Studio, preserve esse diretório. Os arquivos do modelo padrão permanecem no local de instalação original. Renomear só muda o nome de exibição, não a identidade do modelo.

## Validação de desenvolvimento

`tests/test_models.py` e `tests/test_model_package.py` usam modelos temporários e um programa principal falso, sem tocar nos servos. `tests/test_models_e2e.py` usa HTTP real, análise de pacote de implantação, biblioteca de modelos e Edge headless; o pré-verificador e a interface de execução nos testes são explicitamente substituídos por implementações de teste locais. Esses testes confirmam o fluxo de software e não substituem a aceitação de movimento do modelo real em hardware.
