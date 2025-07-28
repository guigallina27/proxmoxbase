# Habilitando o Clipboard (xterm.js) em VMs Linux no Proxmox

**Versão:** 1.2 (Revisada para Markdown)
**Data:** 21 de Julho de 2025

## 1. Objetivo

Este tutorial detalha o processo para habilitar uma console de texto interativa (`xterm.js`) para máquinas virtuais Linux no Proxmox VE. O objetivo é permitir o uso nativo da área de transferência (copiar/colar com `Ctrl+V`) diretamente na interface web, melhorando drasticamente a usabilidade para administração de servidores em modo CLI.

Este método mantém a console de vídeo padrão (`noVNC`) como uma opção de fallback funcional.

## 2. Pré-requisito: Configuração da VM no Proxmox

Este passo inicial é **idêntico** para todas as distribuições Linux e deve ser feito na interface web do Proxmox.

1.  **Selecione a sua Máquina Virtual** no painel esquerdo.
2.  Navegue até a aba **Hardware**.
3.  **Verifique o Display:** Confirme que a `Graphic card` está configurada como `Default (std)`. Não utilize a opção `serial` aqui.
4.  **Adicione a Porta Serial:**
    -   Clique no botão **Add**.
    -   Selecione **Serial Port**.
    -   Mantenha as opções padrão (`Port 0`) e clique em **Add**.

O hardware da VM está pronto para a configuração do sistema operacional.

## 3. Configuração do Sistema Operacional Convidado

A seguir, estão as instruções específicas para os dois principais ramos de distribuições Linux.

---

### 3.1. Para Distribuições baseadas em RHEL (AlmaLinux, Rocky Linux, CentOS, Fedora)

Estes sistemas utilizam o `grub2-mkconfig` para gerar a configuração do bootloader.

#### Passo 1: Editar o arquivo de configuração do GRUB
Abra o arquivo `/etc/default/grub` com um editor de texto de sua preferência (ex: `nano`).
```bash
sudo nano /etc/default/grub
```

#### Passo 2: Modificar e adicionar as linhas necessárias
-   **Para o GRUB usar ambas as consoles (vídeo e serial):**
    ```ini
    # Altere esta linha para incluir "serial"
    GRUB_TERMINAL_OUTPUT="console serial"
    
    # Adicione esta linha, preferencialmente ao final do arquivo
    GRUB_SERIAL_COMMAND="serial --unit=0 --speed=115200"
    ```

-   **Para o Kernel do Linux usar ambas as consoles:**
    Localize a linha que começa com `GRUB_CMDLINE_LINUX`. Adicione `console=tty0 console=ttyS0,115200` ao final da string, dentro das aspas.
    
    *Exemplo de linha modificada:*
    ```ini
    GRUB_CMDLINE_LINUX="... rhgb quiet console=tty0 console=ttyS0,115200"
    ```

#### Passo 3: Salvar e aplicar a configuração
1.  Salve as alterações e saia do editor (em `nano`, é `Ctrl+X`, `Y`, `Enter`).
2.  Execute o comando para gerar o novo arquivo de configuração do GRUB:
    ```bash
    sudo grub2-mkconfig -o /boot/grub2/grub.cfg
    ```

#### Passo 4: Reiniciar a VM
```bash
sudo reboot
```

---

### 3.2. Para Distribuições baseadas em Debian (Debian, Ubuntu, Mint, etc.)

Estes sistemas utilizam o comando `update-grub`, que simplifica o processo.

#### Passo 1: Editar o arquivo de configuração do GRUB
Abra o arquivo `/etc/default/grub` com um editor de texto.
```bash
sudo nano /etc/default/grub
```

#### Passo 2: Modificar e adicionar as linhas necessárias
-   **Para o GRUB usar ambas as consoles:**
    ```ini
    # Altere ou adicione esta linha para incluir "serial"
    GRUB_TERMINAL="console serial"
    
    # Adicione esta linha, preferencialmente ao final do arquivo
    GRUB_SERIAL_COMMAND="serial --unit=0 --speed=115200"
    ```

-   **Para o Kernel do Linux usar ambas as consoles:**
    Localize a linha que começa com `GRUB_CMDLINE_LINUX_DEFAULT`. Adicione `console=tty0 console=ttyS0,115200` ao final da string, dentro das aspas.
    
    *Exemplo de linha modificada:*
    ```ini
    GRUB_CMDLINE_LINUX_DEFAULT="quiet console=tty0 console=ttyS0,115200"
    ```

#### Passo 3: Salvar e aplicar a configuração
1.  Salve as alterações e saia do editor.
2.  Execute o comando de atualização específico para sistemas Debian:
    ```bash
    sudo update-grub
    ```

#### Passo 4: Reiniciar a VM
```bash
sudo reboot
```

---

## 4. Resultado e Verificação

Após a reinicialização da VM, ao clicar em **Console** na interface do Proxmox, a janela principal será um terminal **xterm.js** interativo.

-   **Clipboard:** Você poderá copiar texto de qualquer lugar e colá-lo diretamente na console com `Ctrl+V` ou um clique do botão direito/meio do mouse.
-   **Console de Vídeo:** A console `noVNC` continua disponível. Para acessá-la, clique na seta para baixo ao lado de `> Console` no painel esquerdo e selecione `noVNC`.
