# UV Deadline Plugin

A high-performance Deadline plugin that uses [Astral UV](https://github.com/astral-sh/uv) for Python environment and package management.

## Overview

- **80x faster** virtual environment creation compared to micromamba/conda
- **7x+ faster** package installation
- **Built-in Python version management** — no need to pre-install Python
- **Better package caching** — reduces redundant downloads across workers
- **Simpler architecture** — temporary venvs instead of named environments

## Architecture

### Environment Lifecycle

1. **Pre-Render**: Create temporary venv in `/tmp/uv-venvs/{random_hex}`
2. **Install Base Packages**: Install `python-dotenv` required by Runner.py
3. **Install Job Requirements**: Install packages from requirements.txt (if provided)
4. **Execute Task**: Run script using venv's Python interpreter
5. **Cleanup**: Delete temporary venv directory

## Installation

### Worker Requirements

1. **Install UV** on all farm workers:
   ```bash
   curl -LsSf https://astral.sh/uv/install.sh | sh
   ```

2. **Pre-install Python versions** (optional — UV auto-downloads as needed):
   ```bash
   uv python install 3.11
   uv python install 3.12
   ```

### Deadline Repository Setup

1. Copy the plugin contents to your Deadline repository's custom plugins folder:
   ```
   <DeadlineRepository>/custom/plugins/UV/
   ```

2. Restart Deadline workers to load the new plugin.

## Plugin Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `PythonVersion` | string | No | Python version (e.g., '3.11'). Defaults to '3.11' |
| `ScriptFile` | path | Yes | Path to the Python script to execute |
| `EnvironmentFile` | path | No | Path to .env file with environment variables |
| `RequirementsFile` | path | No | Path to requirements.txt for package installation |
| `Arguments` | string | No | Command-line arguments to pass to the script |
| `StartupDirectory` | path | Yes | Working directory for script execution |
| `CacheDirectory` | path | No | UV package cache directory. Defaults to `/tmp/uv-cache` |
| `SingleFramesOnly` | boolean | No | Render one frame at a time. Defaults to True |

## Performance Tuning

### Cache Directory

The UV cache stores downloaded packages for reuse across jobs:

- **Shared Network Storage**: Mount a shared cache directory across workers for maximum reuse
- **Local SSD**: Use local fast storage if network latency is high
- **Cleanup**: Periodically clean old cache entries

### Python Pre-installation

Pre-install Python versions on workers to avoid per-job download overhead:

```bash
uv python install 3.11
uv python install 3.12
```

## Troubleshooting

### UV Not Found

Ensure UV is installed and in PATH:
```bash
which uv
```

### Permission Errors on Cleanup

Check permissions on the venv directory:
```bash
chmod 1777 /tmp/uv-venvs/
```

### Slow Package Installation

1. Use a shared cache directory across workers
2. Pre-populate cache with common packages
3. Use locked requirements for reproducibility:
   ```bash
   uv pip compile requirements.in -o requirements.txt --universal
   ```

## References

- [UV Documentation](https://docs.astral.sh/uv/)
- [Deadline Plugin Development](https://docs.thinkboxsoftware.com/products/deadline/10.1/1_User%20Manual/manual/manual-plugins.html)
