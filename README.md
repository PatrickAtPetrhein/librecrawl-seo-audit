# librecrawl-seo-audit

Coolify deployment glue for the [LibreCrawl technical SEO audit MCP](https://github.com/adityaarsharma/librecrawl-technical-seo-audit-mcp) (MIT), the "Screaming Frog inside Claude" crawler used for astropet.de audits.

This repo holds one file, `docker-compose.yml`. It builds the two upstream images straight from GitHub at a pinned commit (no fork to maintain) and adds what Coolify cannot add to a compose service on its own:

- a Traefik router for `https://librecrawl.astropet.de` with **basic auth** in front of the MCP service (the upstream server has no authentication);
- a Traefik middleware that rewrites the `Host` header to `127.0.0.1:5081`, because the upstream server builds its FastMCP app without a `host`, which turns on the MCP SDK's localhost-only Host check and makes every proxied request fail with `421 Misdirected Request`.

## Where it runs

- Coolify Cloud (app.coolify.io), app `librecrawl-seo-audit`, Docker Compose build pack, auto-deploy off.
- Endpoint for MCP clients: `https://librecrawl.astropet.de/mcp` with HTTP basic auth (user `astropet`).
- Claude Code registration: `claude mcp add --transport http librecrawl https://librecrawl.astropet.de/mcp --header "Authorization: Basic <base64 user:password>"`.

## Updating upstream

Change the commit hash in both `build.context` lines, commit, and redeploy the Coolify app.

## Rotating the password

Replace the `basicauth.users` label. The value is an htpasswd entry in `{SHA}` form (no `$` characters, so neither Compose interpolation nor Coolify's label escaping touch it):

```
printf 'astropet:{SHA}%s\n' "$(printf '%s' "$PASSWORD" | openssl dgst -binary -sha1 | base64)"
```

Then update the header in the Claude Code MCP config and redeploy.
