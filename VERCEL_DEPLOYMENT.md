# Vercel Deployment Guide for JioSaavn API

This guide will help you deploy the JioSaavn API to Vercel with zero-configuration.

## Prerequisites

- Git repository (GitHub, GitLab, or Bitbucket)
- Vercel account (https://vercel.com)
- Node.js 18.x or higher (Vercel handles this)

## Deployment Steps

### Option 1: Deploy via Vercel Dashboard (Recommended)

1. **Connect Your Repository**
   - Go to https://vercel.com/new
   - Click "Import Git Repository"
   - Select your GitHub/GitLab/Bitbucket account
   - Choose the `jiosaavn-api` repository

2. **Configure Project**
   - Framework Preset: **Other** (or leave empty)
   - Build Command: `npm run build` (should auto-detect)
   - Output Directory: `public` (should auto-detect)
   - Install Command: `npm install` (default)

3. **Environment Variables**
   - No environment variables required by default
   - Optional: Set `ENABLE_RATE_LIMIT=true` if you want rate limiting

4. **Deploy**
   - Click "Deploy"
   - Wait for deployment to complete (~2-3 minutes)
   - Your API will be live at: `https://your-project.vercel.app`

### Option 2: Deploy via Vercel CLI

1. **Install Vercel CLI**
   ```bash
   npm install -g vercel
   ```

2. **Login to Vercel**
   ```bash
   vercel login
   ```

3. **Deploy**
   ```bash
   vercel
   ```

4. **Answer Prompts**
   - Set up and deploy: `Y`
   - Which scope: Select your personal account or team
   - Link to existing project: `N` (first deploy)
   - Project name: `jiosaavn-api` (or your preference)
   - Detected: `TypeScript` (confirm)
   - Build command: `npm run build` (confirm)
   - Output directory: `public` (confirm)

5. **Production Deployment**
   ```bash
   vercel --prod
   ```

## Automatic Deployment

Once connected to Vercel, your project will:
- **Automatically deploy** on push to main/master branch
- **Create preview deployments** for pull requests
- **Support multiple environments** (staging, production)

## Post-Deployment

### Test Your API

After deployment, your API will be accessible at:
```
https://your-project.vercel.app
```

Test with:
```bash
# Search songs
curl "https://your-project.vercel.app/search/songs?query=Imagine&page=1&limit=5"

# Get home page
curl "https://your-project.vercel.app/"
```

### Update Your Documentation

Update references from `http://localhost:3000` to:
```
https://your-project.vercel.app
```

## Configuration

### vercel.json

The project includes a fully configured `vercel.json`:

```json
{
  "projectSettings": {
    "nodeVersion": "18.x"
  },
  "buildCommand": "npm run build",
  "outputDirectory": "public",
  "rewrites": [
    { "source": "(.*)", "destination": "/api" }
  ],
  "headers": [...]
}
```

This configuration ensures:
- ✅ Correct Node.js version
- ✅ TypeScript compilation
- ✅ Serverless function routing
- ✅ CORS headers
- ✅ Security headers
- ✅ Caching strategy

### Environment Variables

No environment variables are **required** by default.

Optional variables you can set in Vercel Dashboard:

| Variable | Default | Description |
|----------|---------|-------------|
| `NODE_ENV` | `production` | Set by Vercel automatically |
| `PORT` | `80` | Set by Vercel automatically |
| `ENABLE_RATE_LIMIT` | `false` | Enable request rate limiting |

To set environment variables:
1. Go to Project Settings → Environment Variables
2. Add variable and value
3. Redeploy (or push to trigger new deploy)

## Troubleshooting

### Issue: Build fails with TypeScript error

**Solution:**
```bash
# Ensure TypeScript is properly installed locally
npm install --save-dev typescript

# Try building locally first
npm run build
```

### Issue: "Cannot find module" errors

**Solution:**
```bash
# Clean and reinstall dependencies
rm -rf node_modules package-lock.json
npm install

# Commit and push
git add .
git commit -m "Reinstall dependencies"
git push
```

### Issue: 404 errors on all endpoints

**Solution:**
- Verify `vercel.json` rewrite rule: `"destination": "/api"`
- Check that `api/index.ts` exists
- Ensure Express app is exported correctly
- Check build output includes `/public` directory

### Issue: CORS errors

**Solution:**
- CORS is already configured in `vercel.json`
- If still having issues, verify headers in vercel.json
- Check browser console for specific error

### Issue: Timeout errors (503)

**Solution:**
- API calls to JioSaavn might be slow
- Default timeout: 9.5 seconds
- Optimize requests (use pagination, cache results)
- Check network connectivity

### Issue: Rate limiting issues

**Solution:**
- Set `ENABLE_RATE_LIMIT=true` only if needed
- Implement request queuing in your client
- Use exponential backoff for retries

## Monitoring & Logs

### View Logs

1. **Vercel Dashboard**
   - Go to your project → Deployments
   - Click on deployment
   - View logs in real-time

2. **Via CLI**
   ```bash
   vercel logs
   ```

### Performance Monitoring

Vercel provides:
- ✅ Response time analytics
- ✅ Error rate tracking
- ✅ Function execution time
- ✅ Memory usage

View in: Project → Analytics

## Optimization Tips

### 1. Caching

The `vercel.json` includes caching headers:
```
Cache-Control: public, s-maxage=300, stale-while-revalidate=600
```

This caches responses for:
- 5 minutes on Vercel Edge
- Up to 10 minutes stale responses

### 2. Compression

Vercel automatically compresses responses with gzip/brotli.

### 3. CDN

Your API is automatically distributed across Vercel's global CDN.

### 4. Database (Optional)

If you need to add a database:
- Use Vercel KV (Redis) for caching
- Use Vercel Postgres for persistent storage
- Update `.env` variables in Vercel Dashboard

## Scaling

### Free Tier
- ✅ 100,000 function invocations/month
- ✅ 100 GB bandwidth/month
- ✅ Good for testing and small projects

### Pro Tier
- ✅ Unlimited function invocations
- ✅ Unlimited bandwidth
- ✅ Advanced analytics
- ✅ Priority support

For Vercel pricing: https://vercel.com/pricing

## Custom Domain

To add a custom domain:

1. **In Vercel Dashboard**
   - Project → Settings → Domains
   - Add your domain

2. **Update DNS Records**
   - Follow Vercel's instructions for your DNS provider
   - Typical records:
     ```
     A: 76.76.19.163
     AAAA: 2606:4700:4700::1111
     ```

3. **SSL Certificate**
   - Automatically provisioned by Vercel
   - No additional setup needed

## Security Best Practices

✅ **Already Implemented:**
- CORS headers properly configured
- Security headers (X-Frame-Options, X-Content-Type-Options)
- HTTPS enforced by default
- Input validation on all endpoints
- Error handling that doesn't expose internals

📋 **Additional Recommendations:**
1. Enable IP filtering if needed (Pro plan)
2. Use environment variables for sensitive data
3. Monitor logs for suspicious activity
4. Keep dependencies updated
5. Enable preview deployments protection (Pro plan)

## Continuous Deployment Workflow

### Git Workflow

```bash
# Create feature branch
git checkout -b feature/new-endpoint

# Make changes
# ... edit files ...

# Commit
git add .
git commit -m "Add new endpoint"

# Push (creates preview deployment)
git push origin feature/new-endpoint

# Test preview deployment
curl "https://jiosaavn-api-pr-123.vercel.app/search/songs?query=test"

# Create Pull Request on GitHub

# After review and approval, merge to main
git merge feature/new-endpoint
git push origin main

# Automatic production deployment happens!
```

## Rollback

If you need to revert to a previous deployment:

1. Go to Vercel Dashboard → Deployments
2. Find the previous working deployment
3. Click "Promote to Production"

Or via CLI:
```bash
vercel list --prod
vercel promote <deployment-url>
```

## API Reference After Deployment

All endpoints work exactly the same:

```bash
# Before: http://localhost:3000/search/songs?query=Imagine
# After:  https://your-project.vercel.app/search/songs?query=Imagine

# All 15 endpoints available:
GET /                                    # Landing page
GET /search/all                          # Search everything
GET /search/songs                        # Search songs
GET /search/albums                       # Search albums
GET /search/playlists                    # Search playlists
GET /search/artists                      # Search artists
GET /songs                               # Get song details
GET /albums                              # Get album details
GET /artists                             # Get artist details
GET /artists/:artistId/songs             # Artist songs
GET /artists/:artistId/albums            # Artist albums
GET /artists/:artistId/recommendations/:songId  # Top songs
GET /playlists                           # Get playlist
GET /lyrics                              # Get lyrics
GET /modules                             # Get modules
```

## Support & Documentation

- **Vercel Docs:** https://vercel.com/docs
- **Vercel CLI:** https://vercel.com/cli
- **Project Docs:** https://docs.saavn.me
- **GitHub Issues:** https://github.com/sumitkolhe/jiosaavn-api/issues

## Common Vercel Features

### Environment-Specific Deployments

```yaml
# .vercel/project.json (auto-generated)
{
  "projectId": "your-project-id",
  "orgId": "your-org-id"
}
```

### Analytics

Vercel provides built-in analytics:
- Page views
- Response times
- Error rates
- Top pages

Access at: Project → Analytics

### Metrics

Real-time metrics available for:
- CPU usage
- Memory usage
- Request count
- Error count

## Verification Checklist

Before considering deployment complete:

- ✅ Build completes without errors
- ✅ All endpoints accessible
- ✅ CORS headers working
- ✅ Search endpoints return results
- ✅ Song/Album/Artist endpoints work
- ✅ Lyrics endpoint working
- ✅ Modules endpoint working
- ✅ Error handling working (test invalid ID)
- ✅ Performance acceptable (<2s response time)
- ✅ Custom domain working (if configured)

## Quick Deployment Checklist

Before deploying:

- [ ] TypeScript builds locally: `npm run build`
- [ ] No TypeScript errors: `npm run build`
- [ ] Dependencies updated: `npm update`
- [ ] `.env.example` created with all variables
- [ ] `vercel.json` properly configured
- [ ] `api/index.ts` exports Express app
- [ ] All routes tested locally
- [ ] Git committed and pushed

## Next Steps

1. Deploy to Vercel
2. Share your API URL with others
3. Add custom domain (optional)
4. Monitor performance in Vercel Dashboard
5. Set up GitHub notifications for deployments

---

**Your JioSaavn API is now production-ready! 🚀**

For questions, check: https://docs.saavn.me
