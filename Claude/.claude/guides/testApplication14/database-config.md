# Step 3: Database Configuration

Configure the database connection for TestApplication by creating the necessary environment files.

## Required Information

Before proceeding, gather your database configuration:
- Database host and port (e.g., 127.0.0.1:3306)
- Database username and password
- Database name pattern (e.g., plugin_name for the base name)

## Create Environment Files

### 1. Create tests/TestApplication/.env

```bash
cat > tests/TestApplication/.env << 'EOF'
###> sylius/test-application ###
SYLIUS_TEST_APP_CONFIGS_TO_IMPORT="../../../../tests/TestApplication/config/config.yaml"
SYLIUS_TEST_APP_ROUTES_TO_IMPORT="../../../../tests/TestApplication/config/routes.yaml"
SYLIUS_TEST_APP_BUNDLES_PATH="tests/TestApplication/config/bundles.php"
###< sylius/test-application ###

###> doctrine/doctrine-bundle ###
DATABASE_URL=mysql://root:root@127.0.0.1:3306/plugin_name_%kernel.environment%?serverVersion=8.0
###< doctrine/doctrine-bundle ###

# Add plugin-specific environment variables here (only if needed)
###> your-plugin ###
###< your-plugin ###
EOF
```

### 2. Create tests/TestApplication/.env.test

```bash
cat > tests/TestApplication/.env.test << 'EOF'
###> sylius/test-application ###
SYLIUS_TEST_APP_CONFIGS_TO_IMPORT="../../../../tests/TestApplication/config/config.yaml"
SYLIUS_TEST_APP_ROUTES_TO_IMPORT="../../../../tests/TestApplication/config/routes.yaml"
SYLIUS_TEST_APP_BUNDLES_PATH="tests/TestApplication/config/bundles.php"
###< sylius/test-application ###

# Add plugin-specific environment variables here (only if needed)
###> your-plugin ###
###< your-plugin ###
EOF
```

### 3. Create tests/TestApplication/.env.local

**Not committed** - your specific database configuration:

```bash
cat > tests/TestApplication/.env.local << 'EOF'
###> doctrine/doctrine-bundle ###
DATABASE_URL=mysql://YOUR_USER:YOUR_PASSWORD@YOUR_HOST:YOUR_PORT/YOUR_DATABASE_%kernel.environment%?serverVersion=8.0
###< doctrine/doctrine-bundle ###

# Add plugin-specific environment overrides here (only if needed)
###> your-plugin ###
###< your-plugin ###
EOF
```

### 4. Create tests/TestApplication/.env.test.local

**Not committed** - your specific test database configuration:

```bash
cat > tests/TestApplication/.env.test.local << 'EOF'
###> doctrine/doctrine-bundle ###
DATABASE_URL=mysql://YOUR_USER:YOUR_PASSWORD@YOUR_HOST:YOUR_PORT/YOUR_DATABASE_%kernel.environment%?serverVersion=8.0
###< doctrine/doctrine-bundle ###

# Add plugin-specific environment overrides here (only if needed)
###> your-plugin ###
###< your-plugin ###
EOF
```

## Important Notes

- Replace `YOUR_USER`, `YOUR_PASSWORD`, `YOUR_HOST`, `YOUR_PORT`, and `YOUR_DATABASE` with your actual values
- The `.env.local` and `.env.test.local` files are already in `.gitignore` and won't be committed
- Choose the correct path structure based on your project (new vs old structure)

## Path Structure Guidelines

**For new structure** (config files in `tests/TestApplication/config/`):
- Use the paths shown above with `../../../../tests/TestApplication/config/`

**For old structure** (if migrating from `tests/Application/config/`):
- Use `@YourPlugin/tests/Application/config/config.yaml` format instead

## Commit Changes

After creating the environment files:

```bash
git add tests/TestApplication/.env tests/TestApplication/.env.test
git commit -m "feat: configure database for testApplication

- Add testApplication environment configuration files
- Configure database connection patterns
- Set up test-specific environment variables"
```

## Next Step

Continue to [testapp-config.md](testapp-config.md) to set up the TestApplication configuration files.