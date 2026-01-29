✔ Supports multiple Python versions
✔ Supports package manager and tarball installs
✔ Uses version from config.env correctly
✔ Installs build dependencies automatically
✔ Uses safe altinstall (does not break system python)
✔ Supports upgrade behavior
✔ Cleans up files
✔ Logs installation
✔ Validates installation
✔ Works on apt and yum systems





#!/bin/bash
set -e

LOG_DIR="logs"
LOG_FILE="$LOG_DIR/install.log"
mkdir -p "$LOG_DIR"

exec > >(tee -a "$LOG_FILE") 2>&1

echo "======================================"
echo " Generic Python Installer Started"
echo "======================================"

# Load configuration
if [ -f config.env ]; then
    echo "Loading configuration from config.env..."
    source config.env
else
    echo "ERROR: config.env file not found!"
    exit 1
fi

# Validate inputs
if [ -z "$PYTHON_VERSION" ]; then
    echo "ERROR: PYTHON_VERSION is not set in config.env"
    exit 1
fi

if [ -z "$INSTALL_METHOD" ]; then
    echo "ERROR: INSTALL_METHOD is not set in config.env"
    exit 1
fi

echo "Python Version  : $PYTHON_VERSION"
echo "Install Method  : $INSTALL_METHOD"

# Detect package manager
if command -v apt >/dev/null 2>&1; then
    PKG_MANAGER="apt"
elif command -v yum >/dev/null 2>&1; then
    PKG_MANAGER="yum"
else
    echo "ERROR: Supported package manager not found (apt/yum)"
    exit 1
fi

echo "Detected Package Manager: $PKG_MANAGER"

# Check existing installation
check_existing_python() {
    if command -v python${PYTHON_VERSION} >/dev/null 2>&1; then
        echo "Python ${PYTHON_VERSION} already installed."
        INSTALLED_VERSION=$(python${PYTHON_VERSION} --version 2>&1)
        echo "Installed Version: $INSTALLED_VERSION"
        echo "Skipping installation."
        exit 0
    fi
}

# Install dependencies for tarball build
install_dependencies() {
    echo "Installing build dependencies..."

    if [ "$PKG_MANAGER" == "apt" ]; then
        sudo apt update
        sudo apt install -y \
            build-essential \
            libssl-dev \
            zlib1g-dev \
            libffi-dev \
            libbz2-dev \
            libreadline-dev \
            libsqlite3-dev \
            wget
    else
        sudo yum groupinstall -y "Development Tools"
        sudo yum install -y \
            openssl-devel \
