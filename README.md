# Plane for Home Assistant

Self-hosted [Plane](https://plane.so/) project management as a Home Assistant add-on.

Runs the full Plane stack (PostgreSQL, Valkey, RabbitMQ, MinIO) in a single addon container.

## Installation

Add this repository to your Home Assistant add-on store, then install the Plane add-on.

[![Open your Home Assistant instance and show the add add-on repository dialog.](https://my.home-assistant.io/badges/supervisor_add_addon_repository.svg)](https://my.home-assistant.io/redirect/supervisor_add_addon_repository/?repository_url=https%3A%2F%2Fgithub.com%2Fnkcreation%2Fplane-ha-app)

## Configuration

Set `domain_name` to the hostname you'll use to access Plane (e.g., `plane.example.com`), then access it on port **8080**.

See the [full documentation](plane/DOCS.md) for details.

## Requirements

- Home Assistant Yellow or equivalent (aarch64/amd64)
- 4 GB+ RAM
- 20 GB+ disk space
