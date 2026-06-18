# claude-md

Ruleset condiviso per **Claude** (e agenti AI in generale) usato in **Tensory-srls**.

## Perché esiste questa repo

Questa repo nasce per dare a chiunque lavori in Tensory-srls un **set di regole comune per Claude**, così che lo sviluppo assistito da AI sia **agevole, coerente e prevedibile** su tutti i progetti del team.

Il file [`CLAUDE.md`](./CLAUDE.md) è il "manuale operativo" che ogni agente legge a inizio sessione: definisce regole non negoziabili, workflow, quality gate di CI, convenzioni di stack e anti-pattern da evitare. Centralizzandolo qui evitiamo che ognuno reinventi le proprie regole e garantiamo lo stesso standard di qualità ovunque.

Stack di riferimento: **Next.js (App Router) · React · TypeScript · Payload CMS · PostgreSQL (Neon / Supabase)**.

## Cosa contiene

- **`CLAUDE.md`** — il ruleset principale. Copre:
  - regole non negoziabili (verification gate prima di ogni push, mai committare segreti, ecc.);
  - il workflow obbligatorio: research → plan → execute → review → ship;
  - disciplina su contesto e prompt-engineering;
  - uso della documentazione (context7 MCP + wiki interna);
  - TDD e quality bar;
  - il gate di verifica CI;
  - convenzioni di stack e anti-pattern.

## Come usarlo in un progetto

Copia (o referenzia) `CLAUDE.md` nella root del tuo progetto. Claude Code lo carica automaticamente all'avvio della sessione.

```bash
# esempio: portare il ruleset in un progetto esistente
curl -sSL https://raw.githubusercontent.com/Tensory-srls/claude-md/main/CLAUDE.md -o CLAUDE.md
```

Dettagli specifici di un progetto vanno in `.claude/rules/` con glob di path, **non** dentro questo file: `CLAUDE.md` deve restare ad alto segnale e generico.

## Contribuire

Le regole evolvono. Se proponi una modifica:

1. Apri una PR con la motivazione del cambiamento ("perché", non solo "cosa").
2. Mantieni il file **ad alto segnale**: l'enfasi (`IMPORTANT` / `YOU MUST`) è razionata e riservata alle regole portanti.
3. Dettagli di progetto → `.claude/rules/`, non qui.
