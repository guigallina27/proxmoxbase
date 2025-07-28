### Guia Genérico e Completo para Configurar o Hostname no Proxmox

Este guia utiliza os seguintes valores como exemplo:
*   **IP do Servidor:** `1.2.3.4`
*   **Nome Curto (hostname):** `pvenode`
*   **Nome de Domínio Completo (FQDN):** `pvenode.seudominio.com`

Substitua esses valores pelos que correspondem ao seu ambiente.

---

#### **1. Definir o Nome Curto do Host (Hostname)**

Este comando configura o nome principal do nó no sistema operacional. Use apenas o nome curto.

```bash
hostnamectl set-hostname <nome-curto>
```
*Exemplo:*
```bash
hostnamectl set-hostname pvenode
```

---

#### **2. Estruturar Corretamente o Arquivo `/etc/hosts`**

Este arquivo é crucial para que o servidor consiga resolver o próprio nome para o endereço de IP correto.

1.  Abra o arquivo para edição:
    ```bash
    nano /etc/hosts
    ```

2.  Assegure-se de que o conteúdo siga esta estrutura. A ordem das entradas é importante.

    ```ini
    # 1. Mapeamento do loopback (essencial para o sistema)
    127.0.0.1       localhost

    # 2. Mapeamento principal do servidor Proxmox
    <ip-do-servidor>   <nome-completo-fqdn> <nome-curto>

    # 3. Linhas recomendadas para IPv6
    ::1             ip6-localhost ip6-loopback
    fe00::0         ip6-localnet
    ff00::0         ip6-mcastprefix
    ff02::1         ip6-allnodes
    ff02::2         ip6-allrouters
    ```

    **Exemplo prático preenchido:**
    ```ini
    127.0.0.1       localhost
    1.2.3.4         pvenode.seudominio.com pvenode

    # The following lines are desirable for IPv6 capable hosts
    ::1             ip6-localhost ip6-loopback
    fe00::0         ip6-localnet
    ff00::0         ip6-mcastprefix
    ff02::1         ip6-allnodes
    ff02::2         ip6-allrouters
    ```

3.  Salve o arquivo e saia (`Ctrl + X`, `Y`, `Enter`).

---

#### **3. Sincronizar a Configuração do Proxmox**

O Proxmox cria um diretório de configuração com o nome do nó. Se você alterou o hostname nos passos anteriores, este diretório também precisa ser renomeado para corresponder ao novo nome.

1.  Navegue até a pasta de nós para ver o nome do diretório atual:
    ```bash
    cd /etc/pve/nodes/
    ls
    ```
2.  Renomeie o diretório:
    ```bash
    mv <nome-antigo> <nome-curto-novo>
    ```
    *Exemplo:*
    ```bash
    mv pve-antigo pvenode
    ```

> **Adendo:** Este passo é necessário **apenas se você estiver alterando um hostname existente**. Se o nome do diretório listado pelo comando `ls` já for igual ao `<nome-curto-novo>` que você definiu, **não é preciso fazer nada** e você pode pular esta etapa.

---

#### **4. Reiniciar os Serviços do Proxmox (Aplicação Rápida)**

Para que as alterações tenham efeito imediato na interface web e nos serviços do Proxmox, reinicie-os. Isso permite uma verificação rápida sem a necessidade de um reboot completo.

Execute os seguintes comandos, um de cada vez:
```bash
systemctl restart pveproxy
systemctl restart pvedaemon
```
Após executar os comandos, atualize a página da interface web do Proxmox no seu navegador para ver se as alterações foram aplicadas corretamente.

---

#### **5. Reinicialização Completa (Aplicação Final)**

Embora a reinicialização dos serviços geralmente seja suficiente, um reboot completo garante que absolutamente todos os componentes do sistema operacional e do Proxmox reconheçam o novo hostname. Esta é a forma mais segura de finalizar o processo.

```bash
reboot
```
---
