sveltekit vendure storefront with starting point coming from this repo: https://github.com/pevey/sveltekit-storefront

vendure needs stripe plugin.

# SvelteKit eCommerce Store Starter App for Vendure

Currently, this starter uses Stripe checkout only.  Your backend must be configured for Stripe payments.

## Creating a project

```bash
# install degit
npm install -g degit

# create a new project in storefront
degit https://github.com/NikoJunttila/sveltekit-storefront.git storefront
```

## Installing packages

```bash
cd storefront
npm install
```

## Configuring a project

```bash
mv .env.example .env
```
- Open .env and add any required settings

- To make development a bit easier, you can change the urls in the EmailPlugin configuration options in vendure-config.ts to match the default SvelteKit dev url: http://localhost:5173:

`vendure-config.ts`
```js
verifyEmailAddressUrl: 'http://localhost:5173/verify',
passwordResetUrl: 'http://localhost:5173/auth',
changeEmailAddressUrl: 'http://localhost:5173/account'
```

## Run Codegen

YOU CANNOT SKIP THIS STEP.

To generate the types from your own backend to use in developing your frontend, you need to run codegen.  The types cannot be generated against a graphql endpoint in production mode (with introspection turned off).  Introsepction must be turned on.

```bash
npm codegen
```

This will output your types in src/lib/gql.

## Running the app

Make sure your backend is running first and that you configure the api urls in your .env file or your shell environment.

```bash
npm dev
```

For analytics get google tag manager add to .env, when tag is correctly configured<br><br>

For 1gb vm builds<br>
NODE_OPTIONS="--max-old-space-size=768" npm run build
