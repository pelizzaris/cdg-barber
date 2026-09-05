# Guia para Desenvolvedores Juniores - FSW Barber

Bem-vindo ao FSW Barber! Este guia foi criado para ajudar você a entender a estrutura e o funcionamento do projeto.

## 1. Visão Geral do Projeto

FSW Barber é uma plataforma de agendamento de serviços de barbearia. Os usuários podem encontrar barbearias, ver os serviços oferecidos e agendar um horário.

## 2. Tecnologias Utilizadas

O projeto foi construído com as seguintes tecnologias:

- **Framework:** [Next.js](https://nextjs.org/) (utilizando o App Router)
- **Linguagem:** [TypeScript](https://www.typescriptlang.org/)
- **Estilização:** [Tailwind CSS](https://tailwindcss.com/)
- **Componentes de UI:** [shadcn/ui](https://ui.shadcn.com/)
- **ORM:** [Prisma](https://www.prisma.io/)
- **Banco de Dados:** [PostgreSQL](https://www.postgresql.org/)
- **Autenticação:** [NextAuth.js](https://next-auth.js.org/)

## 3. Estrutura de Pastas

A estrutura de pastas principal está dentro de `/app`, seguindo o padrão do App Router do Next.js.

```
/app
├── _actions/         # Server Actions para mutações de dados
├── _components/      # Componentes React reutilizáveis
│   └── ui/           # Componentes base do shadcn/ui
├── _constants/       # Constantes usadas na aplicação
├── _data/            # Funções para buscar dados do servidor
├── _lib/             # Configurações de bibliotecas (Prisma, NextAuth)
├── _providers/       # Provedores de contexto React
├── api/              # Rotas de API (ex: para autenticação)
├── barbershops/      # Rota e páginas relacionadas às barbearias
├── bookings/         # Rota e páginas de agendamentos do usuário
├── layout.tsx        # Layout principal da aplicação
└── page.tsx          # Página inicial (Home)
```

### Detalhes Importantes:

- **Pastas com `_` (underscore):** Pastas como `_components`, `_lib`, etc., são uma convenção para indicar que elas não criam rotas públicas. O conteúdo delas é usado para construir as páginas.
- **Server Actions (`_actions`):** Usamos Server Actions para realizar operações no servidor, como criar ou deletar um agendamento. Essas funções são chamadas diretamente a partir dos componentes do cliente. Ex: `createBooking` em `_actions/create-booking.ts`.
- **Acesso a Dados (`_data`):** Funções assíncronas que buscam dados do banco de dados e são usadas principalmente dentro de Server Components para renderização inicial. Ex: `getConfirmedBookings` em `_data/get-confirmed-bookings.ts`.

## 4. Banco de Dados

Utilizamos o Prisma como ORM para interagir com nosso banco de dados PostgreSQL. O esquema do banco de dados está definido em `prisma/schema.prisma`.

### Tabelas Principais:

- `User`: Armazena informações dos usuários que se autenticam na plataforma.
- `Barbershop`: Contém os dados das barbearias, como nome, endereço, imagem, etc.
- `BarbershopService`: Representa os serviços oferecidos por uma barbearia, com preço, nome e descrição.
- `Booking`: Tabela que armazena os agendamentos. Ela relaciona um `User`, um `BarbershopService` e uma `date` (data e hora do agendamento).

As tabelas `Account`, `Session`, e `VerificationToken` são utilizadas pelo NextAuth.js para gerenciar o fluxo de autenticação.

## 5. Autenticação

A autenticação é gerenciada pelo NextAuth.js.

- **Configuração:** O arquivo principal de configuração é `app/_lib/auth.ts`. Nele, definimos os provedores de autenticação (atualmente, apenas o Google) e os callbacks.
- **Rota de API:** A rota `app/api/auth/[...nextauth]/route.ts` expõe as rotas GET e POST que o NextAuth.js precisa para funcionar.
- **Sessão do Usuário:** Para obter a sessão do usuário no lado do servidor (em Server Components ou Server Actions), usamos a função `getServerSession(authOptions)`.

## 6. Fluxo de Agendamento

1.  O usuário seleciona uma barbearia e um serviço.
2.  Um calendário é exibido, e o usuário escolhe uma data e um horário.
3.  Ao confirmar, o componente do cliente chama a Server Action `createBooking`.
4.  A action `createBooking` verifica se o usuário está autenticado (`getServerSession`).
5.  Se estiver autenticado, um novo registro é criado na tabela `Booking` com os dados do agendamento.
6.  A função `revalidatePath` é chamada para atualizar a interface do usuário nas páginas de agendamentos e da barbearia.

## 7. Outros Fluxos de Usuário

### 7.1. Visualização de Agendamentos

O usuário pode ver seus agendamentos na página `/bookings`.

1.  A página (`app/bookings/page.tsx`) é um Server Component que busca os agendamentos do usuário logado.
2.  Ela chama duas funções de busca de dados: `getConfirmedBookings` e `getConcludedBookings` que estão na pasta `app/_data`.
3.  `getConfirmedBookings` retorna os agendamentos com data futura.
4.  `getConcludedBookings` retorna os agendamentos com data passada.
5.  Os agendamentos são listados na interface, separados por "Confirmados" e "Finalizados", usando o componente `BookingItem`.

### 7.2. Cancelamento de Agendamento

Apenas agendamentos confirmados (com data futura) podem ser cancelados.

1.  Dentro do componente `BookingItem` (`app/_components/booking-item.tsx`), que é um Client Component, há um botão "Cancelar Reserva".
2.  Este botão abre um diálogo de confirmação.
3.  Ao confirmar o cancelamento, a função `handleCancelBooking` é chamada.
4.  Essa função invoca a Server Action `deleteBooking` (`app/_actions/delete-booking.ts`), passando o ID do agendamento.
5.  A Server Action deleta o registro correspondente no banco de dados.
6.  Após o sucesso da operação, um "toast" (notificação) informa ao usuário que a reserva foi cancelada.

### 7.3. Busca por Barbearias

O usuário pode encontrar barbearias de duas maneiras principais na página inicial:

1.  **Campo de Busca:**
    - O componente `Search` (`app/_components/search.tsx`) contém um formulário com um campo de input.
    - Ao submeter o formulário, o usuário é redirecionado para a página `/barbershops` com um parâmetro de busca na URL (ex: `/barbershops?title=MinhaBarbearia`).
    - A página de barbearias (`app/barbershops/page.tsx`) irá ler esse parâmetro, filtrar as barbearias pelo nome e exibir os resultados.

2.  **Busca Rápida por Serviços:**
    - Na página inicial, existem botões de "Busca Rápida" (ex: "Cabelo", "Barba").
    - Clicar em um desses botões redireciona o usuário para `/barbershops` com um parâmetro de serviço na URL (ex: `/barbershops?service=Cabelo`).
    - A página de barbearias filtrará e exibirá as barbearias que oferecem o serviço pesquisado.

## 8. Como Rodar o Projeto Localmente

1.  **Instale as dependências:**

    ```bash
    npm install
    ```

2.  **Configure as variáveis de ambiente:**
    - Crie um arquivo `.env` na raiz do projeto.
    - Adicione as seguintes variáveis (você precisará criar um projeto no Google Cloud para obter as credenciais do OAuth):
      ```
      DATABASE_URL="postgresql://USER:PASSWORD@HOST:PORT/DATABASE"
      GOOGLE_CLIENT_ID="SEU_CLIENT_ID_DO_GOOGLE"
      GOOGLE_CLIENT_SECRET="SEU_CLIENT_SECRET_DO_GOOGLE"
      NEXT_AUTH_SECRET="QUALQUER_STRING_SECRETA"
      ```

3.  **Execute as migrações do banco de dados:**

    ```bash
    npx prisma migrate dev
    ```

4.  **(Opcional) Popule o banco com dados iniciais:**

    ```bash
    npx prisma db seed
    ```

5.  **Inicie o servidor de desenvolvimento:**
    ```bash
    npm run dev
    ```

Abra [http://localhost:3000](http://localhost:3000) no seu navegador para ver o resultado.
