### Estrutura dos Arquivos Principais

**README.md:** Explica o propósito, arquitetura, benchmark (UnKEBench), requisitos e como rodar o método.

**unke.py:** Script principal que executa o processo de edição. Carrega dados, modelo, aplica as edições e salva os resultados.

**config.py:** Arquivo de configuração com parâmetros do modelo, caminhos de dados, hiperparâmetros, etc.

**compute_z.py:** Funções auxiliares para cálculos internos do método.

**nethook.py:** Serve para interceptar camadas do modelo durante o forward, permitindo capturar e modificar representações internas. Permite aplicar mudanças de forma precisa em camadas específicas, sem alterar o restante do modelo.

**evaluate.py e eval_fact_score.py:** Scripts para avaliação dos resultados das edições por meio do gpt e também usa algumas métricas automáticas para comparar a similaridade entre repsosta gerada e repsota correta (bleu, rouge e bertscore).

**alpaca_data.json:** É um dataset de instruções genéricas, usado como exemplo para edição ou como dados auxiliares, não é focado em factualidade, mas em tarefas abertas e instruções variadas.

**final_data_v2 e final_data_v3:** São os principais datasets para edição e avaliação factual no UnKE. O v3 tem mais informações relacionadas ao benchmark MMLU.

**mmlu_shot.jsonl:** Usado para avaliação no benchmark MMLU, que testa conhecimento factual e acadêmico em múltiplas áreas.


### Passo a passo de como mexer no repo:

**Configuração:** Ajustar os parâmetros em config.py (modelo base, taxa de aprendizado, batch size, camadas a editar...).

**Execução:** Rodar o script principal (unke.py) para editar o modelo conforme os dados.
    - O unke.py é mais geral, serve para editar e analisar o efeito das edições em perguntas e suas variações. Não faz avaliação automática de acurácia em benchmarks
    - Já o unke_mmlu.py, além de editar o modelo, faz avaliação automática usando o benchmark MMLU, bom para medir o efeito da edição em conheciemnto factual estruturado (não é o nosso caso).

**Avaliação:** Usar o script evaluate.py para medir o desempenho do modelo editado no benchmark proposto, o UnKEBench.

### Como ocorre a edição dos modelos?

1) O método intercepta camadas específicas do Transformer (que são definidas em config.layers). No artigo usa-se a camada 7 **(tivemos essa dúvida na última reunião)**

2) Calcula representações internas antes e depois da edição.

3) Aplica gradientes para modificar os parâmetros do modelo, distribuindo as mudanças entre as camadas.

4) O processo é feito em batches, e os pesos originais podem ser restaurados após cada batch, se desejar.

### Dúvidas e levantamentos:

- Vamos usar qual llm? (Aqui é usada essa aqui 'LLama2-7B-Chat' ou 'Qwen1.5-7B-Chat')
  - Pra usar outra llm:
    - Adaptar ou criar funções de máscara de atenção e de formatação de entrada/saída específicas para o novo modelo escolhido, garantindo compatibilidade com a arquitetura e o processamento do UnKE.
    - Ajustar o carregamento do modelo/tokenizer.
    - Garantir que a estrutura interna (nomes das camadas...) seja reconhecida pelo UnKE.
- Como escolhemos as camadas que devemos editar?
- O dataset de entrada tem essas colunas (question, para_question, answer, sub_question, sub_answer)
- Aqui no UNKE são perguntas e respostas de fatos, no nosso vai ser extamente o que?
- Como a gente pode analisar se a edição foi boa? compara com o treinado em português
- Avaliar as metáforas talvez valha incluir as métricas que já usamos, né? Nos casos em que errou!
