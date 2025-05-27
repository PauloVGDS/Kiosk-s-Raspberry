# Exibição de Dashboard no Raspberry Pi

Configuração de um **Raspberry Pi 3 B** com **Raspberry Pi OS Lite** para exibir um dashboard do **Google Looker Studio** em modo kiosk, em tela cheia, com o cursor do mouse oculto. A configuração utiliza o **LightDM** para login automático, o **Openbox** como gerenciador de janelas leve, o **Chromium** para renderizar o dashboard e o **Unclutter** para esconder o mouse após um período de tempo.

## Pré-requisitos
- **Hardware**:
  - Raspberry Pi 3 B
  - Cartão MicroSD (mínimo 8 GB, recomendado Classe 10)
  - Adaptador de energia (5V, 2,5A)
  - Monitor ou TV compatível com HDMI
  - Opcional: Teclado, mouse e monitor para configuração inicial (ou acesso via SSH)
- **Software**:
  - Raspberry Pi Imager (baixe em [raspberrypi.com/software](https://www.raspberrypi.com/software/))
  - Raspberry Pi OS Lite (32 bits)
  - Conexão com a internet (Ethernet ou Wi-Fi)
  - Dashboard com acesso público ou compartilhado

## Passos para Configuração

### Passo 1: Instalar o Raspberry Pi OS Lite
1. **Baixe e instale o Raspberry Pi Imager**:
   - Acesse [raspberrypi.com/software](https://www.raspberrypi.com/software/) e instale o Raspberry Pi Imager no seu computador (Windows, macOS ou Linux).

2. **Grave o Raspberry Pi OS Lite no cartão MicroSD**:
   - Insira o cartão MicroSD no computador.
   - Abra o Raspberry Pi Imager, selecione **Raspberry Pi 3** em Device, em Sistema Operacional **Raspberry Pi OS Lite (32-bit)**, escolha o cartão MicroSD e clique em **Next**.

3. **Habilite SSH e Wi-Fi**:
   - Depois selecione configurações personalizadas, configure o **usuário e senha** juntamente com a **Rede Wifi** de preferência.
   - Ative o SSH para acesso remoto(caso não tenha teclado).
   - Salve as configurações e aguarde a instalação.

4. **Inicie o Raspberry Pi**:
   - Insira o cartão MicroSD no Raspberry Pi 3 B, conecte ao monitor via HDMI e ligue a alimentação.
   - Faça login com as credenciais padrão(caso não tenha alterado na instalação):
     - Usuário: `pi`
     - Senha: `raspberry`

5. **Atualize o sistema**:
   ```bash
   sudo apt update
   sudo apt full-upgrade -y
   sudo reboot
   ```

6. **Configure ajustes básicos**:
   - Execute:
     ```bash
     sudo raspi-config
     ```
   - Defina o fuso horário: **Localisation Options > Change Timezone > America > Sao_Paulo**.
   - Defina o Boot em modo Desktop: **1 System Options > S5 Boot > B2 Desktop GUI**
   - Defina o Login Automático no modo Desktop: **1 System Options > S6 Auto Login > Would you like to automatically log in to the Desktop? > Yes > Ok**
   - Salve as alterações e reinicie:
     ```bash
     sudo reboot
     ```

### Passo 2: Instalar o Ambiente Gráfico e o LightDM
Como o Raspberry Pi OS Lite não inclui interface gráfica, instale um ambiente gráfico mínimo e o LightDM para gerenciar sessões.

1. **Instale o Xorg, Openbox e LightDM**:
   ```bash
   sudo apt install xorg openbox lightdm -y
   ```
   
2. **Instale o Chromium e Unclutter**:
   ```bash
   sudo apt install chromium-browser unclutter -y
   ```

### Passo 3: Configurar o LightDM para Login Automático
Para que o dashboard inicie automaticamente, configure o LightDM para login automático com o usuário `pi`.

1. **Edite a configuração do LightDM**:
   ```bash
   sudo nano /etc/lightdm/lightdm.conf
   ```
   Adicione ou modifique a seção `[Seat:*]`:
   ```bash
   [Seat:*]
   autologin-user=pi
   autologin-user-timeout=0
   user-session=openbox
   xserver-command=X -s 0 -dpms
   ```
   - `autologin-user=pi`: Faz login automático como usuário `pi`.
   - `autologin-user-timeout=0`: Remove atraso no login automático.
   - `user-session=openbox`: Usa o Openbox como gerenciador de janelas.
   - `xserver-command=X -s 0 -dpms`: Desativa o protetor de tela e o gerenciamento de energia.

2. **Salve e saia**:
   - Pressione `Ctrl+O`, `Enter` e `Ctrl+X`.

3. **Teste o LightDM**:
   ```bash
   sudo systemctl restart lightdm
   ```
   Verifique o status:
   ```bash
   systemctl status lightdm
   ```

### Passo 4: Configurar o Chromium para Modo Kiosk
Configure o Chromium para abrir o dashboard em modo kiosk, em tela cheia.

1. **Crie um script de inicialização**:
   ```bash
   mkdir -p ~/.config/autostart
   nano ~/.config/autostart/dashboard.sh
   ```
   Adicione:
   ```bash
   #!/bin/bash
   chromium-browser --kiosk --start-maximized --noerrdialogs --disable-infobars --enable-features=OverlayScrollBar https://url-para-qualquer-site
   ```
   Substitua `https://url-para-qualquer-site` pela URL do seu dashboard.

2. **Torne o script executável**:
   ```bash
   chmod +x ~/.config/autostart/dashboard.sh
   ```

3. **Configure o Openbox para executar o script e Unclutter para esconder o mouse**:
   ```bash
   mkdir -p ~/.config/openbox
   nano ~/.config/openbox/autostart
   ```
   Adicione:
   ```bash
   unclutter -idle 0.5 &
   ~/.config/autostart/dashboard.sh &
   ```
   Salve e saia.



## Solução de Problemas

### Falha do LightDM
**Problema**: O LightDM falhou ao iniciar com `Active: failed (Result: exit-code)` devido a configurações de sessão incorretas (`LXDE-pi-x` e `pi-greeter`).

**Solução**:
- Modificar `/etc/lightdm/lightdm.conf` para usar `user-session=openbox` e remover `greeter-session=pi-greeter`.
- Reinstalar dependências, se necessário:
  ```bash
  sudo apt install --reinstall lightdm xorg openbox chromium-browser -y
  ```
- Verifique os logs:
  ```bash
  journalctl -u lightdm.service -b
  cat /var/log/lightdm/lightdm.log
  ```
  
## Arquivos de Configuração Finais

1. **`/etc/lightdm/lightdm.conf`**:
   ```bash
   [Seat:*]
   autologin-user=pi
   autologin-user-timeout=0
   user-session=openbox
   xserver-command=X -s 0 -dpms
   ```

2. **`~/.config/openbox/autostart`**:
   ```bash
   unclutter -idle 0.5 &
   ~/.config/autostart/dashboard.sh &
   ```

3. **`~/.config/autostart/dashboard.sh`**:
   ```bash
   #!/bin/bash
   chromium-browser --kiosk --start-maximized --noerrdialogs --disable-infobars --enable-features=OverlayScrollBar https://url-para-qualquer-site
   ```


## Uso
1. Grave o cartão MicroSD com o Raspberry Pi OS Lite usando o Raspberry Pi Imager.
2. Siga os passos para instalar dependências, configurar o LightDM e o Chromium.
3. Atualize `dashboard.sh` com a URL do seu dashboard.
4. Reinicie o Raspberry Pi:
   ```bash
   sudo reboot
   ```
5. O sistema irá:
   - Fazer login automático via LightDM.
   - Iniciar o Openbox e o Chromium em modo kiosk.
   - Exibir o dashboard em tela cheia.


**Script `install.sh` opcional**:
```bash
#!/bin/bash
sudo apt update
sudo apt install xorg openbox lightdm chromium-browser -y
mkdir -p ~/.config/openbox
mkdir -p ~/.config/autostart
chmod +x ~/.config/autostart/dashboard.sh
# Preencha os arquivos.
```
