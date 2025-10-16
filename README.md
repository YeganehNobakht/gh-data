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
      - master
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - name: Get code
        uses: actions/checkout@v3
      - name: Install dependencies
        run: npm ci
      - name: Lint code
        run: npm run lint
      - name: Test code
        run: npm run test
  build:
    needs: test
    runs-on: ubuntu-latest
    outputs:
      script-file: ${{ steps.publish.outputs.script-file }}
    steps:
      - name: Get code
        uses: actions/checkout@v3
      - name: Install dependencies
        run: npm ci
      - name: Build website
        run: npm run build
      - name: Publish JS filename
        id: publish
        run: |
          FILE=$(find dist/assets -name '*.js' | head -n 1)
          echo "script-file=$FILE" >> $GITHUB_OUTPUT 
      - name: Upload artifact
        uses: actions/upload-artifact@v4
        with: 
          name: dist-files
          path: dist
  deploy:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - name: Get build artifacts
        uses: actions/download-artifact@v4
        with: 
          name: dist-files
      - name: Output contents
        run: ls
      - name: Output filename
        run: echo "${{ needs.build.outputs.script-file }}"
      - name: Deploy
        run: echo "Deploying..."


---

The above workflow will:
1. Trigger when code is pushed to the **main** branch.  
2. Get the latest code from the repository.  
3. Install dependencies using `npm ci`.  
4. Build the project with `npm run build`.  
5. Upload the `dist` folder and `package.json` as an artifact for download.
6. Download this artifact later in another job (for example, deployment).Access the JS Filename (Step 7 – Key Step):

7. Accessing the JS Filename

We need to share the generated JS file name between jobs.

1️⃣ Define an output in the build job:

outputs:
  script-file: ${{ steps.publish.outputs.script-file }}


2️⃣ Save the filename in a step:

echo "script-file=$(find dist/assets -name '*.js' | head -n 1)" >> $GITHUB_OUTPUT


This finds the first .js file in dist/assets and stores it as a job output named script-file.

3️⃣ Use the output in the deploy job:

echo "${{ needs.build.outputs.script-file }}"


This makes the filename available for deployment, logging, or testing.