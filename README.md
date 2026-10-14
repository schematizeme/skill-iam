# schematize-iam — o broker de identidade da casa

> **Identidade não é e-mail, autorização não é identidade, e o login não é um muro.** O desenho do
> `auth.<domínio>`: OIDC/OAuth 2.1 com PKCE, passkeys no núcleo, 2FA baseline sem muro pré-login, sessão
> longa com refresh rotativo e logout irreversível, ReBAC multi-tenant, recuperação tão forte quanto o login
> e risk engine adaptativo.

Pacote de **skill normativa para [Claude Code](https://claude.com/claude-code)**.
Parte do catálogo **schematize skills**.

## Por que ela existe

O piso de IAM estava na `schematize-engineering`, o cliente na web, a implementação em cada linguagem e o
teste no pentest. Faltava a dona do domínio e um gate que lesse o código de auth: o implicit flow que sobra,
o `alg: none` aceito, o token em `localStorage`, o fluxo de código sem PKCE, o e-mail usado como ID.

## Instalar

```bash
schematize install iam    # requer o CLI schematize instalado
```

Manual:

```bash
git clone https://github.com/schematizeme/skill-iam.git /tmp/skill-iam
bash /tmp/skill-iam/install.sh .        # instala em .claude/ do projeto atual
```

## O que tem dentro

- **SKILL.md** — o contrato: 19 pisos inegociáveis, a fronteira com as outras skills e o mapa de references.
- **references/** — `topologia`, `protocolos`, `fatores`, `sessao`, `authz-rebac`, `recuperacao-risco`, `stack-versoes`.
- **scripts/** — `check-iam.sh` (anti-padrões em código e config) e `check-oidc.sh` (discovery do IdP), cada um com `.test.sh`.
- **assets/** — `authz/model.fga`, `commands/`, `CLAUDE.md`.

## Comandos

| Comando | O que faz |
|---|---|
| `/iam-help` | lista os comandos |
| `/iam-load` | carrega à força o corpo normativo |
| `/iam-review` | gates + leitura do que a máquina não lê |
| `/iam-client` | integra um cliente ao broker (code + PKCE) |
| `/iam-authz` | modela a autorização ReBAC de um serviço |
| `/iam-recover` | recuperação e step-up com prova de reachability |
| `/iam-audit` | cobertura contra o checklist F0-F6 da base |
| `/iam-claude` | `CLAUDE.md` sempre-on de IAM |
| `/iam-cc` · `/iam-handoff` | context compact / handoff |

## Verificação

```bash
bash scripts/check-iam.test.sh     # vermelho primeiro do gate de código
bash scripts/check-oidc.test.sh    # vermelho primeiro do gate de discovery
```

## Licença

MIT. Ver `LICENSE`.
