# QuizCap Deployment Checklist

Use this checklist to ensure a smooth deployment to Vercel.

## ✅ Pre-Deployment Checklist

### 1. Environment Variables
- [ ] Get Stripe publishable key from [Stripe Dashboard](https://dashboard.stripe.com/test/apikeys)
- [ ] Get Stripe secret key
- [ ] Create Stripe monthly price ID ([Products](https://dashboard.stripe.com/test/products))
- [ ] Create Stripe yearly price ID
- [ ] Set up Stripe webhook secret
- [ ] Determine production API base URL

### 2. Code Preparation
- [x] `vercel.json` configuration created
- [x] README.md updated with project info
- [x] DEPLOYMENT.md created with detailed instructions
- [ ] Review and update `index.html` title/description if needed
- [ ] Run `npm run check:safe` to validate (already passed ✓)

### 3. Git Repository
- [ ] Initialize git repository: `git init`
- [ ] Add all files: `git add .`
- [ ] Create initial commit: `git commit -m "Initial commit: QuizCap deployment"`
- [ ] Create GitHub repository
- [ ] Add remote: `git remote add origin <your-repo-url>`
- [ ] Push code: `git push -u origin main`

### 4. Vercel Setup
- [ ] Create Vercel account at [vercel.com](https://vercel.com)
- [ ] Import project from GitHub
- [ ] Verify framework detected as "Vite"
- [ ] Add environment variables in Vercel dashboard

### 5. Environment Variables in Vercel

Add these in **Vercel Dashboard → Project Settings → Environment Variables**:

#### Required Variables
```
VITE_STRIPE_PUBLISHABLE_KEY=pk_live_your_production_key
VITE_STRIPE_MONTHLY_PRICE_ID=prod_your_monthly_id
VITE_STRIPE_YEARLY_PRICE_ID=prod_your_yearly_id
VITE_API_BASE_URL=https://your-backend-api.com
```

For testing, use Stripe test keys:
```
VITE_STRIPE_PUBLISHABLE_KEY=pk_test_...
```

### 6. Deployment
- [ ] Click "Deploy" in Vercel
- [ ] Wait for build to complete (typically 1-2 minutes)
- [ ] Check deployment logs for errors
- [ ] Visit deployed URL to test

### 7. Post-Deployment Testing
- [ ] Homepage loads correctly
- [ ] Can paste study material
- [ ] Quiz generation works
- [ ] Questions display properly
- [ ] Answers can be selected
- [ ] Results screen shows
- [ ] Stripe integration works (if configured)
- [ ] Daily limit tracking works

### 8. Domain Setup (Optional)
- [ ] Add custom domain in Vercel
- [ ] Update DNS records
- [ ] Wait for SSL certificate
- [ ] Test custom domain

## 🚨 Common Issues

### Build Fails
**Issue:** Build fails on Vercel
**Solution:**
- Check build logs for specific errors
- Verify all environment variables are set
- Ensure Node.js version is 18+

### Routing Issues
**Issue:** 404 on page refresh
**Solution:**
- Verify `vercel.json` exists with rewrites configuration
- Check Vercel build logs

### API Errors
**Issue:** Quiz generation fails
**Solution:**
- Verify `VITE_API_BASE_URL` is correct
- Check backend API is running
- Review OpenAI API integration

### Stripe Errors
**Issue:** Payment flow doesn't work
**Solution:**
- Verify publishable key matches environment (test/live)
- Check price IDs exist in Stripe dashboard
- Configure webhook endpoints

## 📋 Build Configuration

Vercel should auto-detect these settings:

```
Framework Preset: Vite
Build Command: npm run build
Output Directory: dist
Install Command: npm install
Node Version: 18.x
```

## 🔄 Continuous Deployment

Once set up, Vercel will automatically:
- Deploy on every push to `main` branch
- Create preview deployments for pull requests
- Run build checks before deployment

## 📞 Need Help?

- **Vercel Docs:** [vercel.com/docs](https://vercel.com/docs)
- **Stripe Docs:** [stripe.com/docs](https://stripe.com/docs)
- **Project Docs:** See DEPLOYMENT.md for comprehensive guide

---

**Status:** Ready for deployment ✅

All TypeScript and ESLint checks passed. Project is configured and ready to deploy to Vercel.
