# Database Support

This package provides unified database support for SQLite, PostgreSQL, and MySQL databases using GORM.

## Supported Databases

- **SQLite** - File-based database (default)
- **PostgreSQL** - Full-featured relational database
- **MySQL** - Popular relational database

## Configuration

### Database Initialization Control

For faster startup times, you can skip database initialization (AutoMigrate and InitData) by setting the `SKIP_DB_INIT` environment variable to `1`.

### Performance Impact

- **With initialization** (default): Full schema migration and data initialization (~11 seconds)
- **Without initialization** (`SKIP_DB_INIT=1`): Skip migration and data init (~1.8 seconds, **6x faster**)

### Usage

```bash
# Fast startup (skip initialization)
SKIP_DB_INIT=1 ./lazy-rabbit-secretary job --list

# Normal startup (with initialization) - default behavior
./lazy-rabbit-secretary job --list
```

### When to Use

- **Skip initialization (`SKIP_DB_INIT=1`)**:
  - Production environments where database is already initialized
  - Development when database schema is stable
  - CI/CD pipelines where speed is important
  - Testing scenarios

- **Enable initialization (default)**:
  - First-time setup
  - After schema changes
  - When database needs to be reset
  - Development with frequent schema changes

## Database Logging

The database package includes configurable SQL query logging using GORM's built-in logger. This is separate from the application's unified logger and specifically controls SQL query visibility.

#### Log Levels

| Level | Description | Use Case |
|-------|-------------|----------|
| `silent` | No SQL logging | Production (minimal overhead) |
| `error` | Only log SQL errors | Production (recommended default) |
| `warn` | Log SQL errors and slow queries | Production monitoring |
| `info` | Log all SQL queries with timing | Development and debugging |
| `debug` | Same as info (most verbose) | Development and debugging |

#### Configuration Methods

**Config File (config.yaml):**
```yaml
database:
  log_level: "error"  # silent, error, warn, info, debug
```

**Environment Variable:**
```bash
DB_LOG_LEVEL=info ./lazy-rabbit-secretary
```

**Default Behavior:**
- Default level: `error` (reduces noise while showing important issues)
- Environment variables override config file settings

### Environment Variables

You can configure the database using environment variables:

```bash
# Database type (sqlite, postgres, mysql)
export DB_TYPE=postgres

# Database connection details
export DB_HOST=localhost
export DB_PORT=5432
export DB_USERNAME=postgres
export DB_PASSWORD=your_password
export DB_NAME=lazy-rabbit-secretary

# PostgreSQL specific
export DB_SSL_MODE=disable

# MySQL specific  
export DB_CHARSET=utf8mb4

# SQLite specific
export DB_FILE_PATH=lazy-rabbit-secretary.db

# Database logging
export DB_LOG_LEVEL=error  # silent, error, warn, info, debug

# Database initialization control
export SKIP_DB_INIT=1  # Set to 0 to skip AutoMigrate and InitData for faster startup

# Default user credentials
export ADMIN_USERNAME=admin
export ADMIN_PASSWORD=admin123
export DEFAULT_EMAIL=admin@example.com
```

### Configuration File

Alternatively, you can configure the database in `config/config.yaml`:

```yaml
database:
  type: sqlite  # sqlite, postgres, mysql
  host: localhost
  port: 5432
  username: postgres
  password: ""
  database: lazy-rabbit-secretary
  ssl_mode: disable
  charset: utf8mb4
  file_path: lazy-rabbit-secretary.db
```

## Usage

### Initialize Database

```go
import "github.com/walterfan/lazy-rabbit-secretary/pkg/database"

func main() {
    // Initialize database connection
    if err := database.InitDB(); err != nil {
        log.Fatal("Failed to initialize database:", err)
    }
    defer database.CloseDB()
    
    // Use the database
    db := database.GetDB()
    // ... your database operations
}
```

### Database Types

#### SQLite (Default)
```bash
export DB_TYPE=sqlite
export DB_FILE_PATH=lazy-rabbit-secretary.db
```

#### PostgreSQL
```bash
export DB_TYPE=postgres
export DB_HOST=localhost
export DB_PORT=5432
export DB_USERNAME=postgres
export DB_PASSWORD=your_password
export DB_NAME=lazy-rabbit-secretary
export DB_SSL_MODE=disable
```

#### MySQL
```bash
export DB_TYPE=mysql
export DB_HOST=localhost
export DB_PORT=3306
export DB_USERNAME=root
export DB_PASSWORD=your_password
export DB_NAME=lazy-rabbit-secretary
export DB_CHARSET=utf8mb4
```

## Docker Deployment

### PostgreSQL with Docker Compose

```yaml
services:
  postgres:
    image: postgres:15
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: your_password
      POSTGRES_DB: lazy-rabbit-secretary
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data

  lazy-rabbit-secretary:
    build: .
    environment:
      DB_TYPE: postgres
      DB_HOST: postgres
      DB_PORT: 5432
      DB_USERNAME: postgres
      DB_PASSWORD: your_password
      DB_NAME: lazy-rabbit-secretary
    depends_on:
      - postgres
```

### MySQL with Docker Compose

```yaml
services:
  mysql:
    image: mysql:8.0
    environment:
      MYSQL_ROOT_PASSWORD: your_password
      MYSQL_DATABASE: lazy-rabbit-secretary
      MYSQL_USER: user
      MYSQL_PASSWORD: user_password
    ports:
      - "3306:3306"
    volumes:
      - mysql_data:/var/lib/mysql

  lazy-rabbit-secretary:
    build: .
    environment:
      DB_TYPE: mysql
      DB_HOST: mysql
      DB_PORT: 3306
      DB_USERNAME: user
      DB_PASSWORD: user_password
      DB_NAME: lazy-rabbit-secretary
    depends_on:
      - mysql
```

## Features

- **Auto-migration**: Automatically creates and updates database schema
- **Environment-based configuration**: Override settings with environment variables
- **Default data initialization**: Creates default admin user and loads prompts
- **Connection pooling**: Managed by GORM
- **Error handling**: Comprehensive error handling and logging

## Migration from SQLite

To migrate from SQLite to PostgreSQL or MySQL:

1. Export your SQLite data
2. Update your configuration
3. Import data to the new database
4. Restart the application

The application will automatically handle schema migration and data initialization. 