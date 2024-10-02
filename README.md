
# ponderada_semana_8_OpenTelemetry

Esse repositório contém o código de exemplo e a documentação de execução para a implementação de OpenTelemetry em uma aplicação Go, com integração a plataforma SigNoz. A implementação segue o tutorial disponível no artigo ["Complete Guide to Implementing OpenTelemetry in Go Applications"](https://signoz.io/blog/implementing-opentelemetry-in-go-applications/).

## Índice

- [ponderada\_semana\_8\_OpenTelemetry](#ponderada_semana_8_opentelemetry)
  - [Índice](#índice)
  - [Descrição](#descrição)
  - [Requisitos](#requisitos)
  - [Configuração do Ambiente](#configuração-do-ambiente)
  - [Execução dos Exemplos](#execução-dos-exemplos)
  - [Ampliando Comentários do Código](#ampliando-comentários-do-código)
    - [1. **Configuração do Tracer:**](#1-configuração-do-tracer)
    - [2. **Instrumentação da Aplicação com Middleware:**](#2-instrumentação-da-aplicação-com-middleware)
    - [3. **Coleta de Métricas Personalizadas:**](#3-coleta-de-métricas-personalizadas)
  - [Conclusão](#conclusão)

## Descrição

O OpenTelemetry é um projeto open-source cujo o objetivo é padronizar a geração e a coleta de dados de telemetria, incluindo métricas e rastreamentos. Essa implementação demonstra como instrumentar uma aplicação Go para coletar essas informações e visualizar no SigNoz.

## Requisitos

- Go 1.18 ou superior
- Docker (necessário para rodar o SigNoz)
- SigNoz instalado localmente para visualizar os dados de telemetria

## Configuração do Ambiente

1. **Instalação do SigNoz:**

   ```bash
   git clone -b main https://github.com/SigNoz/signoz.git
   cd signoz/deploy/
   ./install.sh
   ```

2. **Configuração das Variáveis de Ambiente:**
   No arquivo `main.go`, configure as variáveis de ambiente que serão usadas para conectar o OpenTelemetry ao SigNoz:

   ```bash
   export SERVICE_NAME=goGinApp
   export OTEL_EXPORTER_OTLP_ENDPOINT=localhost:4317
   export INSECURE_MODE=true
   ```

## Execução dos Exemplos

Após a configuração do ambiente, execute os seguintes passos para rodar os exemplos do tutorial:

1. **Clonar o repositório de exemplo:**

   ```bash
   git clone https://github.com/SigNoz/sample-golang-app.git
   cd sample-golang-app
   ```

2. **Executar a aplicação Go instrumentada:**

   ```bash
   go run main.go
   ```

3. **Interagir com a aplicação para gerar dados de monitoramento:**
   Utilize a URL `http://localhost:8090/books` e faça várias requisições para gerar carga e coletar dados de telemetria.


4. **Visualizar os dados no SigNoz:**
   Acesse a interface do SigNoz em `http://localhost:3301/` para visualizar os dados da aplicação Go monitorada.

## Ampliando Comentários do Código

### 1. **Configuração do Tracer:**
   O tracer é responsável por capturar os spans e coletar as informações de rastreamento. A função `initTracer()` é onde o tracer é configurado, utilizando o SDK do OpenTelemetry.

   ```go
   func initTracer() func(context.Context) error {
       ...
       exporter, err := otlptrace.New(
           context.Background(),
           otlptracegrpc.NewClient(
               otlptracegrpc.WithInsecure(),
               otlptracegrpc.WithEndpoint(collectorURL),
           ),
       )
       ...
   }
   ```

   - A função `initTracer` inicializa o tracer configurando o exportador OTLP que envia dados de rastreamento para o SigNoz.
   - `WithInsecure` é utilizado no exemplo para simplificar a conexão sem TLS.

### 2. **Instrumentação da Aplicação com Middleware:**
   A aplicação Go usa o Gin como framework, e o OpenTelemetry fornece um middleware para instrumentar automaticamente as requisições HTTP.

   ```go
   r := gin.Default()
   r.Use(otelgin.Middleware(serviceName))
   ```

   - O middleware `otelgin.Middleware` injeta spans em cada requisição recebida pela aplicação, permitindo que o OpenTelemetry capture o início e o fim de cada transação.

### 3. **Coleta de Métricas Personalizadas:**
   Para adicionar métricas ou eventos personalizados, o OpenTelemetry permite a criação de spans com atributos adicionais.

   ```go
   span := trace.SpanFromContext(c.Request.Context())
   span.SetAttributes(attribute.String("controller", "books"))
   ```

   - Esse código adiciona um atributo personalizado `controller="books"` ao span atual, facilitando a análise dos rastreamentos no SigNoz.

## Conclusão

A implementação do OpenTelemetry em uma aplicação Go, como demonstrado neste tutorial, permite melhorar significativamente a observabilidade e o monitoramento de serviços distribuídos. Com a integração ao SigNoz, uma poderosa ferramenta open-source, é possível visualizar e analisar métricas e rastreamentos, facilitando a identificação de problemas de desempenho e a otimização dos serviços. Ao seguir as etapas de configuração, instrumentação e análise, é possível garantir que sua aplicação atenda aos requisitos de monitoramento e mantenha um desempenho eficiente em ambientes de produção. A adoção de OpenTelemetry, especialmente em conjunto com uma ferramenta como SigNoz, é uma boa opção para times que buscam maior controle e visibilidade sobre suas aplicações.
