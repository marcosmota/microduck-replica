# CAD e modelos unificados no repositório complementar

> 🌐 Tradução não oficial para pt-BR do [original](https://github.com/fanhao375/microduck-replica). Em caso de divergência, vale o original. Licença: CC BY-NC-SA 4.0 (texto alterado: traduzido do chinês/inglês).

**Baixe as peças estruturais, os conjuntos montados, os arquivos-fonte SolidWorks / STEP e os projetos de impressão em [fanhao375/microduck-replica-cad](https://github.com/fanhao375/microduck-replica-cad).**

Esse repositório é o ponto de entrada unificado dos modelos e inclui a escolha de versão, a BOM de montagem, os diagramas de componentes e as instruções de instalação. Em 2026-09-28, o Feetech HD-1910 está na [v2.1](https://github.com/fanhao375/microduck-replica-cad/releases/tag/v2.1) e o XL330 na [v1.1](https://github.com/fanhao375/microduck-replica-cad/releases/tag/v1.1). Antes de baixar, confira o tipo de servo e a versão específica do anexo; para as observações sobre impressão, veja [print/README](../print/README.md).

Os 16 STL que existiam neste diretório eram prévias de montagem geradas a partir do MJCF do upstream, não sincronizadas com o CAD físico mais recente, e foram removidos em 2026-09-28. Para consultá-los, veja o histórico do Git; os arquivos mais recentes devem ser obtidos sempre no repositório complementar.

As renderizações de análise em `assembly-drawings/` deste repositório, os modelos de treinamento e os modelos do console web de depuração ainda usam geometria do upstream e não devem servir de base para usinagem ou impressão das peças estruturais mais recentes. O [script de exportação](../scripts/export_assembly_stl.py) serve apenas para análise do modelo do upstream e por padrão grava em `analysis-output/cad-upstream/`, que é ignorado pelo Git.

**English:** CAD assemblies, mechanical parts and print files now have one maintained source: [microduck-replica-cad](https://github.com/fanhao375/microduck-replica-cad). The old upstream assembly-preview STLs were removed from this directory. Simulation and visualization meshes retained elsewhere are not manufacturing files.
