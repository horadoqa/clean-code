# Clean Code

**Clean Code** (Código Limpo) é um conjunto de práticas para escrever código **fácil de ler, entender, testar e manter**. A ideia principal não é simplesmente fazer o código funcionar, mas fazer com que outra pessoa — ou você mesmo daqui a seis meses — consiga entender o que ele faz sem precisar “decifrá-lo”.

Em Python, alguns princípios são especialmente importantes.

## 1. Nomes claros e significativos

Evite nomes genéricos como `x`, `data`, `a` ou `func1`.

**Ruim:**

```python
def calc(x, y):
    return x * y * 0.1
```

O código funciona, mas não deixa claro o que está sendo calculado.

**Melhor:**

```python
def calculate_commission(sales_amount, commission_rate):
    return sales_amount * commission_rate
```

Agora o próprio código explica sua intenção.

> **Regra prática:** se você precisa de um comentário para explicar o significado de uma variável, talvez o nome dela possa ser melhor.

---

## 2. Funções pequenas e com uma única responsabilidade

Uma função deve, idealmente, fazer **uma coisa bem definida**.

**Ruim:**

```python
def process_order(order):
    total = sum(item["price"] for item in order["items"])

    if order["customer"]["type"] == "premium":
        total *= 0.9

    print(f"Total: R$ {total:.2f}")

    with open("orders.txt", "a") as file:
        file.write(f"{order['id']}: {total}\n")

    return total
```

Essa função está fazendo várias coisas:

1. Calculando o total.
2. Aplicando desconto.
3. Exibindo o resultado.
4. Salvando em arquivo.

Podemos separar essas responsabilidades:

```python
def calculate_order_total(order):
    return sum(item["price"] for item in order["items"])


def apply_customer_discount(total, customer_type):
    if customer_type == "premium":
        return total * 0.9

    return total


def save_order(order_id, total):
    with open("orders.txt", "a") as file:
        file.write(f"{order_id}: {total}\n")


def process_order(order):
    total = calculate_order_total(order)
    total = apply_customer_discount(
        total,
        order["customer"]["type"]
    )

    save_order(order["id"], total)

    return total
```

Agora cada função possui uma responsabilidade mais clara.

---

## 3. Evite números mágicos

Considere:

```python
if age >= 18:
    ...
```

O `18` pode ser compreensível, mas em sistemas maiores existem muitos valores cujo significado não é óbvio.

Por exemplo:

```python
if customer_score >= 750:
    approve_credit()
```

Por que `750`?

Podemos tornar isso explícito:

```python
MINIMUM_CREDIT_SCORE = 750

if customer_score >= MINIMUM_CREDIT_SCORE:
    approve_credit()
```

Isso melhora a legibilidade e facilita alterações futuras.

---

## 4. Evite código duplicado — DRY

DRY significa **Don't Repeat Yourself**.

**Ruim:**

```python
price_with_tax = price * 1.10
total_with_tax = total * 1.10
product_with_tax = product_price * 1.10
```

Se a taxa mudar de 10% para 12%, precisamos alterar vários lugares.

**Melhor:**

```python
TAX_RATE = 0.10


def add_tax(amount):
    return amount * (1 + TAX_RATE)


price_with_tax = add_tax(price)
total_with_tax = add_tax(total)
product_with_tax = add_tax(product_price)
```

Agora a regra está centralizada.

---

## 5. Prefira código simples a código “inteligente”

Python permite escrever coisas extremamente compactas.

Por exemplo:

```python
result = [x * 2 for x in numbers if x > 10]
```

Isso é perfeitamente legível.

Mas tentar fazer tudo em uma única expressão pode prejudicar a manutenção:

```python
result = {x["id"]: x["price"] * (0.9 if x["category"] == "premium" else 1) for x in products if x["active"] and x["price"] > 100}
```

Uma versão mais explícita pode ser melhor:

```python
result = {}

for product in products:
    if not product["active"]:
        continue

    if product["price"] <= 100:
        continue

    price = product["price"]

    if product["category"] == "premium":
        price *= 0.9

    result[product["id"]] = price
```

**Clean Code não significa código com menos linhas.**

Significa código que exige **menos esforço mental para ser compreendido**.

---

## 6. Evite `if/else` excessivamente aninhados

**Difícil de ler:**

```python
def process_payment(user):
    if user:
        if user["active"]:
            if user["balance"] > 0:
                if not user["blocked"]:
                    return "Payment processed"

    return "Payment failed"
```

Podemos utilizar **guard clauses**:

```python
def process_payment(user):
    if not user:
        return "Payment failed"

    if not user["active"]:
        return "Payment failed"

    if user["balance"] <= 0:
        return "Payment failed"

    if user["blocked"]:
        return "Payment failed"

    return "Payment processed"
```

Ou, quando fizer sentido, extrair regras para funções específicas.

---

## 7. Use exceções adequadamente

Evite esconder problemas:

```python
try:
    user = find_user(user_id)
except Exception:
    pass
```

Isso é perigoso porque qualquer erro pode ser ignorado.

Prefira capturar exceções específicas:

```python
try:
    user = find_user(user_id)
except UserNotFoundError:
    print("Usuário não encontrado.")
```

Melhor ainda, dependendo da aplicação, podemos deixar a exceção subir para uma camada responsável por tratá-la.

---

## 8. Utilize Type Hints

Type hints tornam o código mais fácil de entender e permitem que ferramentas como linters e IDEs encontrem problemas.

**Sem type hints:**

```python
def calculate_total(price, quantity):
    return price * quantity
```

**Com type hints:**

```python
def calculate_total(price: float, quantity: int) -> float:
    return price * quantity
```

Podemos ir além:

```python
def calculate_total(
    price: float,
    quantity: int
) -> float:
    return price * quantity
```

Isso deixa explícito:

* `price` é `float`;
* `quantity` é `int`;
* o resultado é `float`.

---

## 9. Classes devem ter responsabilidades claras

Imagine uma classe que faz tudo:

```python
class User:
    def register(self):
        # salva no banco
        # envia e-mail
        # gera relatório
        # registra log
        # valida CPF
        # envia SMS
        pass
```

Isso cria uma classe difícil de testar e manter.

Uma abordagem melhor é separar responsabilidades:

```python
class UserService:
    def __init__(self, user_repository, email_service):
        self.user_repository = user_repository
        self.email_service = email_service

    def register(self, user):
        self.user_repository.save(user)
        self.email_service.send_welcome_email(user)
```

Agora o serviço coordena o processo, enquanto cada componente possui uma responsabilidade específica.

---

## 10. Código limpo também envolve testes

Uma função simples é muito mais fácil de testar.

```python
def calculate_discount(price: float, percentage: float) -> float:
    return price - (price * percentage)
```

Podemos criar um teste:

```python
def test_calculate_discount():
    result = calculate_discount(100, 0.10)

    assert result == 90
```

Se a função tiver muitas responsabilidades, o teste também tende a ficar complicado.

---

# Exemplo prático: antes e depois

Imagine um sistema de vendas.

### ❌ Código difícil de manter

```python
def process(p):
    if p["active"] == True:
        total = p["price"] * p["quantity"]

        if p["type"] == "vip":
            total = total * 0.9

        if total > 1000:
            total = total * 0.95

        print("Total:", total)

        return total

    return 0
```

Temos vários problemas:

* `p` não explica o que representa;
* `active == True` é desnecessário;
* existem números mágicos;
* a função possui várias regras;
* não há type hints;
* `process` é um nome genérico.

### ✅ Uma versão mais limpa

```python
VIP_DISCOUNT = 0.10
HIGH_VALUE_DISCOUNT = 0.05
HIGH_VALUE_THRESHOLD = 1000


def calculate_subtotal(price: float, quantity: int) -> float:
    return price * quantity


def apply_vip_discount(total: float, customer_type: str) -> float:
    if customer_type == "vip":
        return total * (1 - VIP_DISCOUNT)

    return total


def apply_high_value_discount(total: float) -> float:
    if total > HIGH_VALUE_THRESHOLD:
        return total * (1 - HIGH_VALUE_DISCOUNT)

    return total


def calculate_order_total(order: dict) -> float:
    if not order["active"]:
        return 0

    total = calculate_subtotal(
        order["price"],
        order["quantity"]
    )

    total = apply_vip_discount(
        total,
        order["customer_type"]
    )

    return apply_high_value_discount(total)
```

O resultado é praticamente o mesmo, mas agora as **regras de negócio estão explícitas e isoladas**.

---

# Os principais princípios para guardar

Uma boa forma de pensar em Clean Code é:

| Princípio             | Pergunta que você deve fazer                                |
| --------------------- | ----------------------------------------------------------- |
| **Nomes claros**      | "Outra pessoa entenderia o significado?"                    |
| **Funções pequenas**  | "Essa função está fazendo uma única coisa?"                 |
| **DRY**               | "Estou repetindo uma mesma regra?"                          |
| **Simplicidade**      | "Existe uma maneira mais simples de fazer isso?"            |
| **Baixo acoplamento** | "Uma alteração aqui quebra muitas outras partes?"           |
| **Alta coesão**       | "As coisas dessa classe/função realmente pertencem juntas?" |
| **Testabilidade**     | "Consigo testar isso facilmente?"                           |
| **Type hints**        | "Os tipos esperados estão claros?"                          |
| **Exceções**          | "Estou tratando erros de maneira específica?"               |
| **Legibilidade**      | "Consigo entender o código rapidamente?"                    |

### Em uma frase:

> **Clean Code é escrever código pensando não apenas no computador que vai executá-lo, mas principalmente no ser humano que vai precisar entendê-lo e modificá-lo.**

E um ponto importante: **Clean Code não é seguir regras cegamente**. Às vezes uma função de 20 linhas é melhor que cinco funções de quatro linhas; às vezes uma abstração deixa o código pior. O objetivo é sempre **reduzir complexidade e tornar a intenção do código evidente**.

