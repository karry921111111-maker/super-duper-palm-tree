# super-duper-palm-tree# 1. 创建本地项目目录
mkdir chronic-disease-health-profile-saas
cd chronic-disease-health-profile-saas

# 2. 初始化git
git init

# 3. 创建README（暂时）
echo "# Chronic Disease Health Profile SaaS" > README.md

# 4. 配置git
git add README.md
git commit -m "Initial commit"
git branch -M main
git remote add origin https://github.com/karry921111111-maker/chronic-disease-health-profile-saas.git
git push -u origin main
#!/bin/bash

# 创建所有目录结构
mkdir -p backend/src/main/java/com/chronicdisease/{config,controller,dto,entity,exception,filter,repository,service,util}
mkdir -p backend/src/main/resources
mkdir -p frontend/src/{components,pages,services,store,types,hooks,utils,styles}
mkdir -p frontend/src/store/slices
mkdir -p frontend/src/components/{Layout,Dashboard,Patient,Analytics,Auth}
mkdir -p frontend/public
mkdir -p ml-service/{models,routes,data,utils,ml_models}
mkdir -p .github/workflows
mkdir -p docs

# 创建空的 __init__.py 和 .gitkeep 文件
touch ml-service/__init__.py
touch ml-service/models/__init__.py
touch ml-service/routes/__init__.py
touch ml-service/data/__init__.py
touch ml-service/utils/__init__.py
touch ml-service/ml_models/.gitkeep
touch frontend/public/.gitkeep
touch docs/.gitkeep

echo "✅ Directory structure created successfully!"
