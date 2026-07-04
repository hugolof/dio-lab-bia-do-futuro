# Como Executar o PoupaSonho no Colab Google

Para replicar o ambiente e rodar o assistente, siga a sequência de execução das células no seu Google Colab:

## 1. Configuração do Ambiente

    Atenção: Antes de rodar qualquer célula, vá em Ambiente de execução > Alterar tipo de ambiente de execução e selecione T4 GPU. Sem isso, o modelo não carregará corretamente.

## 2. Fluxo de Execução

Siga a ordem lógica das células no notebook:

    Instalação & Ollama: Executa a instalação de dependências e baixa o modelo gemma4.

    Geração de Dados: Cria a estrutura de pastas /data e gera os arquivos JSON e CSV (Perfil, Transações, Desafios e Histórico).

    Setup do App: Cria o arquivo app.py com a lógica do Streamlit e o System Prompt.

    Execução do Servidor: Inicia o Streamlit e gera o link do localtunnel.

## 3. Acesso à Interface

Após rodar a última célula:

    O sistema imprimirá um endereço IP no console. Copie esse número.

    Clique no link gerado pelo localtunnel (ex: [https://xxxx.loca.lt](https://xxxx.loca.lt)).

    Uma página de segurança será aberta. Cole o IP copiado no campo "Endpoint IP" e clique em Submit.

    O PoupaSonho estará pronto para uso na sua aba.

## Evidência de Execução

<img width="1260" height="1734" alt="image" src="https://github.com/user-attachments/assets/9881e419-c998-419b-b147-67c9b597afde" />
