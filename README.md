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

---

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

---

### Task 3 – Shared Workflow Repository & Release
- Created a shared workflow to tag docker images
```yaml
name: Reusable Workflow to Tag Docker Images

on:
  workflow_call:
    inputs:
      image_name:
        required: true
        type: string
      source_tag:
        required: true
        type: string
      target_tag:
        required: true
        type: string
    secrets:
      DOCKER_USERNAME:
        required: true
      DOCKER_PASSWORD:
        required: true

jobs:
  tag-image:
    runs-on: ubuntu-latest

    steps:
      - name: Login to Docker Hub
        uses: docker/login-action@v4
        with:
          username: ${{ secrets.DOCKER_USERNAME }}
          password: ${{ secrets.DOCKER_PASSWORD }}

      - name: Pull existing image
        run: |
          docker pull ${{ secrets.DOCKER_USERNAME }}/${{ inputs.image_name }}:${{ inputs.source_tag }}

      - name: Tag image
        run: |
          docker tag \
          ${{ secrets.DOCKER_USERNAME }}/${{ inputs.image_name }}:${{ inputs.source_tag }} \
          ${{ secrets.DOCKER_USERNAME }}/${{ inputs.image_name }}:${{ inputs.target_tag }}

      - name: Push new tag
        run: |
          docker push ${{ secrets.DOCKER_USERNAME }}/${{ inputs.image_name }}:${{ inputs.target_tag }}
```

- Created caller workflow that calls shared workflow to tag images
```yaml
name: Tag Docker Image on Push using github shared workflow

on:
  push:
    branches:
      - develop
      - main

jobs:
  tag-staging:
    if: ${{ github.ref_name == 'develop' }}
    uses: amandhal/workflow-repository/.github/workflows/docker-ci.yml@v1.0.0
    with:
      image_name: frontend
      source_tag: latest
      target_tag: staging-${{ github.sha }}
    secrets:
      DOCKER_USERNAME: ${{ secrets.DOCKER_USERNAME }}
      DOCKER_PASSWORD: ${{ secrets.DOCKER_PASSWORD }}

  tag-prod:
    if: ${{ github.ref_name == 'main' }}
    uses: amandhal/workflow-repository/.github/workflows/docker-ci.yml@v1.0.0
    with:
      image_name: frontend
      source_tag: latest
      target_tag: prod-${{ github.run_number }}
    secrets:
      DOCKER_USERNAME: ${{ secrets.DOCKER_USERNAME }}
      DOCKER_PASSWORD: ${{ secrets.DOCKER_PASSWORD }}
```
<img width="1919" height="691" alt="image" src="https://github.com/user-attachments/assets/d6b160e6-8c8e-4107-993a-5f948f789166" />
<img width="1919" height="732" alt="image" src="https://github.com/user-attachments/assets/6c9a035f-eb4a-4977-8a1d-fc55129eaa59" />

---

### Task 4 – Security & Notifications

