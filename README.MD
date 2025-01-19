# Backups Library for Obsidian Server Portal

The `backups` library is a Git-dependency-only Rust library for the [Obsidian Server Portal](https://github.com/Obsidian-Minecraft-Server-Portal). It provides utilities for managing, scheduling, creating full or incremental backups, and handling backup-related tasks for Minecraft servers.

## Features

- **Database-backed Backup Management**: Store, retrieve, and manage backups using SQLite.
- **Full and Incremental Backups**: Create full backups of the server directory or incremental backups based on changes.
- **Hashing Utilities**: Efficient file hashing to detect changed files for incremental backups.
- **Backup Scheduling**: Schedule recurring backups using custom intervals.
- **WorldEdit Backup Support**: Handle WorldEdit-specific backup tasks for Minecraft worlds.
- **Thread Safety and Concurrency**: Uses `rayon` for handling backups in parallel.
- **Time and Path Formatting**: Utility functions to seamlessly handle system times and paths.

## Installation

This library is not published on [crates.io](https://crates.io). Add it as a Git dependency in your `Cargo.toml`:

```toml
[dependencies]
backups = { git = "https://github.com/Obsidian-Minecraft-Server-Portal/backups.git" }
```

### Required Git Sub-Dependencies

The `backups` library depends on the following Obsidian libraries:

- [obsidian-cryptography](https://github.com/Obsidian-Minecraft-Server-Portal/cryptography.git)
- [obsidian-sqlite](https://github.com/Obsidian-Minecraft-Server-Portal/sqlite.git)
- [obsidian-zip](https://github.com/Obsidian-Minecraft-Server-Portal/zip.git)

Ensure your `Cargo.toml` includes these dependencies properly. If you are using this library, these dependencies should already be pulled automatically.

## Usage

### Initialization

Before creating or managing backups, you must initialize the database and environment:

```rust
use backups::initialize;

fn main() {
    // Initialize the backup and file hash databases
    initialize();
}
```

### Creating Full Backups

Create a full backup of a server directory:

```rust
use backups::backup_item::{BackupItem, BackupCreationMethod, BackupType};

let server_id = 1;
let server_directory = "/path/to/server/directory";

let backup = BackupItem::create_backup(
    server_id,
    server_directory,
    BackupCreationMethod::MANUAL,
    BackupType::Full,
);
match backup {
    Ok(item) => println!("Backup created: {:?}", item),
    Err(e) => eprintln!("Failed to create backup: {}", e),
}
```

### Creating Incremental Backups

Incremental backups only archive files that have changed since the last backup:

```rust
let backup = BackupItem::create_backup(
    server_id,
    server_directory,
    BackupCreationMethod::AUTO,
    BackupType::Incremental,
);
```

### Managing Backups

- **Listing Backups**:

```rust
let backups = BackupItem::list();
for backup in backups {
    println!("Backup ID: {} - Path: {:?}", backup.id, backup.path);
}
```

- **Fetching a Backup by ID**:

```rust
let backup = BackupItem::from_id(1);
if let Some(item) = backup {
    println!("Backup: {:?}", item);
} else {
    println!("No backup found with ID 1");
}
```

- **Deleting a Backup**:

```rust
BackupItem::delete(1);
println!("Backup with ID 1 deleted.");
```

- **Trimming Old Backups**:

Keep only the most recent `n` backups for a server:

```rust
BackupItem::trim(server_id, 5);
println!("Backups trimmed for server ID: {}", server_id);
```

### Scheduling Backups

The library provides functions to manage scheduled backups. Example to add a new schedule:

```rust
use backups::backup_schedules_db::insert;
use backups::backup_item::BackupType;

let result = insert(
    server_id,
    BackupType::Full,
    86400, // Interval in seconds (1 day)
    true,  // Execute even if the server directory is empty
    true,  // Execute even if the server is offline
);

if result.is_ok() {
    println!("Backup schedule added successfully!");
} else {
    eprintln!("Failed to add backup schedule: {:?}", result);
}
```

### Hashing Utilities

Retrieve and store file hashes for smooth incremental backups:

```rust
use backups::file_hash_db;

file_hash_db::insert("/path/to/file", "file_hash").unwrap();
let exists = file_hash_db::exists("/path/to/file").unwrap();
println!("File exists in hash database: {}", exists);
```

## Testing

Unit tests are included for various utilities (e.g., time formatting, database operations). Use the following command to run tests:

```sh
cargo test
```

## Contributions

Contributions and feedback are welcome! If you encounter any bugs or have feature requests, please [open an issue](https://github.com/Obsidian-Minecraft-Server-Portal/backups/issues).

## License

This project is licensed under the **LGPL-3.0 License**. See the [LICENSE](LICENSE) file for details.

## Acknowledgments

Special thanks to the Obsidian Server Portal team for their dedication to creating tools for the Minecraft server community.