# Installation and configuration

## Try with npx

You can try n8n without installing it using npx.
From the terminal, run:

```bash
npx n8n
```

## Install globally with npm

To install n8n (LTS) globally, use `npm`:

```bash
npm install n8n -g
```

To install or update to a specific version of n8n use the `@` syntax to specify the version. For example:

```bash
npm install -g n8n@0.126.1
```

To install `next` (beta):

```bash
npm install -g n8n@next
```

After the installation, start n8n by running:

```bash
n8n
# or
n8n start
```

## Updating

To update your n8n instance to the latest (LTS) version, run:

```bash
npm update -g n8n
```

To install the `next` version:

```bash
npm install -g n8n@next
```

Docs at: [npm | n8n Docs](https://docs.n8n.io/hosting/installation/npm/)

---

## Configuration steps

After installing n8n, you can configure it by setting environment variables or using a configuration file.

### Environment variables

```bash
WEBHOOK_URL
N8N_PORT
N8N_PROTOCOL 
N8N_SECURE_COOKIE=true
```

