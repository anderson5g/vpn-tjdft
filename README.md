# Tutorial VPN TJDFT - Linux (Ubuntu/Mint/Debian)

Este tutorial foi criado para ajudar usuários de distribuições Linux baseadas em Debian (Ubuntu, Linux Mint, etc.) a configurar e utilizar a VPN do TJDFT de forma simples e eficiente. O guia inclui todos os passos necessários, desde a instalação das dependências até a criação de um atalho personalizado no sistema.

---

## 📋 Pré-requisitos

- Sistema Linux baseado em Debian (Ubuntu 20.04+, Linux Mint 20+, Debian 10+)
- Conexão com a internet
- Privilégios de administrador (sudo)

---

## 🔧 Instalação e Configuração

### 1. Atualização do Sistema

Primeiro, vamos atualizar o sistema para garantir que temos as versões mais recentes dos pacotes:

```bash
sudo apt update && sudo apt upgrade -y
```

### 2. Instalação de Dependências

Instale todas as dependências necessárias:

```bash
sudo apt install -y \
    python3-dev \
    python3-pip \
    python3-gi \
    python3-gi-cairo \
    gir1.2-gtk-3.0 \
    gir1.2-webkit2-4.0 \
    libgirepository1.0-dev \
    libcairo2-dev \
    libffi-dev \
    build-essential \
    pkg-config \
    meson \
    ninja-build \
    wget \
    openconnect
```

### 3. Configuração do Ambiente Python

Crie um diretório específico para a VPN e configure o ambiente Python:

```bash
# Criar diretório
mkdir -p ~/vpn
cd ~/vpn

# Criar ambiente virtual com acesso aos pacotes do sistema
python3 -m venv --system-site-packages vpn-env

# Ativar ambiente virtual
source vpn-env/bin/activate

# Instalar o gp-saml-gui
pip install https://github.com/dlenski/gp-saml-gui/archive/master.zip
```

**✅ Verificação:** Teste se a instalação funcionou:
```bash
gp-saml-gui --help
```

### 4. Configuração do OpenSSL

Crie o arquivo de configuração do OpenSSL:

```bash
nano ~/vpn/openssl.cnf
```

Adicione o seguinte conteúdo:

```ini
openssl_conf = openssl_init

[openssl_init]
ssl_conf = ssl_sect

[ssl_sect]
system_default = system_default_sect

[system_default_sect]
Options = UnsafeLegacyRenegotiation
```

**💡 Dica:** Para salvar no nano: `Ctrl+O` → `Enter` → `Ctrl+X`

### 5. Criar Script de Conexão VPN

Crie o script principal da VPN:

```bash
nano ~/vpn/vpn_tjdft.sh
```

Adicione o seguinte conteúdo:

```bash
#!/bin/bash

# Cores para melhor visualização
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
BLUE='\033[0;34m'
NC='\033[0m' # No Color

# Função para exibir mensagens coloridas
echo_info() {
    echo -e "${BLUE}[INFO]${NC} $1"
}

echo_success() {
    echo -e "${GREEN}[SUCESSO]${NC} $1"
}

echo_warning() {
    echo -e "${YELLOW}[AVISO]${NC} $1"
}

echo_error() {
    echo -e "${RED}[ERRO]${NC} $1"
}

# Verificar se o ambiente virtual existe
if [ ! -d "$HOME/vpn/vpn-env" ]; then
    echo_error "Ambiente virtual não encontrado. Execute a instalação primeiro."
    exit 1
fi

# Ativar o ambiente virtual
echo_info "Ativando ambiente virtual..."
source ~/vpn/vpn-env/bin/activate

# Configuração do OpenSSL
export OPENSSL_CONF=~/vpn/openssl.cnf
echo_info "Configuração OpenSSL carregada"

# Verificar se gp-saml-gui está disponível
if ! command -v gp-saml-gui &> /dev/null; then
    echo_error "gp-saml-gui não encontrado. Verifique a instalação."
    exit 1
fi

# Função para conectar à VPN
conectar_vpn() {
    local gateway=$1
    echo_info "Conectando ao gateway: $gateway"
    echo_warning "Uma janela de login será aberta. Insira suas credenciais do TJDFT."
    
    # Tentar conectar
    if gp-saml-gui --gateway --clientos=Linux "$gateway" -S; then
        echo_success "Conexão estabelecida com sucesso!"
        return 0
    else
        echo_error "Falha na conexão com $gateway"
        return 1
    fi
}

# Função para verificar conexão
verificar_conexao() {
    echo_info "Verificando conectividade..."
    if ping -c 3 8.8.8.8 > /dev/null 2>&1; then
        echo_success "Conexão com a internet OK"
    else
        echo_warning "Problemas de conectividade detectados"
    fi
}

# Banner de início
echo -e "${BLUE}"
echo "╔══════════════════════════════════════╗"
echo "║           VPN TJDFT - Linux          ║"
echo "║                                      ║"
echo "║  Pressione Ctrl+C para desconectar   ║"
echo "╚══════════════════════════════════════╝"
echo -e "${NC}"

# Verificar conexão inicial
verificar_conexao

# Gateways (adicione mais se necessário)
gateways=(
    "rpv-dev.tjdft.jus.br"
)

# Tentar conectar a cada gateway
for gateway in "${gateways[@]}"; do
    if conectar_vpn "$gateway"; then
        break
    fi
    echo_warning "Tentando próximo gateway..."
done

# Manter o terminal aberto
echo_info "VPN ativa. Mantenha esta janela aberta."
echo_warning "Para desconectar, pressione Ctrl+C"

# Função de limpeza ao sair
cleanup() {
    echo_info "Desconectando VPN..."
    deactivate 2>/dev/null || true
    echo_success "Desconectado com sucesso!"
    exit 0
}

# Capturar sinal de interrupção
trap cleanup SIGINT SIGTERM

# Manter ativo
exec $SHELL
```

### 6. Tornar Script Executável

```bash
chmod +x ~/vpn/vpn_tjdft.sh
```

---

## 🖥️ Criação do Atalho no Desktop

### 7. Baixar Ícone (Opcional)

```bash
wget -O ~/vpn/tjdft-icon.png "https://www.tjdft.jus.br/favicon.ico" || \
curl -o ~/vpn/tjdft-icon.png "https://www.tjdft.jus.br/favicon.ico"
```

### 8. Criar Arquivo .desktop

```bash
nano ~/.local/share/applications/vpn-tjdft.desktop
```

**⚠️ IMPORTANTE:** Substitua `SEU_USUARIO` pelo seu nome de usuário real:

```desktop
[Desktop Entry]
Name=VPN TJDFT
Comment=Conectar à VPN do TJDFT
Exec=gnome-terminal -- /home/SEU_USUARIO/vpn/vpn_tjdft.sh
Icon=/home/SEU_USUARIO/vpn/tjdft-icon.png
Terminal=true
Type=Application
Categories=Network;Internet;
StartupNotify=true
```

**💡 Para descobrir seu usuário:** execute `whoami` no terminal

### 9. Configurar Permissões

```bash
chmod +x ~/.local/share/applications/vpn-tjdft.desktop
chmod +x ~/vpn/tjdft-icon.png 2>/dev/null || true
```

### 10. Atualizar Cache de Aplicações

```bash
update-desktop-database ~/.local/share/applications 2>/dev/null || true
```

---

## 🚀 Como Usar

### Método 1: Terminal
```bash
~/vpn/vpn_tjdft.sh
```

### Método 2: Menu do Sistema
1. Pressione `Super` (tecla Windows)
2. Digite "VPN TJDFT"
3. Clique no ícone que aparecer

### Método 3: Criar Alias (Opcional)
Adicione ao seu `~/.bashrc` ou `~/.zshrc`:
```bash
alias vpn-tjdft='~/vpn/vpn_tjdft.sh'
```

Depois execute: `source ~/.bashrc` (ou `~/.zshrc`)

---

## 🔧 Solução de Problemas

### Problema: "openconnect: comando não encontrado"
**Solução:**
```bash
sudo apt install openconnect
```

### Problema: "gp-saml-gui não encontrado"
**Solução:**
```bash
cd ~/vpn
source vpn-env/bin/activate
pip install --upgrade https://github.com/dlenski/gp-saml-gui/archive/master.zip
```

### Problema: Erro de dependências no Ubuntu/Mint mais antigo
**Solução:** Adicione repositório do Ubuntu Focal:
```bash
sudo add-apt-repository "deb http://archive.ubuntu.com/ubuntu focal main universe"
sudo apt update
```

### Problema: Janela de login não abre
**Solução:** Instale dependências adicionais:
```bash
sudo apt install gir1.2-webkit2-4.0 python3-gi-cairo
```

### Problema: SSL/TLS errors
**Solução:** Verifique se o arquivo `openssl.cnf` foi criado corretamente

---

## 🔐 Segurança e Boas Práticas

- ✅ **Mantenha suas credenciais seguras** - nunca as compartilhe
- ✅ **Verifique os gateways** antes de conectar
- ✅ **Use apenas em ambiente seguro** e confiável
- ✅ **Desconecte sempre** quando terminar de usar
- ✅ **Mantenha o sistema atualizado** regularmente

---

## 📝 Notas Importantes

- **Distribuições testadas:** Ubuntu 20.04+, Linux Mint 20+, Debian 10+
- **Requisitos mínimos:** Python 3.8+, GTK 3.0+
- **Conexão:** Mantenha a janela do terminal aberta durante o uso
- **Firewall:** Pode ser necessário configurar exceções

---

## 🆘 Suporte

Se encontrar problemas:

1. **Verifique os logs** no terminal
2. **Teste a conectividade** com `ping 8.8.8.8`
3. **Reinstale** seguindo os passos novamente
4. **Consulte** a documentação oficial do gp-saml-gui

---

## 📚 Referências

- [Repositório oficial gp-saml-gui](https://github.com/dlenski/gp-saml-gui)
- [Documentação Python venv](https://docs.python.org/3/library/venv.html)
- [Ubuntu Desktop Files](https://help.ubuntu.com/community/UnityLaunchersAndDesktopFiles)

---

**✨ Tutorial testado e otimizado para máxima compatibilidade com distribuições Linux baseadas em Debian.**