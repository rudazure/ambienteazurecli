# ANTES DE COMECAR

1. criar RG para jumpbox
RG = uolhosting

2. criar 1 vm win2016 e 1 vm linux ubuntu para management
vm win = ruduolwin.eastus.cloudapp.azure.com (52.170.232.74)
vm linux = ruduollinux.eastus.cloudapp.azure.com (52.170.233.245)


ATIVIDADE JÁ COM ALUNO

1. acessar a vm windows e linux minha

	1.1. acessar a VM Linux via MOBA
	
	1.2. instalar o Azure CLI 2.0
	https://docs.microsoft.com/pt-br/cli/azure/install-azure-cli?view=azure-cli-latest
	
	para sistemas 64bits:
	1.
	echo "deb [arch=amd64] https://packages.microsoft.com/repos/azure-cli/ wheezy main" | \
     sudo tee /etc/apt/sources.list.d/azure-cli.list
	2.
	sudo apt-key adv --keyserver packages.microsoft.com --recv-keys 52E16F86FEE04B979B07E28DB02C46DF417A0893
	sudo apt-get install apt-transport-https
	sudo apt-get update && sudo apt-get install azure-cli
	3. 
	testar com o comando az
	
	1.3. conectar na subscricao via AZ CLI
			az login
			az account show (para verificar se está selecionado a subscription correta)
			az account list --output table (para listar todas subscriptions)
			az account set --subscription "My Demos" (para selecionar subscription com base no nome ou ID)
			
	
	OBSERVACAO: RODAR BASH NO WINDOWS:
	INSTALAR O BASH on ubuntu FOR WINDOWS.
	https://blogs.msdn.microsoft.com/commandline/2016/04/06/bash-on-ubuntu-on-windows-download-now-3/
	

2. criar via CLI o RG : rgparticipante1

	az group create --name rgparticipante1 --location eastus
	
	
3. criar via CLI a VNET, SUBNET (manag, front, back)

	3.1 criar a vnet com uma subnet default
	
	az network vnet create \
--name uolhvnet   \
--resource-group rgparticipante1 \
--location east us \
--address-prefix 192.168.0.0/16 \
--subnet-name manag \
--subnet-prefix 192.168.1.0/24

3.2 adicionar outras subnets via CLI

Front

az network vnet subnet create \
--address-prefix 192.168.2.0/24 \
--name Front \
--resource-group rgparticipante1 \
--vnet-name uolhvnet

Back

az network vnet subnet create \
--address-prefix 192.168.3.0/24 \
--name Back \
--resource-group rgparticipante1 \
--vnet-name uolhvnet


3.3 verificando sua nova vnet

az network vnet show \
-g rgparticipante1 \
-n uolhvnet \
--query '{Name:name,Where:location,Group:resourceGroup,Status:provisioningState,SubnetCount:subnets | length(@)}' \
-o table

3.4. verificando detalhes da subnet

az network vnet subnet list \
-g rgparticipante1 \
--vnet-name uolhvnet \
--query '[].{Name:name,CIDR:addressPrefix,Status:provisioningState}' \
-o table


4. criar / configurar via CLI a NSG

4.1. criar a NSG

para a subnet manag

az network nsg create \
--resource-group rgparticipante1 \
--name NSG-manag \
--location eastus

para a subnet front

az network nsg create \
--resource-group rgparticipante1 \
--name NSG-Front \
--location eastus

para a subnet back

az network nsg create \
--resource-group rgparticipante1 \
--name NSG-Back \
--location eastus

4.2 conferindo as regras default

az network nsg show \
-g rgparticipante1 \
-n nsg-front \
--query 'defaultSecurityRules[].{Access:access,Desc:description,DestPortRange:destinationPortRange,Direction:direction,Priority:priority}' \
-o table

4.3 adicionando uma nova regra "ALLOW ACCESS TO PORT 3389 (RDP) e PORT 22 (SSH)" para cada NSG

RDP para subnet Front

az network nsg rule create \
--resource-group rgparticipante1 \
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
--resource-group rgparticipante1 \
--nsg-name NSG-Front \
--name rdp-rule \
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
--resource-group rgparticipante1 \
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
--resource-group rgparticipante1 \
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
--resource-group rgparticipante1 \
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
--resource-group rgparticipante1 \
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
--vnet-name uolhvnet \
--name manag \
--resource-group rgparticipante1 \
--network-security-group NSG-manag

NSG DA SUBNET FRONT
az network vnet subnet update \
--vnet-name uolhvnet \
--name Front \
--resource-group rgparticipante1 \
--network-security-group NSG-Front

NSG DA SUBNET BACK
az network vnet subnet update \
--vnet-name uolhvnet \
--name Back \
--resource-group rgparticipante1 \
--network-security-group NSG-Back


5. criar via CLI as 2 vms para manag



6. criar via CLI a VM WIN (PLESK) e Linux (CPanel) na subnet Front

6.1 criar a vm Linux 


7. criar via CLI a VM Linux (Ubuntu ou CentOS) para subnet Back

7.1 criar uma aplicacao web e conectar no Banco MySQL (LAMP)

#listando as imagens disponiveis
az vm image list --offer CentOS --all --output table

#listando os tamanhos de VMs disponiveis
az vm list-sizes --location eastus --output table

ex.: Standard_A1

#criando a VM NA SUBNET FRONT

    az network public-ip create --resource-group rgparticipante1   \
     --name part1ippubfront --allocation-method dynamic --idle-timeout 4

    az network nic create \
        -n part1nicnamefront \
        -g rgparticipante1 \
        --subnet front \
        --public-ip-address part1ippubfront \
        --vnet-name  uolhvnet

az vm create \
  --resource-group rgparticipante1 \
  --name part1vmfront \
  --image UbuntuLTS \
  --size Standard_DS1_v2 \
  --nics part1nicnamefront
  --data-disk-sizes-gb 128 \
  --admin-username participante1 \
  --admin-password Go010101you! \
  --authentication-type password

  ADICIONANDO O LAMP:
sudo apt update && sudo apt install lamp-server^
faca o teste acessando o ip publico da VM
(a porta 80 para acesso via internet foi habilitada em step anterior durante criacao das regras NSG da subnet)


#criando a VM NA SUBNET BACK
RELEMBRANDO: 
ATÉ AQUI CRIAMOS O RG, A VNET, A SUBNET, O NSG. 
ENTÃO ANTES DE CRIAR A VM PRECISAMOS CRIAR A NIC E O IP PUBLICO A USAR.

    az network public-ip create --resource-group rgparticipante1   \
     --name part1ippubback --allocation-method dynamic --idle-timeout 4

    az network nic create \
        -n part1nicnameback \
        -g rgparticipante1 \
        --subnet Back \
        --public-ip-address part1ippubback \
        --vnet-name  uolhvnet

az vm create \
  --resource-group rgparticipante1 \
  --name part1vmback \
  --image UbuntuLTS \
  --size Standard_DS1_v2 \
  --nics part1nicname
  --data-disk-sizes-gb 128 \ 
  --admin-username participante1 \
  --admin-password Go010101you! \
  --authentication-type password
  #OBSERVACAO: O DISCO APENAS É ADICIONADO A VM, MAS NAO EH ENTREGUE FORMATADO DENTRO DO LINUX.

  ADICIONANDO O MYSQL
  
  (AINDA NAO INSTALEI NESSA VM)
  

7.2 criar uma Custom Image usando CLI

7.2.1 criando custom image no windows


7.2.2 criando custom image no linux

# criando primeiro uma vm para destruir

    az network public-ip create --resource-group rgparticipante1   \
     --name part1destroy --allocation-method dynamic --idle-timeout 4

    az network nic create \
        -n part1destroy \
        -g rgparticipante1 \
        --subnet front \
        --public-ip-address part1destroy \
        --vnet-name  uolhvnet

az vm create \
  --resource-group rgparticipante1 \
  --name part1destroy \
  --image UbuntuLTS \
  --size Standard_DS1_v2 \
  --nics part1destroy
  --admin-username participante1 \
  --admin-password Go010101you! \
  --authentication-type password
#OBSERVACAO: O DISCO APENAS É ADICIONADO A VM, MAS NAO EH ENTREGUE FORMATADO DENTRO DO LINUX.

#preparando a VM para salvar a imagem

#aplicar os comandos abaixo na VM que tera a imagem salva
sudo waagent -deprovision+user -force

#a partir daqui, usar os comandos na sua workstation de trabalho
az vm deallocate --resource-group rgparticipante1 --name part1destroy
az vm generalize --resource-group rgparticipante1 --name part1destroy 
az image create \
    --resource-group rgparticipante1 \
    --name imageteste \
    --source part1destroy
	
#listando as imagens disponiveis
az image list \
  --resource-group rgparticipante1
  
 #deletando uma imagem
 az image delete \
    --name imageteste \
    --resource-group rgparticipante1
	
#criando uma nova vm a partir da imagem criada

az vm create \
    --resource-group rgparticipante1 \
    --name myVMfromImage \
    --image imageteste \
    --admin-username rudneir \
    --generate-ssh-keys

8. criar GW para VPN via CLI

9. criar o servico backup das 2 VMs de operacao (win e linux) via CLI e Portal

9.1 Criando o Vault de Backup
criado na VM Linux de Front
https://docs.microsoft.com/en-us/azure/virtual-machines/linux/tutorial-backup-vms

9.2 Restaurando um arquivo

#baixe o arquivo
On your local computer, sign in to the Azure portal.
In the menu on the left, select Virtual machines.
From the list, select the VM.
On the VM blade, in the Settings section, click Backup. The Backup blade opens.
In the menu at the top of the blade, select File Recovery. The File Recovery blade opens.
In Step 1: Select recovery point, select a recovery point from the drop-down.
In Step 2: Download script to browse and recover files, click the Download Executable button. Save the downloaded file to your local computer.
Click Download script to download the script file locally.

#copie o arquivo para a VM a ter o arquivo recuperado
usei o filezilla

#execute o arquivo
chmod +x nomedoarquivo.sh
./<nome_arquivo>.sh
sudo cp ~/discomontado/Volume1/var/www/html/index.nginx-debian.html /<diretório_a_ser_copiado>

10. Monitoramento básico da VM via Portal

11. Segurança (Security Center)

12. Outras ferramentas
	- auto-shutdown
	- disaster recovery (ver alternativa com ummanaged disk)
	- alert rules
	- redeploy
	- acompanhar custos
	
	novas features:
	
	Azure AD Managed Service Identity
	https://azure.microsoft.com/en-us/blog/keep-credentials-out-of-code-introducing-azure-ad-managed-service-identity/
	
	Just-In-Time VM Access
	https://azure.microsoft.com/pt-br/blog/announcing-the-just-in-time-vm-access-public-preview/

	
	
13. USANDO JENKINS

https://docs.microsoft.com/en-us/azure/virtual-machines/linux/tutorial-jenkins-github-docker-cicd

		Create a Jenkins VM
		Install and configure Jenkins
		Create webhook integration between GitHub and Jenkins
		Create and trigger Jenkins build jobs from GitHub commits
		Create a Docker image for your app
		Verify GitHub commits build new Docker image and updates running app
		
		Create a Jenkins VM
		
	az network public-ip create --resource-group rgparticipante1   \
     --name part1jenkinsip --allocation-method dynamic --idle-timeout 4

    az network nic create \
        -n part1jenkinsnic \
        -g rgparticipante1 \
        --subnet manag \
        --public-ip-address part1jenkinsip \
        --vnet-name  uolhvnet

az vm create \
  --resource-group rgparticipante1 \
  --name part1jenkinsvm \
  --image UbuntuLTS \
  --size Standard_DS1_v2 \
  --nics part1jenkinsnic
  --admin-username participante1 \
  --admin-password Go010101you! \
  --authentication-type password \
  --custom-data cloud-init-jenkins.txt
  
  #OBSERVACAO:
   create the file in the Cloud Shell not on your local machine. 
   Enter sensible-editor cloud-init-jenkins.txt to create the file and see a list of available editors. 
   
   #liberando as portas
   
   8080 para subnet manag

az network nsg rule create \
--resource-group rgparticipante1 \
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

   1337 para subnet manag

az network nsg rule create \
--resource-group rgparticipante1 \
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
 
14. criar um cluster kubernetes como exemplo









