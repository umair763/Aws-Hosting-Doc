# Setting up GitHub Self-Hosted Runner and CI/CD Workflow

This guide explains how to connect your AWS EC2 instance as a **GitHub Actions self-hosted runner**, configure environment variables, and set up a **Node.js CI/CD pipeline**.

---

## 🔧 Step 1: Configure GitHub Self-Hosted Runner
1. Go to your **GitHub backend repository**.
2. Navigate to:
   - **Settings** from the top menu.
   - From the left sidebar, select **Actions → Runners**.
3. Click **New self-hosted runner**.
4. Choose **Linux** as the operating system.
5. Make sure that from **PowerShell (Windows)** or another terminal (Linux/macOS), you are already logged into your AWS EC2 server using the **SSH command** from the previous section.
6. Run the commands provided by GitHub for the runner setup, in your EC2 terminal:
   - Start with the **Download** section.
   - Continue with the **Configure** section.
   - **Skip** the `./run.sh` command usually shown at the end of the configure section.
7. Keep all settings as **default** unless you are a professional user and want advanced configurations.
8. Once you reach the step where `./run.sh` is mentioned, instead run the following commands to install and start the runner as a **service**:

   ```bash
   sudo ./svc.sh install
   sudo ./svc.sh start
### 🔐 Step 2: Configure Environment Variables in GitHub

1. Go back to your **GitHub repository settings**.
2. From the sidebar, select **Secrets and variables → Actions**.
3. Click **New variable** to create a new environment variable for each value your project needs (e.g., `DB_URI`, `JWT_SECRET`, `PORT`, etc.).
4. Add all your environment variables here.
5. Optionally, create one combined secret called `PROD_ENV` that contains all variables in `KEY=VALUE` format (one per line).  
   This will be used to generate a `.env` file inside the runner during deployment.


### ⚙️ Step 3: Configure GitHub Actions Workflow for Node.js Deployment

1. Go to your GitHub repository.
2. Click on the **Actions** tab.
3. Select a **Node.js** workflow template to start with.
4. Navigate to the `.github/workflows/` directory.
5. Open or create a file named `deploy.yml`.

Paste the following content into `deploy.yml`:

```yaml
name: Node.js CI/CD

on:
  push:
    branches: [ "main" ]

jobs:
  build:
    runs-on: self-hosted
    strategy:
      matrix:
        node-version: [18.x]
      # See supported Node.js release schedule at https://nodejs.org/en/about/releases/

    steps:
      - uses: actions/checkout@v3

      - name: Use Node.js ${{ matrix.node-version }}
        uses: actions/setup-node@v3
        with:
          node-version: ${{ matrix.node-version }}
          cache: "npm"

      - run: npm ci

      - name: Create .env file from secret
        run: |
          touch .env
          cat <<EOF > .env
          ${{ secrets.PROD_ENV }}
          EOF

      - name: Restart PM2 process (nodejs server)
        run: pm2 restart backendApi


