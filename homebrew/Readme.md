## Install Homebrew on Windows 10 WSL2

Follow these steps to install the Homebrew as a package manager on WSL2

```
sudo apt update
```

Install required tools:

```
sudo apt-get install build-essential curl file git
```

Download and install Homebrew

```
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install.sh)"
```

Add Homebrew to the PATH

```
test -d ~/.linuxbrew && eval $(~/.linuxbrew/bin/brew shellenv)
```

```
test -d /home/linuxbrew/.linuxbrew && eval $(/home/linuxbrew/.linuxbrew/bin/brew shellenv)
```

```
test -r ~/.bash_profile && echo "eval \$($(brew --prefix)/bin/brew shellenv)" >>~/.bash_profile
```

```
echo "eval \$($(brew --prefix)/bin/brew shellenv)" >>~/.profile
```

### HomeBrew Commands

Install package

```
brew install {package-name}
```

Update package

```
brew update
```
