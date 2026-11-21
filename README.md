# App skeleton

A starting point for running your own single sign-on server (OpenID Connect) with your own look.
You get a Docker Compose file and a `theme/` folder. The server itself comes from the SSO image.

## Start

1. Copy the settings: `cp .env.example .env`, then set `ADMIN_PASSWORD`.
2. Get the SSO image, either way:
   - build it: `docker compose build`. This builds the php-sso release named in `docker-compose.yml` straight from
     [GitHub](https://github.com/php-sso/php-sso), so you need no checkout. To build from a checkout of your own,
     set `SSO_SOURCE` in `.env` to its folder (for example `../php-sso`), or
   - pull a published one: set `SSO_IMAGE` in `.env`, then `docker compose pull`.
3. `docker compose up -d`
4. Open http://localhost:9000/admin and sign in as `admin` with your password. Create your groups,
   users and clients (your apps) there.

Discovery document for your apps: http://localhost:9000/.well-known/openid-configuration

To update php-sso, change the version after the `#` in the `build.context` of `docker-compose.yml` (the release it builds, for
example `v0.1.0`), then run `docker compose build` and `docker compose up -d`. Back up the `sso-data` volume first.

## Settings

`.env` holds the settings. The ones you are most likely to want are listed in `.env.example`:

- `SSO_LOGIN_IDENTIFIER`: what people type to sign in (username, email, or either).
- `SSO_MFA_POLICY`: who must set up two-step verification (optional, admins or everyone).
- `SSO_TRUSTED_PROXIES`: your reverse proxies, so IP rules see the visitor's address.
- `SSO_GOOGLE_*`, `SSO_GITHUB_*`, `SSO_OIDC_*`: sign in with another provider. Each one turns on when its
  client id is set.

SAML connections for a company's own identity provider, SCIM provisioning, consent notices and IP rules are set
up in the admin panel, not in `.env`.

## Make it yours

Everything people see on sign-in and in account settings is a Twig template in `theme/`.
Edit a file and reload the page; no rebuild or restart is needed.

- `theme/assets/theme.css`: colours, fonts and spacing, through a few CSS variables.
- `theme/assets/logo.svg`: your logo.
- `theme/templates/layout.html.twig`: header and footer of every page.

Sign-in is several pages, and this skeleton has a starter file for each one that can take extra text:

| File | Page |
| --- | --- |
| `login.html.twig` | Step 1: who are you (and the buttons for other providers) |
| `login_password.html.twig` | Step 2: the password |
| `login_mfa.html.twig` | Step 3, when asked: the code from the authenticator app |
| `mfa_setup.html.twig` | Setting up the authenticator app, when signing in and from the account page |
| `mfa_recovery.html.twig` | The recovery codes, shown once |
| `password_change.html.twig` | Choosing a new password, when an administrator requires it |
| `consent.html.twig` | A notice to accept before signing in |
| `account.html.twig` | The account page: profile, password, connected accounts, two-step verification, sessions |

Each starter extends the built-in page and changes one block, so the forms keep working. To change more, override
more blocks, or copy the built-in page from the SSO source (`templates/pages/`) and keep its field names.

The pages with no starter here are `no_access`, `device`, `logout_confirm`, `signed_out` and `error`. They have no
blocks to override, so adding a file replaces the whole page. Do that only when you want to write the page
yourself, and copy the built-in one as your starting point.

Every variable, block and form field is documented in [theme/README.md](theme/README.md). That file is a copy of
the SSO server's `docs/theming.md`: replace it when you update the server.

Keep `SSO_DEBUG=1` in `.env` while you work, so template mistakes show up in the browser. Turn it off in
production: a broken template then falls back to the built-in page instead of breaking sign-in.

## Going to production

- Put the server behind HTTPS (a reverse proxy such as Caddy or Traefik) and set `SSO_PUBLIC_URL` to the
  public `https://` address. Apps validate tokens against this issuer, so don't change it afterwards.
- Set `SSO_TRUSTED_PROXIES` to the proxy's address. Without it every request looks like it comes from the proxy,
  and IP rules and session addresses are wrong.
- Choose a strong `ADMIN_PASSWORD`, sign in once, then change it from the account page.
- Consider `SSO_MFA_POLICY=admins` or `everyone`, so two-step verification is set up from the first sign-in.
- Back up the `sso-data` volume. It holds the database and the signing keys.
- Themes are code that runs on the server. Only mount themes you trust.

## License

MIT. See [LICENSE](LICENSE).
