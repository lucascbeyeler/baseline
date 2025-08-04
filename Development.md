# Development and Testing Guide

This document outlines the steps to set up your development environment and run tests using `molecule`.

## 1. Prepare the Environment with `venv`

It is recommended to use a Python virtual environment (`venv`) to manage dependencies.

1.  **Create a virtual environment:**
    ```bash
    python3 -m venv venv
    ```

2.  **Activate the virtual environment:**
    *   On Linux/macOS:
        ```bash
        source venv/bin/activate
        ```
    *   On Windows:
        ```bash
        .\venv\Scripts\activate
        ```

3.  **Install dependencies:**
    ```bash
    pip install -r requirements.txt
    ```

## 2. Run Molecule Tests

`Molecule` is used for testing Ansible roles. Before running `molecule` tests, ensure your Docker environment is correctly configured.

1.  **Check Docker Context (if applicable):**
    If you are using Docker Desktop or have multiple Docker contexts, you might need to set the `DOCKER_HOST` environment variable.

    First, list your Docker contexts:
    ```bash
    docker context ls
    ```
    Example output:
    ```
    NAME              DESCRIPTION                               DOCKER ENDPOINT                                         ERROR
    default           Current DOCKER_HOST based configuration   unix:///var/run/docker.sock                             
    desktop-linux *   Docker Desktop                            unix:///home/lucasbeyeler/.docker/desktop/docker.sock  
    ```

    If `desktop-linux` (or a similar context pointing to Docker Desktop) is not the active context, you need to export the `DOCKER_HOST` variable.

2.  **Set `DOCKER_HOST` (if necessary):**
    ```bash
    export DOCKER_HOST=unix:///home/lucasbeyeler/.docker/desktop/docker.sock
    ```
    *Note: Adjust the path to `docker.sock` based on your `docker context ls` output.*

3.  **Run Molecule tests:**
    Navigate to the `molecule/default` directory (or the specific scenario you want to test) and run the `molecule test` command.

    ```bash
    cd molecule/default
    molecule test
    ```
    This command will perform the following steps: `destroy`, `create`, `prepare`, `converge`, `idempotence`, `lint`, and `verify`.