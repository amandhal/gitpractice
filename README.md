# GitHub Actions CI/CD Lab Exam

### Task 1 – Hello World Workflow
```yaml
name: Hello World Workflow

on: push  

jobs:
  hello-world:
    runs-on: ubuntu-latest
    steps:
      - run: echo "Hello GitHub Actions!"
```
<img width="1918" height="549" alt="image" src="https://github.com/user-attachments/assets/9c6861d6-d3d8-4f21-9333-180ba8f829ce" />

### Task 2 – Basic Docker Build & Push
- Created workflow to build and push docker images to dockerhub
```yaml
name: Build and Push Frontend Image

on:
  push:
    branches:
      - main
    paths:
      - "frontend/**"
      - ".github/workflows/build-push-images.yml"
      
env:
  REPOSITORY: frontend      
  IMAGE_TAG: ${{ github.sha }}   

jobs:
  build-push-frontend-image:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout Repository
        uses: actions/checkout@v6

      - name: Log in to Docker Hub
        uses: docker/login-action@v4
        with:
          username: ${{ secrets.DOCKER_USERNAME }}
          password: ${{ secrets.DOCKER_PASSWORD }}

      - name: Build and Push Image
        uses: docker/build-push-action@v7
        with:
          push: true
          tags: |
            ${{ secrets.DOCKER_USERNAME }}/${{ env.REPOSITORY }}:latest
            ${{ secrets.DOCKER_USERNAME }}/${{ env.REPOSITORY }}:${{ env.IMAGE_TAG }}
          
      - name: Remove Local Image
        run: |
          docker rmi ${{ secrets.DOCKER_USERNAME }}/${{ env.REPOSITORY }}:latest
          docker rmi ${{ secrets.DOCKER_USERNAME }}/${{ env.REPOSITORY }}:${{ env.IMAGE_TAG }}
```
<img width="1919" height="621" alt="image" src="https://github.com/user-attachments/assets/b4b85757-bff8-4664-88b8-a38f5f9383c8" />
<img width="1919" height="669" alt="image" src="https://github.com/user-attachments/assets/45d5d6c8-9a88-4fc0-862e-9b554112cebc" />

