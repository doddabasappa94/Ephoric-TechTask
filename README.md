# CICD Pipeline
 Stages of CICD Pipeline
Clone: checks out the source code to jenkins workspace.
Sonar Scanning: Sonar Qube scanning is performed against the source code.
Quality Gate: Validateds the Quality of the source code, allows only if its passing.
Build & Test: Installs the dependencies and run the unit test.
Dependency Vulnerability Check : snyk will perform the vulnerablity scanning on the dependencies.
Build and Sign Docker image: Container image is created and Signed by the notary.
Image Scanning: Build image is scanned by Snyk for any vulnerabilities.
Push To Docker Repo: Docker image is now pushed to Dockerhub.
Deploy: Helm Chart is deployed to dev namespace under Dev Cluster.
