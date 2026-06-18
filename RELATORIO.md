# Design Pattern: Decorator

> **Disciplina:** Engenharia de Software
> **Aluno:** Cauã de Carvalho Paiva
> **Data:** Junho de 2026

---

## Sumário

1. [Introdução](#1-introdução)
2. [Motivação e Problema](#2-motivação-e-problema)
3. [Estrutura e Participantes](#3-estrutura-e-participantes)
4. [Implementação em Python](#4-implementação-em-python)
5. [Vantagens e Desvantagens](#5-vantagens-e-desvantagens)
6. [Relação com os Princípios SOLID](#6-relação-com-os-princípios-solid)
7. [Aplicações no Mundo Real](#7-aplicações-no-mundo-real)
8. [Comparação com Padrões Relacionados](#8-comparação-com-padrões-relacionados)
9. [Conclusão](#9-conclusão)
10. [Referências](#referências)

---

## 1. Introdução

O Decorator é um dos 23 padrões de projeto catalogados pelo grupo Gang of Four (GoF) no livro *Design Patterns: Elements of Reusable Object-Oriented Software*, publicado em 1994. Ele faz parte da categoria dos padrões estruturais e tem como objetivo permitir que novas funcionalidades sejam adicionadas a um objeto de forma dinâmica, sem precisar modificar a classe original.

A ideia central é simples: em vez de criar subclasses para cada combinação possível de comportamento, o Decorator "envolve" o objeto original com camadas adicionais que estendem seu comportamento. Cada camada é independente e pode ser combinada livremente.

> **Definição (GoF):** *"Attach additional responsibilities to an object dynamically. Decorators provide a flexible alternative to subclassing for extending functionality."*

---

## 2. Motivação e Problema

Imagine um jogo de RPG em que o personagem principal é um guerreiro. Durante a aventura, ele pode equipar diferentes itens: espada, escudo, armadura, entre outros. O problema surge quando tentamos modelar isso usando apenas herança.

Para cobrir todas as combinações possíveis de equipamentos, seríamos obrigados a criar uma classe para cada combinação:

- `GuerreiroComEspada`
- `GuerreiroComEscudo`
- `GuerreiroComArmadura`
- `GuerreiroComEspadaEEscudo`
- `GuerreiroComEspadaEArmadura`
- `GuerreiroComEscudoEArmadura`
- `GuerreiroComEspadaEscudoEArmadura`
- *(e assim por diante para cada novo item)*

Isso é o que chamamos de **explosão de subclasses**: quanto mais itens existirem no jogo, mais classes precisaríamos criar. Isso torna o código difícil de manter e praticamente impossível de escalar.

O Decorator resolve esse problema de forma elegante: em vez de criar subclasses, cada equipamento é implementado como um decorador independente. Assim, podemos combinar espada, escudo e armadura dinamicamente, adicionando ou removendo equipamentos em tempo de execução sem modificar a classe do personagem.

---

## 3. Estrutura e Participantes

O padrão Decorator é formado por quatro elementos principais:

| Participante | Responsabilidade |
|---|---|
| **Component** | Interface ou classe abstrata que define o contrato comum entre objetos reais e decoradores. |
| **ConcreteComponent** | O objeto base (o personagem, no nosso caso) que implementa a interface Component. |
| **Decorator** | Classe abstrata que também implementa Component e mantém uma referência para outro Component. Delega as operações para o objeto interno. |
| **ConcreteDecorator** | Cada equipamento concreto (espada, escudo, armadura). Adiciona comportamento específico antes ou depois de delegar ao componente interno. |

---

## 4. Implementação em Python

O exemplo a seguir simula um personagem RPG que pode receber equipamentos dinamicamente. O Decorator permite adicionar espada, escudo e armadura ao personagem sem modificar sua classe original.

### 4.1 Interface e Componente Concreto

Definimos a interface `Personagem` com o método `status()`, que retorna as informações do personagem. O `Guerreiro` é o componente concreto, sem nenhum equipamento inicialmente.

```python
# component.py
from abc import ABC, abstractmethod

class Personagem(ABC):
    @abstractmethod
    def status(self) -> str:
        pass

class Guerreiro(Personagem):
    def status(self) -> str:
        return "Guerreiro"
```

### 4.2 Decorator Base

O `EquipamentoDecorator` também implementa a interface `Personagem` e mantém uma referência para o personagem que está sendo decorado. Por padrão, ele apenas delega a chamada para o objeto interno.

```python
# decorator.py
from component import Personagem

class EquipamentoDecorator(Personagem):
    def __init__(self, personagem: Personagem) -> None:
        self._personagem = personagem

    def status(self) -> str:
        return self._personagem.status()
```

### 4.3 Decoradores Concretos

Cada equipamento é um decorador concreto que estende o `EquipamentoDecorator`. Ao chamar `status()`, cada um adiciona sua própria descrição ao retorno do personagem interno.

```python
# equipamentos.py
from decorator import EquipamentoDecorator

class Espada(EquipamentoDecorator):
    def status(self) -> str:
        return self._personagem.status() + " + Espada"

class Escudo(EquipamentoDecorator):
    def status(self) -> str:
        return self._personagem.status() + " + Escudo"

class Armadura(EquipamentoDecorator):
    def status(self) -> str:
        return self._personagem.status() + " + Armadura"
```

### 4.4 Exemplo de Utilização

```python
# main.py
from component import Guerreiro
from equipamentos import Espada, Escudo, Armadura

# Personagem sem equipamento
personagem = Guerreiro()
print(personagem.status())

# Adicionando espada
personagem = Espada(personagem)
print(personagem.status())

# Adicionando escudo
personagem = Escudo(personagem)
print(personagem.status())

# Adicionando armadura
personagem = Armadura(personagem)
print(personagem.status())
```

### 4.5 Saída Esperada

```
Guerreiro
Guerreiro + Espada
Guerreiro + Espada + Escudo
Guerreiro + Espada + Escudo + Armadura
```

Vale destacar que os equipamentos podem ser combinados em qualquer ordem e a qualquer momento da execução, sem nenhuma mudança na classe `Guerreiro`.

---

## 5. Vantagens e Desvantagens

### 5.1 Vantagens

- **Extensão sem modificação:** novas funcionalidades são adicionadas sem alterar o código existente, respeitando o Princípio Aberto/Fechado (OCP).
- **Composição flexível:** os decoradores podem ser combinados livremente em tempo de execução.
- **Responsabilidade única:** cada decorador tem uma única responsabilidade bem definida.
- **Alternativa à herança:** evita a explosão de subclasses para cobrir todas as combinações possíveis.
- **Transparência para o cliente:** quem usa o objeto não precisa saber se ele está decorado ou não.

### 5.2 Desvantagens

- **Depuração mais difícil:** cadeias longas de decoradores podem dificultar o rastreamento de erros.
- **Identidade do objeto:** um objeto decorado não é igual ao original em comparações por referência.
- **Ordem importa:** a sequência em que os decoradores são aplicados pode afetar o resultado.
- **Muitos objetos pequenos:** um número grande de decoradores pode tornar o design difícil de entender.

---

## 6. Relação com os Princípios SOLID

O Decorator tem relação direta com alguns princípios SOLID, especialmente o SRP e o OCP:

| Princípio | Relação |
|---|---|
| **SRP — Responsabilidade Única** | Cada decorador tem uma única responsabilidade bem definida. |
| **OCP — Aberto/Fechado** | Novas funcionalidades são adicionadas sem modificar o código existente. |
| **LSP — Substituição de Liskov** | Decoradores respeitam o contrato da interface original. |
| **DIP — Inversão de Dependência** | Dependência sobre abstrações, não sobre implementações concretas. |

---

## 7. Aplicações no Mundo Real

O Decorator não é apenas um conceito teórico. Ele aparece em diversas bibliotecas e frameworks amplamente utilizados.

### 7.1 Java I/O Streams

A biblioteca padrão do Java é o exemplo mais clássico. Classes como `InputStream`, `OutputStream` e suas variantes seguem exatamente a estrutura do padrão Decorator, permitindo combinar comportamentos de leitura e escrita de dados.

### 7.2 Django — Middlewares

O framework Django utiliza middlewares como decoradores de requisição e resposta HTTP. Cada middleware envolve o próximo na cadeia, podendo modificar a requisição antes de passá-la adiante ou alterar a resposta antes de retorná-la ao cliente.

### 7.3 Sistemas de Logging

Sistemas de log frequentemente usam o Decorator para adicionar funcionalidades como formatação, filtragem por nível e envio para múltiplos destinos (console, arquivo, rede) de forma composável e independente.

---

## 8. Comparação com Padrões Relacionados

| Padrão | Diferença em relação ao Decorator |
|---|---|
| **Proxy** | Controla o acesso ao objeto; o Decorator adiciona comportamento. |
| **Composite** | Agrega vários objetos-filhos; o Decorator envolve apenas um. |
| **Strategy** | Substitui o algoritmo interno; o Decorator estende o comportamento sem substituir. |
| **Chain of Responsibility** | Passa a requisição até um único handler; o Decorator sempre delega para o próximo. |

---

## 9. Conclusão

O Decorator é um padrão bastante útil quando precisamos adicionar funcionalidades a objetos de forma flexível, sem criar uma hierarquia de classes extensa. No contexto do RPG apresentado neste trabalho, ficou evidente como ele resolve o problema da explosão de subclasses de forma simples e elegante.

A principal lição que o padrão traz é que, em muitas situações, **compor comportamentos é melhor do que herdá-los**. Isso torna o código mais fácil de manter, de testar e de expandir, já que cada decorador é independente e pode ser reutilizado em diferentes combinações.

Em resumo, o Decorator vale a pena sempre que: o número de combinações de comportamentos for grande; as funcionalidades precisarem ser adicionadas ou removidas em tempo de execução; ou quando se quer respeitar o Princípio Aberto/Fechado sem abrir mão da flexibilidade.

---

## Referências

- GAMMA, E. et al. *Design Patterns: Elements of Reusable Object-Oriented Software.* Addison-Wesley, 1994.
- REFACTORING.GURU. *Decorator Pattern.* Disponível em: https://refactoring.guru/design-patterns/decorator
- MARTIN, R. C. *Clean Architecture.* Prentice Hall, 2017.
