# api-monitorizada-conteinerizada
Api conteinerizada com telemetria e observabilidade

Hoje o Docker tem se tornado um requesito mínimo de atuação para os desenvolvedores.
Neste humilde artigo vos trago uma pequena api .net core 6 que lança os logs no elasticsearch que podem ser vistos no kibana, coletagem de métricas com prometheus que podem ser vistos no grafana.
O objetivo deste artigo é de uma forma abreviada mostrar um projeto pronto funcionando com poucos passos, a reunião destas ferramentas todas agendo em harmonia com Docker.

Arquitetura do projeto:

![CI-CD](https://user-images.githubusercontent.com/111398584/212097940-7c1141fa-129e-4247-9d26-6bcaa4c850a9.png)

A api envia métricas no container do prometheus, e o prometheus é adicionado no grafana.
A api envia logs no container do elastic que por sua vez envia para o kibana.

Conseguimos monitorar uma api com poucos passos.

Em vermelho encontramos as bibiliotecams que temos de instalar no nuguet:


# Como inicializar o projeto?
1- Entrar na primeira pasta cd e rodar o comando:

    docker compose up -d
   
2- Entrar na segunda pasta cd e rodar o seguinte comando:
   
    docker compose up -d
   
3- Entrar na terceira pasta cd rodar o seguinte comando:

    docker compose up -d
   
4- Para buildar a api e conectar ela na rede do prometheus e elastic:
4.1 - Vai na pasta principal onde se encontra o Dockerfile
4.2 - Executa o seguinte comando:

      docker image build --no-cache -t apoioprodesp/apicontainer:v1 . 
      
4.3 - Executa o seguinte comando:

      docker run -d --network  cd_n_easybox --name easybox -p 8001:80 apoioprodesp/apicontainer:v1


Nota: este repositório apoioprodesp/apicontainer:v1 é um repositório docker, seria mais útil inserir o teu repositório.


# Como verificar se tudo está rodando?

Para o elastic vai na seguinte url: http://localhost:9200/

Para o kibana faça as seguintes comfigurações : http://localhost:5601/app/kibana#/home?_g=()

Para o prometheus faça o seguinte: http://localhost:9090/graph

Para o grafana Faça o seguinte: http://localhost:3000/login

Para a api : http://localhost:8001/swagger/index.html


# Prometheus 

Primeiramente devemos saber o seguinte:
Metricas são medições numéricas de dados relacionados a elemosntos do seu software ou da infraestrutura.
São dados relacionados numa linha temporal.

Prometheus é bastante útil para Métricas dentre elas:

Métricas de Sistema:
Quantidade de requisições
Quantidade de erros
Consumo de recursos
Apis mais acessadas
Tempo de acesso a um recurso

Metricas de Negócio
Usuário  acessando a aplicação
Boletos emitidos
Compras de produto
Transação aceite

Métricas de infraestrutura
Nós ativos
Cluster funcionando

# Métricas não são logs!


Métricas -->
Dados Numéricos,
Gráficos,
Agregações,
Performance

Logs -->
Dados Textuais,
Mensagens de erro,
Informação,
Buscáveis



