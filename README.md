# Exercise for GitHub Actions

After running:

npm install
npm run build

The app produces a "dist" folder, which contains the output files of this application.

We want to access these produced files so that users can:
- Download them
- Inspect or test them locally
- Upload them to a hosting provider

To make this possible, we should store them as an artifact.

---

## GitHub Actions Configuration

Below is an example workflow that installs dependencies, builds the project, and uploads the build output as an artifact.

name: Deploy website

on:
  push:
    branches:
      - main

jobs:
  build:
    needs: test
    runs-on: ubuntu-latest
    steps:
      - name: Get code
        uses: actions/checkout@v3

      - name: Install dependencies
        run: npm ci

      - name: Build website
        run: npm run build

      - name: Upload artifact
        uses: actions/upload-artifact@v
        with: 
          name: dist-files
          path: |
            dist
            package.json

---

The above workflow will:
1. Trigger when code is pushed to the **main** branch.  
2. Get the latest code from the repository.  
3. Install dependencies using `npm ci`.  
4. Build the project with `npm run build`.  
5. Upload the `dist` folder and `package.json` as an artifact for download.
