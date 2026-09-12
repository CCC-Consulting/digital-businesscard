# How it all fits together

Four diagrams to make sense of GitHub, GitHub Actions, and Azure before you
dive into the [setup guide](03_SETUP-GUIDE.md). If you've never worked with any
of these, start at the top and work down.

## 1. The big picture: from your laptop to a live URL

```mermaid
flowchart LR
    A["You edit your site's files"] --> B["git push to GitHub"]
    B --> C["GitHub Actions workflow runs automatically"]
    C --> D["Azure Static Web Apps uploads your files"]
    D --> E["Live at yourcard.azurestaticapps.net"]
    E --> F(["Someone taps NFC or scans your QR code"])
```

- **GitHub** just stores your code and its history — nothing runs there by
  itself.
- **GitHub Actions** is GitHub's automation layer: it watches your repo for
  events (a push, a schedule, a manual click) and runs scripts in response.
  Each script is a **workflow**, defined as a YAML file in
  `.github/workflows/`.
- **Azure Static Web Apps** is where your site actually lives on the
  internet. It doesn't know or care about GitHub directly — the workflow is
  what uploads your files to it.

## 2. What happens inside a workflow run

```mermaid
flowchart TD
    A["Trigger fires<br/>(push / schedule / manual click)"] --> B["GitHub spins up a fresh temporary VM"]
    B --> C["Checks out your repo's code onto it"]
    C --> D["Logs in to Azure using a short-lived credential"]
    D --> E["Runs the deployment step"]
    E --> F["VM is destroyed — nothing persists"]
```

Every workflow run starts from a completely clean machine, does its job,
and throws the machine away afterward. Nothing about your workflow run is
kept around between runs except what you explicitly commit back to the
repo (like a refreshed `articles.json`).

## 3. How GitHub Actions authenticates to Azure — the advanced path

The **easy path** in the [setup guide](03_SETUP-GUIDE.md) has Azure generate a
deployment secret for you automatically, stored in GitHub as an encrypted
repo secret. That's fine, and simplest to start with.

The **advanced path** — what the reference build actually uses — skips
storing any secret at all, using OpenID Connect (OIDC) federated
credentials instead:

```mermaid
sequenceDiagram
    participant GH as GitHub Actions
    participant Entra as Microsoft Entra ID
    participant Azure as Azure

    GH->>Entra: "I'm this exact workflow, in this exact repo — here's proof"
    Entra->>Entra: Check the proof against a pre-registered trust rule
    Entra-->>GH: Short-lived access token (expires in minutes)
    GH->>Azure: Deploy, using that short-lived token
    Azure-->>GH: Deployment result
```

Nothing long-lived is ever stored in GitHub — the "proof" is generated
fresh by GitHub for each run and is only valid for that run, from that
exact workflow, in that exact repo. If you want this path, tell your AI
assistant "set this up with OIDC instead of a stored deployment token" and
it can walk you through registering the trust rule in Azure (an "App
Registration" with a "federated credential").

## 4. How the content-refresh automation works

If your card has a "latest articles" or "latest videos" section, this loop
runs independently of the main deploy workflow:

```mermaid
flowchart TD
    A["Cron schedule fires<br/>e.g. daily at 06:00 UTC"] --> B["A workflow runs on GitHub's servers"]
    B --> C["Fetches the RSS/YouTube feed<br/>server-to-server — no CORS involved"]
    C --> D["Writes the latest few items to<br/>a JSON file, e.g. articles.json"]
    D --> E{"Did the file's content change?"}
    E -->|No| F["Nothing to do — workflow ends"]
    E -->|Yes| G["Commits and pushes the updated JSON file"]
    G --> H["That push triggers the normal deploy workflow"]
    H --> I["Site redeploys with the fresh JSON included"]
    I --> J["Your page's own JavaScript fetches<br/>the JSON from your own site"]
```

The key idea: your browser never talks to the external feed directly. A
scheduled job does that on your behalf, from a context where cross-origin
restrictions don't apply, and hands your page a plain same-origin file
instead.

## 5. Bonus: pull request preview environments

A nice Azure Static Web Apps feature, if you ever collaborate with someone
or just want to review a change before it goes live:

```mermaid
flowchart LR
    A["Open a pull request"] --> B["GitHub Actions builds a temporary preview"]
    B --> C["Preview URL, e.g. red-water-123.azurestaticapps.net"]
    D["Merge or close the PR"] --> E["GitHub Actions tears the preview down"]
```

Each open PR gets its own throwaway URL to review changes on, separate
from your live site, cleaned up automatically once you're done with it.

---

Ready to build? Head back to **[the setup guide](03_SETUP-GUIDE.md)**.
