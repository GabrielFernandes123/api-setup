# Guia de Configuração do Ambiente (Windows + Yarn)

## 1. Instalação das Bibliotecas (Via Yarn)

Abra o terminal na raiz do projeto e rode:

### Dependências de Produção

```powershell
yarn add @prisma/client class-validator class-transformer @nestjs/swagger @nestjs/config @scalar/nestjs-api-reference
```

### Dependências de Desenvolvimento

```powershell
yarn add -D prisma husky lint-staged @commitlint/cli @commitlint/config-conventional @commitlint/types eslint-plugin-unused-imports eslint-config-prettier eslint-plugin-prettier globals @eslint/js typescript-eslint
```

---

## 2. Inicialização (Banco e Husky)

```powershell
# Inicializa o arquivo do Prisma
npx prisma init

# Inicializa o Husky (Cria a pasta .husky e ajusta o package.json)
yarn husky init
```

---

## 3. Criação Manual dos Arquivos de Configuração

Como você está no Windows, a maneira mais segura é criar esses arquivos
diretamente no VS Code.

---

### A. Arquivos da Pasta `.husky`

#### Arquivo: `.husky/commit-msg`

```sh
npx --no -- commitlint --edit $1
```

#### Arquivo: `.husky/pre-commit`

```sh
yarn lint-staged
```

---

### B. Arquivos na Raiz do Projeto

#### Arquivo: `nest-cli.json`

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

---

#### Arquivo: `commitlint.config.ts`

```ts
import type { UserConfig } from "@commitlint/types";

const Configuration: UserConfig = {
  extends: ["@commitlint/config-conventional"],
  rules: {
    "type-enum": [
      2,
      "always",
      [
        "feat",
        "fix",
        "docs",
        "style",
        "refactor",
        "perf",
        "test",
        "build",
        "ci",
        "chore",
        "revert",
      ],
    ],
    "scope-enum": [0],
    "scope-empty": [0],
  },
};

export default Configuration;
```

---

#### Arquivo: `eslint.config.mjs`

```js
import eslint from "@eslint/js";
import tseslint from "typescript-eslint";
import unusedImports from "eslint-plugin-unused-imports";
import prettierPlugin from "eslint-plugin-prettier/recommended";
import globals from "globals";
import { fileURLToPath } from "url";
import { dirname } from "path";

const __filename = fileURLToPath(import.meta.url);
const __dirname = dirname(__filename);

export default tseslint.config(
  // 1. Configurações de ignorar arquivos (substitui .eslintignore)
  {
    ignores: ["eslint.config.mjs", "dist", "node_modules", "coverage"],
  },

  // 2. Configurações base recomendadas (JS + TS + Prettier)
  eslint.configs.recommended,
  ...tseslint.configs.recommended,
  prettierPlugin,

  // 3. Sua configuração personalizada
  {
    languageOptions: {
      parserOptions: {
        project: "tsconfig.json",
        tsconfigRootDir: __dirname,
        sourceType: "module",
      },
      globals: {
        ...globals.node,
        ...globals.jest,
      },
    },
    plugins: {
      "unused-imports": unusedImports,
    },
    rules: {
      // --- SUAS REGRAS ORIGINAIS ---

      // Segurança: Obriga await ou catch em promises
      "@typescript-eslint/no-floating-promises": "error",

      // Limpeza de variáveis e imports
      "no-unused-vars": "off",
      "@typescript-eslint/no-unused-vars": "off",
      "unused-imports/no-unused-imports": "error",
      "unused-imports/no-unused-vars": [
        "warn",
        {
          vars: "all",
          varsIgnorePattern: "^_",
          args: "after-used",
          argsIgnorePattern: "^_",
        },
      ],

      // Regras desligadas para agilidade (Pragmatic NestJS)
      "@typescript-eslint/interface-name-prefix": "off",
      "@typescript-eslint/explicit-function-return-type": "off",
      "@typescript-eslint/explicit-module-boundary-types": "off",
      "@typescript-eslint/no-explicit-any": "off",
      "@typescript-eslint/no-unsafe-assignment": "off",
      "@typescript-eslint/no-unsafe-member-access": "off",
    },
  }
);
```

---

#### Arquivo: `.lintstagedrc.json`

```json
{
  "*.{ts,tsx}": ["eslint --fix", "prettier --write"]
}
```

---

#### Arquivo: `.prettierrc`

```json
{
  "singleQuote": true,
  "trailingComma": "all",
  "printWidth": 100
}
```

#### Arquivo: `tsconfig.json`

```json
{
  "compilerOptions": {
    "module": "commonjs",
    "moduleResolution": "node",
    "esModuleInterop": true,
    "isolatedModules": true,
    "declaration": true,
    "removeComments": true,
    "emitDecoratorMetadata": true,
    "experimentalDecorators": true,
    "allowSyntheticDefaultImports": true,
    "target": "es2020",
    "sourceMap": true,
    "outDir": "./dist",
    "baseUrl": "./",
    "incremental": true,
    "skipLibCheck": true,
    "strictNullChecks": true,
    "forceConsistentCasingInFileNames": true,
    "noImplicitAny": false,
    "strictBindCallApply": false,
    "noFallthroughCasesInSwitch": false
  }
}
```

#### Arquivo: `main.ts`

```ts
import { NestFactory } from "@nestjs/core";
import { AppModule } from "./app.module";
import { ValidationPipe } from "@nestjs/common";
import { DocumentBuilder, SwaggerModule } from "@nestjs/swagger";
import { json, urlencoded } from "express";
import { apiReference } from "@scalar/nestjs-api-reference";

async function bootstrap() {
  const app = await NestFactory.create(AppModule, { cors: true });

  app.useGlobalPipes(
    new ValidationPipe({
      transform: true,
    })
  );
  app.use(json({ limit: "50mb" }));
  app.use(urlencoded({ extended: true, limit: "50mb" }));

  const config = new DocumentBuilder()
    .setTitle("Documentação")
    .setDescription("")
    .setVersion("1.0")
    .build();

  const document = SwaggerModule.createDocument(app, config);
  SwaggerModule.setup("api", app, document);

  app.use(
    "/reference",
    apiReference({
      spec: {
        content: document,
      },
    })
  );

  await app.listen(3333);
}

bootstrap().catch((err) => {
  console.error("Erro ao iniciar a aplicação:", err);
  process.exit(1);
});
```

---

### C. Configuração do VS Code

Crie a pasta `.vscode` caso ainda não exista.

#### Arquivo: `.vscode/settings.json`

```json
{
  "editor.formatOnSave": true,
  "editor.codeActionsOnSave": {
    "source.fixAll.eslint": "explicit",
    "source.organizeImports": "explicit"
  },
  "editor.defaultFormatter": "esbenp.prettier-vscode",
  "[typescript]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode"
  }
}
```

---

## 4. Ajuste Final no `package.json`

Garanta que existe o script `prepare`:

```json
"scripts": {
  "prepare": "husky"
}
```

## 5. Criar Prisma Module

### Criar modulo de banco de dados

```powershell
nest g module modules/database
nest g service services/database/prisma --flat --project modules/database
```

#### Arquivo: `src/services/database/prisma.service.ts`:

```ts
import { Injectable, OnModuleInit } from "@nestjs/common";
import { PrismaClient } from "@prisma/client";

@Injectable()
export class PrismaService extends PrismaClient implements OnModuleInit {
  async onModuleInit() {
    await this.$connect();
  }
}
```

#### Arquivo: `src/modules/database/database.module.ts`:

```ts
import { Global, Module } from "@nestjs/common";
import { PrismaService } from "../../services/database/prisma.service";

@Global() // Importante: Isso torna o Prisma disponível no app todo sem precisar reimportar
@Module({
  providers: [PrismaService],
  exports: [PrismaService],
})
export class DatabaseModule {}
```

---

## Pronto para usar! 🎉

- **Swagger**: Gera sozinho.\
- **Lint**: Roda automaticamente ao salvar.\
- **Commits**: Validação automática com padrão convencional.\
- **Backend seguro**: ESLint garante `await` obrigatório em Promises.
