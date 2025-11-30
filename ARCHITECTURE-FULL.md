# Diretrizes de Arquitetura e Desenvolvimento Ágil (Pragmatic NestJS)

Este documento serve como a fonte da verdade para o desenvolvimento do
projeto e como contexto para Agentes de IA gerarem código alinhado com
nossa arquitetura.

---

## 1. Filosofia do Projeto

Utilizamos uma abordagem **Pragmatic NestJS**. O foco é velocidade de
desenvolvimento (Time-to-Market) sem perder a tipagem forte.

- **Eliminamos:** Camadas de DDD purista (Entidades de Domínio
  manuais, Mappers manuais, Repositórios abstratos).
- **Mantemos:** Injeção de dependência, SOLID e organização modular.
- **Fonte da Verdade:** O arquivo `schema.prisma` é a definição final
  dos dados.

---

## 2. Estrutura de Pastas

Adotamos uma estrutura **Modular Híbrida**. Cada módulo contém suas
próprias pastas de `controllers` e `services` para manter a organização
visual e escalável.

```text
src/
├── database/
├── [nome-do-modulo]/
│   ├── [nome-do-modulo].module.ts
│   ├── controllers/
│   │   ├── transactions.controller.ts
│   │   └── invoices.controller.ts
│   └── services/
│       ├── transactions.service.ts
│       └── invoices.service.ts
```

---

## 3. Comandos CLI (Cheatsheet)

### Criar Novo Módulo

```bash
nest g mo [nome-do-modulo]
# Exemplo: nest g mo finance
```

### Criar Controller

O controller deve ser criado dentro da pasta `controllers` do módulo
específico.

```bash
nest g co [nome-do-modulo]/controllers/[nome-do-recurso] --flat --no-spec
# Exemplo: nest g co finance/controllers/transactions --flat --no-spec
```

### Criar Service

O service deve ser criado dentro da pasta `services` do módulo
específico.

```bash
nest g s [nome-do-modulo]/services/[nome-do-recurso] --flat --no-spec
# Exemplo: nest g s finance/services/transactions --flat --no-spec
```

---

## 4. Regras de Implementação (Para IAs e Devs)

### A. Banco de Dados (Prisma)

- Sem Repositórios: Injete o `PrismaService` diretamente no Service.
- Tipagem: Utilize os tipos gerados automaticamente pelo
  `@prisma/client`.
- Complexidade: Se a query for extremamente complexa, isole em um
  método privado.

### B. DTOs (Data Transfer Objects)

- Swagger Automático: Não utilize `@ApiProperty`. O plugin já gera
  automaticamente.
- Updates: Utilize `PartialType` do `@nestjs/swagger`.

```ts
// create-transaction.dto.ts
export class CreateTransactionDto {
  title: string;
  amount: number;
}

// update-transaction.dto.ts
import { PartialType } from "@nestjs/swagger";
export class UpdateTransactionDto extends PartialType(CreateTransactionDto) {}
```

### C. Tratamento de Erros

- Utilize exceções nativas do NestJS.
- Exemplos:
  - Registro não encontrado:
    `throw new NotFoundException('Transaction not found');`
  - Duplicidade/Conflito:
    `throw new ConflictException('Already exists');`

### D. Controllers

- Devem ser "magros": apenas recebem requisição, chamam o Service e
  retornam resposta.
- Nunca devem conter regras de negócio.

---

## 5. Exemplo de Fluxo de Trabalho (Workflow)

CRUD de Produtos no módulo "Estoque":

1.  Modelagem: Adicione `model Product { ... }` ao `schema.prisma` e
    rode:

```bash
npx prisma db push
```

2.  Criar Módulo:

```bash
nest g mo stock
```

3.  Controller:

```bash
nest g co stock/controllers/products --flat --no-spec
```

4.  Service:

```bash
nest g s stock/services/products --flat --no-spec
```

5.  Codificação:

- No Service: injete `PrismaService`. Crie métodos `create`,
  `findAll`, etc.
- No Controller: chame os métodos do Service.

---

## 6. Configuração Global (nest-cli.json)

```json
{
  "$schema": "https://json.schemastore.org/nest-cli",
  "collection": "@nestjs/schematics",
  "sourceRoot": "src",
  "compilerOptions": {
    "deleteOutDir": true,
    "plugins": [
      {
        "name": "@nestjs/swagger",
        "options": {
          "classValidatorShim": true,
          "introspectComments": true
        }
      }
    ]
  },
  "generateOptions": {
    "spec": false
  }
}
```
