# Guia de Configuração do Ambiente (Windows + Yarn)

## 1. Instalação das Bibliotecas (Via Yarn)

Abra o terminal na raiz do projeto e rode:

### Dependências de Produção

``` powershell
yarn add @prisma/client class-validator class-transformer @nestjs/swagger @nestjs/config
```

### Dependências de Desenvolvimento

``` powershell
yarn add -D prisma husky lint-staged @commitlint/cli @commitlint/config-conventional @commitlint/types eslint-plugin-unused-imports eslint-config-prettier eslint-plugin-prettier
```

------------------------------------------------------------------------

## 2. Inicialização (Banco e Husky)

``` powershell
# Inicializa o arquivo do Prisma
npx prisma init

# Inicializa o Husky (Cria a pasta .husky e ajusta o package.json)
yarn husky init
```

------------------------------------------------------------------------

## 3. Criação Manual dos Arquivos de Configuração

Como você está no Windows, a maneira mais segura é criar esses arquivos
diretamente no VS Code.

------------------------------------------------------------------------

### A. Arquivos da Pasta `.husky`

#### Arquivo: `.husky/commit-msg`

``` sh
npx --no -- commitlint --edit $1
```

#### Arquivo: `.husky/pre-commit`

``` sh
yarn lint-staged
```

------------------------------------------------------------------------

### B. Arquivos na Raiz do Projeto

#### Arquivo: `nest-cli.json`

``` json
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

------------------------------------------------------------------------

#### Arquivo: `commitlint.config.ts`

``` ts
import type { UserConfig } from '@commitlint/types';

const Configuration: UserConfig = {
  extends: ['@commitlint/config-conventional'],
  rules: {
    'type-enum': [
      2,
      'always',
      [
        'feat', 'fix', 'docs', 'style', 'refactor',
        'perf', 'test', 'build', 'ci', 'chore', 'revert'
      ],
    ],
    'scope-enum': [0],
    'scope-empty': [0],
  },
};

export default Configuration;
```

------------------------------------------------------------------------

#### Arquivo: `.eslintrc.js`

``` js
module.exports = {
  parser: '@typescript-eslint/parser',
  parserOptions: {
    project: 'tsconfig.json',
    tsconfigRootDir: __dirname,
    sourceType: 'module',
  },
  plugins: ['@typescript-eslint/eslint-plugin', 'unused-imports'],
  extends: [
    'plugin:@typescript-eslint/recommended',
    'plugin:prettier/recommended',
  ],
  root: true,
  env: {
    node: true,
    jest: true,
  },
  ignorePatterns: ['.eslintrc.js'],
  rules: {
    '@typescript-eslint/no-floating-promises': 'error',

    'no-unused-vars': 'off',
    '@typescript-eslint/no-unused-vars': 'off',
    'unused-imports/no-unused-imports': 'error',
    'unused-imports/no-unused-vars': [
      'warn',
      { 'vars': 'all', 'varsIgnorePattern': '^_', 'args': 'after-used', 'argsIgnorePattern': '^_' }
    ],

    '@typescript-eslint/interface-name-prefix': 'off',
    '@typescript-eslint/explicit-function-return-type': 'off',
    '@typescript-eslint/explicit-module-boundary-types': 'off',
    '@typescript-eslint/no-explicit-any': 'off',
    '@typescript-eslint/no-unsafe-assignment': 'off',
    '@typescript-eslint/no-unsafe-member-access': 'off',
  },
};
```

------------------------------------------------------------------------

#### Arquivo: `.lintstagedrc.json`

``` json
{
  "*.{ts,tsx}": [
    "eslint --fix",
    "prettier --write"
  ]
}
```

------------------------------------------------------------------------

#### Arquivo: `.prettierrc`

``` json
{
  "singleQuote": true,
  "trailingComma": "all",
  "printWidth": 100
}
```

------------------------------------------------------------------------

### C. Configuração do VS Code

Crie a pasta `.vscode` caso ainda não exista.

#### Arquivo: `.vscode/settings.json`

``` json
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

------------------------------------------------------------------------

## 4. Ajuste Final no `package.json`

Garanta que existe o script `prepare`:

``` json
"scripts": {
  "prepare": "husky"
}
```

------------------------------------------------------------------------

## Pronto para usar! 🎉

-   **Swagger**: Gera sozinho.\
-   **Lint**: Roda automaticamente ao salvar.\
-   **Commits**: Validação automática com padrão convencional.\
-   **Backend seguro**: ESLint garante `await` obrigatório em Promises.
