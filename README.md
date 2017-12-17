## Workshop Azure

### 1. acessar a vm inicial

1.1. acessar a VM Linux conforme info acima.
	
1.2. instalar o Azure CLI 2.0
	```bash
	para sistemas 64bits:
	
	1.2.1.

echo "deb [arch=amd64] https://packages.microsoft.com/repos/azure-cli/ wheezy main" | \sudo tee /etc/apt/sources.list.d/azure-cli.list
	 
	1.2.2.

sudo apt-key adv --keyserver packages.microsoft.com --recv-keys 52E16F86FEE04B979B07E28DB02C46DF417A0893

sudo apt-get install apt-transport-https

sudo apt-get update && sudo apt-get install azure-cli

	1.2.3. testar com o comando:

az

az –version
```
Referência:
https://docs.microsoft.com/pt-br/cli/azure/install-azure-cli?view=azure-cli-latest 

	1.3. conectar na subscricao via AZ CLI

```bash
az login

 *** NÃO USAR CTRL+C e CTRL+V, SENÃO O COMANDO SERÁ ANULADO ***
- usar seu usuário rgparticipante”x”@outlook.com
Senha: Go010101you!

 *** Ao voltar ao terminal, aguarde o comando completar! ***

az account show (para verificar se está selecionado a subscription correta)

OPCIONAL:
az account list --output table (caso queira listar todas subscriptions)

CASO TENHA MAIS DE UMA SUBSCRIÇÃO E A QUE PRETENDE USAR NÃO É A DEFAULT:
az account set --subscription "<nome_subscription> ou <ID>" (para selecionar subscription com base no nome ou ID)
	
	```
	### OBSERVAÇÃO: caso necessite rodar no Windows:
	INSTALAR O BASH on ubuntu FOR WINDOWS.

Referência:	
https://blogs.msdn.microsoft.com/commandline/2016/04/06/bash-on-ubuntu-on-windows-download-now-3/ 
	

### 2. criar via CLI o RG : rgparticipante"x"

az group create --name rgparticipante"x" --location eastus

	
3. criar via CLI a VNET, SUBNET (manag, front, back)

3.1	criar a vnet com uma subnet inicial (chamada manag)

```bash
az network vnet create \
--name uolhvnet"x"   \
--resource-group rgparticipante"x" \
--location eastus \
--address-prefix 192.168.0.0/16 \
--subnet-name manag \
--subnet-prefix 192.168.1.0/24

3.2 adicionar outras subnets via CLI

Front

az network vnet subnet create \
--address-prefix 192.168.2.0/24 \
--name front \
--resource-group rgparticipante"x" \
--vnet-name uolhvnet"x"

Back

az network vnet subnet create \
--address-prefix 192.168.3.0/24 \
--name back \
--resource-group rgparticipante"x" \
--vnet-name uolhvnet"x"

3.3 verificando sua nova vnet

az network vnet show \
-g rgparticipante"x" \
-n uolhvnet"x" \
--query '{Name:name,Where:location,Group:resourceGroup,Status:provisioningState,SubnetCount:subnets | length(@)}' \
-o table

3.4. verificando detalhes da subnet

az network vnet subnet list \
-g rgparticipante"x" \
--vnet-name uolhvnet"x" \
--query '[].{Name:name,CIDR:addressPrefix,Status:provisioningState}' \
-o table
```

### 4. criar / configurar via CLI a NSG (Network Security Group)

```bash
4.1. criar a NSG

para a subnet manag

az network nsg create \
--resource-group rgparticipante"x" \
--name NSG-manag \
--location eastus

para a subnet front

az network nsg create \
--resource-group rgparticipante"x" \
--name NSG-front \
--location eastus

para a subnet back

az network nsg create \
--resource-group rgparticipante"x" \
--name NSG-back \
--location eastus

4.2 conferindo as regras default do NSG (para a NSG Front)

az network nsg show \
-g rgparticipante"x" \
-n nsg-front \
--query 'defaultSecurityRules[].{Access:access,Desc:description,DestPortRange:destinationPortRange,Direction:direction,Priority:priority}' \
-o table

4.3 adicionando as regras necessárias para cada NSG

RDP para subnet Front

az network nsg rule create \
--resource-group rgparticipante"x" \
--nsg-name NSG-Front \
--name rdp-rule \
--access Allow \
--protocol Tcp \
--direction Inbound \
--priority 100 \
--source-address-prefix Internet \
--source-port-range "*" \
--destination-address-prefix "*" \
--destination-port-range 3389

SSH para subnet Front

az network nsg rule create \
--resource-group rgparticipante"x" \
--nsg-name NSG-Front \
--name ssh-rule \
--access Allow \
--protocol Tcp \
--direction Inbound \
--priority 101 \
--source-address-prefix Internet \
--source-port-range "*" \
--destination-address-prefix "*" \
--destination-port-range 22

SSH para subnet Backend

az network nsg rule create \
--resource-group rgparticipante"x" \
--nsg-name NSG-Back \
--name sshrule \
--access Allow \
--protocol Tcp \
--direction Inbound \
--priority 101 \
--source-address-prefix Internet \
--source-port-range "*" \
--destination-address-prefix "*" \
--destination-port-range 22

RDP para a subnet Manag

az network nsg rule create \
--resource-group rgparticipante"x" \
--nsg-name NSG-manag \
--name rdp-rule \
--access Allow \
--protocol Tcp \
--direction Inbound \
--priority 100 \
--source-address-prefix Internet \
--source-port-range "*" \
--destination-address-prefix "*" \
--destination-port-range 3389

SSH para subnet manag

az network nsg rule create \
--resource-group rgparticipante"x" \
--nsg-name NSG-manag \
--name sshrule \
--access Allow \
--protocol Tcp \
--direction Inbound \
--priority 101 \
--source-address-prefix Internet \
--source-port-range "*" \
--destination-address-prefix "*" \
--destination-port-range 22


HTTP para a subnet Front

az network nsg rule create \
--resource-group rgparticipante"x" \
--nsg-name NSG-Front \
--name web-rule \
--access Allow \
--protocol Tcp \
--direction Inbound \
--priority 200 \
--source-address-prefix Internet \
--source-port-range "*" \
--destination-address-prefix "*" \
--destination-port-range 80

4.4 BIND THE NSG TO SUBNET

NSG DA SUBNET MANAG

az network vnet subnet update \
--vnet-name uolhvnet"x" \
--name manag \
--resource-group rgparticipante"x" \
--network-security-group NSG-manag

NSG DA SUBNET FRONT

az network vnet subnet update \
--vnet-name uolhvnet"x" \
--name Front \
--resource-group rgparticipante"x" \
--network-security-group NSG-Front

NSG DA SUBNET BACK

az network vnet subnet update \
--vnet-name uolhvnet"x" \
--name Back \
--resource-group rgparticipante"x" \
--network-security-group NSG-Back

```

### 6. criar via CLI a VM Linux (para CPanel) na subnet Front

*** a VM Windows será criada no final do Workshop. ***

```bash
6.1 listando as imagens disponiveis (exemplos)

az vm image list --offer Ubuntu --all --output table

az vm image list --offer CentOs --all --output table

az vm image list --offer Windows --all --output table

*** Utilizaremos uma imagem Ubuntu durante o workshop. ***


6.2 listando os tamanhos de VMs disponiveis

az vm list-sizes --location eastus --output table

ex.: Standard_A1

*** Cada localidade poderá oferecer diferentes tipos de VMs. ***
```

6.3 criando a VM NA SUBNET FRONT

Relembrando: 
Até aqui criamos o rg, a vnet, a subnet, o nsg. 
Então antes de criar a vm precisamos criar a nic e o ip publico da vm.

```bash
az network public-ip create --resource-group rgparticipante"x"   \
--name pip-frontpart”x” --allocation-method dynamic --idle-timeout 4

az network nic create \
-n nic-frontpart”x” \
-g rgparticipante"x" \
--subnet front \
--public-ip-address pip-vmpart”x” \
--vnet-name  uolhvnet"x"

az vm create \
--resource-group rgparticipante"x" \
--name vmfront”x” \
--image UbuntuLTS \
--size Standard_DS1_v2 \
--nics nic-vmpart”x” \
--data-disk-sizes-gb 128 \
--admin-username participante”x” \
--admin-password Go010101you! \
--authentication-type password
```

Após criação da VM Linux, acessá-la através de seu ip público.  
Você poderá anotar o ip público ao final da execução do comando ou rodar o comando CLI:

az network public-ip list --resource-group rgparticipante"x" --query [].ipAddress

6.4 adicionando o apache via stack lamp

Conectar na nova VM criada:

ssh participante”x”@<ip_público>

Executar o comando:

sudo apt update && sudo apt install lamp-server^

Faça o teste acessando o ip publico da VM em um browser.
(a porta 80 para acesso via internet foi habilitada em step anterior durante criação das regras NSG da subnet)

### 7. criando uma VM Linux na Subnet Back para o MySQL

Relembrando mais uma vez: 
Até aqui criamos o rg, a vnet, a subnet, o nsg. 
Então antes de criar a vm precisamos criar a nic e o ip publico a usar.

```bash
az network public-ip create --resource-group rgparticipante"x"   \
--name pip-backpart”x” --allocation-method dynamic --idle-timeout 4

az network nic create \
-n nic-backpart”x” \
-g rgparticipante"x" \
--subnet back \
--public-ip-address pip-backpart”x” \
--vnet-name  uolhvnet"x"

az vm create \
--resource-group rgparticipante"x" \
--name vmback”x” \
--image UbuntuLTS \
--size Standard_DS1_v2 \
--nics nic-backpart”x” \
--data-disk-sizes-gb 128 \
--admin-username participante”x” \
--admin-password Go010101you! \
--authentication-type password
```

### Observação: o disco apenas é adicionado a vm, mas nao é entregue formatado dentro do linux.

  	7.1 adicionando o mysql (via stack LAMP)
  
sudo apt update && sudo apt install lamp-server^
  
7.2 testar a conexão entre as vms das Subnets Front e Back (via ip privado)

*** Não está coberto nesse workshop a configuração do MySQL. ***


8. criar uma Custom Image usando CLI

8.2 criando custom image no linux

8.2.1 criando uma nova VM para criarmos posteriormente sua imagem “generalizada”

(há diferença entre Imagem Generalizada e Backup da Imagem)

```bash
az network public-ip create --resource-group rgparticipante"x"   \
--name pip-destroy”x” --allocation-method dynamic --idle-timeout 4

az network nic create \
-n nic-destroy”x” \
-g rgparticipante"x" \
--subnet front \
--public-ip-address pip-destroy”x” \
--vnet-name  uolhvnet"x"

az vm create \
--resource-group rgparticipante"x" \
--name vmdestroy”x” \
--image UbuntuLTS \
--size Standard_DS1_v2 \
--nics nic-destroy”x” \
--admin-username participante”x” \
--admin-password Go010101you! \
--authentication-type password

8.2.2 preparando a VM para salvar a imagem

(aplicar os comandos abaixo na VM que terá a imagem salva)

sudo waagent -deprovision+user –force

(a partir daqui, usar os comandos na sua workstation de trabalho)

az vm deallocate --resource-group rgparticipante"x" --name vmdestroy”x”
az vm generalize --resource-group rgparticipante"x" --name vmdestroy”x”  
az image create \
--resource-group rgparticipante"x" \
--name imageteste”x” \
--source vmdestroy”x”

8.2.3 listando as imagens disponiveis

az image list \
--resource-group rgparticipante"x"
  
 8.2.4 criar uma vm a partir de uma imagem salva
  
(criaremos primeiro o ip público e a NIC como nas outras VMs)

 az network public-ip create --resource-group rgparticipante"x"   \
--name pip-fromimg”x” --allocation-method dynamic --idle-timeout 4

az network nic create \
-n nic-fromimg”x”  \
-g rgparticipante"x" \
--subnet front \
--public-ip-address pip-fromimg”x”  \
--vnet-name  uolhvnet"x"

az vm create \
  --resource-group rgparticipante"x" \
  --name vmfromimg”x” \
  --image <nomedaimagem> ou <enderecodaimagem> \
  --size Standard_DS1_v2 \
  --nics nic-fromimg”x”  \
  --admin-username participante"x" \
  --admin-password Go010101you! \
  --authentication-type password
  
 ```

8.2.5 deletando uma imagem

az image delete \
--name <nomedaimagem> \
--resource-group rgparticipante"x"

10. criar o servico backup de 1 VM em operacao (VM linux) via CLI e Portal

10.1 Criando o Vault de Backup via Portal
https://docs.microsoft.com/en-us/azure/virtual-machines/linux/tutorial-backup-vms 

(será criado na VM Linux de Front, demonstrado no Portal)

10.2 Criar um backup imediato (levará alguns minutos)

10.3 Fazer um teste de Restore de um arquivo

(Procedimento macro – VIA PORTAL)
1.	On your local computer, sign in to the Azure portal.
2.	In the menu on the left, select Virtual machines.
3.	From the list, select the VM.
4.	On the VM blade, in the Settings section, click Backup. The Backup blade opens.
5.	In the menu at the top of the blade, select File Recovery. The File Recovery blade opens.
6.	In Step 1: Select recovery point, select a recovery point from the drop-down.
7.	In Step 2: Download script to browse and recover files, click the Download Executable button. Save the downloaded file to your local computer.
8.	Click Download script to download the script file locally.

10.4 copie o arquivo para a VM a ter o arquivo recuperado
(poderá ser usado o filezilla)

10.5 execute o arquivo

chmod +x nomedoarquivo.sh
./<nome_arquivo>.sh


10.6 acesse o arquivo que necessita recuperar e copie o para alguma pasta

(navegue pelo caminho onde está seu arquivo e faça a cópia. Abaixo apenas um exemplo)

sudo cp ~/discomontado/Volume1/var/www/html/index.nginx-debian.html /<diretório_a_ser_copiado>


11. Monitoramento básico da VM via Portal

(Explorar as opções de Métrica e Alertas)



12. Segurança (Security Center)

(Explorar via Portal)

13. Outras opções dentro da VM
o	auto-shutdown
o	disaster recovery (ver alternativa com ummanaged disk)
o	redeploy
	
LABs EXTRA

14. criar uma VM Windows

az network public-ip create --resource-group rgparticipante"x"   \
--name pip-win”x” --allocation-method dynamic --idle-timeout 4

az network nic create \
-n nic-win”x” \
-g rgparticipante"x" \
--subnet front \
--public-ip-address part1ippubfront \
--vnet-name  uolhvnet"x"

az vm create \
--resource-group rgparticipante"x" \
--name vmWINpart”x” \
--os-disk-name diskwin”x” \
--image win2016datacenter \
--size Standard_DS1_v2 \
--nics part1nicnamefront
--data-disk-sizes-gb 128 \
--admin-username participante1 \
--admin-password Go010101you! \
--authentication-type password


15. criar uma Imagem baseada em Windows Server

16. criar um cluster kubernetes como exemplo

Referência:
https://docs.microsoft.com/pt-br/azure/aks/kubernetes-walkthrough 

17. adicionando Role customizado:

Logando via Powershell

Login-AzureRmAccount
Select-AzureRmSubscription –SubscriptionName 'Microsoft Azure Internal Consumption' 

$role = Get-AzureRmRoleDefinition "Contributor"
$role.Id = $null
$role.Name = "tudo menos deletar"
$role.Description = "pode fazer tudo menos deletar recurso"
$role.Actions.Clear()
$role.Actions.Add("*")
$role.NotActions.Add("*/delete")
$role.AssignableScopes.Clear()
$role.AssignableScopes.Add("/subscriptions/560c947d-7c94-4413-a569-e391f56128a0")
New-AzureRmRoleDefinition -Role $role

#Listar roles
Get-AzureRmRoleDefinition | FT Name, Description

18. criar uma arquitetura Altamente disponível com Availability Set e Load Balancer











Implementação de duas VMs CentOS (em um mesmo Availability Set) com servidor web Apache, usando load balancer externo com NAT para SSH (portas 50001, 50002).
https://github.com/matiasma/azureeverywhere
(material criado e cedido pelo Marcelo Matias)


19. Criar um VPN Gateway e configurar uma conexão P2S (Point to Site)

Referência:
https://docs.microsoft.com/en-us/azure/vpn-gateway/vpn-gateway-howto-point-to-site-resource-manager-portal 

20. USANDO JENKINS

Referência:
https://docs.microsoft.com/en-us/azure/virtual-machines/linux/tutorial-jenkins-github-docker-cicd 
		
PASSOS MACROS:
		
1.	Create a Jenkins VM
2.	Install and configure Jenkins
3.	Create webhook integration between GitHub and Jenkins
4.	Create and trigger Jenkins build jobs from GitHub commits
5.	Create a Docker image for your app
6.	Verify GitHub commits build new Docker image and updates running app
		
20.1 Create a Jenkins VM

20.1.1 Criar o arquivo cloud-init-jenkins.txt

OBSERVAÇÃO:
o arquivo cloud-init-jenkins.txt devera estar no local em que será rodado o comando “az vm create ... “

CONTEÚDO DO ARQUIVO:

#cloud-config
package_upgrade: true
write_files:
  - path: /etc/systemd/system/docker.service.d/docker.conf
    content: |
      [Service]
        ExecStart=
        ExecStart=/usr/bin/dockerd
  - path: /etc/docker/daemon.json
    content: |
      {
        "hosts": ["fd://","tcp://127.0.0.1:2375"]
      }
runcmd:
  - wget -q -O - https://jenkins-ci.org/debian/jenkins-ci.org.key | apt-key add -
  - sh -c 'echo deb http://pkg.jenkins-ci.org/debian-stable binary/ > /etc/apt/sources.list.d/jenkins.list'
  - apt-get update && apt-get install jenkins -y
  - curl -sSL https://get.docker.com/ | sh
  - usermod -aG docker azureuser
  - usermod -aG docker jenkins
  - service jenkins restart

		
20.1.2 criando a VM para o Jenkins


az network public-ip create --resource-group rgparticipante"x"   \
--name jenkinsip”x” --allocation-method dynamic --idle-timeout 4

az network nic create \
-n jenkinsnic”x” \
-g rgparticipante"x" \
--subnet manag \
--public-ip-address jenkinsip”x”  \
--vnet-name  uolhvnet"x"

az vm create \
--resource-group rgparticipante"x" \
--name part1jenkinsvm \
--image UbuntuLTS \
--size Standard_DS1_v2 \
--nics jenkinsnic”x”  \
--admin-username participante1 \
--admin-password Go010101you! \
--authentication-type password \
--custom-data cloud-init-jenkins.txt

  
20.1.3 liberando as portas para o Jenkins e Node.js
   
(8080 liberado na subnet manag via NSG)

az network nsg rule create \
--resource-group rgparticipante"x" \
--nsg-name NSG-manag \
--name jenkinsrule \
--access Allow \
--protocol Tcp \
--direction Inbound \
--priority 102 \
--source-address-prefix Internet \
--source-port-range "*" \
--destination-address-prefix "*" \
--destination-port-range 8080


(1337 liberado na subnet manag via NSG)

az network nsg rule create \
--resource-group rgparticipante"x" \
--nsg-name NSG-manag \
--name nodejssrule \
--access Allow \
--protocol Tcp \
--direction Inbound \
--priority 103 \
--source-address-prefix Internet \
--source-port-range "*" \
--destination-address-prefix "*" \
--destination-port-range 1337


20.1.4. Adquirindo a senha inicial do Jenkins

sudo cat /var/lib/jenkins/secrets/initialAdminPassword

Continua em:
https://docs.microsoft.com/en-us/azure/virtual-machines/linux/tutorial-jenkins-github-docker-cicd 





