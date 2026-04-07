# QuizCap - Deployment Guide

## 🚀 Project Overview

**QuizCap** is an AI-powered quiz generation platform built with React 19 and Vite. It transforms study material into interactive "Who Wants to Be a Millionaire?"-style quizzes using OpenAI's GPT API.

## 🛠️ Technology Stack

### Frontend Framework
- **React 19** with TypeScript
- **Vite** (build tool and dev server)
- **TanStack Router** for file-based routing
- **TanStack Query** for server state management
- **Tailwind CSS v4** for styling
- **shadcn/ui** component library

### Backend & Services
- Platform-hosted persistent database with ORM
- OpenAI GPT API integration (MaaS_4.1 model)
- Stripe payment integration
- JWT authentication

### Key Features
- AI-powered quiz generation with progressive difficulty (basic, understanding, application)
- Freemium model (1 free quiz per day)
- Premium subscriptions (unlimited quizzes)
- Session persistence for quiz progress
- Auto or custom question count
- Responsive purple/gold gradient design

## 📦 Local Development Setup

### Prerequisites
- **Node.js** 18+ and npm
- Stripe account (for payment features)
- OpenAI API access (provided by platform)

### Installation Steps

1. **Install dependencies:**
   ```bash
   npm install
   ```

2. **Configure environment variables:**

   Copy `.env.example` to create `.env`:
   ```bash
   cp .env.example .env
   ```

   Update the following variables in `.env`:
   ```env
   # Stripe Configuration (get from https://dashboard.stripe.com/test/apikeys)
   VITE_STRIPE_PUBLISHABLE_KEY=pk_test_your_key_here
   VITE_STRIPE_MONTHLY_PRICE_ID=prod_your_monthly_id
   VITE_STRIPE_YEARLY_PRICE_ID=prod_your_yearly_id

   # Backend API Configuration
   VITE_API_BASE_URL=http://localhost:3001

   # Backend-only (DO NOT prefix with VITE_)
   STRIPE_SECRET_KEY=sk_test_your_secret_key
   STRIPE_WEBHOOK_SECRET=whsec_your_webhook_secret

   # Server Configuration
   PORT=3001
   FRONTEND_URL=http://localhost:3000
   ```

3. **Run development server:**
   ```bash
   npm run dev
   ```

   The app will be available at `http://localhost:3000`

4. **Validate your build:**
   ```bash
   npm run check:safe
   ```

   This runs TypeScript type checking and ESLint validation with a 20-second timeout.

### Development Commands

| Command | Description |
|---------|-------------|
| `npm run dev` | Start development server on port 3000 |
| `npm run build` | Build for production (runs check:safe first) |
| `npm run serve` | Preview production build locally |
| `npm run check:safe` | TypeScript + ESLint validation (20s timeout) |
| `npm run format` | Format code with Biome |

## 🌐 Vercel Deployment

### Prerequisites
- Vercel account ([vercel.com](https://vercel.com))
- GitHub/GitLab repository (recommended)

### Deployment Steps

#### Option 1: Deploy via Vercel Dashboard (Recommended)

1. **Push your code to GitHub/GitLab:**
   ```bash
   git add .
   git commit -m "Prepare for Vercel deployment"
   git push origin main
   ```

2. **Import project to Vercel:**
   - Go to [vercel.com/new](https://vercel.com/new)
   - Click "Import Project"
   - Select your repository
   - Vercel will auto-detect the framework (Vite)

3. **Configure environment variables:**

   In the Vercel dashboard, add these environment variables:
   ```
   VITE_STRIPE_PUBLISHABLE_KEY=pk_live_your_production_key
   VITE_STRIPE_MONTHLY_PRICE_ID=prod_your_monthly_id
   VITE_STRIPE_YEARLY_PRICE_ID=prod_your_yearly_id
   VITE_API_BASE_URL=https://your-backend-api.com
   ```

4. **Deploy:**
   - Click "Deploy"
   - Vercel will automatically build and deploy your project

#### Option 2: Deploy via Vercel CLI

1. **Install Vercel CLI:**
   ```bash
   npm install -g vercel
   ```

2. **Login to Vercel:**
   ```bash
   vercel login
   ```

3. **Deploy:**
   ```bash
   vercel
   ```

   For production deployment:
   ```bash
   vercel --prod
   ```

### Build Configuration

The project includes `vercel.json` with the following configuration:

```json
{
  "rewrites": [
    {
      "source": "/(.*)",
      "destination": "/index.html"
    }
  ],
  "headers": [
    {
      "source": "/assets/(.*)",
      "headers": [
        {
          "key": "Cache-Control",
          "value": "public, max-age=31536000, immutable"
        }
      ]
    }
  ]
}
```

This ensures:
- SPA routing works correctly (all routes serve `index.html`)
- Static assets are cached for 1 year

### Build Settings (Auto-detected by Vercel)

- **Framework Preset:** Vite
- **Build Command:** `npm run build`
- **Output Directory:** `dist`
- **Install Command:** `npm install`
- **Node Version:** 18.x (or higher)

## 🔒 Environment Variables

### Frontend Variables (prefixed with `VITE_`)
These are embedded in the client-side bundle and publicly accessible:

| Variable | Description | Required |
|----------|-------------|----------|
| `VITE_STRIPE_PUBLISHABLE_KEY` | Stripe public key | Yes |
| `VITE_STRIPE_MONTHLY_PRICE_ID` | Stripe monthly price ID | Yes |
| `VITE_STRIPE_YEARLY_PRICE_ID` | Stripe yearly price ID | Yes |
| `VITE_API_BASE_URL` | Backend API URL | Yes |

### Backend Variables (NO `VITE_` prefix)
These should only be used server-side and kept secret:

| Variable | Description | Required |
|----------|-------------|----------|
| `STRIPE_SECRET_KEY` | Stripe secret key | Yes |
| `STRIPE_WEBHOOK_SECRET` | Stripe webhook secret | Yes |
| `PORT` | Server port | No (defaults to 3001) |
| `FRONTEND_URL` | Frontend URL for CORS | Yes |

## 🗂️ Project Structure

```
/home/user/vite-template/
├── src/
│   ├── routes/              # File-based routing
│   │   ├── index.tsx        # Main quiz app page
│   │   └── __root.tsx       # Root layout
│   ├── components/
│   │   ├── ui/              # shadcn/ui components
│   │   ├── quiz/            # Quiz-specific components
│   │   │   ├── LessonInput.tsx
│   │   │   ├── QuizGame.tsx
│   │   │   ├── QuizResults.tsx
│   │   │   └── WelcomeScreen.tsx
│   │   └── QuizApp.tsx      # Main quiz orchestrator
│   ├── lib/
│   │   ├── quizService.ts   # Quiz generation & management
│   │   └── subscriptionService.ts
│   ├── sdk/
│   │   ├── api-clients/     # API client functions
│   │   ├── database/orm/    # Database ORM models
│   │   └── core/            # Core SDK utilities
│   ├── types/
│   │   └── quiz.ts          # TypeScript interfaces
│   ├── assets/              # Static assets
│   └── main.tsx             # App entry point
├── index.html               # HTML template
├── vercel.json             # Vercel deployment config
├── vite.config.js          # Vite configuration
├── package.json            # Dependencies & scripts
├── tailwind.config.js      # Tailwind CSS config
├── tsconfig.json           # TypeScript config
└── .env.example            # Environment template
```

## 🎨 Customization

### Update Branding

1. **App Title & Description:**
   Edit `index.html`:
   - Line 14: `<title>QuizCap - AI Learning Game</title>`
   - Line 10: `<meta name="description" content="...">`

2. **Theme Colors:**
   The app uses a purple/gold gradient theme:
   - Purple: `#2D0F55`, `#3A1478`, `#1E1B4D`
   - Gold: `#F59E0B`, `#D97706`, `#B45309`

   Update in component files or Tailwind config.

### Quiz Configuration

Edit `src/types/quiz.ts`:
```typescript
export const DAILY_QUIZ_LIMIT = 1;           // Free tier daily limit
export const DEFAULT_QUESTION_COUNT = 15;    // Default questions
```

## 🧪 Testing & Validation

### Type Checking
```bash
npm run check:safe
```

### Build Test
```bash
npm run build
npm run serve
```

Visit `http://localhost:4173` to test the production build.

## 📝 Important Notes

### Platform Integration
This app uses platform-hosted services:
- **Database:** Persistent ORM-based database (pre-configured)
- **API Clients:** OpenAI GPT integration via platform SDK
- **Authentication:** JWT-based auth context

### Stripe Configuration
To enable payments:
1. Create products in [Stripe Dashboard](https://dashboard.stripe.com/test/products)
2. Create monthly and yearly price IDs
3. Update environment variables
4. Configure webhook endpoints for subscription events

### Database Schema
The app uses these ORM models:
- `QuizSessionORM` - Quiz session tracking
- `QuizQuestionORM` - Individual questions
- `UserUsageTrackingORM` - Daily usage limits

## 🆘 Troubleshooting

### Build Fails on Vercel
- Check that all environment variables are set
- Verify Node.js version is 18+
- Check build logs for specific errors

### Routing Issues After Deployment
- Ensure `vercel.json` is present with rewrites configuration
- Verify SPA mode is enabled

### TypeScript Errors
Run locally first:
```bash
npm run check:safe
```

### Stripe Integration Issues
- Verify publishable key matches secret key (test/live)
- Check webhook secret is configured correctly
- Ensure price IDs exist in Stripe dashboard

## 📞 Support

For issues related to:
- **Platform SDK/Database:** Contact platform support
- **Stripe:** [Stripe Support](https://support.stripe.com)
- **Vercel:** [Vercel Support](https://vercel.com/support)

## 📄 License

This project is part of the CREAO platform. Contact your platform administrator for licensing details.
