# Spring Batch - Processamento de Dados em Lote

Projeto de aprendizado focado em processar grandes volumes de dados de forma eficiente usando Spring Batch. Ideal para trabalhos recorrentes, migrações de dados e relatórios em lote.

## 📌 O que é Spring Batch?

Spring Batch é um framework leve para processar grandes volumes de dados em lotes (batches). É perfeito para:
- Importação/exportação de dados
- Processamento de logs
- Transformação de dados
- Relatórios agendados
- Migrações

## 🛠️ Tecnologias

- **Java 17**
- **Spring Boot 4.0.4**
- **Spring Batch** - Processamento em lote
- **H2 Database** - Armazenamento
- **Maven**

## 🎯 Caso de Uso: Sistema de Revendedora de Carros

O projeto implementa um cenário real: processar dados de vendas de uma revendedora de carros, aplicar regras de negócio e salvar os resultados.

**Fluxo:**
```
CSV/DB Input → Spring Batch Job → Processamento → Database Output
```

## 🚀 Como Executar

### Pré-requisitos
- Java 17+
- Maven 3.6+

### Passos

```bash
# 1. Clone
git clone https://github.com/lutheone/spring-batch.git
cd spring-batch

# 2. Execute
mvn spring-boot:run

# 3. Job será executado automaticamente no startup
# Ou acione via endpoint HTTP
curl http://localhost:8080/api/batch/execute
```

## 🔄 Componentes do Batch

### 1. **Reader** (Leitor)
Lê dados de uma fonte (arquivo, banco, API):
```java
@Bean
public FlatFileItemReader<CarDealerData> reader() {
    return new FlatFileItemReaderBuilder<CarDealerData>()
        .name("carReader")
        .resource(new ClassPathResource("cars.csv"))
        .delimited()
        .names("id", "modelo", "preco", "estoque")
        .targetType(CarDealerData.class)
        .build();
}
```

### 2. **Processor** (Processador)
Aplica lógica de negócio aos dados:
```java
@Component
public class CarPriceProcessor implements ItemProcessor<CarDealerData, CarDealerData> {
    @Override
    public CarDealerData process(CarDealerData car) {
        // Aplica desconto se estoque > 5 unidades
        if (car.getEstoque() > 5) {
            car.setPreco(car.getPreco() * 0.9);
        }
        return car;
    }
}
```

### 3. **Writer** (Escritor)
Salva dados processados no destino:
```java
@Bean
public RepositoryItemWriter<CarDealerData> writer() {
    RepositoryItemWriter<CarDealerData> writer = new RepositoryItemWriter<>();
    writer.setRepository(carRepository);
    writer.setMethodName("save");
    return writer;
}
```

## 📊 Estrutura do Job

```java
@Bean
public Job carDealerJob(JobRepository jobRepository, Step step) {
    return new JobBuilder("carDealerJob", jobRepository)
        .start(step)
        .build();
}

@Bean
public Step step(JobRepository jobRepository, 
                PlatformTransactionManager tm,
                FlatFileItemReader<CarDealerData> reader,
                CarPriceProcessor processor,
                RepositoryItemWriter<CarDealerData> writer) {
    return new StepBuilder("step", jobRepository)
        .<CarDealerData, CarDealerData>chunk(10)  // Processa 10 registros por vez
        .reader(reader)
        .processor(processor)
        .writer(writer)
        .transactionManager(tm)
        .build();
}
```

## 🗄️ Banco de Dados

Spring Batch usa tabelas próprias para rastrear jobs:
- `BATCH_JOB_INSTANCE` - Instâncias de jobs
- `BATCH_JOB_EXECUTION` - Execuções de jobs
- `BATCH_STEP_EXECUTION` - Execuções de steps

Você pode consultar o status:
```bash
SELECT * FROM BATCH_JOB_EXECUTION;
SELECT * FROM BATCH_STEP_EXECUTION;
```

## 📈 Monitoramento

Acesse o console H2 para visualizar execuções:
```
http://localhost:8080/h2-console
JDBC URL: jdbc:h2:mem:testdb
```

## 💡 Casos de Uso Práticos

1. **Importação de Dados**
   - Ler CSV, validar, salvar no BD

2. **Processamento de Vendas**
   - Calcular comissões, gerar relatórios

3. **Sincronização de Sistemas**
   - Ler dados de um sistema, transformar, enviar para outro

4. **Limpeza de Dados**
   - Remover duplicatas, normalizar informações

5. **Geração de Relatórios**
   - Processar dados e exportar em formato específico

## 🧪 Configurações por Ambiente

### Desenvolvimento
```properties
spring.batch.job.enabled=true
spring.batch.jdbc.initialize-database=always
```

### Produção
```properties
spring.batch.job.enabled=false
# Controlar jobs via scheduler/API
```

## 🔧 Troubleshooting

**Job não executa?**
```bash
# Verifique se spring.batch.job.enabled=true
# Ou acione manualmente via JobLauncher
```

**Erro ao ler arquivo?**
```bash
# Certifique-se que o arquivo está em src/main/resources
# Verifique delimitadores (CSV, TSV, etc)
```

## 📚 Conceitos Chave

| Conceito | Descrição |
|----------|----------|
| **Job** | Trabalho completo a ser executado |
| **Step** | Etapa de um job (read-process-write) |
| **Chunk** | Quantidade de registros processados por vez |
| **ItemReader** | Lê dados da fonte |
| **ItemProcessor** | Transforma/valida dados |
| **ItemWriter** | Persiste dados no destino |
| **Tasklet** | Tarefa simples sem itemização |