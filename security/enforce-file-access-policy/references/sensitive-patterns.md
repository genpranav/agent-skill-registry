# Sensitive Name Patterns

Canonical list of name fragments that must be covered by deny rules. Read this
file in step 3 of `SKILL.md` before building the diff against existing rules.

Each fragment requires **five deny rules** — one per tool verb — in the form
`Verb(**/*<fragment>*)`. See the complete JSON array at the bottom of this file.

---

## Fragment list

Organised by concept. Separator variants (hyphen, underscore, concatenated) are
listed explicitly so the validation step can check each one individually.

### Passwords
| Fragment | Matches |
|---|---|
| `password` | password.txt, my-password.env, database-password.conf |
| `passwords` | passwords.json, all-passwords.txt |
| `passwd` | passwd, .passwd, system-passwd |

### Secrets
| Fragment | Matches |
|---|---|
| `secret` | secret.key, app-secret.env |
| `secrets` | secrets.json, .secrets, k8s-secrets.yaml |

### Credentials
| Fragment | Matches |
|---|---|
| `credential` | credential.json |
| `credentials` | credentials.json, aws-credentials, gcp-credentials |

### API keys
| Fragment | Matches |
|---|---|
| `apikey` | apikey.txt, myapikey.env |
| `api-key` | api-key.json, my-api-key.env |
| `api_key` | api_key.txt, stripe_api_key.env |

### Access tokens
| Fragment | Matches |
|---|---|
| `access-token` | access-token.json |
| `access_token` | access_token.env |
| `accesstoken` | accesstoken.txt |

### OAuth tokens
| Fragment | Matches |
|---|---|
| `oauth-token` | oauth-token.json |
| `oauth_token` | oauth_token.env |

### JWT
| Fragment | Matches |
|---|---|
| `jwt` | jwt.txt, my-jwt.env, jwt-token.json, jwt_token.json |

### Bearer tokens
| Fragment | Matches |
|---|---|
| `bearer-token` | bearer-token.txt |
| `bearer_token` | bearer_token.env |

### Private keys
| Fragment | Matches |
|---|---|
| `private-key` | private-key.pem |
| `private_key` | private_key.pem |
| `privatekey` | privatekey.pem |

### SSH keys
| Fragment | Matches |
|---|---|
| `ssh-key` | ssh-key, id_rsa-ssh-key |
| `ssh_key` | ssh_key.pem |

### Service accounts
| Fragment | Matches |
|---|---|
| `service-account` | service-account.json |
| `service_account` | service_account.json |

### Vault tokens
| Fragment | Matches |
|---|---|
| `vault-token` | vault-token.txt |
| `vault_token` | vault_token.env |

### Git credentials
| Fragment | Matches |
|---|---|
| `git-credentials` | .git-credentials |

### Cloud credentials
| Fragment | Matches |
|---|---|
| `aws-credentials` | .aws-credentials, aws-credentials.json |
| `gcp-credentials` | gcp-credentials.json |
| `azure-credentials` | azure-credentials.json |

### Database passwords
| Fragment | Matches |
|---|---|
| `database-password` | database-password.conf |
| `db-password` | db-password.env |
| `db_password` | db_password.env |

### Client secrets
| Fragment | Matches |
|---|---|
| `client-secret` | client-secret.json |
| `client_secret` | client_secret.env |

### Signing keys
| Fragment | Matches |
|---|---|
| `signing-key` | signing-key.pem |
| `signing_key` | signing_key.pem |

### Encryption keys
| Fragment | Matches |
|---|---|
| `encryption-key` | encryption-key.bin |
| `encryption_key` | encryption_key.bin |

### Master keys
| Fragment | Matches |
|---|---|
| `master-key` | master-key.pem |
| `master_key` | master_key.env |

### Root keys
| Fragment | Matches |
|---|---|
| `root-key` | root-key.pem |
| `root_key` | root_key.env |

---

## Notes on coverage

**Why list separator variants separately?**
Glob patterns do not support alternation (`{-,_}`). Each separator variant
(`api-key`, `api_key`, `apikey`) is a distinct string that requires its own
rule. The validation step must check each variant individually — do not treat
them as interchangeable.

**Do these patterns cover subdirectory names?**
`**/*fragment*` matches a path component containing the fragment anywhere in
the tree. It covers:
- Files named `*fragment*` at any depth — `secrets.json`, `dir/secrets.json`.
- Directories named `*fragment*` — `secrets/`, `my-secrets/`.

Files *inside* a directory named `*fragment*` (e.g. `secrets/apikey.txt`)
are matched because `apikey.txt` itself contains the `apikey` fragment, not
because of the parent directory. If you need to block all files under a
secrets-named directory regardless of their own name, add companion rules of
the form `Read(**/*fragment*/**)`. Those rules are not in the required
baseline, but document their absence in the report if relevant.

**Absolute paths:**
Rules using `../` block relative traversal above the repo root. They do not
block absolute paths like `/etc/passwd` or `C:\Windows\System32`. Document
this gap in the report.

**Shell tool bypass:**
`Bash` and `PowerShell` deny rules match against the full command string, not
a file path, so per-fragment rules (as in Group B) are not applicable to them.
The required baseline adds `Bash(*../**)` and `PowerShell(*../**)` to block
path traversal via shell. Commands that enumerate or read files within the
repo (`ls`, `cat`, `find`, `Get-Content`) remain unrestricted unless the user
chooses one of the Group C hardening options in `SKILL.md` step 3. Document
this residual exposure in the report regardless of which option is chosen.

---

## Complete required JSON array

Paste this as the value of `permissions.deny` when creating from scratch, or
diff individual entries against an existing array during validation.

```json
[
  "Read(../**)",
  "Edit(../**)",
  "Write(../**)",
  "Glob(../**)",
  "Grep(../**)",
  "Bash(*../**)",
  "PowerShell(*../**)",

  "Read(**/*password*)",
  "Edit(**/*password*)",
  "Write(**/*password*)",
  "Glob(**/*password*)",
  "Grep(**/*password*)",

  "Read(**/*passwords*)",
  "Edit(**/*passwords*)",
  "Write(**/*passwords*)",
  "Glob(**/*passwords*)",
  "Grep(**/*passwords*)",

  "Read(**/*passwd*)",
  "Edit(**/*passwd*)",
  "Write(**/*passwd*)",
  "Glob(**/*passwd*)",
  "Grep(**/*passwd*)",

  "Read(**/*secret*)",
  "Edit(**/*secret*)",
  "Write(**/*secret*)",
  "Glob(**/*secret*)",
  "Grep(**/*secret*)",

  "Read(**/*secrets*)",
  "Edit(**/*secrets*)",
  "Write(**/*secrets*)",
  "Glob(**/*secrets*)",
  "Grep(**/*secrets*)",

  "Read(**/*credential*)",
  "Edit(**/*credential*)",
  "Write(**/*credential*)",
  "Glob(**/*credential*)",
  "Grep(**/*credential*)",

  "Read(**/*credentials*)",
  "Edit(**/*credentials*)",
  "Write(**/*credentials*)",
  "Glob(**/*credentials*)",
  "Grep(**/*credentials*)",

  "Read(**/*apikey*)",
  "Edit(**/*apikey*)",
  "Write(**/*apikey*)",
  "Glob(**/*apikey*)",
  "Grep(**/*apikey*)",

  "Read(**/*api-key*)",
  "Edit(**/*api-key*)",
  "Write(**/*api-key*)",
  "Glob(**/*api-key*)",
  "Grep(**/*api-key*)",

  "Read(**/*api_key*)",
  "Edit(**/*api_key*)",
  "Write(**/*api_key*)",
  "Glob(**/*api_key*)",
  "Grep(**/*api_key*)",

  "Read(**/*access-token*)",
  "Edit(**/*access-token*)",
  "Write(**/*access-token*)",
  "Glob(**/*access-token*)",
  "Grep(**/*access-token*)",

  "Read(**/*access_token*)",
  "Edit(**/*access_token*)",
  "Write(**/*access_token*)",
  "Glob(**/*access_token*)",
  "Grep(**/*access_token*)",

  "Read(**/*accesstoken*)",
  "Edit(**/*accesstoken*)",
  "Write(**/*accesstoken*)",
  "Glob(**/*accesstoken*)",
  "Grep(**/*accesstoken*)",

  "Read(**/*oauth-token*)",
  "Edit(**/*oauth-token*)",
  "Write(**/*oauth-token*)",
  "Glob(**/*oauth-token*)",
  "Grep(**/*oauth-token*)",

  "Read(**/*oauth_token*)",
  "Edit(**/*oauth_token*)",
  "Write(**/*oauth_token*)",
  "Glob(**/*oauth_token*)",
  "Grep(**/*oauth_token*)",

  "Read(**/*jwt*)",
  "Edit(**/*jwt*)",
  "Write(**/*jwt*)",
  "Glob(**/*jwt*)",
  "Grep(**/*jwt*)",

  "Read(**/*jwt-token*)",
  "Edit(**/*jwt-token*)",
  "Write(**/*jwt-token*)",
  "Glob(**/*jwt-token*)",
  "Grep(**/*jwt-token*)",

  "Read(**/*jwt_token*)",
  "Edit(**/*jwt_token*)",
  "Write(**/*jwt_token*)",
  "Glob(**/*jwt_token*)",
  "Grep(**/*jwt_token*)",

  "Read(**/*bearer-token*)",
  "Edit(**/*bearer-token*)",
  "Write(**/*bearer-token*)",
  "Glob(**/*bearer-token*)",
  "Grep(**/*bearer-token*)",

  "Read(**/*bearer_token*)",
  "Edit(**/*bearer_token*)",
  "Write(**/*bearer_token*)",
  "Glob(**/*bearer_token*)",
  "Grep(**/*bearer_token*)",

  "Read(**/*private-key*)",
  "Edit(**/*private-key*)",
  "Write(**/*private-key*)",
  "Glob(**/*private-key*)",
  "Grep(**/*private-key*)",

  "Read(**/*private_key*)",
  "Edit(**/*private_key*)",
  "Write(**/*private_key*)",
  "Glob(**/*private_key*)",
  "Grep(**/*private_key*)",

  "Read(**/*privatekey*)",
  "Edit(**/*privatekey*)",
  "Write(**/*privatekey*)",
  "Glob(**/*privatekey*)",
  "Grep(**/*privatekey*)",

  "Read(**/*ssh-key*)",
  "Edit(**/*ssh-key*)",
  "Write(**/*ssh-key*)",
  "Glob(**/*ssh-key*)",
  "Grep(**/*ssh-key*)",

  "Read(**/*ssh_key*)",
  "Edit(**/*ssh_key*)",
  "Write(**/*ssh_key*)",
  "Glob(**/*ssh_key*)",
  "Grep(**/*ssh_key*)",

  "Read(**/*service-account*)",
  "Edit(**/*service-account*)",
  "Write(**/*service-account*)",
  "Glob(**/*service-account*)",
  "Grep(**/*service-account*)",

  "Read(**/*service_account*)",
  "Edit(**/*service_account*)",
  "Write(**/*service_account*)",
  "Glob(**/*service_account*)",
  "Grep(**/*service_account*)",

  "Read(**/*vault-token*)",
  "Edit(**/*vault-token*)",
  "Write(**/*vault-token*)",
  "Glob(**/*vault-token*)",
  "Grep(**/*vault-token*)",

  "Read(**/*vault_token*)",
  "Edit(**/*vault_token*)",
  "Write(**/*vault_token*)",
  "Glob(**/*vault_token*)",
  "Grep(**/*vault_token*)",

  "Read(**/*git-credentials*)",
  "Edit(**/*git-credentials*)",
  "Write(**/*git-credentials*)",
  "Glob(**/*git-credentials*)",
  "Grep(**/*git-credentials*)",

  "Read(**/*aws-credentials*)",
  "Edit(**/*aws-credentials*)",
  "Write(**/*aws-credentials*)",
  "Glob(**/*aws-credentials*)",
  "Grep(**/*aws-credentials*)",

  "Read(**/*gcp-credentials*)",
  "Edit(**/*gcp-credentials*)",
  "Write(**/*gcp-credentials*)",
  "Glob(**/*gcp-credentials*)",
  "Grep(**/*gcp-credentials*)",

  "Read(**/*azure-credentials*)",
  "Edit(**/*azure-credentials*)",
  "Write(**/*azure-credentials*)",
  "Glob(**/*azure-credentials*)",
  "Grep(**/*azure-credentials*)",

  "Read(**/*database-password*)",
  "Edit(**/*database-password*)",
  "Write(**/*database-password*)",
  "Glob(**/*database-password*)",
  "Grep(**/*database-password*)",

  "Read(**/*db-password*)",
  "Edit(**/*db-password*)",
  "Write(**/*db-password*)",
  "Glob(**/*db-password*)",
  "Grep(**/*db-password*)",

  "Read(**/*db_password*)",
  "Edit(**/*db_password*)",
  "Write(**/*db_password*)",
  "Glob(**/*db_password*)",
  "Grep(**/*db_password*)",

  "Read(**/*client-secret*)",
  "Edit(**/*client-secret*)",
  "Write(**/*client-secret*)",
  "Glob(**/*client-secret*)",
  "Grep(**/*client-secret*)",

  "Read(**/*client_secret*)",
  "Edit(**/*client_secret*)",
  "Write(**/*client_secret*)",
  "Glob(**/*client_secret*)",
  "Grep(**/*client_secret*)",

  "Read(**/*signing-key*)",
  "Edit(**/*signing-key*)",
  "Write(**/*signing-key*)",
  "Glob(**/*signing-key*)",
  "Grep(**/*signing-key*)",

  "Read(**/*signing_key*)",
  "Edit(**/*signing_key*)",
  "Write(**/*signing_key*)",
  "Glob(**/*signing_key*)",
  "Grep(**/*signing_key*)",

  "Read(**/*encryption-key*)",
  "Edit(**/*encryption-key*)",
  "Write(**/*encryption-key*)",
  "Glob(**/*encryption-key*)",
  "Grep(**/*encryption-key*)",

  "Read(**/*encryption_key*)",
  "Edit(**/*encryption_key*)",
  "Write(**/*encryption_key*)",
  "Glob(**/*encryption_key*)",
  "Grep(**/*encryption_key*)",

  "Read(**/*master-key*)",
  "Edit(**/*master-key*)",
  "Write(**/*master-key*)",
  "Glob(**/*master-key*)",
  "Grep(**/*master-key*)",

  "Read(**/*master_key*)",
  "Edit(**/*master_key*)",
  "Write(**/*master_key*)",
  "Glob(**/*master_key*)",
  "Grep(**/*master_key*)",

  "Read(**/*root-key*)",
  "Edit(**/*root-key*)",
  "Write(**/*root-key*)",
  "Glob(**/*root-key*)",
  "Grep(**/*root-key*)",

  "Read(**/*root_key*)",
  "Edit(**/*root_key*)",
  "Write(**/*root_key*)",
  "Glob(**/*root_key*)",
  "Grep(**/*root_key*)"
]
```

**Total: 227 rules** (5 file-tool boundary + 2 shell-tool boundary + 44 fragments × 5 tool verbs).

---

## Validation quick-reference

Use these tables when auditing an existing `permissions.deny` array. Tick each
cell when the corresponding rule is present.

Boundary rules (Groups A + C):

| Rule | Present |
|---|---|
| `Read(../**)` | ☐ |
| `Edit(../**)` | ☐ |
| `Write(../**)` | ☐ |
| `Glob(../**)` | ☐ |
| `Grep(../**)` | ☐ |
| `Bash(*../**)` | ☐ |
| `PowerShell(*../**)` | ☐ |

Sensitive-name rules (Group B), one column per tool verb:

| Fragment | Read | Edit | Write | Glob | Grep |
|---|---|---|---|---|---|
| `password` | ☐ | ☐ | ☐ | ☐ | ☐ |
| `passwords` | ☐ | ☐ | ☐ | ☐ | ☐ |
| `passwd` | ☐ | ☐ | ☐ | ☐ | ☐ |
| `secret` | ☐ | ☐ | ☐ | ☐ | ☐ |
| `secrets` | ☐ | ☐ | ☐ | ☐ | ☐ |
| `credential` | ☐ | ☐ | ☐ | ☐ | ☐ |
| `credentials` | ☐ | ☐ | ☐ | ☐ | ☐ |
| `apikey` | ☐ | ☐ | ☐ | ☐ | ☐ |
| `api-key` | ☐ | ☐ | ☐ | ☐ | ☐ |
| `api_key` | ☐ | ☐ | ☐ | ☐ | ☐ |
| `access-token` | ☐ | ☐ | ☐ | ☐ | ☐ |
| `access_token` | ☐ | ☐ | ☐ | ☐ | ☐ |
| `accesstoken` | ☐ | ☐ | ☐ | ☐ | ☐ |
| `oauth-token` | ☐ | ☐ | ☐ | ☐ | ☐ |
| `oauth_token` | ☐ | ☐ | ☐ | ☐ | ☐ |
| `jwt` | ☐ | ☐ | ☐ | ☐ | ☐ |
| `jwt-token` | ☐ | ☐ | ☐ | ☐ | ☐ |
| `jwt_token` | ☐ | ☐ | ☐ | ☐ | ☐ |
| `bearer-token` | ☐ | ☐ | ☐ | ☐ | ☐ |
| `bearer_token` | ☐ | ☐ | ☐ | ☐ | ☐ |
| `private-key` | ☐ | ☐ | ☐ | ☐ | ☐ |
| `private_key` | ☐ | ☐ | ☐ | ☐ | ☐ |
| `privatekey` | ☐ | ☐ | ☐ | ☐ | ☐ |
| `ssh-key` | ☐ | ☐ | ☐ | ☐ | ☐ |
| `ssh_key` | ☐ | ☐ | ☐ | ☐ | ☐ |
| `service-account` | ☐ | ☐ | ☐ | ☐ | ☐ |
| `service_account` | ☐ | ☐ | ☐ | ☐ | ☐ |
| `vault-token` | ☐ | ☐ | ☐ | ☐ | ☐ |
| `vault_token` | ☐ | ☐ | ☐ | ☐ | ☐ |
| `git-credentials` | ☐ | ☐ | ☐ | ☐ | ☐ |
| `aws-credentials` | ☐ | ☐ | ☐ | ☐ | ☐ |
| `gcp-credentials` | ☐ | ☐ | ☐ | ☐ | ☐ |
| `azure-credentials` | ☐ | ☐ | ☐ | ☐ | ☐ |
| `database-password` | ☐ | ☐ | ☐ | ☐ | ☐ |
| `db-password` | ☐ | ☐ | ☐ | ☐ | ☐ |
| `db_password` | ☐ | ☐ | ☐ | ☐ | ☐ |
| `client-secret` | ☐ | ☐ | ☐ | ☐ | ☐ |
| `client_secret` | ☐ | ☐ | ☐ | ☐ | ☐ |
| `signing-key` | ☐ | ☐ | ☐ | ☐ | ☐ |
| `signing_key` | ☐ | ☐ | ☐ | ☐ | ☐ |
| `encryption-key` | ☐ | ☐ | ☐ | ☐ | ☐ |
| `encryption_key` | ☐ | ☐ | ☐ | ☐ | ☐ |
| `master-key` | ☐ | ☐ | ☐ | ☐ | ☐ |
| `master_key` | ☐ | ☐ | ☐ | ☐ | ☐ |
| `root-key` | ☐ | ☐ | ☐ | ☐ | ☐ |
| `root_key` | ☐ | ☐ | ☐ | ☐ | ☐ |
