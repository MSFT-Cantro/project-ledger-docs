# OAuth Setup Instructions

This guide will help you set up Google and Microsoft OAuth authentication for your Project Ledger application.

## Prerequisites

1. **Google Cloud Console Account** - For Google OAuth
2. **Microsoft Azure Account** - For Microsoft OAuth
3. **Development Environment** - Backend running on `http://localhost:3001`, Frontend on `http://localhost:3000`

## Google OAuth Setup

### 1. Create a Google Cloud Project

1. Go to [Google Cloud Console](https://console.cloud.google.com/)
2. Create a new project or select an existing one
3. Enable the Google+ API or People API

### 2. Configure OAuth Consent Screen

1. Navigate to **APIs & Services** > **OAuth consent screen**
2. Choose **External** user type (for testing) or **Internal** (for G Suite)
3. Fill in required fields:
   - App name: "Project Ledger"
   - User support email: Your email
   - Developer contact information: Your email

### 3. Create OAuth Credentials

1. Go to **APIs & Services** > **Credentials**
2. Click **Create Credentials** > **OAuth 2.0 Client IDs**
3. Choose **Web application**
4. Configure:
   - **Name**: Project Ledger Web Client
   - **Authorized JavaScript origins**: `http://localhost:3000`, `http://localhost:3001`
   - **Authorized redirect URIs**: `http://localhost:3001/api/auth/google/callback`

5. Save and copy the **Client ID** and **Client Secret**

## Microsoft OAuth Setup

### 1. Register Application in Azure

1. Go to [Azure Portal](https://portal.azure.com/)
2. Navigate to **Azure Active Directory** > **App registrations**
3. Click **New registration**
4. Configure:
   - **Name**: Project Ledger
   - **Supported account types**: Accounts in any organizational directory and personal Microsoft accounts
   - **Redirect URI**: Web - `http://localhost:3001/api/auth/microsoft/callback`

### 2. Configure API Permissions

1. In your app registration, go to **API permissions**
2. Add a permission > **Microsoft Graph** > **Delegated permissions**
3. Add: `User.Read` (to read user profile)

### 3. Create Client Secret

1. Go to **Certificates & secrets**
2. Click **New client secret**
3. Add description and set expiration
4. Copy the **Value** (this is your client secret)

## Environment Configuration

### Backend (.env file)

Create or update `apps/backend/.env`:

```bash
# Database
DATABASE_URL="postgresql://username:password@localhost:5432/project_ledger_dev"

# JWT
JWT_SECRET="your-super-secret-jwt-key-here"

# Google OAuth
GOOGLE_CLIENT_ID="your-google-client-id.apps.googleusercontent.com"
GOOGLE_CLIENT_SECRET="your-google-client-secret"
GOOGLE_CALLBACK_URL="http://localhost:3001/api/auth/google/callback"

# Microsoft OAuth
MICROSOFT_CLIENT_ID="your-microsoft-application-id"
MICROSOFT_CLIENT_SECRET="your-microsoft-client-secret"
MICROSOFT_CALLBACK_URL="http://localhost:3001/api/auth/microsoft/callback"

# Frontend URL
FRONTEND_URL="http://localhost:3000"

# Server Configuration
PORT=3001
NODE_ENV="development"
```

### Frontend (.env file)

Create `apps/frontend/.env`:

```bash
REACT_APP_BACKEND_URL=http://localhost:3001
```

## Database Migration

Run the database migration to add OAuth support:

```bash
cd apps/backend
npx prisma migrate dev --name add_oauth_support
```

## Testing OAuth Flow

1. Start both backend and frontend servers
2. Navigate to login page
3. Click "Continue with Google" or "Continue with Microsoft"
4. Complete OAuth flow in popup/redirect
5. Should be redirected back to the application dashboard

## Production Configuration

For production deployment:

1. **Update callback URLs** in OAuth providers to use production domain
2. **Update environment variables** with production URLs
3. **Configure HTTPS** - OAuth providers require HTTPS in production
4. **Review OAuth consent screens** - Remove testing status for public use

## Troubleshooting

### Common Issues:

1. **Redirect URI mismatch**: Ensure callback URLs match exactly in OAuth provider settings
2. **CORS errors**: Make sure frontend URL is allowed in backend CORS configuration
3. **Token validation errors**: Check JWT_SECRET is set and consistent
4. **OAuth consent errors**: Verify OAuth consent screen is configured properly

### Debug Mode:

Enable debug logging in backend by adding:
```bash
DEBUG="passport*"
```

This will show detailed OAuth flow information in console.
