Claro! Aqui está sua **terceira etapa** revisada com mais clareza, fluidez e padronização:

---

# Tutorial Arquitetura Hexagonal - CRUD de Usuários | API + MongoDB (NoSQL) + Kafka (Mensageria)

Aprenda na prática como aplicar a **Arquitetura Hexagonal** em microsserviços utilizando **Java**, **Spring Boot**, **MongoDB** e **Kafka**.

Neste projeto, construiremos um **CRUD de Clientes**, explorando todas as camadas da arquitetura de forma clara e orientada.

---

## 🔁 Etapa 4: Camada de infrastructure — Criação do Adaptador para Consulta de Endereço via CEP (API Externa - ViaCEP)

Após modelarmos as classes de domínio `Customer` e `Address`, e o caso de uso `CreateCustomerUseCase`, nosso próximo passo é criar o **adaptador da porta de saída** que irá implementar a interface `AddressLookupOutputPort`.

Este adaptador será responsável por buscar o endereço a partir de um CEP utilizando a **API pública do ViaCEP**.

---

### 🎯 Objetivo da etapa:

Criar um **cliente HTTP (adaptador)** para:

-   ✅ **Isolar a lógica de integração externa**, mantendo o domínio e os casos de uso independentes.
-   ✅ **Permitir testes simples e previsíveis**, substituindo a chamada real por mocks ou fakes durante os testes.
-   ✅ **Desacoplar a aplicação da tecnologia de infraestrutura**, seguindo os princípios da arquitetura hexagonal.

---

### ⚠️ Importante:

Como a resposta da API do ViaCEP vem em **formato JSON**, precisaremos criar uma classe auxiliar chamada `ViaCepResponseDTO`, que servirá como **modelo temporário para receber a resposta da API** e convertê-la em um objeto do nosso domínio: `Address`.

---

Excelente pergunta para o tutorial! Aqui está uma explicação clara, objetiva e didática:

---

### 📌 O que é um adaptador?

Um **adaptador** é uma **classe que conecta a aplicação ao mundo externo**.

Na arquitetura hexagonal, ele é responsável por **implementar uma porta** (interface) definida pela aplicação. O adaptador sabe **como** se comunicar com outras tecnologias, como:

-   APIs externas (ex: ViaCEP)
-   Banco de dados
-   Fila de mensagens (ex: Kafka)
-   Interfaces web (ex: controllers)

---

### 🔁 Por que usar adaptadores?

-   ✅ **Desacoplamento**: sua aplicação não depende diretamente de bibliotecas ou serviços externos.
-   ✅ **Facilidade de testes**: você pode simular (mockar) os adaptadores nos testes.
-   ✅ **Reusabilidade**: você pode trocar a implementação sem mudar a regra de negócio.

---

### 🧱 Exemplo no seu projeto

| Componente                | Papel                                             |
| ------------------------- | ------------------------------------------------- |
| `AddressLookupOutputPort` | Porta de saída (interface da aplicação)           |
| `ViaCepAddressAdapter`    | Adaptador (implementa a interface e acessa a API) |

---

---

## ✏️ Parte 1: Criação do adaptador de saída do cliente (`ViaCepAddressAdapter`)

> Nesta etapa, já tinhamos uma **porta de saída** chamada `AddressLookupOutputPort`, responsável pela interface de entrada na aplicação
>
> Sua implementação concreta, chamada `ViaCepAddressAdapter`, realizará a **consulta HTTP à API pública do ViaCEP**, acessando a URL:
>
> ```
> https://viacep.com.br/ws/{cep}/json
> ```
>
> Exemplo real:
> [https://viacep.com.br/ws/28300000/json](https://viacep.com.br/ws/28300000/json)
>
> A resposta da API retorna os dados do endereço no formato JSON, que transformamos em um objeto `Address` do domínio.
>
> Essa comunicação externa é encapsulada dentro do adaptador, garantindo que o restante da aplicação continue desacoplado da lógica HTTP.

Vamos lá!

Possíveis nomes:
| Nome da Implementação | Motivo |
| ------------------------------ | -------------------------------------------------------------------------------- |
| `AddressLookupRestAdapter` | Se for uma chamada via REST/HTTP para um serviço externo |
| `ViaCepAddressAdapter` | Se for uma implementação específica que usa a API ViaCEP |
| `HttpAddressLookupAdapter` | Se a busca for feita via cliente HTTP genérico |
| `AddressLookupExternalService` | Se quiser indicar que a implementação se conecta a um serviço externo |
| `AddressLookupRestClient` | Comum quando a implementação usa um client HTTP (como RestTemplate ou WebClient) |

Utilizaremos `ViaCepAddressAdapter`

-   Acesse a pasta src/main/java/com/example/hexagonal/infrastructure/adapter/output/client

-   Dentro de client crie a classe ViaCepAddressAdapter.java

### 📌 O que essa classe fará?

Ela será responsável por **buscar o endereço do cliente** com base no CEP, consultando uma **API externa (ViaCEP)**. Essa consulta será feita **através de uma porta de saída**, respeitando o princípio de desacoplamento da arquitetura hexagonal.

---

### 🧱 O que implementar no método:

-   O método precisa receber:

    -   Um **cliente (`Customer`)**
    -   Um **CEP (`zipCode`)**

-   Dentro do método:

    -   Vamos **usar o CEP para buscar o endereço** em uma **aplicação externa** (o microserviço do ViaCEP).
    -   Essa comunicação será feita através da **porta `AddressLookupOutputPort`**, que será implementada pelo adaptador `ViaCepAddressAdapter`.

-   Após buscar o endereço:

    -   O endereço será **associado ao cliente**
    -   E em seguida, o cliente será salvo no banco de dados.

> 💡 Importante: **não acessamos diretamente o banco de dados** nem o serviço externo. Tudo será feito **por meio das portas** (interfaces), mantendo o domínio da aplicação isolado e testável.

---

**Passo 1: Criando o adapatdor**

O adaptador é implementação concreta da porta de saída `AddressLookupOutputPort` da camada application

Mas para utilizarmos o adaptador, precisaríamos também de criar uma classe ViaCepResponseDTO.java que irá converter a resposta em Address.

A API do ViaCEP retorna os dados de endereço em formato JSON. Para que o Java consiga entender e utilizar essa resposta, precisamos de uma classe que represente essa estrutura.

Essa classe é chamada de DTO (Data Transfer Object). Ela serve como um "modelo temporário" para receber os dados da API e depois convertê-los em um objeto do nosso domínio (Address).

Onde incluir a classe ViaCepResponse.java?
Como ela é usada apenas para representar a resposta da API externa (ViaCEP), a melhor prática é colocá-la junto com o adaptador que consome essa API.

src/main/java/com/example/hexagonal/infrastructure/adapter/output/client/ViaCepResponse.java

Assim crie o classe ViaCepResponse.java em client

```lua
└── infrastructure
    └── adapter
        └── output
            └── client
                ├── ViaCepAddressAdapter.java   ✅ adaptador da porta
                └── ViaCepResponse.java         ✅ resposta da API ViaCEP

```

```java
package com.example.hexagonal.infrastructure.adapter.output.client;

public class ViaCepResponseDTO {

    private String logradouro;
    private String localidade;
    private String uf;

    public String getLogradouro() {
        return logradouro;
    }

    public void setLogradouro(String logradouro) {
        this.logradouro = logradouro;
    }

    public String getLocalidade() {
        return localidade;
    }

    public void setLocalidade(String localidade) {
        this.localidade = localidade;
    }

    public String getUf() {
        return uf;
    }

    public void setUf(String uf) {
        this.uf = uf;
    }
}

```

Criamos a classe ViaCepResponseDTO para deserializar a resposta da API externa (ViaCEP) e transformá-la em um Address, que é a classe usada dentro da nossa aplicação.

Com a classe de conversão pronta, podemo criar a classe ViaCepAddressAdapter

```java
package com.example.hexagonal.infrastructure.adapter.output.client;

import org.springframework.stereotype.Component;
import org.springframework.web.client.RestTemplate;

import com.example.hexagonal.application.port.output.AddressLookupOutputPort;
import com.example.hexagonal.domain.Address;

@Component
public class ViaCepAddressAdapter implements AddressLookupOutputPort {

    private final RestTemplate restTemplate = new RestTemplate();

    @Override
    public Address findByZipCode(String zipcode) {
        String url = "https://viacep.com.br/ws/" + zipcode + "/json";
        ViaCepResponseDTO response = restTemplate.getForObject(url, ViaCepResponseDTO.class);

        return new Address(
            response.getLogradouro(),   // street
            response.getLocalidade(),   // city
            response.getUf()            // state
        );
    }
}

```

Isso mantém a responsabilidade bem isolada: infraestrutura lida com o mundo externo, enquanto o domínio e a aplicação continuam limpos e desacoplados.

---

| Termo técnico                  | Significado claro no tutorial                                  |
| ------------------------------ | -------------------------------------------------------------- |
| API externa (external service) | Serviço fora da aplicação que fornece dados via internet       |
| ViaCEP                         | Serviço gratuito de consulta de endereço via CEP               |
| Requisição HTTP GET            | Pedido que busca informações em uma URL                        |
| Porta de saída (OutputPort)    | Interface que define como a aplicação se comunica para fora    |
| Adaptador (Adapter)            | Classe que implementa uma porta de saída e faz o trabalho real |

---

---

## ✏️ Parte 2: Criar um teste de integração real (chamando a API)

Agora que temos o `CreateCustomerUseCase` implementado, vamos criar um **teste de integração simples** para garantir que o caso de uso funciona corretamente, integrando a busca de endereço e a persistência do cliente.

---

### ✅ Objetivo do teste:

-   Cria a instância do adaptador.
-   Executar o método findByZipCode do adaptador passando o cep
-   Confirmar se os dados foram corretos

---

### 📁 Local:

Crie o arquivo em:
`src/test/java/com/example/hexagonal/infrastructure/adapter/output/client/ViaCepAddressAdapterTest.java
`

---

### 🧪 Código do teste:

```java
package com.example.hexagonal;

import static org.junit.jupiter.api.Assertions.assertEquals;
import static org.junit.jupiter.api.Assertions.assertNotNull;

import org.junit.jupiter.api.Test;

import com.example.hexagonal.domain.Address;
import com.example.hexagonal.infrastructure.adapter.output.client.ViaCepAddressAdapter;

public class ViaCepAddressAdapterTest {

    @Test
    void deveBuscarEnderecoRealDaApiViaCep() {
        // Arrange
        ViaCepAddressAdapter adapter = new ViaCepAddressAdapter();

        // Act
        Address address = adapter.findByZipCode("28300000"); // CEP válido da Praça da Sé - SP

        System.out.println("Estado: " + address.getCity());
        System.out.println("UF: " + address.getState());

        // Assert
        assertNotNull(address);
        assertEquals("Itaperuna", address.getCity());
        assertEquals("RJ", address.getState());
        // logradouro pode variar, então não fixamos aqui
    }
}


```

No terminal, execute na raiz do projeto:

```bash
mvn test
```

Para rodar somente o teste da classe AddressTest.java com Maven, use o seguinte comando:

```bash
mvn -Dtest=ViaCepAddressAdapterTest test
```

---

## Explicação rápida do teste de integração

Este teste verifica se o adaptador `ViaCepAddressAdapter` consegue buscar um endereço real usando a API pública do ViaCEP.

-   **Arrange:** Cria a instância do adaptador.
-   **Act:** Chama o método `findByZipCode` com um CEP válido.
-   **Assert:** Confirma que o endereço retornado não é nulo e que a cidade e estado correspondem ao esperado.

Esse teste valida a integração com o serviço externo ViaCEP, garantindo que a comunicação e o mapeamento do JSON para o objeto `Address` funcionem corretamente.

---

**Observações:** Use com cuidado: se a API do ViaCEP estiver fora do ar, o teste vai falhar.

### 📌 Próximos passos:

4. **Implementar o Adapter (porta de saída)**

    - Implementação concreta do repositório usando MongoDB.

5. **Criar o Controller (porta de entrada)**

    - Para expor o endpoint REST e permitir a criação de clientes via HTTP.

6. **Adicionar testes unitários para o use case.**

---

Se quiser, posso escrever a estrutura da classe `CreateCustomerUseCase` para você com exemplos. Deseja isso?

https://github.com/DaniloArantesSilva/hexagonal-architecture
