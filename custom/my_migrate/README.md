# My Migrate Module

This module defines the migrations for importing program data, program categories, and images into your Drupal site using the Migrate API. The migrations are defined with configuration files and are designed to be run using Drupal's Migrate system.

## Table of Contents
- [Module Overview](#module-overview)
- [Migration Files](#migration-files)
- [Migration Configuration](#migration-configuration)
- [Running the Migrations](#running-the-migrations)

## Module Overview

The `my_migrate` module includes several migration configurations that import different types of data:

1. **Programs**: This migration imports program data from a CSV file into a custom content type `program`.
2. **Program Categories**: This migration imports taxonomy terms (categories) from a CSV file into a vocabulary named `Program Categories`.
3. **Program Images**: This migration imports image files from a directory and associates them with the respective content in Drupal.
4. **Migration Group**: The migration group `my_migrate` groups all migration tasks related to programs, categories, and images.

## Migration Files

The following files define the migrations for the `my_migrate` module:

1. **migrate_plus.migration.my_migrate_programs.yml**:
   - Defines the migration for importing programs from a CSV file.
   - Maps program data (e.g., title, type, level) to the corresponding fields in the `program` content type.
   
2. **migrate_plus.migration.program_categories.yml**:
   - Defines the migration for importing taxonomy terms (program categories) into the `Program Categories` vocabulary.
   
3. **migrate_plus.migration.program_image.yml**:
   - Defines the migration for importing images into Drupal and storing them in the `/sites/default/files/migrate_destination` directory.
   
4. **migrate_plus.migration_group.my_migrate.yml**:
   - Defines the migration group `my_migrate` to group all related migrations together.

5. **my_migrate_data.yml**:
   - Contains additional configuration and data mapping for the migrations. This file could include specific transformation rules or additional settings for your migrations.

## Migration Configuration

### `my_migrate_programs.yml`

This migration imports program data from a CSV file (`programs.csv`). The fields in the CSV are mapped to the `program` content type.

- **Source**: CSV file containing program data.
- **Process**: Maps columns such as `title`, `type`, `level`, and `tags` to fields in the `program` content type.
- **Destination**: The `program` content type.

### `program_categories.yml`

This migration imports program categories from a CSV file into a taxonomy vocabulary (`Program Categories`).

- **Source**: CSV file containing program category data.
- **Process**: Maps the category names to taxonomy terms.
- **Destination**: `Program Categories` taxonomy vocabulary.

### `program_image.yml`

This migration imports images into Drupal from a directory (`modules/custom/my_migrate/migrate_source/`). The images are stored in the `/sites/default/files/migrate_destination` directory.

- **Source**: Image files in the `migrate_source` directory.
- **Process**: Uses the `file_copy` process plugin to copy the images to the `public://migrate_destination` directory.
- **Destination**: Stores images as file entities.

### `my_migrate_group.yml`

Defines the migration group for organizing all related migrations under the `my_migrate` group.

## Running the Migrations

To run the migrations:

1. **Install the Migrate Module**:
   - Ensure that the Migrate module is enabled on your site.

2. **Enable the `my_migrate` Module**:
   - Enable the module via Drupal's admin interface or using Drush:
     ```bash
     drush en my_migrate -y
     ```

3. **Run the Migrations**:
   - You can run the migrations via Drush:
     ```bash
     drush migrate-import my_migrate_programs
     drush migrate-import program_categories
     drush migrate-import program_image
     ```
   - Alternatively, you can run all migrations in the group at once:
     ```bash
     drush migrate-import --group=my_migrate
     ```

4. **Check the Status of the Migrations**:
   - To check the status of your migrations, use:
     ```bash
     drush migrate-status
     ```

