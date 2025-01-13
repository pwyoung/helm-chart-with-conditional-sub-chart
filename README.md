# Flexible Database Helm Chart

This Helm chart demonstrates a flexible application deployment pattern that can either install a PostgreSQL database as a subchart or connect to an external database based on configuration.

## Overview

The chart provides two deployment options:
1. Install PostgreSQL alongside your application using Bitnami's PostgreSQL chart
2. Connect to an existing external PostgreSQL database

This flexibility allows the same chart to be used in different environments - development, staging, or production - where database requirements might differ.

## Chart Structure

```
my-application/
├── Chart.yaml
├── values.yaml
└── templates/
    ├── configmap.yaml
    ├── secret.yaml
    └── deployment.yaml
```

### Chart.yaml

The main chart definition file that specifies metadata and dependencies.

```yaml
apiVersion: v2
name: my-application
description: A Helm chart that conditionally installs a database
version: 0.1.0
dependencies:
  - name: postgresql
    version: "12.5.6"
    repository: https://charts.bitnami.com/bitnami
    condition: postgresql.enabled
```

Key points:
- Defines PostgreSQL as a conditional dependency
- Uses Bitnami's PostgreSQL chart version 12.5.6
- The `condition: postgresql.enabled` makes the dependency optional

### values.yaml

Default configuration values that control the chart's behavior.

```yaml
postgresql:
  enabled: false
  auth:
    postgresPassword: ""
    database: myapp

externalDatabase:
  host: ""
  port: 5432
  database: myapp
  user: postgres
  password: ""
```

Configuration sections:
- `postgresql`: Controls the PostgreSQL subchart
  - `enabled`: Toggle for installing PostgreSQL
  - `auth`: PostgreSQL authentication settings
- `externalDatabase`: External database connection details
  - Used when `postgresql.enabled` is false
  - All fields required except `port` (defaults to 5432)

### Template Files

#### configmap.yaml

Creates a ConfigMap containing non-sensitive database configuration:
- Database host
- Port
- Database name
- Username

Uses conditional logic to set values based on the chosen database option.

#### secret.yaml

Creates a Secret containing sensitive database credentials:
- Database password (base64 encoded)

Uses the same conditional logic to select between PostgreSQL and external database passwords.

#### deployment.yaml

Creates a Deployment for your application that:
- Mounts database configuration from ConfigMap and Secret
- Makes configuration available as environment variables
- Works transparently with either database option

## Installation

### With PostgreSQL Subchart

```bash
helm install myapp . \
  --set postgresql.enabled=true \
  --set postgresql.auth.postgresPassword=mypassword
```

This will:
1. Install the PostgreSQL subchart
2. Create a database named 'myapp'
3. Set up the specified password
4. Configure your application to connect to the installed PostgreSQL

### With External Database

```bash
helm install myapp . \
  --set externalDatabase.host=db.example.com \
  --set externalDatabase.password=mypassword
```

This will:
1. Skip the PostgreSQL installation
2. Configure your application to connect to the specified external database
3. Use the provided credentials for authentication

## Configuration Reference

| Parameter | Description | Default |
|-----------|-------------|---------|
| `postgresql.enabled` | Install PostgreSQL subchart | `false` |
| `postgresql.auth.postgresPassword` | PostgreSQL password | `""` |
| `postgresql.auth.database` | PostgreSQL database name | `"myapp"` |
| `externalDatabase.host` | External database host | `""` |
| `externalDatabase.port` | External database port | `5432` |
| `externalDatabase.database` | External database name | `"myapp"` |
| `externalDatabase.user` | External database user | `"postgres"` |
| `externalDatabase.password` | External database password | `""` |

## Environment Variables

Your application will receive the following environment variables:

- `DB_HOST`: Database hostname
- `DB_PORT`: Database port
- `DB_NAME`: Database name
- `DB_USER`: Database username
- `DB_PASSWORD`: Database password

These variables are set consistently regardless of which database option is chosen.

## Security Considerations

- Sensitive data (passwords) is stored in Kubernetes Secrets
- Non-sensitive configuration is stored in ConfigMaps
- PostgreSQL passwords must be provided at installation time
- External database configuration is validated before deployment

## Troubleshooting

Common issues and solutions:

1. **Chart fails to install with PostgreSQL enabled**
   - Ensure you've provided `postgresql.auth.postgresPassword`
   - Check if you have sufficient cluster resources

2. **Chart fails to install with external database**
   - Verify all required external database parameters are provided
   - Ensure the external database is accessible from your cluster

3. **Application cannot connect to database**
   - Check the ConfigMap and Secret were created correctly
   - Verify network connectivity to the database
   - Ensure database credentials are correct

## Development

To modify this chart:

1. Update dependencies:
```bash
helm dependency update
```

2. Test the chart:
```bash
helm template . --debug
```

3. Validate values:
```bash
helm lint
```

## Contributing

When contributing to this chart:

1. Update documentation when changing values
2. Test both PostgreSQL and external database scenarios
3. Follow Helm best practices for templating
4. Consider backwards compatibility