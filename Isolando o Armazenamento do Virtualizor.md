### **Isolando o Armazenamento do Virtualizor**

Este guia detalha como criar um espaço de armazenamento dedicado para o diretório `/var/virtualizor` em um host Proxmox. O objetivo é isolar os arquivos do painel Virtualizor (como templates e ISOs) para prevenir o esgotamento do disco do sistema operacional e permitir um gerenciamento de espaço mais eficaz.

O procedimento varia dependendo da tecnologia de armazenamento utilizada pelo seu sistema: **ZFS** ou **LVM**. Siga a seção correspondente à sua configuração.

#### **Pré-requisitos Comuns**

*   Acesso de administrador (root) ao terminal do servidor Proxmox (via SSH ou console).
*   Espaço em disco livre suficiente no seu grupo de volumes (LVM) ou pool (ZFS).

---

### **Cenário 1: Usando ZFS**

Este método é ideal para sistemas Proxmox instalados sobre ZFS. Ele utiliza um *dataset* para criar um volume isolado de forma eficiente.

#### **Passo 1.1: Identificar o Pool ZFS**

Primeiro, identifique o nome do seu pool ZFS principal. O nome padrão geralmente é `rpool`.

```bash
zpool list
```
Anote o nome que aparece na coluna `NAME`.

#### **Passo 1.2: Criar um Novo Dataset ZFS**

Crie um *dataset* (um "diretório inteligente") para os dados do Virtualizor. Substitua `rpool` pelo nome do seu pool, se necessário.

```bash
zfs create rpool/virtualizor_data
```

#### **Passo 1.3: Definir o Ponto de Montagem**

Configure o ZFS para montar o novo dataset diretamente no diretório `/var/virtualizor`.

```bash
zfs set mountpoint=/var/virtualizor rpool/virtualizor_data
```

#### **Passo 1.4 (Recomendado): Definir uma Cota de Espaço**

Para evitar o consumo excessivo de disco, defina um limite de armazenamento (quota). Ajuste o valor `500G` conforme sua necessidade.

```bash
zfs set quota=500G rpool/virtualizor_data
```

#### **Passo 1.5: Verificar a Configuração ZFS**

Confirme se o dataset foi montado corretamente e se a cota foi aplicada.

```bash
df -h /var/virtualizor
```
A saída deve mostrar `rpool/virtualizor_data` montado em `/var/virtualizor` com o tamanho da cota definida.

---

### **Cenário 2: Usando LVM (Logical Volume Manager)**

Este método é para sistemas Proxmox instalados com `ext4` ou `xfs` sobre LVM. Ele envolve a criação de um novo Volume Lógico e a montagem no diretório desejado.

#### **Passo 2.1: Identificar o Grupo de Volumes (VG)**

Primeiro, identifique o nome do seu Grupo de Volumes (Volume Group) que contém espaço livre. O nome padrão do Proxmox é `pve`.

```bash
vgs
```
Anote o nome da coluna `VG` e verifique o espaço livre na coluna `VFree`.

#### **Passo 2.2: Criar um Novo Volume Lógico (LV)**

Crie um novo Volume Lógico (Logical Volume) com o tamanho desejado.
*   Substitua `pve` pelo nome do seu VG, se for diferente.
*   Ajuste o tamanho em `-L 500G` conforme sua necessidade.
*   `virtualizor_lv` será o nome do novo volume.

```bash
lvcreate -L 500G -n virtualizor_lv pve
```

#### **Passo 2.3: Formatar o Novo Volume**

Para que o sistema operacional possa usar o novo volume, ele precisa ser formatado com um sistema de arquivos. O `ext4` é uma escolha segura e comum.

```bash
mkfs.ext4 /dev/pve/virtualizor_lv
```
*(Se seu VG não for `pve`, o caminho será `/dev/SEU_VG/virtualizor_lv`)*

#### **Passo 2.4: Montar o Volume Permanentemente**

Para garantir que o volume seja montado automaticamente toda vez que o servidor reiniciar, adicione uma entrada no arquivo `/etc/fstab`.

1.  **Primeiro, faça um backup do `fstab` (medida de segurança):**
    ```bash
    cp /etc/fstab /etc/fstab.bak
    ```

2.  **Adicione a linha de montagem ao final do arquivo `/etc/fstab`:**
    Use o comando `echo` para adicionar a linha de forma segura.

    ```bash
    echo '/dev/pve/virtualizor_lv /var/virtualizor ext4 defaults 0 0' >> /etc/fstab
    ```

3.  **Monte o volume:**
    O comando `mount -a` lê o arquivo `fstab` e monta todas as entradas que ainda não foram montadas.

    ```bash
    mount -a
    ```

#### **Passo 2.5: Verificar a Configuração LVM**

Confirme se o novo volume lógico está corretamente montado no local esperado.

```bash
df -h /var/virtualizor
```
A saída deve mostrar o dispositivo `/dev/mapper/pve-virtualizor_lv` montado em `/var/virtualizor` com o tamanho que você definiu.

---

### **Observação Final Importante (Para Ambos os Cenários)**

Este armazenamento é de **uso exclusivo do Virtualizor**. Ele não deve ser adicionado como um "Storage" na interface web do Proxmox (em `Datacenter -> Storage`). O painel Virtualizor gerenciará o conteúdo deste diretório de forma independente, e adicioná-lo ao Proxmox pode causar conflitos de gerenciamento.
