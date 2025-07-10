Claro! Aqui está sua **terceira etapa** revisada com mais clareza, fluidez e padronização:

---

# Tutorial Arquitetura Hexagonal - CRUD de Usuários | API + MongoDB (NoSQL) + Kafka (Mensageria)

Aprenda na prática como aplicar a **Arquitetura Hexagonal** em microsserviços utilizando **Java**, **Spring Boot**, **MongoDB** e **Kafka**.

Neste projeto, construiremos um **CRUD de Clientes**, explorando todas as camadas da arquitetura de forma clara e orientada.

---

## 🔁 Etapa 3: Camada de Aplicação - Criação do Use Case

Depois de modelarmos as classes de domínio `Customer` e `Address`, o próximo passo é iniciar a **camada de aplicação**, responsável por orquestrar os **casos de uso** do sistema.

Nesta terceira etapa, nosso foco será a **criação do primeiro caso de uso** da aplicação, com os seguintes objetivos:

-   ✅ **Isolar a lógica do caso de uso da aplicação**
-   ✅ **Garantir testabilidade e clareza da regra de negócio**
-   ✅ **Promover o desacoplamento entre as camadas**

Essa separação permite manter o código mais **limpo, reutilizável e fácil de evoluir**, algo essencial em sistemas baseados em microsserviços.

---

### 📌 O que é um Use Case?

Um **Use Case (caso de uso)** representa uma **ação específica** que o sistema realiza para atender a uma necessidade de negócio. Exemplos comuns incluem:

-   Criar um cliente
-   Buscar um cliente por ID
-   Atualizar os dados de um cliente
-   Excluir um cliente

Na Arquitetura Hexagonal, os use cases atuam como a **ponte entre o domínio e as interfaces externas**, centralizando a lógica de aplicação e mantendo a separação de responsabilidades.

---

## ✏️ Parte 1: Criação do caso de uso "Criar Cliente" (`CreateCustomerUseCase`)

(criar, registrar, inserir)

Vamos começar criando a classe `CreateCustomerUseCase`, responsável por:

-   Receber os dados de entrada (geralmente por DTO ou parâmetros simples);
-   Criar uma instância da entidade `Customer`;
-   Delegar a persistência ao repositório (interface/porta de saída);
-   Retornar uma resposta ou confirmação da operação.

# Camada Application

```
src/
├── application/
│   └── usecase/
│       └── CreateCustomerUseCase.java
```

Vamos lá!

-   Acesse a pasta src/main/java/com/example/hexagonal/application/usecase

-   Dentro de usecase crie a classe CreateCustomerUseCase.java

-   Dentro da classe adicione o método para criar o cliente
    -   O método precisa receber por parâmetro o cliente (customer) e o cep(zipCode). Oberve utilizar o cep para acessar uma outra aplicação (api externa) que nos infrominformará o endereço do cliente.
    -   Dentro do método iremos pegar o endereço do cliente
        -   Precisamos criar a conexão com o microserviço. Contudo não vamos ter acesso direto e sim a comunicação desacoplada, através de portas.
    -   Dentro do método iremos criar o cliente
        -   Precisamos criar a conexão com o banco de dados.Contudo não vamos ter acesso direto e sim a comunicação desacoplada, através de portas.

**Passo 1: Criando o método create e pegando o enedereço**

```java
package com.example.hexagonal.application.usecase;

import com.example.hexagonal.domain.Customer;

public class CreateCustomerUseCase {

    public void create(Customer customer, String zipCode) {
        var address = ???
    }

}

```

Veja que precisamos acessar o serviço que nos dará as informações mediante o envio do cep. Assim precisaríamos nos comunicar de forma desacoplada atravpes de portas.

Claro! Aqui está o texto **melhorado, traduzido e mais didático**, ideal para iniciantes:

---

### ✅ **Passo 2: Criando a porta de saída da aplicação** - interface do serviço (Porta de saída)

Neste passo, vamos criar uma **porta de saída**, ou seja, uma interface que permitirá que a aplicação se comunique com **um serviço externo de busca de endereço via CEP**.

Quando nomeamos essa interface, temos duas abordagens possíveis:

-   `FindAddressByZipCodeOutputPort`: destaca a **ação específica** (buscar por CEP).
-   `AddressLookupOutputPort`: destaca o **papel da interface** (responsável por consultar endereços).

Ambos estão corretos, mas para manter o código mais claro e alinhado às boas práticas, vamos optar por um nome mais simples e semântico:

### 🔹 Nome escolhido: `AddressLookupOutputPort`

Esse nome:

-   É fácil de entender;
-   Mostra que a interface é uma **porta de saída** (`OutputPort`);
-   Reflete bem o propósito: **buscar um endereço** externo com base no CEP.

---

Acesse src/main/java/com/example/hexagonal/application/port/output

Crie o arquivo `AddressLookupOutputPort.java`

```java
package com.example.hexagonal.application.port.output;

import com.example.hexagonal.domain.Address;

public interface AddressLookupOutputPort {

    Address findByZipCode(String zipcode);

}

```

Uma **interface** em Java é um **contrato** que define **métodos que uma classe deve implementar**, mas **sem fornecer a implementação** desses métodos.

-   A interface **declara** que quem implementá-la deve fornecer um método `findByZipCode`.
-   Serve para **desacoplar** a lógica da aplicação da forma como os dados são obtidos (ex: API externa).
-   Na arquitetura hexagonal, isso permite trocar a implementação (ex: mudar a API) sem alterar a lógica principal.

👉 **Resumo**: Interface é uma forma de dizer “_alguém vai fazer isso_, mas eu não me importo _como_”.

Agora que temos uma forma de desacoplar o usecase, vamos implementá-lo.

### ✅ **Passo 3: Adicionando a porta no nosso caso de uso**

Precisamos agora adicionar o atributo addressLookupOutputPort.
Depois adicionamos o serviço de buscar o cep por injeção de dependência no construtor da aplicação.
Veja que não usaremos os decorator do famework springboot para injetar a dependência do serviço de buscar o cep pois iremos desacoplar o usecase das tecnologias.
Depois com o enedereço recebido, podemos definir o endereço do cliente

```java
package com.example.hexagonal.application.usecase;

import com.example.hexagonal.application.port.output.AddressLookupOutputPort;
import com.example.hexagonal.domain.Customer;

public class CreateCustomerUseCase {

    private final AddressLookupOutputPort addressLookupOutputPort;

    public CreateCustomerUseCase(AddressLookupOutputPort addressLookupOutputPort) {
        this.addressLookupOutputPort = addressLookupOutputPort;
    }

    public void create(Customer customer, String zipCode) {
        var address = addressLookupOutputPort.findByZipCode(zipCode);
        customer.setAddress(address);
    }

}

```

Veja que via porta de saída temos como acessar o serviço da interface. Claro que depois teremos que criar uam classe concreta que implementará a interface para realmente acessar a api externa.

### ✅ **Passo 4: Inserir o cliente**

Precisamos agora adicionar o cliente, sem ter acesso a base dados diretamente. Assim precisamos criar mais um porta de saída para acessar o banco de dados.

**Criar a interface do repositório (Porta de saída)**

Acesse src/main/java/com/example/hexagonal/application/port/output

Crie o arquivo `CustomerPersistenceOutputPort.java`

```java
package com.example.hexagonal.application.port.output;

import com.example.hexagonal.domain.Customer;

public interface CustomerPersistenceOutputPort {

    void save(Customer customer);

}
```

Podemos agora usar a porta de saída `CustomerPersistenceOutputPort`

```java
package com.example.hexagonal.application.usecase;

import com.example.hexagonal.application.port.output.AddressLookupOutputPort;
import com.example.hexagonal.application.port.output.CustomerPersistenceOutputPort;
import com.example.hexagonal.domain.Customer;

public class CreateCustomerUseCase {

    private final AddressLookupOutputPort addressLookupOutputPort;
    private final CustomerPersistenceOutputPort customerPersistenceOutputPort;

    public CreateCustomerUseCase(AddressLookupOutputPort addressLookupOutputPort,
            CustomerPersistenceOutputPort customerPersistenceOutputPort) {
        this.addressLookupOutputPort = addressLookupOutputPort;
        this.customerPersistenceOutputPort = customerPersistenceOutputPort;
    }

    public void create(Customer customer, String zipCode) {
        var address = addressLookupOutputPort.findByZipCode(zipCode);
        customer.setAddress(address);
        customerPersistenceOutputPort.save(customer);
    }

}

```

## Estudo complemenetar:

---

## Como dar nomes para as portas que se comunicam com o mundo externo?

Na arquitetura hexagonal, quando sua aplicação precisa conversar com algo **fora dela** — como um serviço externo, banco de dados ou sistema de mensagens — usamos **interfaces chamadas de portas de saída (output ports)**.

Por exemplo:

-   Queremos buscar o endereço de um cliente usando o CEP num serviço externo.
-   Essa comunicação é uma **porta de saída** porque nossa aplicação está **saindo para buscar informação**.

---

## Exemplos de nomes para essa porta de saída:

-   `FindAddressByZipCodeOutputPort`
-   `AddressLookupOutputPort`
-   `AddressLookupGateway`
-   `AddressLookupAdapter`
-   `AddressLookupService`

---

## Por que escolher `AddressLookupOutputPort`?

-   Ele deixa claro que é uma **porta de saída** (OutputPort).
-   Mostra o propósito: **buscar um endereço** (Address Lookup).
-   Facilita a organização do código, ajudando a identificar que essa interface é uma conexão com algo externo.

---

         |

obs.: A camada **domain** deve ser independente de qualquer tecnologia externa: ela não pode conter dependências de frameworks, nem ser acessada diretamente por camadas externas sem passar pelas interfaces (portas) da aplicação. Veja que importamos os getter e setter na mão e não utilizamos a tecnologia lombok do framework Spring.

## ✏️ Parte 2: Criando o Teste de Integração do Use Case

Agora que temos o `CreateCustomerUseCase` implementado, vamos criar um **teste de integração simples** para garantir que o caso de uso funciona corretamente, integrando a busca de endereço e a persistência do cliente.

---

### ✅ Objetivo do teste:

-   Simular a chamada do método `create()`.
-   Verificar se o endereço foi adicionado ao cliente.
-   Verificar se o cliente foi salvo corretamente.

---

### 📁 Local:

Crie o arquivo em:
`src/test/java/com/example/hexagonal/CreateCustomerUseCaseIntegrationTest.java`

---

### 🧪 Código do teste:

```java
package com.example.hexagonal;

import static org.junit.jupiter.api.Assertions.assertEquals;
import static org.junit.jupiter.api.Assertions.assertNotNull;

import org.junit.jupiter.api.Test;

import com.example.hexagonal.application.port.output.AddressLookupOutputPort;
import com.example.hexagonal.application.port.output.CustomerPersistenceOutputPort;
import com.example.hexagonal.application.usecase.CreateCustomerUseCase;
import com.example.hexagonal.domain.Address;
import com.example.hexagonal.domain.Customer;

public class CreateCustomerUseCaseIntegrationTest {

    @Test
    void deveCriarClienteComEndereco() {
        // // CustomerPersistenceOutputPort persistence = new InMemoryCustomerPersistencePort();
        // Criando a implementação em memória da porta de persistência
        // para armazenar o cliente durante o teste
        InMemoryCustomerPersistencePort persistence = new InMemoryCustomerPersistencePort();

        // Criando a implementação fake da porta de busca de endereço,
        // que sempre retorna um endereço fixo para o teste
        AddressLookupOutputPort addressService = new FakeAddressLookupOutputPort();

        // Criando o caso de uso com as implementações das portas
        CreateCustomerUseCase useCase = new CreateCustomerUseCase(addressService, persistence);

        // Criando o cliente com nome e CPF
        Customer customer = new Customer();
        customer.setName("Maria");
        customer.setCpf("12345678900");

        // Executando o caso de uso para criar o cliente com endereço pelo CEP
        useCase.create(customer, "12345678");

        // Imprimindo dados para verificar manualmente (opcional)
        System.out.println("Name " + persistence.getSavedCustomer().getName());
        System.out.println("Rua " + persistence.getSavedCustomer().getAddress().getStreet());

        // Verificando se o cliente foi salvo corretamente (não nulo)
        assertNotNull(persistence.getSavedCustomer());

        // Verificando se o nome do cliente está correto
        assertEquals("Maria", persistence.getSavedCustomer().getName());

        // Verificando se o endereço atribuído ao cliente é o esperado
        assertEquals("Rua Teste", persistence.getSavedCustomer().getAddress().getStreet());
    }

    // Implementação em memória da porta de persistência do cliente,
    // que guarda o cliente internamente para podermos verificar depois
    static class InMemoryCustomerPersistencePort implements CustomerPersistenceOutputPort {
        private Customer savedCustomer;

        @Override
        public void save(Customer customer) {
            this.savedCustomer = customer;
        }

        // Método para recuperar o cliente salvo e verificar no teste
        public Customer getSavedCustomer() {
            return savedCustomer;
        }
    }

    // Implementação fake da porta de busca de endereço que sempre
    // retorna um endereço fixo, simulando uma busca externa via CEP
    static class FakeAddressLookupOutputPort implements AddressLookupOutputPort {
        @Override
        public Address findByZipCode(String zipcode) {
            return new Address("Rua Teste", "Cidade Teste", "Estado Teste");
        }
    }
}
```

No terminal, execute na raiz do projeto:

```bash
mvn test
```

Para rodar somente o teste da classe AddressTest.java com Maven, use o seguinte comando:

```bash
mvn -Dtest=CreateCustomerUseCaseIntegrationTest test
```

---

## Explicação rápida do teste de integração

Este teste verifica o funcionamento do caso de uso **CreateCustomerUseCase**, integrando duas portas de saída simuladas (fake):

-   **InMemoryCustomerPersistencePort**: armazena o cliente "em memória" para que possamos verificar se o cliente foi salvo.
-   **FakeAddressLookupOutputPort**: simula a busca de endereço externo pelo CEP, retornando um endereço fixo para o teste.

No teste:

1. Criamos um cliente com nome e CPF.
2. Chamamos o método `create` do caso de uso, passando o cliente e o CEP.
3. O caso de uso obtém o endereço via `FakeAddressLookupOutputPort` e define no cliente.
4. Salva o cliente no `InMemoryCustomerPersistencePort`.
5. Verificamos se o cliente foi salvo (`assertNotNull`).
6. Verificamos se o nome do cliente está correto.
7. Verificamos se o endereço do cliente foi corretamente atribuído.

---

### Por que fazer assim?

-   Não dependemos de banco de dados real ou serviços externos.
-   O teste é rápido e confiável.
-   Validamos a integração entre a lógica do caso de uso e as portas de saída.

---

---

Você pode ir além, criando um teste com Spring Boot, simulando a infraestrutura real (ex: MongoDB + serviço externo).

## Dúvidas sobre Nomeclaturas:

## Explicando outras portas do código:

| Nome em inglês             | Tradução / Significado                                                  |
| -------------------------- | ----------------------------------------------------------------------- |
| `AddressLookupService`     | Serviço que pesquisa endereço externo                                   |
| `CustomerPersistencePort`  | Porta para persistência (armazenamento) do cliente                      |
| `CpfValidationMessagePort` | Porta para enviar mensagem de validação do CPF (por exemplo, via Kafka) |

---

## Tabela comparativa de nomes:

| **Nome possível**                | **Nome Sugerido**          | **O que faz**                                   | **Vantagem do nome sugerido**                                      |
| -------------------------------- | -------------------------- | ----------------------------------------------- | ------------------------------------------------------------------ |
| `FindAddressByZipCodeOutputPort` | `AddressLookupService`     | Busca endereço via CEP em serviço externo       | ✅ Nome curto e fácil de entender<br>✅ Mostra a função claramente |
| `InsertCustomerOutputPort`       | `CustomerPersistencePort`  | Salva dados do cliente no banco                 | ✅ Nome genérico que pode ser usado para várias operações          |
| `SendCpfForValidationOutputPort` | `CpfValidationMessagePort` | Envia CPF para validação via mensageria (Kafka) | ✅ Nome claro que indica uso de mensagens para validação           |

Observações:
Os nomes originais focavam mais na ação específica (Find, Insert, Send).
Os nomes sugeridos focam mais no papel da interface dentro da aplicação — o "o que ela representa", não apenas o que ela faz.
Isso melhora a leitura, manutenção e testes, além de seguir melhores práticas de arquitetura limpa e hexagonal.

### 📌 Próximos passos:

4. **Implementar o Adapter (porta de saída)**

    - Implementação concreta do repositório usando MongoDB.

5. **Criar o Controller (porta de entrada)**

    - Para expor o endpoint REST e permitir a criação de clientes via HTTP.

6. **Adicionar testes unitários para o use case.**

---

Se quiser, posso escrever a estrutura da classe `CreateCustomerUseCase` para você com exemplos. Deseja isso?

https://github.com/DaniloArantesSilva/hexagonal-architecture
