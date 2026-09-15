# Configurador SAGE → OTS

Ferramenta para identificar e configurar, na base do OTS (Elipse Power), os
pontos de supervisão que já existem no SAGE e ainda não foram importados.

Executa inteiramente no navegador, como uma página única, sem instalação e
sem servidor. Nenhum arquivo carregado é transmitido para fora do
computador do usuário — todo o processamento (leitura, comparação e geração
do arquivo atualizado) acontece localmente, no próprio navegador.

---

## Sumário

- [Visão geral](#visão-geral)
- [Como funciona](#como-funciona)
- [Requisitos e arquivos de entrada](#requisitos-e-arquivos-de-entrada)
- [Passo a passo de uso](#passo-a-passo-de-uso)
- [Interpretando o diagnóstico](#interpretando-o-diagnóstico)
- [Conferências antes de considerar definitivo](#conferências-antes-de-considerar-definitivo)
- [Limitações conhecidas](#limitações-conhecidas)
- [Privacidade e segurança dos dados](#privacidade-e-segurança-dos-dados)
- [Publicação e manutenção (GitHub Pages)](#publicação-e-manutenção-github-pages)
- [Suporte](#suporte)

---

## Visão geral

Toda vez que um equipamento novo é modelado no Elipse Power Studio, os
pontos de medição e comando correspondentes já existem na base do SAGE, mas
precisam ser manualmente adicionados ao arquivo **Configuração da
Importação** (`Configuracao_da_importacao.xlsm`) para que o OTS passe a
simulá-los. Esse processo é sujeito a erro e demorado quando feito à mão,
principalmente em subestações com centenas de pontos.

Esta ferramenta automatiza o diagnóstico e a maior parte da configuração:
compara os pontos existentes no SAGE com os já configurados no OTS,
resolve automaticamente o equipamento correspondente quando ele já está
modelado, e gera uma cópia atualizada do arquivo de importação — pronta
para revisão e uso.

## Como funciona

1. **Leitura dos três arquivos de entrada** (todos processados localmente,
   descritos em [Requisitos e arquivos de entrada](#requisitos-e-arquivos-de-entrada)).
2. **Comparação**: para a subestação escolhida, cada ponto do SAGE (medida
   analógica, comando ou medida discreta) é verificado contra o que já
   existe na Configuração da Importação.
3. **Resolução do equipamento**: para cada ponto que falta, a ferramenta
   procura o equipamento correspondente no modelo elétrico já cadastrado
   (aba `EstruturaEqps` e, se fornecido, no arquivo de export do modelo).
   Se o equipamento existe, o ponto é considerado configurável.
4. **Sugestão de configuração**: os valores de cada campo (tipo de medida,
   textos de estado, terminal, etc.) são sugeridos a partir de pontos com o
   mesmo padrão já configurados em outras subestações do mesmo arquivo —
   o mesmo princípio dos "modelos" (*templates*) usado manualmente hoje.
5. **Geração do arquivo atualizado**: uma cópia do arquivo de Configuração
   da Importação é gerada com as linhas novas inseridas nas abas corretas,
   preservando integralmente o restante do arquivo — incluindo macros,
   fórmulas, tabelas e validações já existentes.

## Requisitos e arquivos de entrada

| Arquivo | Obrigatório | Descrição |
|---|---|---|
| **Base do SAGE** (`Template_ems.xls` ou equivalente) | Sim | Extrato do SAGE com os pontos da(s) instalação(ões) a processar. Pode conter mais de uma subestação no mesmo arquivo. |
| **Configuração da Importação** (`Configuracao_da_importacao.xlsm`) | Sim | Arquivo de importação do OTS que será lido e atualizado. |
| **Export do Modelo Elétrico** (`Model_AAAAMMDD_HHMMSS.xlsx`) | Recomendado | Export direto do modelo elétrico do Elipse Power Studio. Amplia a base de equipamentos reconhecidos, reduzindo falsos "requer modelagem elétrica". |

Todos os arquivos devem ser carregados a cada sessão de uso — a ferramenta
não armazena nada entre uma execução e outra.

## Passo a passo de uso

1. Abra a página no navegador.
2. Carregue os três arquivos de entrada.
3. Selecione, no menu, a subestação a processar (a lista é preenchida
   automaticamente a partir do arquivo do SAGE carregado).
4. Clique em **Rodar diagnóstico** e revise o resumo apresentado.
5. Clique em **Aplicar e baixar planilha atualizada** para gerar e baixar a
   cópia atualizada do arquivo de Configuração da Importação.
6. Abra o arquivo baixado e realize as conferências descritas em
   [Conferências antes de considerar definitivo](#conferências-antes-de-considerar-definitivo)
   antes de substituir o arquivo em uso pela equipe.

## Interpretando o diagnóstico

Cada ponto identificado como faltante recebe uma classificação:

| Classificação | Significado | Ação necessária |
|---|---|---|
| **Pronto para revisão** | Equipamento já modelado; todos os pontos com o mesmo padrão, em outras subestações, concordam nos valores sugeridos. | Conferir e aprovar. |
| **Revisar divergência** | Equipamento já modelado; há divergência de valores entre pontos do mesmo padrão em diferentes subestações. | Conferir manualmente antes de aprovar. |
| **Poucos exemplos** | Equipamento já modelado; poucos pontos de referência para basear a sugestão. | Conferir manualmente antes de aprovar. |
| **Requer modelagem elétrica** | O equipamento correspondente ainda não existe no modelo elétrico do OTS. | Modelar o equipamento no Elipse Power Studio; o ponto fica listado na aba de pendências do arquivo gerado, para ser reprocessado depois. |

## Conferências antes de considerar definitivo

- Revisar toda linha marcada como divergência ou com poucos exemplos antes
  de aprovar.
- As colunas **PAC** (Comandos discretos) e **CopiCmdType** não são
  preenchidas automaticamente — o pareamento entre comando e medida de
  retorno ainda depende de conferência manual.
- Conferir a aba de pendências do arquivo gerado e planejar a modelagem
  elétrica dos itens listados nela.
- Reabrir o arquivo gerado em uma cópia de teste antes de substituir o
  arquivo em produção, confirmando que os valores calculados (colunas de
  fórmula) aparecem corretamente.

## Limitações conhecidas

- A ferramenta não modela equipamentos novos no OTS — a modelagem elétrica
  continua sendo um passo manual no Elipse Power Studio.
- O pareamento automático entre um comando e sua medida de retorno (PAC)
  não é inferido; permanece como conferência manual.
- A qualidade da sugestão de configuração depende do volume de pontos já
  configurados com o mesmo padrão em outras subestações do arquivo. Padrões
  novos, sem precedente no arquivo, não recebem sugestão automática.

## Privacidade e segurança dos dados

Todo o processamento — leitura, comparação e geração do arquivo — ocorre
localmente, no navegador do usuário. Nenhum arquivo carregado é enviado a
servidores externos. A página, ao ser aberta, carrega duas bibliotecas
públicas de leitura/escrita de planilhas a partir de uma rede de
distribuição de conteúdo (CDN); isso requer conexão à internet no
carregamento inicial da página, mas não implica envio de dados.

## Publicação e manutenção (GitHub Pages)

Este repositório contém um único arquivo estático (`index.html`),
publicável diretamente via GitHub Pages:

1. Em **Settings → Pages**, selecione a branch de publicação e a pasta
   `/ (root)`.
2. A publicação é concluída em poucos minutos, disponibilizando a
   ferramenta em `https://<usuário>.github.io/<repositório>/`.
3. Atualizações são feitas substituindo o arquivo `index.html` pela versão
   mais recente; a publicação é atualizada automaticamente.

Por padrão, o GitHub Pages publica o conteúdo do repositório de forma
acessível a quem tiver o link, independentemente da visibilidade do
repositório de origem. Como o repositório não contém dados da empresa —
apenas a lógica da ferramenta — isso não representa exposição de
informação sensível. Caso haja necessidade de restringir o acesso à página
publicada, consulte a documentação vigente do GitHub, já que as condições
de disponibilidade variam conforme o plano contratado.

## Suporte

Para dúvidas sobre o funcionamento, divergências no diagnóstico ou
sugestões de melhoria, contatar a equipe responsável pela manutenção deste
repositório.
