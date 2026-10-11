# Deploy a static site to cPanel over SFTP with an SSH key

Every push to this branch uploads the site to a cPanel account over SFTP, authenticated with an SSH key instead of a password, using the [DEP 9000](https://github.com/Black-HOST/deployer) GitHub Action. The result is live at [cicd.guide/sftp/](https://cicd.guide/sftp/).

Three files:

| File | Role |
|---|---|
| `index.html` | the site |
| `.github/workflows/deploy.yml` | the pipeline |
| `README.md` | this page, excluded from the upload |

## Use it

**1. Get the files.** For a new project:

```sh
git clone -b sftp --single-branch https://github.com/Black-HOST/CI-CD.git my-site
cd my-site
git remote set-url origin <your repository>
```

For an existing project, copy `.github/workflows/deploy.yml` into it.

**2. Create a deploy key.** A key that exists only for this pipeline. In cPanel, open **SSH Access > Manage SSH Keys > Generate a New Key**: give it a name such as `deploy_key`, choose **RSA** with **4096** bits and click **Generate Key**. A key password is optional: the workflow passes it as `password: ${{ secrets.SFTP_KEY_PASS }}` and DEP 9000 unlocks the key with it. For a key without a password, simply do not create that secret.

cPanel generates RSA keys only. The modern standard for SSH keys is ed25519: shorter, faster, nothing to size. If you prefer it, create the key on your computer instead (`-N ""` means no password, set one if you want):

```sh
ssh-keygen -t ed25519 -f deploy_key -N "" -C "github deploy"
```

This gives `deploy_key` (private, goes to GitHub) and `deploy_key.pub` (public, goes to the server). Do not add them to the repository. In cPanel, open **Manage SSH Keys > Import Key**, paste the content of `deploy_key.pub` as the public key and click **Import**.

**3. Authorize the key.** Under **Manage SSH Keys > Public Keys**, click **Manage** next to the key, then **Authorize**. SSH access has to be enabled on the account; if the page is missing, ask your host.

**4. Set the branch that deploys.** The example deploys the `sftp` branch; most repositories deploy `main`:

```yaml
    branches: [ "main" ]
```

**5. Add the secrets** in your repository, under Settings > Secrets and variables > Actions:

| Secret | Value |
|---|---|
| `SFTP_HOST` | your domain, or the server hostname shown in cPanel |
| `SFTP_PORT` | the SSH port, `22` unless your host says otherwise |
| `SFTP_USER` | your cPanel username |
| `SFTP_KEY` | the private key as text, not a file name: paste the whole content, `BEGIN` and `END` lines included, from **Manage SSH Keys > Private Keys > View/Download** in cPanel, or from the `deploy_key` file if you created it yourself |
| `SFTP_KEY_PASS` | the key password, only if you set one |

Only the key and its password are real secrets. The others are kept as secrets too, so there is one place to look; if you prefer, host and port can be repository variables instead (`server: ${{ vars.SFTP_HOST }}`), which are visible in the settings and in the logs. That helps when a connection fails, because secrets are masked as `***` in the log.

**6. Push.** The Actions tab shows the run; the site is live when it finishes.

## What the workflow does

```yaml
name: Deploy

on:
  push:
    branches: [ "sftp" ]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Code Deployment
        uses: Black-HOST/deployer@v1
        with:
          protocol: sftp
          server: ${{ secrets.SFTP_HOST }}
          port: ${{ secrets.SFTP_PORT }}
          username: ${{ secrets.SFTP_USER }}
          ssh_key: ${{ secrets.SFTP_KEY }}
          # the key password, if the key has one; an empty secret is ignored
          password: ${{ secrets.SFTP_KEY_PASS }}
          # SFTP paths are real paths, relative to the login's home directory
          remote_dir: "public_html/sftp"
          # a custom list replaces the default one, so the dotfile rules stay in front
          exclude: ".*,.*/,*.md"
```

- `protocol: sftp` with `ssh_key` instead of `password`. `ssh_key` takes the key itself, the text of the private key file, not a path to it; the deployer writes it to a file inside the job's container. Nothing to type, nothing to rotate when someone leaves: delete the key in cPanel and the pipeline is locked out. Both together, `ssh_key` and `password`, unlock a password-protected key.
- Optional, for more security: `host_key`. By default the deployer trusts whatever host key the server presents on each run. Pin it and a server that presents any other key is refused before the private key or any file is sent. Get it once, from a network you trust, with `ssh-keyscan -p <port> -t ed25519 <server>`, and put the key type and the key, without the host name, into the workflow: `host_key: "ssh-ed25519 AAAA..."`. It is a public key, so it can live in the repository. Several keys fit, one per line.
- `remote_dir: "public_html/sftp"` has no leading slash on purpose. Over FTP, `/` is the account's home; over SFTP, `/` is the root of the server's filesystem. SFTP paths are relative to the login's home directory, so `public_html` is the web root and this example uses the `sftp` folder below it, which is why it shows up at `cicd.guide/sftp/`. For your own site use `public_html`.
- `port` comes from a secret like the host. It is 22 on most servers; this one listens on 4004.
- `exclude` keeps `README.md` off the server. Setting `exclude` replaces the default list, so the dotfile rules `.*,.*/` are repeated: `.git/` and `.env` must never reach a web server. To upload a dotfile such as `.htaccess`, add `include: ".htaccess"`.
- Nothing is deleted on the server. To remove files that no longer exist in the repository, add `delete: "true"`; the deployer refuses to run when the local directory is empty, so a broken checkout cannot wipe the site.

Every option is listed in the [DEP 9000 configuration](https://github.com/Black-HOST/deployer#%EF%B8%8F-configuration).

## Let your agent do it

Building with Claude Code, Cursor or a similar tool? Paste this:

> Set up deployment for this repository to my cPanel hosting with the GitHub Action `Black-HOST/deployer@v1`, over SFTP with an SSH key. Follow the example at https://github.com/Black-HOST/CI-CD/tree/sftp: create `.github/workflows/deploy.yml` that deploys the `main` branch to `public_html`, using the secrets `SFTP_HOST`, `SFTP_PORT`, `SFTP_USER`, `SFTP_KEY` and `SFTP_KEY_PASS` (the key password, if any). Tell me how to create and authorize the deploy key in cPanel, or the `ssh-keygen` command if I make it myself, and which secrets to add in GitHub. Do not ask for the private key; I will add the secrets myself.

The agent writes the pipeline. The private key goes into a GitHub secret, never into the chat or the repository.

## More examples

All examples, one per branch, are listed on the [`main` branch](https://github.com/Black-HOST/CI-CD).
