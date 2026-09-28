# AWS Backend Setup

This workspace has three apps:

- `sibling-shoe-BE`: Serverless AWS backend
- `sibiling-site`: storefront on `http://localhost:3000`
- `sibiling-admin`: admin app on `http://localhost:3002`

## 1. Configure AWS credentials

Do not paste AWS keys into chat.

Open a new PowerShell window and set credentials only in that local terminal:

```powershell
$env:AWS_ACCESS_KEY_ID="YOUR_ACCESS_KEY_ID"
$env:AWS_SECRET_ACCESS_KEY="YOUR_SECRET_ACCESS_KEY"
$env:AWS_REGION="ap-south-1"
```

The AWS identity needs permission to create/update:

- CloudFormation
- IAM roles/policies
- Lambda
- API Gateway HTTP APIs
- DynamoDB
- Cognito User Pools
- CloudWatch Logs

For first local deployment, an admin/deployer IAM user is simplest. Tighten permissions before production.

## 2. Confirm R2 values

The backend `.env` should already contain:

```powershell
CLOUDFLARE_R2_ENDPOINT
CLOUDFLARE_R2_BUCKET_NAME
CLOUDFLARE_R2_ACCESS_KEY_ID
CLOUDFLARE_R2_SECRET_ACCESS_KEY
CLOUDFLARE_R2_PUBLIC_URL
```

## 3. Deploy backend

From the same PowerShell window where AWS credentials are set:

```powershell
cd C:\Users\rathi\OneDrive\Desktop\sibling_shoe\sibling-shoe-BE
npm run deploy:dev
```

Copy these outputs after deploy:

- `ApiEndpoint`
- `UserPoolId`
- `UserPoolClientId`
- `UserPoolDomain`

Current deployed `dev` values:

```powershell
ApiEndpoint=https://lmkpb95524.execute-api.ap-south-1.amazonaws.com
UserPoolId=ap-south-1_Zshcdfk6Y
UserPoolClientId=6778pjsjsl5v2t0hqvoksdf05c
UserPoolDomain=ecommerce-backend-dev-642433525591.auth.ap-south-1.amazoncognito.com
CarouselsTableName=sibling-shoe-carousels-dev
CategoriesTableName=sibling-shoe-categories-dev
ProductsTableName=sibling-shoe-products-dev
GeneralTableName=sibling-shoe-general-dev
```

## 4. Update frontend env files

Update both frontend env files:

```powershell
C:\Users\rathi\OneDrive\Desktop\sibling_shoe\sibiling-site\.env
C:\Users\rathi\OneDrive\Desktop\sibling_shoe\sibiling-admin\.env
```

Set:

```powershell
NEXT_PUBLIC_API_URL=<ApiEndpoint>
NEXT_PUBLIC_COGNITO_REGION=ap-south-1
NEXT_PUBLIC_COGNITO_USER_POOL_ID=<UserPoolId>
NEXT_PUBLIC_COGNITO_CLIENT_ID=<UserPoolClientId>
```

The storefront may only need `NEXT_PUBLIC_API_URL`; the admin needs the Cognito values.

## 5. Create an admin user

The admin app uses Cognito username/password auth and expects the user to be in the `Admins` group.

If AWS CLI is installed:

```powershell
aws cognito-idp admin-create-user `
  --region ap-south-1 `
  --user-pool-id <UserPoolId> `
  --username admin@example.com `
  --user-attributes Name=email,Value=admin@example.com Name=email_verified,Value=true

aws cognito-idp admin-set-user-password `
  --region ap-south-1 `
  --user-pool-id <UserPoolId> `
  --username admin@example.com `
  --password "Use-A-Strong-Password-123" `
  --permanent

aws cognito-idp admin-add-user-to-group `
  --region ap-south-1 `
  --user-pool-id <UserPoolId> `
  --username admin@example.com `
  --group-name Admins
```

If AWS CLI is not installed, do the same from the AWS Console:

1. Go to Cognito.
2. Open the deployed user pool.
3. Create a user with email verified.
4. Set a permanent password.
5. Add the user to the `Admins` group.

## 6. Restart apps

```powershell
cd C:\Users\rathi\OneDrive\Desktop\sibling_shoe\sibiling-site
npm run dev
```

```powershell
cd C:\Users\rathi\OneDrive\Desktop\sibling_shoe\sibiling-admin
npx next dev --port 3002
```
