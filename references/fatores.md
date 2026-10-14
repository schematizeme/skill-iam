# Fatores, senha, passkeys e step-up

Dona: `schematize-iam`. Base: `schematize-engineering → references/iam.md` §2-§4. Onde houver
divergência, a base manda e esta página é o defeito.

## 1. Identidade antes de credencial

- `sub` = ID interno opaco e imutável (ULID/UUIDv7). **E-mail e telefone nunca são ID**: são
  identificadores 1..N por usuário, cada um com estado de verificação. Trocar e-mail não muda `sub`.
- Identificador só vale **verificado** (não loga nem recupera sem verificação).
- Ter 2+ e-mails é incentivado (resiliência a brick de provedor). **Nudge** de e-mail secundário:
  detecta o provedor do atual e sugere um de **outro provedor**, com tooltip explicando o motivo;
  é sugestão, nunca obrigação.
- SSO nunca é ponto único de falha: cadastro via SSO força >= 1 fator de recuperação local.
  **Account linking** só com prova de posse de ambos os lados; nunca por e-mail não verificado.

## 2. Tiers de fator (NIST SP 800-63B)

| Tier | Fatores | Uso |
|---|---|---|
| Alto (resistente a phishing) | **passkey/WebAuthn**, chave FIDO2, push aprovado | ações sensíveis, troca de fator, admin, cross-tenant, billing, recuperação |
| Médio | TOTP | reforço do 2º fator |
| Baixo (fallback) | **Email OTP**, SMS/voz | 2º fator baseline; não autoriza ação sensível sozinho |

## 3. A regra que mata o círculo infinito

**Senha + Email OTP já é 2FA baseline** e dá acesso baseline pleno desde o cadastro. Fator forte
(passkey/TOTP) é **incentivado (nudge) e exigido just-in-time só em ação sensível**. É **vetado** barrar
o login por falta de fator forte: não entra sem forte / não enrola forte sem entrar = bug de bootstrap.
Toda exigência degrada o sensível, não bloqueia o baseline, e tem rampa self-service a partir do
estado zero (prova por estado de conta: só senha+e-mail, fator forte perdido, SSO sem local, convidado).

## 4. Senha (padrão cultural, opcional por escolha)

- **argon2id** com o piso numérico da base: `m >= 19456 KiB`, `t >= 2`, `p = 1`, sal >= 16 B CSPRNG,
  saída >= 32 B; calibre para ~0,5-1 s no hardware real e registre em ADR. String codificada completa
  guardada (`$argon2id$v=19$m=...`), nunca só o digest.
- **argon2id-only para senha nova.** bcrypt/PBKDF2/scrypt só como **legado lido e regravado** em argon2id
  no próximo login bem-sucedido (rehash preguiçoso); o leitor legado leva waiver `iam-ok: legado-rehash`.
- Verificar a senha escolhida contra base de vazadas (HIBP, k-anonymity: só o prefixo do SHA-1 sai).
  Comprimento mínimo 8 (12+ recomendado), máximo >= 64, aceitar todos os caracteres, sem regra de
  composição e sem expiração periódica (NIST 800-63B).
- Pepper opcional, em KMS/env fora do banco. Nunca logar senha, hash, sal ou pepper.
- O seletor de modos permite marcar a senha como opcional e viver de passkey/OTP/app.

## 5. Passkeys/WebAuthn (núcleo, fluxo real)

- Registro (attestation) **e** asserção implementados de ponta a ponta; mapear a string `passkey => AAL_ALTO`
  sem fluxo é o furo comum.
- `rpId` = domínio do broker (`auth.<domínio>` ou o domínio registrável escolhido, uma vez e para
  sempre: mudar invalida as passkeys); `origin` conferida no servidor; `challenge` CSPRNG, single-use,
  com TTL curto; contador de assinatura e flags UV/UP conferidos; `userVerification: required` em
  ação sensível. Biblioteca madura de WebAuthn; nunca parser CBOR/COSE próprio.
- Permitir **várias passkeys por conta** e listar/remover (view de dispositivos). Passkeys sincronizadas
  contam como resistentes a phishing, mas sua recuperação depende do cofre do provedor: por isso a conta
  mantém o caminho de recuperação independente (`recuperacao-risco.md`).
- Conditional UI (autofill) no campo de login para fluxo passwordless-first.

## 6. TOTP, push e OTP

- TOTP (RFC 6238): segredo **cifrado at-rest** (não em claro no banco), janela +-1 passo, **anti-replay**
  (o mesmo código não vale duas vezes), comparação em tempo constante.
- Email/SMS OTP: 6 dígitos de CSPRNG, **single-use**, TTL curto (<= 10 min), um novo invalida o anterior,
  guardado hasheado, rate-limit por conta+IP+destino, backoff exponencial. Reenvio com cooldown e teto diário.
- Push-approval: mostra contexto (local, dispositivo, ação) e exige número/confirmação para barrar
  fadiga de MFA.

## 7. Escolha de método e invariante de troca

- Login: sem app -> Email OTP; com app -> pergunta app ou e-mail; vários fatores -> **lista todos** e o
  usuário escolhe.
- **Invariante:** para mutar o fator X, apresente um fator Y != X no maior AAL disponível. Toda mudança
  notifica todos os canais verificados. Remover o **último fator forte** é ação com **atraso cancelável**.

## 8. Step-up

Rotas sensíveis declaram o AAL mínimo e a idade máxima da autenticação (`max_age`). O PEP responde
`insufficient_user_authentication` (RFC 9470) com o `acr`/`max_age` exigidos; o cliente leva o usuário ao
broker, que pede o fator forte na hora (e deixa enrolar um, verificado pelo Email OTP). O token novo
carrega `aal` e `auth_time`.
