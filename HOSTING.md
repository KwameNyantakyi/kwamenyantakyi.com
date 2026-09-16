# Hosting and DNS for kwamenyantakyi.com

Last verified: 16 September 2026.

## Where things live

| Piece | Where | Notes |
|---|---|---|
| Site files | This repo, branch `main`, root folder | Static export. `CNAME` file must contain `kwamenyantakyi.com`. |
| Hosting | GitHub Pages | Settings → Pages. Custom domain `kwamenyantakyi.com`, "Enforce HTTPS" on. |
| Domain registrar and DNS | Names.co.uk (team.blue) | Log in at admin.names.co.uk. Whois shows "Register S.p.A.", which is the same company. |
| Nameservers | `ns0.phase8.net`, `ns1.phase8.net`, `ns2.phase8.net` | Names.co.uk's own. Set on the domain's Nameservers page, not the account-level "Default nameservers" page. |

The site is not on AWS. Nothing in any AWS account is needed for it.

## DNS records (Names.co.uk → Manage DNS)

| Type | Host | Value |
|---|---|---|
| A | @ | 185.199.108.153 |
| A | @ | 185.199.109.153 |
| A | @ | 185.199.110.153 |
| A | @ | 185.199.111.153 |
| CNAME | www | kwamenyantakyi.github.io |

Do not use Names.co.uk "web forwarding" for this domain.

## Publishing a change

Push to `main`. GitHub Pages rebuilds in about a minute. Check status with:

```
gh api repos/KwameNyantakyi/kwamenyantakyi.com/pages --jq '{status, cname, https_enforced}'
```

## If the site goes down

1. `dig +short A kwamenyantakyi.com` should return the four 185.199.x.x addresses.
   If it returns nothing, the nameservers or records above have changed at Names.co.uk.
2. `dig +norecurse +noall +authority NS kwamenyantakyi.com @a.gtld-servers.net` shows what
   the registry thinks the nameservers are. They must be the three phase8 ones.
3. If DNS is right but the page 404s, check the `CNAME` file is still in the repo root and
   the Pages source is still `main` / root.

## If HTTPS stops working or a new certificate never appears

GitHub issues the Let's Encrypt certificate itself. After a DNS change it can pass its own
domain check yet never request the certificate. The fix that worked in September 2026:

```
# remove the custom domain, wait a minute, add it back
gh api -X PUT repos/KwameNyantakyi/kwamenyantakyi.com/pages --input - <<< '{"cname":null}'
sleep 75
gh api -X PUT repos/KwameNyantakyi/kwamenyantakyi.com/pages --input - <<< '{"cname":"kwamenyantakyi.com"}'

# watch for "approved", then enforce HTTPS
gh api repos/KwameNyantakyi/kwamenyantakyi.com/pages --jq '.https_certificate'
gh api -X PUT repos/KwameNyantakyi/kwamenyantakyi.com/pages --input - <<< '{"https_enforced":true}'
```

The same steps work in the browser: Settings → Pages → clear the custom domain → Save →
wait a minute → enter it again → Save → tick "Enforce HTTPS" once the certificate shows.

GitHub's own DNS check is at `gh api repos/KwameNyantakyi/kwamenyantakyi.com/pages/health`.
The first call only queues the check and returns `{}`; call it again after 20 seconds.

## Other domains, for reference

- vorim.ai: registrar and DNS at GoDaddy, hosted on EC2 in the vorim.ai AWS account.
- agpwrld.com: registrar GoDaddy; hosting not yet restored after the old AWS account closed.
