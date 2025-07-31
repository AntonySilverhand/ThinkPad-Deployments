# Deployment Logs

This directory contains deployment tracking and resource management files.

## Files

- `resource-registry.json` - Central registry of allocated ports, domains, and shared services
- `{project-name}-deployment.json` - Individual project deployment logs (created per project)

## Project Deployment Log Structure

Each project gets its own deployment log file named `{project-name}-deployment.json` with:

```json
{
  "project_name": "example-project",
  "deployment_date": "2025-07-31",
  "directory": "/home/antony/coding/deployments/example-project",
  "ports": [3000, 5432],
  "domains": ["example.local", "api.example.local"],
  "services": {
    "main": "docker-compose.yml",
    "database": "shared:postgres-main"
  },
  "environment": {
    "NODE_ENV": "production",
    "DB_HOST": "postgres-main"
  },
  "shared_resources": ["postgres-main"],
  "notes": "Additional deployment notes"
}
```

## Resource Management

- Check `resource-registry.json` before allocating new resources
- Update registry when deploying new projects
- Use shared services when possible to avoid conflicts