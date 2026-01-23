#!/bin/bash
set -e

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

# Installation logic
install_with_package_manager() {
    echo "Installing Python using package manager..."

    if [ "$PKG_MANAGER" == "apt" ]; then
        sudo apt update
        sudo apt install -y python3 python3-pip
    else
        sudo yum install -y python3 python3-pip
    fi
}

install_with_tarball() {
    echo "Installing Python from tarball..."

    VERSION="$PYTHON_VERSION"
    URL="https://www.python.org/ftp/python/$VERSION/Python-$VERSION.tgz"

    echo "Downloading: $URL"
    wget $URL

    echo "Extracting source..."
    tar -xzf Python-$VERSION.tgz
    cd Python-$VERSION

    echo "Configuring build..."
    ./configure --enable-optimizations

    echo "Compiling source..."
    make

    echo "Installing..."
    sudo make install

    cd ..
}

# Execute based on install method
if [ "$INSTALL_METHOD" == "package" ]; then
    install_with_package_manager
elif [ "$INSTALL_METHOD" == "tarball" ]; then
    install_with_tarball
else
    echo "ERROR: Invalid INSTALL_METHOD. Use 'package' or 'tarball'."
    exit 1
fi

# Validation
echo "======================================"
echo " Validating Installation"
echo "======================================"

python3 --version
which python3
python3 -c "print('Python OK')"

echo "======================================"
echo " Python Installation Completed Successfully"
echo "======================================"
