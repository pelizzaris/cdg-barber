# 💈 FSW Barber

> Plataforma moderna de agendamento para barbearias desenvolvida durante a Full Stack Week

[![Next.js](https://img.shields.io/badge/Next.js-14-black)](https://nextjs.org/)
[![TypeScript](https://img.shields.io/badge/TypeScript-5-blue)](https://www.typescriptlang.org/)
[![Prisma](https://img.shields.io/badge/Prisma-5-2D3748)](https://www.prisma.io/)
[![Tailwind CSS](https://img.shields.io/badge/Tailwind-3-38B2AC)](https://tailwindcss.com/)

**[🔗 Demo ao vivo](https://fullstackweek-barber.vercel.app)**

---

## 📋 Sobre o Projeto

FSW Barber é uma aplicação completa de agendamento para barbearias que permite aos usuários:

- 🔍 **Buscar** barbearias próximas
- 📅 **Agendar** serviços com data e horário específicos
- 📱 **Gerenciar** seus agendamentos
- 🔐 **Autenticar** de forma segura com Google

O projeto foi desenvolvido com foco em **performance**, **acessibilidade** e **experiência do usuário**, utilizando as tecnologias mais modernas do ecossistema React.

---

## ✨ Funcionalidades

### 🏠 Página Inicial
- Saudação personalizada para usuários autenticados
- Lista de agendamentos confirmados
- Seção de barbearias recomendadas
- Seção de barbearias populares
- Busca por nome de barbearia

### 💈 Detalhes da Barbearia
- Informações completas da barbearia (endereço, telefones, imagem)
- Lista de serviços oferecidos com preços
- Sistema de agendamento integrado

### 📅 Sistema de Agendamento
- Calendário interativo para seleção de data
- Lista de horários disponíveis por dia
- Validação de horários já agendados
- Resumo do agendamento antes da confirmação
- Notificações de sucesso com ação para visualizar agendamentos

### 📊 Página de Agendamentos
- Visualização de agendamentos confirmados
- Histórico de agendamentos finalizados
- Cancelamento de agendamentos
- Proteção de rota (apenas usuários autenticados)

### 🔐 Autenticação
- Login social com Google (NextAuth.js)
- Gerenciamento de sessão
- Proteção de rotas e funcionalidades

### 🎨 Interface
- Design moderno e responsivo
- Suporte a tema dark/light (next-themes)
- Componentes reutilizáveis (shadcn/ui)
- Animações suaves (tailwindcss-animate)

---

## 🛠️ Tecnologias Utilizadas

### Frontend
- **[Next.js 14](https://nextjs.org/)** - Framework React com App Router
- **[React 18](https://react.dev/)** - Biblioteca UI
- **[TypeScript](https://www.typescriptlang.org/)** - Tipagem estática
- **[Tailwind CSS](https://tailwindcss.com/)** - Framework CSS utility-first
- **[shadcn/ui](https://ui.shadcn.com/)** - Componentes React (Radix UI)
- **[Lucide React](https://lucide.dev/)** - Ícones
- **[next-themes](https://github.com/pacocoursey/next-themes)** - Gerenciamento de tema

### Backend & Database
- **[Prisma](https://www.prisma.io/)** - ORM TypeScript-first
- **[PostgreSQL](https://www.postgresql.org/)** - Banco de dados relacional
- **[NextAuth.js](https://next-auth.js.org/)** - Autenticação

### Formulários & Validação
- **[React Hook Form](https://react-hook-form.com/)** - Gerenciamento de formulários
- **[Zod](https://zod.dev/)** - Validação de schema TypeScript

### Utils
- **[date-fns](https://date-fns.org/)** - Manipulação de datas
- **[Sonner](https://sonner.emilkowal.ski/)** - Notificações toast

---

## 🗄️ Modelo de Dados

```prisma
model User {
  id            String    @id @default(cuid())
  name          String?
  email         String?   @unique
  emailVerified DateTime?
  image         String?
  bookings      Booking[]
}

model Barbershop {
  id       String    @id @default(uuid())
  name     String
  address  String
  imageUrl String
  services Service[]
  bookings Booking[]
}

model Service {
  id           String     @id @default(uuid())
  name         String
  price        Decimal    @db.Decimal(10, 2)
  description  String
  imageUrl     String
  barbershopId String
  barbershop   Barbershop @relation(fields: [barbershopId], references: [id])
  bookings     Booking[]
}

model Booking {
  id           String     @id @default(uuid())
  userId       String
  user         User       @relation(fields: [userId], references: [id])
  serviceId    String
  service      Service    @relation(fields: [serviceId], references: [id])
  barbershopId String
  barbershop   Barbershop @relation(fields: [barbershopId], references: [id])
  date         DateTime
}
```

---

## 🚀 Como Rodar o Projeto

### Pré-requisitos

- Node.js 18+ instalado
- PostgreSQL rodando (localmente ou via Docker)
- Conta Google Cloud para OAuth (credenciais)

### Passo a Passo

1. **Clone o repositório**
```bash
git clone https://github.com/felipemotarocha/fullstackweek-barber.git
cd fullstackweek-barber
```

2. **Instale as dependências**
```bash
npm install
```

3. **Configure as variáveis de ambiente**

Crie um arquivo `.env` na raiz do projeto:

```env
# Database
DATABASE_URL="postgresql://usuario:senha@localhost:5432/fsw_barber"

# NextAuth
NEXTAUTH_SECRET="seu-secret-aqui" # Gere com: openssl rand -base64 32
NEXTAUTH_URL="http://localhost:3000"

# Google OAuth
GOOGLE_CLIENT_ID="seu-google-client-id"
GOOGLE_CLIENT_SECRET="seu-google-client-secret"
```

4. **Configure o Google OAuth**

- Acesse o [Google Cloud Console](https://console.cloud.google.com/)
- Crie um novo projeto ou selecione um existente
- Ative a Google+ API
- Crie credenciais OAuth 2.0
- Adicione `http://localhost:3000` como origem autorizada
- Adicione `http://localhost:3000/api/auth/callback/google` como URI de redirecionamento

5. **Execute as migrações do Prisma**
```bash
npx prisma migrate dev
```

6. **Popule o banco de dados (opcional)**
```bash
npx prisma db seed
```

7. **Inicie o servidor de desenvolvimento**
```bash
npm run dev
```

8. **Acesse a aplicação**

Abra [http://localhost:3000](http://localhost:3000) no navegador

---

## 📦 Scripts Disponíveis

```bash
# Desenvolvimento
npm run dev          # Inicia servidor de desenvolvimento

# Build & Produção
npm run build        # Gera build de produção
npm run start        # Inicia servidor de produção

# Qualidade de Código
npm run lint         # Executa ESLint

# Prisma
npx prisma studio    # Abre Prisma Studio (GUI para banco de dados)
npx prisma generate  # Gera Prisma Client
npx prisma migrate dev  # Cria e aplica migrations
npx prisma db seed   # Popula o banco de dados
```

---

## 📁 Estrutura de Pastas

```
fullstackweek-barber/
├── app/
│   ├── (home)/              # Grupo de rota - Página inicial
│   ├── _actions/            # Server Actions (mutações)
│   ├── _components/         # Componentes reutilizáveis
│   │   └── ui/             # Componentes shadcn/ui
│   ├── _lib/               # Configurações (Prisma, NextAuth)
│   ├── _providers/         # Context Providers
│   ├── api/                # API Routes
│   │   └── auth/           # NextAuth endpoints
│   ├── barbershops/        # Página de barbearias
│   │   └── [id]/          # Página de detalhes (dinâmica)
│   ├── bookings/           # Página de agendamentos
│   └── layout.tsx          # Layout raiz
├── prisma/
│   ├── schema.prisma       # Schema do banco de dados
│   └── seed.ts            # Dados iniciais
├── public/                 # Arquivos estáticos
└── package.json
```

### Convenção de Pastas

- `_actions/` - Server Actions para mutações de dados
- `_components/` - Componentes React reutilizáveis
- `_lib/` - Configurações e utilitários
- `_providers/` - Context Providers do React
- Pastas com `_` não criam rotas (organizacional)

---

## 🏗️ Padrões Arquiteturais

### Server Components vs Client Components

**Server Components** (padrão):
- Renderizados no servidor
- Acesso direto ao banco de dados
- Melhor performance e SEO
- Exemplos: `page.tsx`, listagens

**Client Components** (`"use client"`):
- Interatividade e hooks
- Event handlers
- Exemplos: formulários, calendário, botões

### Server Actions

Funções assíncronas que rodam no servidor:

```typescript
"use server"

export async function saveBooking(data: BookingData) {
  // Validação de sessão
  const session = await getServerSession(authOptions);

  // Operação no banco
  await db.booking.create({ data });

  // Revalidação de cache
  revalidatePath("/bookings");
}
```

---

## 🎨 Design System

### Cores
- **Primary**: Roxo (#8162FF)
- **Dark**: Tema escuro por padrão
- **Gray scale**: Tons de cinza para texto e fundos

### Componentes shadcn/ui
- Button
- Card
- Calendar
- Dialog / Sheet
- Avatar
- Alert Dialog
- Input

---

## 🔒 Segurança

- ✅ Autenticação OAuth 2.0 (Google)
- ✅ Proteção de rotas no servidor
- ✅ Validação de sessão em Server Actions
- ✅ Variáveis de ambiente para credenciais
- ✅ Sanitização de inputs com Zod

---

## 📱 Responsividade

O projeto é totalmente responsivo, funcionando perfeitamente em:
- 📱 Mobile (320px+)
- 📱 Tablet (768px+)
- 💻 Desktop (1024px+)

---

## 🤝 Contribuindo

Contribuições são bem-vindas! Sinta-se à vontade para:

1. Fazer um fork do projeto
2. Criar uma branch para sua feature (`git checkout -b feature/MinhaFeature`)
3. Commit suas mudanças (`git commit -m 'Add: MinhaFeature'`)
4. Push para a branch (`git push origin feature/MinhaFeature`)
5. Abrir um Pull Request

---

## 📄 Licença

Este projeto foi desenvolvido para fins educacionais durante a Full Stack Week.

---

## 👨‍💻 Autor

**Felipe Rocha**
- GitHub: [@felipemotarocha](https://github.com/felipemotarocha)
- YouTube: [Dicas Para Devs](https://www.youtube.com/@dicasparadevs)

---

## 🙏 Agradecimentos

- [Full Stack Club](https://fullstackclub.com.br) - Pela organização da Full Stack Week
- Comunidade React/Next.js
- Todos os contribuidores do projeto

---

<div align="center">
  Feito com 💜 por Felipe Rocha
</div>
