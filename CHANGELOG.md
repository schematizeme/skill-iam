# Changelog — schematize-iam

Todas as mudanças relevantes deste pacote, no formato [Keep a Changelog](https://keepachangelog.com/pt-BR/1.1.0/),
com versionamento [SemVer](https://semver.org/lang/pt-BR/).

## [0.1.0] — 2026-10-14
Primeira versão. O desenho do broker `auth.<domínio>` vivia espalhado: o piso na `schematize-engineering`
(`references/iam.md`), o cliente na web, a implementação em cada linguagem e o teste no pentest, sem uma skill
dona do domínio e sem nenhum gate executável sobre o código de auth.

### Adicionado
- `SKILL.md` com 19 pisos de IAM (broker único, PKCE sempre, JWT assimétrico, `sub` imutável, 2FA baseline sem muro, passkey com fluxo real, argon2id-only, Y != X, ReBAC com outbox e PDP fail-closed, logout irreversível, CSPRNG, efeito externo só em prd) e a tabela de fronteira com engineering, pentest, api, web, mobile e as linguagens.
- `references/`: `topologia`, `protocolos`, `fatores`, `sessao`, `authz-rebac`, `recuperacao-risco`, `stack-versoes` (anexo volátil com fonte e data).
- `scripts/check-iam.sh`: 19 regras sobre código e config (implicit, ROPC, PKCE ausente/plain, redirect com curinga, hash rápido de senha, bcrypt, argon2i/d, JWT `none`/sem verificação/segredo fraco/HS*, token em storage, PRNG fraco, cookie sem HttpOnly, SameSite=None sem Secure, e-mail como ID, chave privada no repo); waiver `iam-ok:`. `check-iam.test.sh`: 24 casos com vermelhos plantados e verdes.
- `scripts/check-oidc.sh`: valida o discovery do IdP (arquivo ou URL) com `jq`; `check-oidc.test.sh`: 12 casos.
- `assets/authz/model.fga` (modelo ReBAC multi-tenant base, não validado por execução), `assets/CLAUDE.md`.
- Comandos `/iam-help`, `-load`, `-claude`, `-handoff`, `-cc`, `-review`, `-client`, `-authz`, `-recover`, `-audit`.
