# API Financeira 99 - Documentação

Esta API permite consultar dados financeiros da 99 por meio da DL Compras.

## Base URLs

| Ambiente | Base URL |
| --- | --- |
| Homologação | `https://api.dlcompras.dev.br/v1` |
| Produção | `https://api.dlcompras.com.br/v1` |

Todas as requisições e respostas usam JSON.

## Autenticação

Antes de consultar os endpoints financeiros, gere um token de acesso.

### Gerar token

`POST /oauth/token`

#### Request

Headers:

```http
Content-Type: application/json
```

Body:

```json
{
  "grant_type": "client_credentials",
  "client_id": "seu_client_id",
  "client_secret": "seu_client_secret"
}
```

#### Response 200

```json
{
  "access_token": "token_de_acesso",
  "token_type": "Bearer",
  "expires_in": 3600
}
```

Use o token retornado no header `Authorization` das próximas chamadas:

```http
Authorization: Bearer token_de_acesso
```

#### Possíveis erros

| HTTP | Código | Descrição |
| --- | --- | --- |
| 400 | `INVALID_REQUEST` | Requisição incompleta ou inválida. |
| 400 | `INVALID_GRANT_TYPE` | `grant_type` deve ser `client_credentials`. |
| 401 | `INVALID_CLIENT` | Credenciais inválidas. |
| 403 | `CLIENT_DISABLED` | Cliente inativo. |
| 500 | `INTERNAL_ERROR` | Erro interno ao processar a autenticação. |

Formato dos erros:

```json
{
  "error": "INVALID_CLIENT",
  "message": "Invalid client credentials.",
  "correlation_id": "corr_xxx"
}
```

## Endpoints Financeiros

Os endpoints financeiros exigem o header:

```http
Authorization: Bearer token_de_acesso
```

## Parâmetros comuns

| Query param | Obrigatório | Formato | Default | Descrição |
| --- | --- | --- | --- | --- |
| `store_id` | Sim | string | - | Identificador da loja. |
| `start_date` | Sim | `YYYY-MM-DD` ou `YYYYMMDD` | - | Data inicial da consulta. |
| `end_date` | Sim | `YYYY-MM-DD` ou `YYYYMMDD` | - | Data final da consulta. |
| `page_no` | Sim | inteiro maior ou igual a 1 | - | Número da página. |
| `page_size` | Não | inteiro maior ou igual a 1 | `200` | Quantidade de registros por página. |

## Consultar dados de faturamento

`GET /finance/99/bill-data`

### Exemplo

```http
GET /finance/99/bill-data?store_id=STORE_ID&start_date=2026-06-01&end_date=2026-06-22&page_no=1&page_size=200
Authorization: Bearer token_de_acesso
```

### Response

Quando a consulta chega na 99, a resposta é repassada exatamente como retornada pela 99, tanto em sucesso quanto em erro.

Para interpretar campos, códigos e mensagens retornados pela 99, consulte a [documentação oficial da 99](https://developer-food.99app.com/pt-BR/openapi).

## Consultar repasses / settlements

`GET /finance/99/settlements`

### Exemplo

```http
GET /finance/99/settlements?store_id=STORE_ID&start_date=20260601&end_date=20260622&page_no=1
Authorization: Bearer token_de_acesso
```

### Response

Quando a consulta chega na 99, a resposta é repassada exatamente como retornada pela 99, tanto em sucesso quanto em erro.

Para interpretar campos, códigos e mensagens retornados pela 99, consulte a [documentação oficial da 99](https://developer-food.99app.com/pt-BR/openapi).

## Responses

### Respostas da 99

Para os endpoints:

- `GET /finance/99/bill-data`
- `GET /finance/99/settlements`

as respostas vindas da 99 são repassadas sem alteração.

Isso significa que:

- O status HTTP retornado será o status recebido da 99.
- O body retornado será o body recebido da 99.
- Em caso de erro retornado pela 99, a interpretação do erro deve seguir a [documentação oficial da 99](https://developer-food.99app.com/pt-BR/openapi).

### Erros gerados antes da consulta na 99

Alguns erros podem ocorrer antes da consulta ser enviada para a 99, por exemplo autenticação, autorização ou parâmetros inválidos.

Formato:

```json
{
  "error": "INVALID_PAGE_NO",
  "message": "page_no must be an integer greater than or equal to 1.",
  "correlation_id": "corr_xxx"
}
```

| HTTP | Código | Descrição |
| --- | --- | --- |
| 400 | `INVALID_REQUEST` | Parâmetros obrigatórios ausentes. |
| 400 | `INVALID_DATE_FORMAT` | `start_date` ou `end_date` fora dos formatos aceitos. |
| 400 | `INVALID_PERIOD` | `start_date` maior que `end_date`. |
| 400 | `INVALID_PAGE_NO` | `page_no` inválido. |
| 400 | `INVALID_PAGE_SIZE` | `page_size` inválido. |
| 401 | `UNAUTHORIZED` | Token ausente, inválido ou expirado. |
| 403 | `FORBIDDEN` | Token sem permissão para o endpoint. |
| 403 | `FORBIDDEN_STORE` | Loja não autorizada para o cliente autenticado. |
| 403 | `PROVIDER_NOT_ENABLED` | Integração financeira 99 não habilitada para a loja ou conta. |
| 500 | `INTERNAL_ERROR` | Erro interno. |
| 504 | `PROVIDER_TIMEOUT` | Timeout ao consultar a 99. |

## Observações

- Guarde o `correlation_id` de respostas de erro locais para suporte.
- O token deve ser renovado quando expirar.
- Os formatos aceitos para datas são `YYYY-MM-DD` e `YYYYMMDD`.
- Se `page_size` não for enviado, será usado `200`.