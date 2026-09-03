# =====================================================================
# Análisis del Provider PostgreSQL
# =====================================================================

## Provider: tages/provider-postgresql v0.1.0

### 1. Managed Resources disponibles

| Managed Resource | API Group | Propósito |
|---|---|---|
| **Database** | `postgresql.postgresql.upbound.io` | Crea y gestiona bases de datos PostgreSQL (`CREATE DATABASE`). Recurso principal de esta PoC. |
| **Role** | `postgresql.postgresql.upbound.io` | Crea roles/usuarios con privilegios configurables (login, createdb, superuser, replication, etc.). |
| **Grant** | `postgresql.postgresql.upbound.io` | Gestiona permisos sobre objetos de base de datos (tablas, esquemas, secuencias) para un rol específico. |
| **GrantRole** | `postgresql.postgresql.upbound.io` | Gestiona membresía de roles (agregar un rol como miembro de otro, herencia de privilegios). |
| **DefaultPrivileges** | `postgresql.postgresql.upbound.io` | Define privilegios por defecto que se aplican automáticamente a objetos futuros creados en un esquema. |
| **Schema** | `postgresql.postgresql.upbound.io` | Crea y gestiona esquemas dentro de una base de datos, con políticas de ownership y acceso. |
| **Extension** | `postgresql.postgresql.upbound.io` | Instala extensiones de PostgreSQL (ej: `uuid-ossp`, `hstore`, `pg_stat_statements`). |
| **Publication** | `replication.postgresql.upbound.io` | Gestiona publicaciones de replicación lógica (lado publisher). |
| **Subscription** | `replication.postgresql.upbound.io` | Gestiona suscripciones de replicación lógica (lado subscriber). |
| **ReplicationSlot** | `replication.postgresql.upbound.io` | Gestiona slots de replicación lógica. |
| **PhysicalReplicationSlot** | `physical.postgresql.upbound.io` | Gestiona slots de replicación física. |
| **Server** | `postgresql.postgresql.upbound.io` | Gestiona servidores de foreign data wrappers (`CREATE SERVER`). |
| **UserMapping** | `user.postgresql.upbound.io` | Gestiona mapeos de usuario para servidores foráneos. |
| **Function** | `postgresql.postgresql.upbound.io` | Crea y gestiona funciones/procedimientos almacenados en PostgreSQL. |

### 2. Campos requeridos del recurso Database

| Campo | Requerido | Descripción |
|---|---|---|
| `name` | Sí | Nombre de la base de datos a crear. |
| `owner` | No | Rol propietario de la base de datos. Si no se especifica, usa el usuario de conexión del ProviderConfig. |
| `encoding` | No | Codificación de caracteres (ej: `UTF8`). |
| `lcCollate` | No | Locale de collation (ej: `en_US.UTF-8`). |
| `lcCtype` | No | Locale de clasificación de caracteres. |
| `template` | No | Base de datos template para la creación (ej: `template0`). |
| `connectionLimit` | No | Número máximo de conexiones concurrentes (-1 = sin límite). |
| `allowConnections` | No | Si se permiten conexiones a la base de datos (default: true). |
| `isTemplate` | No | Si la base de datos puede usarse como template. |
| `tablespace` | No | Tablespace donde se almacenará la base de datos. |

### 3. Información requerida por el ProviderConfig

El ProviderConfig necesita un Secret de Kubernetes que contenga un JSON de conexión. En nuestro taller, el Secret se creó así:
 
```bash
kubectl create secret generic postgresql-credentials \
  --namespace crossplane-system \
  --from-literal=connection='{"host":"postgresql.postgresql.svc.cluster.local","port":"5432","username":"postgres","password":"platform123","database":"postgres","sslmode":"disable"}'
```
 
Los campos del JSON de conexión son:
 
| Campo | Valor en nuestra PoC | Descripción |
|---|---|---|
| `host` | `postgresql.postgresql.svc.cluster.local` | DNS interno del servicio PostgreSQL desplegado con Helm en el namespace `postgresql`. |
| `port` | `5432` | Puerto estándar de PostgreSQL. |
| `username` | `postgres` | Usuario administrador con privilegios para crear bases de datos y roles. |
| `password` | `platform123` | Contraseña configurada en el `helm install` de PostgreSQL (`--set auth.postgresPassword=platform123`). |
| `database` | `postgres` | Base de datos de administración para la conexión inicial del provider. |
| `sslmode` | `disable` | Deshabilitado porque la comunicación es intra-cluster (Kind local). |
 
El ProviderConfig (`provider-config.yaml`) referencia este Secret mediante:
 
```yaml
spec:
  credentials:
    source: Secret
    secretRef:
      namespace: crossplane-system
      name: postgresql-credentials
      key: connection
```
 
Cada Managed Resource o Composition referencia esta configuración con `providerConfigRef.name: postgresql-config`.
