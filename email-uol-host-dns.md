# DNS de e-mail UOL Host para domínio externo (Registro.br)

Documentação da configuração final do e-mail `patricia@angelscomex.com.br` que está hospedado no UOL Host com domínio registrado no Registro.br.

---

## Contexto

- **Domínio:** angelscomex.com.br
- **Registrar/DNS:** Registro.br
- **E-mail:** UOL Host E-mail Profissional (caixa 50 GB)
- **Site:** Netlify (`tiny-zuccutto-152eab.netlify.app`)

## Problema original

Quando configuramos o DNS do site no Registro.br (A records + CNAME pro Netlify), os MX antigos do UOL caíram. Configurei MX errado na primeira tentativa.

### Erros que apareciam

**Recebendo e-mail PARA a Patricia:**
```
554 5.7.1 Relay access denied
host mx-p1.uol.com.br[200.147.35.219]
```

**Enviando e-mail DA Patricia:**
```
550 5.7.1 Envio não autorizado
```

## Causa raiz (importante pra próxima vez)

UOL Host tem **dois conjuntos diferentes** de servidores MX:

| Tipo de domínio | MX correto |
|------------------|------------|
| Registrado **dentro** do UOL Host | `mx-p1.uol.com.br` / `mx-p2.uol.com.br` |
| Registrado **fora** (Registro.br, etc) | `mx.uhserver.com` |

Eu havia configurado o primeiro grupo (errado pro caso da Patricia).

## Solução aplicada — DNS final no Registro.br

| Tipo | Nome | Dados |
|------|------|-------|
| MX | angelscomex.com.br | `10 mx.uhserver.com` |
| TXT | angelscomex.com.br | `v=spf1 include:spf.whservidor.com ~all` |
| CNAME | `pro._domainkey.angelscomex.com.br` | `pro._domainkey.uhserver.com` |
| CNAME | www.angelscomex.com.br | `tiny-zuccutto-152eab.netlify.app` |
| A | angelscomex.com.br | `75.2.60.5` |
| A | angelscomex.com.br | `99.83.190.102` |

**Observação:** o CNAME `pro._domainkey` é o DKIM, autentica os e-mails enviados (anti-phishing). Foi importante adicionar pra entregabilidade.

## Configuração no cliente de e-mail (Outlook, celular, etc.)

| Campo | Valor |
|-------|-------|
| Servidor IMAP (receber) | `imap.uhserver.com` porta 993 SSL/TLS |
| Servidor SMTP (enviar) | `smtps.uhserver.com` porta 465 SSL/TLS |
| Login | `patricia@angelscomex.com.br` |
| Senha | senha da caixa no painel UOL Host |

## Validação

Após salvar, DNS propagou em menos de 5 minutos. Teste de envio/recebimento confirmado.

Pra verificar no futuro:
- [mxtoolbox.com/MXLookup](https://mxtoolbox.com/MXLookup.aspx) → `angelscomex.com.br` → deve mostrar `mx.uhserver.com`
- [mxtoolbox.com/spf.aspx](https://mxtoolbox.com/spf.aspx) → deve mostrar `v=spf1 include:spf.whservidor.com ~all`

## Lição aprendida

Se um domínio com UOL Host email for migrar pra outro DNS provider (Registro.br, Cloudflare, etc.) e o site for hospedado em outro lugar (Netlify, Vercel, etc.), o checklist mínimo é:

1. ✅ Manter MX apontando pra `mx.uhserver.com` (NÃO `mx-p1/p2.uol.com.br`)
2. ✅ SPF: `v=spf1 include:spf.whservidor.com ~all`
3. ✅ DKIM: CNAME `pro._domainkey` → `pro._domainkey.uhserver.com`
4. ⚠️ Opcional: DMARC TXT em `_dmarc` (boa prática extra, anti-spoofing)
