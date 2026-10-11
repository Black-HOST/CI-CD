# Deploy a static site to cPanel over FTP

Every push to this branch uploads the site to a cPanel account over FTPS, using the [DEP 9000](https://github.com/Black-HOST/deployer) GitHub Action. The result is live at [cicd.guide/ftp/](https://cicd.guide/ftp/).

Three files:

| File | Role |
|---|---|
| `index.html` | the site |
| `.github/workflows/deploy.yml` | the pipeline |
| `README.md` | this page, excluded from the upload |

## Use it

**1. Get the files.** For a new project:

```sh
git clone -b ftp --single-branch https://github.com/Black-HOST/CI-CD.git my-site
cd my-site
git remote set-url origin <your repository>
```

For an existing project, copy `.github/workflows/deploy.yml` into it.

**2. Set the branch that deploys.** The example deploys the `ftp` branch; most repositories deploy `main`:

```yaml
    branches: [ "main" ]
```

**3. Add three secrets** in your repository, under Settings > Secrets and variables > Actions:

| Secret | Value |
|---|---|
| `FTP_HOST` | your domain, or the server hostname shown in cPanel |
| `FTP_USER` | your cPanel username, or an account from cPanel > FTP Accounts |
| `FTP_PASS` | its password |

**4. Push.** The Actions tab shows the run; the site is live when it finishes.

## What the workflow does

```yaml
name: Deploy

on:
  push:
    branches: [ "ftp" ]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Code Deployment
        uses: Black-HOST/deployer@v1
        with:
          server: ${{ secrets.FTP_HOST }}
          username: ${{ secrets.FTP_USER }}
          password: ${{ secrets.FTP_PASS }}
          remote_dir: "/public_html/ftp"
          # a custom list replaces the default one, so the dotfile rules stay in front
          exclude: ".*,.*/,*.md"
```

- No `protocol` is set, so the default `ftp` applies: FTPS with certificate verification. For a server without TLS, add `secure: "false"`.
- `remote_dir` is where the files land. `/public_html` is the web root of a cPanel account; this example uses the `ftp` folder below it, which is why it shows up at `cicd.guide/ftp/`. For your own site use `/public_html`, and `/` for an FTP account that was created for a single directory, since it already starts inside it.
- `exclude` keeps `*.md` off the server. Setting `exclude` replaces the default list, so the dotfile rules `.*,.*/` are repeated: `.git/` and `.env` must never reach a web server. To upload a dotfile such as `.htaccess`, add `include: ".htaccess"`.
- Nothing is deleted on the server. To remove files that no longer exist in the repository, add `delete: "true"`; the deployer refuses to run when the local directory is empty, so a broken checkout cannot wipe the site.

Every option is listed in the [DEP 9000 configuration](https://github.com/Black-HOST/deployer#%EF%B8%8F-configuration).

## Let your agent do it

Building with Claude Code, Cursor or a similar tool? Paste this:

> Set up deployment for this repository to my cPanel hosting with the GitHub Action `Black-HOST/deployer@v1`. Follow the example at https://github.com/Black-HOST/CI-CD/tree/ftp: create `.github/workflows/deploy.yml` that deploys the `main` branch over FTPS to `/public_html`. Use the secrets `FTP_HOST`, `FTP_USER` and `FTP_PASS`, and list them for me to add in GitHub. Do not ask for the password; I will add the secrets myself.

The agent writes the pipeline. The credentials go into GitHub secrets, never into the chat or the repository.

## More examples

All examples, one per branch, are listed on the [`main` branch](https://github.com/Black-HOST/CI-CD).
