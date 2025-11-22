.github/workflows/devsecops.yml:

name: DevSecOps Pipeline

on:
  push:
    branches: ["main"]

jobs:
  build-test-secure:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout
      uses: actions/checkout@v3

    - name: Install Node
      uses: actions/setup-node@v3
      with:
        node-version: 18

    - name: Install Dependencies
      run: npm install

    - name: Lint Check
      run: npm run lint

    - name: Security Scan
      run: npm audit --audit-level=high

    - name: Unit Tests
      run: npm test

    - name: Build Chaincode
      run: npm run build

    - name: Build Docker Image
      run: docker build -t mycc:latest .

    - name: Scan Docker Image
      uses: aquasecurity/trivy-action@master
      with:
        image-ref: 'mycc:latest'

    - name: Deploy to Test Network (SSH)
      uses: appleboy/ssh-action@v1.0.0
      with:
        host: ${{ secrets.SERVER_IP }}
        username: ubuntu
        key: ${{ secrets.SERVER_PRIVATE_KEY }}
        script: |
          cd fabric-samples/test-network
          ./network.sh down
          ./network.sh up createChannel -ca
          ./network.sh deployCC -ccn mycc -ccp ../chaincode -ccl javascript



here need to change the folder structure based on my folder structure and my server ip is 178.16.139.239