# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Purpose

This is a deployment management directory for organizing and deploying various projects. Each project gets its own subdirectory with managed environments, resources, and configurations.

## Directory Structure

```
/deployments/
├── deployment-logs/           # Resource tracking and deployment logs
│   ├── resource-registry.json # Central resource allocation registry
│   ├── README.md              # Log structure documentation
│   └── {project}-deployment.json # Individual project logs
└── {project-name}/            # Individual project directories
```

## Deployment Workflow

### Before Deploying Any Project:

1. **Check Resource Registry**: Always read `deployment-logs/resource-registry.json` to avoid conflicts
2. **Allocate Resources**: Reserve ports, domains, and identify shared services
3. **Create Project Directory**: `mkdir {project-name}`
4. **Update Registry**: Add allocated resources to prevent conflicts

### Resource Management Rules:

- **Ports**: Check allocated ports, use next available port ranges
- **Domains**: Use subdomains or unique local domains to avoid conflicts
- **Shared Services**: Reuse databases, caches, etc. when multiple projects need the same service
- **Environment Variables**: Namespace env vars by project when using shared configs

### Required Files Per Deployment:

1. **Project deployment log**: `deployment-logs/{project-name}-deployment.json`
2. **Docker Compose**: Most deployments should use `docker-compose.yml`
3. **Environment file**: `.env` with project-specific variables
4. **README.md**: Basic project info and startup instructions

### Deployment Log Format:

Each project must have a corresponding log file with:
- Project name and deployment date
- Directory path
- Allocated ports and domains
- Services (containerized and shared)
- Environment variables
- Shared resources used
- Deployment notes

### Port Allocation Strategy:

- Web services: 3000-3999 range
- APIs: 4000-4999 range  
- Databases: 5000-5999 range
- Message queues: 6000-6999 range
- Other services: 7000+ range

### Common Shared Services:

- PostgreSQL: Usually shared across projects
- Redis: Shared cache/session store
- Message queues: RabbitMQ/Redis for multiple projects
- Reverse proxy: Nginx for domain routing

## Git Workflow

### Main Repository (Deployment Management):
- This directory is a git repository tracking overall deployment structure
- Commit changes to deployment logs, resource registry, and infrastructure updates
- Use frequent commits for rollback capability

### Individual Projects:
- Each project directory gets its own git repository (`git init` in project dir)  
- Separate version control for each deployed application
- Commit configurations, customizations, and deployment scripts per project
- **Naming Convention**: Use `[project_name][ThinkPad]` for GitHub repositories (e.g., `n8n[ThinkPad]`)

### Git Commands for Deployment Workflow:

```bash
# Main repo - after any infrastructure changes
git add . && git commit -m "update: deployment infrastructure changes"

# Project repos - after project setup/changes
cd {project-name}
git add . && git commit -m "deploy: initial {project-name} setup"
git add . && git commit -m "config: update environment variables"

# Rollback strategies
git log --oneline  # View commit history
git reset --hard HEAD~1  # Rollback last commit
git checkout {commit-hash} -- file.txt  # Restore specific file
```

### Commit Frequency Rules:
- **Before major changes**: Always commit current state
- **After successful deployment**: Commit working configuration
- **After resource allocation**: Update and commit registry changes
- **Before troubleshooting**: Commit to create restore point
- **After fixing issues**: Commit the fix for future reference

## Essential Commands

### Start a deployment:
```bash
# Check current resource usage
cat deployment-logs/resource-registry.json

# Deploy new project
mkdir {project-name}
cd {project-name}

# Initialize project git repo
git init
git branch -m main

# Set up project files...

# Initial project commit
git add . && git commit -m "deploy: initial {project-name} setup"

# Create GitHub repo for project using naming convention  
gh repo create "{project-name}[ThinkPad]" --private --description "ThinkPad deployment of {project-name}" --source=. --push

# Update resource registry and create deployment log (in main repo)
cd ..
# Update files...
git add . && git commit -m "registry: allocate resources for {project-name}"
git push
```

### Manage existing deployments:
```bash
# Start project services
cd {project-name} && docker-compose up -d

# Stop project services  
cd {project-name} && docker-compose down

# View project logs
cd {project-name} && docker-compose logs -f
```

### Resource conflict resolution:
```bash
# Check port usage
netstat -tlnp | grep :{port}

# Find domain conflicts
grep -r "domain.name" */

# List all allocated resources
jq '.ports.allocated' deployment-logs/resource-registry.json
```

## Project Isolation

Each project should be:
- Self-contained in its directory
- Use Docker Compose for orchestration
- Have isolated environment variables
- Document shared resource dependencies
- Cleanly startable/stoppable independently