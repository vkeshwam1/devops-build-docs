Application Deployment E-Commerce Codes


Cloning the repo to GitHub: 
cd ~
git clone https://github.com/sriram-R-krishnan/devops-build devops-build-ecommerce
cd devops-build-ecommerce
ls
# Creating Dockerfile

cat > Dockerfile << 'EOF'
FROM nginx:alpine
COPY build/ /usr/share/nginx/html
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
EOF
cat Dockerfile
# Creating docker-compose.yml
cat > docker-compose.yml << 'EOF'
version: '3.8'
services:
  app:
    image: vkeshwam1/dev:latest
    ports:
      - "80:80"
    restart: always
EOF
cat docker-compose.yml
#Creating build.sh:

cat > build.sh << 'EOF'
#!/bin/bash
set -e
BRANCH=$(git rev-parse --abbrev-ref HEAD)
BUILD_NUM=${BUILD_NUMBER:-local}
if [ "$BRANCH" = "master" ]; then
  IMAGE="vkeshwam1/prod:$BUILD_NUM"
else
  IMAGE="vkeshwam1/dev:$BUILD_NUM"
fi
echo "Building image: $IMAGE"
docker build -t $IMAGE .
docker tag $IMAGE ${IMAGE%:*}:latest
echo "Build complete: $IMAGE"
EOF
chmod +x build.sh
cat build.sh
#Create deploy.sh: 
cat > deploy.sh << 'EOF'
#!/bin/bash
set -e
BRANCH=$(git rev-parse --abbrev-ref HEAD)
if [ "$BRANCH" = "master" ]; then
  IMAGE="vkeshwam1/prod:latest"
else
  IMAGE="vkeshwam1/dev:latest"
fi
echo "Deploying image: $IMAGE"
docker pull $IMAGE
docker stop app 2>/dev/null || true
docker rm app 2>/dev/null || true
docker run -d --name app -p 80:80 --restart always $IMAGE
echo "Deployment complete! Running: $IMAGE"
EOF
chmod +x deploy.sh
cat deploy.sh
#Creating dockerignore & .gitignore
cat > .dockerignore << 'EOF'
node_modules
.git
*.log
.env
EOF
#gitignore
cat > .gitignore << 'EOF'
node_modules/
.env
*.log
.DS_Store
EOF

#Pushing the GitHub on dev branch: 
git remote set-url origin https://github.com/vkeshwam1/devops-build.git
git checkout -b dev 2>/dev/null || git checkout dev
git add .
git status
git commit -m "fresh: Dockerfile, docker-compose, build.sh, deploy.sh, gitignore"
git push origin dev
# Creating Docker build:

cd ~/devops-build-ecommerce 
# Build the image 
docker build -t vkeshwam1/dev:latest .
 # Verify it built 
docker images | grep vkeshwam1

Login and push to docker
# Login to DockerHub
docker login -u vkeshwam1

# Push to dev repo
docker push vkeshwam1/dev:latest

# Tag and push to prod repo
docker tag vkeshwam1/dev:latest vkeshwam1/prod:latest
docker push vkeshwam1/prod:latest

mv ~/Downloads/devops-ecommerce-key-new.pem ~/.ssh/
chmod 400 ~/.ssh/devops-ecommerce-key-new.pem

ssh -i ~/.ssh/devops-ecommerce-key-new.pem ubuntu@52.91.6.8

sudo apt update && sudo apt upgrade -y

# Installing Docker to Instance
sudo apt install -y docker.io
sudo systemctl start docker
sudo systemctl enable docker
sudo usermod -aG docker ubuntu

# Install Java (Jenkins dependency)
sudo apt install -y fontconfig openjdk-17-jre

# Installing  Jenkins to EC2 Instance
sudo wget -O /usr/share/keyrings/jenkins-keyring.asc \
  https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key
echo "deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] https://pkg.jenkins.io/debian-stable binary/" | \
  sudo tee /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt update
sudo apt install -y jenkins
sudo systemctl start jenkins
sudo systemctl enable jenkins

# Getting Jenkins initial password key
sudo cat /var/lib/jenkins/initialAdminPassword


