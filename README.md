# AgroVision

Sistema de gestão agrícola para produtores e administradores, desenvolvido com Vue 3 + Vite. A aplicação permite o cadastro e acompanhamento de usuários, lavouras, defensivos agrícolas e orientações técnicas.

## Visão geral

O projeto foi pensado para um ambiente de gestão rural, com fluxo de login por perfil e painel específico para:

- administradores
- produtores
- acompanhamento de lavouras e recomendações técnicas

A persistência dos dados é feita no `localStorage`, permitindo que as informações permaneçam disponíveis mesmo ao trocar de tela ou recarregar a página.

## Funcionalidades

- Login com seleção de perfil (produtor ou administrador)
- Cadastro de usuários
- Cadastro de lavouras
- Cadastro de defensivos agrícolas
- Registro de orientações técnicas
- Histórico de orientações
- Dashboard com destaque para próxima visita
- Visualização das lavouras por produtor
- Layout responsivo para telas menores

## Stack utilizada

- Vue 3
- Vite
- TypeScript
- HTML + CSS
- localStorage para persistência local
- MDI icons (`@mdi/js`)

## Requisitos

- Node.js 22 ou superior
- npm

## Instalação

1. Clone o projeto
2. Acesse a pasta do projeto
3. Instale as dependências:

```bash
npm install
```

## Como rodar em desenvolvimento

```bash
npm run dev
```

A aplicação ficará disponível no navegador em modo de desenvolvimento, normalmente em:

```bash
http://localhost:5173
```

## Build de produção

```bash
npm run build
```

## Estrutura do projeto

```text
agrovision/
├── public/
│   └── agrovision-logo.png
├── src/
│   ├── assets/
│   ├── components/
│   │   ├── Login.vue
│   │   ├── CadastroUsuarios.vue
│   │   ├── CadastroLavoura.vue
│   │   ├── CadastroDefensivos.vue
│   │   └── CadastroOrientacao.vue
│   ├── mocks/
│   │   └── auth.ts
│   ├── App.vue
│   ├── main.ts
│   └── style.css
├── env.d.ts
├── index.html
├── package.json
├── tsconfig.json
├── tsconfig.app.json
├── tsconfig.node.json
├── vite.config.ts
├── README.md
└── public/
```

## Persistência de dados

A aplicação salva os dados principais em `localStorage`, com chaves como:

- `agrovision:users`
- `agrovision:lavouras`
- `agrovision:defensivos`
- `agrovision:orientacoes`

Isso permite que as informações sejam mantidas mesmo quando o usuário navega entre telas.

## Fluxo principal

### Admin

- Cadastra usuários
- Cadastra defensivos
- Cadastra lavouras
- Cria orientações técnicas
- Visualiza histórico e produtores vinculados

### Produtor

- Acessa sua área específica
- Visualiza suas lavouras
- Consulta orientações atribuídas
- Acompanha próxima visita técnica

## Observações

- A aplicação utiliza dados mockados para login inicial e seed de usuários.
- O projeto foi desenvolvido para demonstração e gestão local de informações agrícolas.
- O objetivo principal é facilitar o acompanhamento operacional de produtores e administradores.

## Licença

Este projeto é de uso interno/demonstração e pode ser adaptado conforme necessidade do usuário ou equipe responsável.

## Autor

Projeto desenvolvido para gestão agrícola em ambiente Vue 3.

