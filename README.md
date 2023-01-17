# api-monitorizada-conteinerizada
Api conteinerizada com telemetria e observabilidade.

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
Metricas são medições numéricas de dados relacionados a elementos do seu software ou da infraestrutura.
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


Prometheus serve para coletar métricas na api, fazer consulta com elas, criar alertas e enviar em diversos outros softwares como Alert manager e Grafana.

![download](https://user-images.githubusercontent.com/111398584/212371744-fbe1e12c-8e9b-41f2-b6c0-e7f2280b49bb.png)


Primeiro criar uma api e na api instalar a seguinte lib:

    prometheus-net.AspNetCore(7.0.0)

Depois devemos na classe Program.cs inserir o seguinte using:

    using Prometheus;
    
    
Depois devemos inserir este linha de código após var "app = builder.Build();" :

    /*INICIO DA CONFIGURAÇÃO - PROMETHEUS*/
    var counter = Metrics.CreateCounter("nome-da-api", "Counts requests to the WebApiMetrics API endpoints",
                  new CounterConfiguration
                  {
                      LabelNames = new[] { "method", "endpoint" }
                  });

    app.Use((context, next) =>
    {
    counter.WithLabels(context.Request.Method, context.Request.Path).Inc();
    return next();
    });

    // Use the prometheus middleware
    app.UseMetricServer();
    app.UseHttpMetrics();
    /*FIM DA CONFIGURAÇÃO - PROMETHEUS*/


<img width="818" alt="Captura de tela_20230113_134822" src="https://user-images.githubusercontent.com/111398584/212374682-998a1f09-739a-499f-81ab-d88fea326a71.png">


Primeiro vamos abordar o docker-compose.yml (Concentre-se nos hashtags #)

    version: '3.1'  # versão do compose
    networks:  # especificação da rede
       n_easybox:  # nome da rede
        driver: bridge # tipo da rede

    services: # serviço

      prometheus: # nome do serviço
       image: prom/prometheus:v2.22.0 # imagem docker do prometheus
       ports:  # portas
         - "9090:9090"
       volumes: # volumes
         - ./prometheus.yml:/etc/prometheus/prometheus.yml
       networks: # a rede
         - n_easybox
       
De forma simples ficaria assim :

    version: '3.1' 
    networks:
       n_easybox:
        driver: bridge

    services:

      prometheus:
       image: prom/prometheus:v2.22.0
       ports:
         - "9090:9090"
       volumes:
         - ./prometheus.yml:/etc/prometheus/prometheus.yml
       networks:
         - n_easybox
       
    
Esta linha :  "- ./prometheus.yml:/etc/prometheus/prometheus.yml" ela é um arquivo a parte .yml que deve ficar no mesmo diretório do docker-compose.yml


        global: 
          scrape_interval: 15s 
          scrape_timeout: 5s 
          evaluation_interval: 15s 
          external_labels: 
           servico: service-prometheus 

        scrape_configs:
          - job_name: prometheus
            static_configs:
              - targets: ["localhost:9090"] # esta é a url do prometheus 
                labels:
                  grupo: "Prometheus"

          - job_name: ApiEasyBox
            scrape_interval: 5s
            scrape_timeout: 1s
            scheme: http
            metrics_path: /metrics # este é o path onde se encontra o caminho das métricas que a api lança
            static_configs:
              - targets: ["easybox:80"] # esta é a api espelhada na porta 80, este nome easybox é o nome da imagem que foi instanciada .
                labels:
                  grupo: "webapi"
                  
                  
 Nota - Sempre que for criar uma nova api e quer que este mesmo prometheus colete as métricas é simplesmente aumentar o range de - targets: ["easybox:80","outra-api"]

Ao ir em targets no Prometheus poderá ver a seguinte imagem:

<img width="926" alt="Captura de tela_20230113_150300" src="https://user-images.githubusercontent.com/111398584/212389091-8415a0a3-f5d9-4858-8349-e5810a6ddf72.png">

Veremos que a nossa api está no ar e que o prometheus coleta métricas de ele mesmo.


# PROMQL

Em resumo como podemos fazer querys de métricas para o prometheus sobre a nossa api?

Nossa aplicação quando a gente navega em /metrics:

<img width="938" alt="Captura de tela_20230112_182359" src="https://user-images.githubusercontent.com/111398584/212389459-51947a2a-4c77-43d6-b67b-e3691e2d6b0a.png">

1- passo ver aqual é a métrica que queremos consultar:

vou consultar:

       http_Prequests_received_total
       
 Teria um formato semelhante as duas imagens:
<img width="949" alt="Captura de tela_20230113_151041" src="https://user-images.githubusercontent.com/111398584/212390913-de031bb8-d5e6-4f25-bab0-ac4d9be73c37.png">

<img width="952" alt="Captura de tela_20230113_151131" src="https://user-images.githubusercontent.com/111398584/212390936-ef73ce33-3102-4ac3-ac93-fcea9fe4220b.png">


Filtrar apenas aqueles endpoints que o status code dá 200:

    http_requests_received_total{code="200"}
    
<img width="952" alt="Captura de tela_20230113_152010" src="https://user-images.githubusercontent.com/111398584/212391730-528b66dd-9415-42d4-9164-576faf675d43.png">

Outras querys básicas mas que fazem muita diferença caso quiser-mos achar os verbos de requisição que foram feitos:

    http_requests_received_total{method="GET"}
    http_requests_received_total{method="POST"}
    
Média de chamadas em 1 minuto:

    rate(http_requests_received_total[1m])
    
Soma de todas as requisições por api

    sum(http_requests_received_total) by (job)
    
    
    
# Enviar as métricas do prometheus para o Grafana!

A intenção é mostrar de forma simples de como juntar o prometheus com grafana e criar um painel com as métricas do prometheus.

1 -Entrar no Grafana:

<img width="949" alt="Captura de tela_20230113_160210" src="https://user-images.githubusercontent.com/111398584/212398429-e11beba6-5b18-4bbe-846a-28b5e291f2c9.png">

2- Vá em comfigurações e datasources:

<img width="950" alt="Captura de tela 2023-01-13 160345" src="https://user-images.githubusercontent.com/111398584/212398790-7173968e-644c-419e-a22e-f270112beca3.png">

3-Adicionar um datasource:

<img width="482" alt="Captura de tela_20230113_160817" src="https://user-images.githubusercontent.com/111398584/212399362-a4cd90a8-6783-40f6-b88a-16bfabb835dc.png">

4-Selecionar Prometheus:

<img width="506" alt="image" src="https://user-images.githubusercontent.com/111398584/212399560-eefcb194-0905-40dc-86fd-fe2c92f1821f.png">

5-Inserir a URL do compose do prometheus (http://prometheus:9090)

<img width="398" alt="image" src="https://user-images.githubusercontent.com/111398584/212399789-9a4b3e55-65d2-4f04-b26e-b0d0d2711061.png">

6-Testar e salvar:

<img width="671" alt="image" src="https://user-images.githubusercontent.com/111398584/212399898-63a2d583-842d-4a25-a345-6c2a9ed5340c.png">

7-Vá em criar dashboard:

<img width="197" alt="image" src="https://user-images.githubusercontent.com/111398584/212400190-9f321ce8-94bb-47cc-9837-ec0b18c99ed0.png">

8-Clica em adicionar um painel:

<img width="581" alt="image" src="https://user-images.githubusercontent.com/111398584/212400356-65f0e57e-cb11-4746-9ff1-3814b71fae8b.png">

9-As métricas são escritas onde tem a ceta - no rótulo métrics?

<img width="956" alt="image" src="https://user-images.githubusercontent.com/111398584/212400642-f3d8ec2e-afc9-4956-b987-b1fd5b311feb.png">

10-Seleciona a soma de toas as requisições por job - nesse caso por serviço coletado:

<img width="958" alt="image" src="https://user-images.githubusercontent.com/111398584/212400860-2ead52e1-6e17-4e80-a6ae-eea62bb44177.png">

11-Ao salvar o painel dé um nome a sua escolha:

<img width="444" alt="image" src="https://user-images.githubusercontent.com/111398584/212400996-4715bea7-76e6-4b71-b448-fe9ab0d9abf7.png">

12 - Caso quiser adicionar outro painel é só clicar onde a seta indica:

<img width="952" alt="image" src="https://user-images.githubusercontent.com/111398584/212401374-347b02d1-fa36-4386-b383-b11f35a8efc1.png">

13- Um painel mais personalizado:

<img width="957" alt="metricas" src="https://user-images.githubusercontent.com/111398584/212401476-a317c1b6-4fc7-432d-82a4-ef0b40d676c6.PNG">


E por final o .yml do Grafana que estamos usando no projeto:


            version: '3.1'
    networks:
       n_easybox:
        driver: bridge

    services:

      grafana:
       image: grafana/grafana:7.3.3
       ports:
         - "3000:3000"
       networks:
         - n_easybox
         
 Nota : os componentes estão dentro da mesma rede que é n_easybox.
 
 
