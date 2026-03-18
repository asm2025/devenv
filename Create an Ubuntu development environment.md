## Mixed Dev Environment

### `WSL2 + Ubuntu 24.04` (Poetry + pnpm/Bun + C/C++ + Rust)

This guide is optimized for **WSL2 / Ubuntu 24.04** with a modern, high-performance stack: **Poetry** for Python, **pnpm/Bun** for Node/TS, **C/C++**, and **Rust**. It ensures **Git**, **VS Code**, and **Chrome** (via Windows) work seamlessly together.

References you provided:

- Microsoft: [Set up a WSL development environment](https://learn.microsoft.com/en-us/windows/wsl/setup/environment)
- DEV.to: [Supercharge your Windows development: the ultimate guide to WSL](https://dev.to/wasp/supercharge-your-windows-development-the-ultimate-guide-to-wsl-195m)
- Repo: [ultimate-windows-web-dev-setup](https://github.com/a99divx/ultimate-windows-web-dev-setup)
- Medium: [Configure a Windows development environment with WSL](https://medium.com/@inzuael/configure-a-windows-development-environment-with-wsl-02b9bd89cbff)

------

## Principles (WSL Performance & Stability)

### 1 Filesystem Rule

- **Always** store code in the Linux filesystem: `~/projects/...`
- **Never** run heavy builds (Node, Rust, C++) on `/mnt/c/...` (Windows drive). IO performance is significantly degraded across the 9P protocol boundary.

### 2 Toolchain Isolation

- **OS deps**: `apt`
- **Shell**: `Zsh` + `Oh-My-Zsh` + `ansi-dark` dircolors
- **Python**: `Poetry` (manages virtualenvs automatically)
- **Node/TS**: `nvm` (versioning) + `pnpm` (fastest/efficient) or `Bun` (runtime/package manager)
- **Rust**: `rustup`

------

## 1) Install WSL2 + Ubuntu (Windows)

### 1.1 Install WSL

Open **PowerShell (Admin)**:

```powershell
wsl --install
```

Reboot if prompted.
 Source: Microsoft WSL guide.

### 1.2 Verify WSL2

```powershell
wsl --list --verbose
```

If version is 1: `wsl --set-version Ubuntu 2`.

------

## 2) First run (Ubuntu): update + baseline toolchain

Open Ubuntu, create your Linux user.

### 2.1 Update packages

```bash
sudo apt update && sudo apt upgrade -y && sudo apt autoremove -y
```

### 2.2 Install baseline utilities + compilers

```bash
sudo apt install -y \
  build-essential \
  curl wget ca-certificates \
  git \
  unzip zip \
  jq \
  gpg \
  openssh-client \
  lsb-release software-properties-common \
  htop \
  cmake ninja-build pkg-config gdb
```

------

## 3) Shell Setup: Zsh + Oh-My-Zsh + Dircolors

### 3.1 Install Zsh

```
sudo apt install -y zsh  
chsh -s "$(which zsh)"  
```

### 3.2 Install Oh-My-Zsh

```
sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"  
```

### 3.3 Configure `ansi-dark` Dircolors

Download the `ansi-dark` theme to `~/dircolors`:

```
curl -L https://raw.githubusercontent.com/seebi/dircolors-solarized/master/dircolors.ansi-dark -o ~/dircolors  
```

Add the following to your `~/.zshrc` to apply the colors:

```
# Add to ~/.zshrc  
if [ -f ~/dircolors ]; then  
    eval $(dircolors -b ~/dircolors)  
fi  
```

------

## 4) Git (WSL) + SSH

### 4.1 Configure identity

```bash
git config --global user.name "Your Name"
git config --global user.email "you@example.com"
```

#### 4.2 Choose Your SSH Path

**Option A: You have an existing key from another PC**
 If you have your `id_ed25519` and `id_ed25519.pub` files (e.g., on a USB or in your Windows Downloads folder):

1. **Move keys into WSL**:

   ```
   mkdir -p ~/.ssh && chmod 700 ~/.ssh  
   # Replace 'YourUser' with your Windows username  
   cp /mnt/c/Users/YourUser/Downloads/id_ed25519* ~/.ssh/  
   ```

2. **Set strict permissions (Mandatory)**:
    SSH will ignore keys with "too open" permissions.

   ```
   chmod 600 ~/.ssh/id_ed25519  
   chmod 644 ~/.ssh/id_ed25519.pub  
   ```

**Option B: You want to generate a new key for this machine**

```
mkdir -p ~/.ssh && chmod 700 ~/.ssh  
ssh-keygen -t ed25519 -C "you@example.com"  
# Press Enter to save in default location, add a passphrase if desired.  
```

#### 4.3 Register the Key with the Agent

To avoid re-entering passphrases, add the agent to your `~/.zshrc`:

```
# Append to ~/.zshrc  
eval "$(ssh-agent -s)" > /dev/null  
ssh-add ~/.ssh/id_ed25519  
```

#### 4.4 Link to Git Host

Copy the public key to your clipboard and add it to GitHub/GitLab settings:

```
cat ~/.ssh/id_ed25519.pub  
```

#### 4.5 Verification

Test the handshake:

```
ssh -T git@github.com  
```

*Expected output: "Hi [username]! You've successfully authenticated..."*

------

## 5) VS Code (Windows) + Remote - WSL

### 5.1 Install VS Code on Windows

Install VS Code on Windows and the **Remote - WSL** extension.

### 5.2 Launch from WSL

```bash
mkdir -p ~/projects
cd ~/projects
code .
```

------

## 6) Chrome as Default Browser (WSL Integration)

### 6.1 Set Windows Default

Set **Google Chrome** as the default browser in Windows Settings.

### 6.2 WSL URL Opener (`wslu`)

```bash
sudo apt install -y wslu
# Map xdg-open to wslview for tool compatibility
sudo update-alternatives --install /usr/bin/xdg-open xdg-open /usr/bin/wslview 100
```

Test: `xdg-open https://example.com` (should open Chrome on Windows).

------

## 7) Python via Poetry (Modern Dependency Management)

Poetry replaces `venv`, `pip`, and `setup.py` with a single `pyproject.toml`.

### 7.1 Install Poetry

```bash
curl -sSL https://install.python-poetry.org | python3 -
# Add to path (usually ~/.local/bin)
export PATH="$HOME/.local/bin:$PATH"
```

### 7.2 Configure Poetry (Recommended)

Force Poetry to create virtualenvs **inside** your project folder (makes VS Code integration easier):

```bash
poetry config virtualenvs.in-project true
```

### 7.3 Usage

```bash
poetry init          # Create new project
poetry add requests  # Install a dependency
poetry shell         # Activate the environment
```

------

## 8) Node.js + pnpm + Bun

### 8.1 Install `nvm` (Node Version Manager)

```bash
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh | bash
# Restart shell, then:
nvm install --lts
```

### 8.2 Install `pnpm` (Primary)

```bash
curl -fsSL https://get.pnpm.io/install.sh | sh -
```

### 8.3 Install `Bun` (Secondary/Alternative)

```bash
curl -fsSL https://bun.sh/install | bash
```

------

## 9) Rust via `rustup`

### 9.1 Install Rust

```bash
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```

### 9.2 Native Dependencies

```bash
sudo apt install -y pkg-config libssl-dev
```

------

## 10) C/C++ Toolchain

Already installed via `build-essential` and `cmake`. For advanced builds:

```bash
sudo apt install -y autoconf automake libtool
```

------

## 11) Verification Checklist

Run in WSL:

```bash
# Python/Poetry
poetry --version

# Node/pnpm/Bun
node -v
pnpm -v
bun -v

# Rust
rustc --version
cargo --version

# C/C++
gcc --version
g++ --version
cmake --version

# Browser
xdg-open https://example.com
```

------

## 12) Final Note on Docker

As requested, **Docker is omitted**. If you ever move this setup to a native Ubuntu machine or a dedicated VM, you should install **Docker Engine** directly via the official Docker Ubuntu repository rather than using Docker Desktop.