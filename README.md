name: Deploy ADK/MCP/A2A Agent

on:
  push:
    branches: [ main ]

jobs:
  deploy-agent:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Use Node 20
        uses: actions/setup-node@v4
        with:
          node-version: '20'
      - name: Install dependencies
        run: npm install
      - name: Run tests
        run: npm test
      - name: Build the agent
        run: npm run build
      - name: Authenticate to Google Cloud
        uses: 'google-github-actions/auth@v2'
        with:
          credentials_json: '${{ secrets.AGENT_RUNTIME_KEY }}'
      - name: Deploy to Cloud Run
        uses: 'google-github-actions/deploy-cloudrun@v2'
        with:
          service: 'agent-runtime'
          region: 'us-central1'
          env_vars: |
            PROJECT_ID=adk-mcp-a2a-486802
            REGION=us-central1
      - name: Upload build artifacts
        uses: actions/upload-artifact@v4
        with:
          name: build-artifacts
          path: ./dist
