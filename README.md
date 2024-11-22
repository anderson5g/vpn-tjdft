# Tutorial VPN TJDFT - Ubuntu 24.04

<video src="vpn_tjdft.webm" autoplay loop muted playsinline style="max-width: 100%; border-radius: 10px;">
  Seu navegador não suporta vídeos em HTML5. Você pode [baixar o vídeo aqui](vpn_tjdft.webm).
</video>

Este tutorial foi criado para ajudar usuários do Ubuntu 24.04 a configurar e utilizar a VPN do TJDFT de forma simples e eficiente. O guia inclui todos os passos necessários, desde a instalação das dependências até a criação de um atalho personalizado no sistema.

---

## Pré-requisitos e Instalação

Antes de começar, precisamos garantir que o sistema está atualizado e com todas as dependências necessárias instaladas.

### Atualização do Sistema

Estes comandos atualizam a lista de pacotes e instalam as atualizações disponíveis no sistema:

```bash
sudo apt update
sudo apt upgrade -y
```

## Instalação de Dependências

Instalação do OpenSSL e pip3, necessários para o funcionamento da VPN:

```bash
sudo apt install -y openssl python3-pip
pip3 install gp-saml-gui
```

## Configuração da VPN

1. **Preparar Diretório e Arquivos**  
Criamos um diretório específico para armazenar os arquivos da VPN: 

```bash
mkdir -p ~/vpn
```

2. **Configuração do OpenSSL**  
Abra o arquivo de configuração:

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

Salvar o arquivo (Ctrl+O, depois Enter). Para Fechar (Ctrl+X). 

3. **Criar Script de Conexão VPN**  

```bash
$ nano ~/vpn/vpn_tjdft.sh
```

```bash
#!/bin/bash

# Configuração do OpenSSL
export OPENSSL_CONF=~/vpn/openssl.cnf

# Função para conectar à VPN
conectar_vpn() {
    local gateway=$1
    echo "Conectando ao gateway: $gateway"
    gp-saml-gui --gateway --clientos=Linux "$gateway" -S
}

# Gateways principais
gateways=(
    "rpv-dev.tjdft.jus.br"
)

# Conectar a cada gateway
for gateway in "${gateways[@]}"; do
    conectar_vpn "$gateway"
done
```

4. **Tornar Script Executável**  

```bash
$ chmod +x ~/vpn/vpn_tjdft.sh
```

## Criação do Atalho no Ubuntu

5. **Baixar Ícone(Opcional)**

```bash
$ wget -O ~/vpn/tjdft-icon.png https://www.tjdft.jus.br/favicon.ico
```

6. **Criar Arquivo .desktop**

```bash
$ nano ~/.local/share/applications/vpn-tjdft.desktop
```

```bash
[Desktop Entry]
Name=VPN TJDFT
Comment=Conectar VPN do TJDFT
Exec=/home/seu_usuario/vpn/vpn_tjdft.sh
Icon=/home/seu_usuario/vpn/tjdft-icon.png
Terminal=true
Type=Application
Categories=Network;
```
> **Nota:** Lembre-se de mudar "home/seu_usuario/" pelo caminho correto. 

7. **Configurar Permissões**

```bash
$ chmod +x ~/.local/share/applications/vpn-tjdft.desktop
$ chmod +x ~/vpn/tjdft-icon.png
```

8. **Atualizar Cache deÍcones**

```bash
$ update-desktop-database ~/.local/share/applications
```

## Uso

- **Método 1:** Execute o script diretamente

```bash
$ ~/vpn/vpn_tjdft.sh
```

- **Método 2:** Use o atalho criado no menu do Ubuntu 

## Solução de Problemas

```bash
$ which gp-saml-gui
$ pip3 install --upgrade gp-saml-gui
```

> **Notas Importantes:** Mantenha suas credenciais seguras, Verifique os gateways antes de usar, Use em ambiente seguro.

> **Referências:** [Repositório oficial gp-saml-gui](https://github.com/dlenski/gp-saml-gui).