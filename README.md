#WORKSHOP UOL HOSTING

##DIAGRAMA

##1. acessar a vm inicial

1.1. acessar a VM Linux conforme info acima.
	
1.2. instalar o Azure CLI 2.0
https://docs.microsoft.com/pt-br/cli/azure/install-azure-cli?view=azure-cli-latest 
	
	para sistemas 64bits:
	
	1.2.1.
```bash
echo "deb [arch=amd64] https://packages.microsoft.com/repos/azure-cli/ wheezy main" | \sudo tee /etc/apt/sources.list.d/azure-cli.list
```
	 
	1.2.2.
```bash
sudo apt-key adv --keyserver packages.microsoft.com --recv-keys 52E16F86FEE04B979B07E28DB02C46DF417A0893
sudo apt-get install apt-transport-https
sudo apt-get update && sudo apt-get install azure-cli
```
	
	1.2.3. testar com o comando:
```bash
az
az –version
```
	
1.3. conectar na subscricao via AZ CLI
```bash
az login
az account show (para verificar se está selecionado a subscription correta)
az account list --output table (para listar todas subscriptions)
az account set --subscription "<nome_subscription> ou <ID>" (para selecionar subscription com base no nome ou ID)
```
		
	OBSERVAÇÃO: caso necessite rodar no Windows:
		INSTALAR O BASH on ubuntu FOR WINDOWS.
		https://blogs.msdn.microsoft.com/commandline/2016/04/06/bash-on-ubuntu-on-windows-download-now-3/ 
	
##2. criar via CLI o RG : rgparticipante"x"

```bash
az group create --name rgparticipante"x" --location eastus
```

##3. criar via CLI a VNET, SUBNET (manag, front, back)

3.1	criar a vnet com uma subnet inicial (chamada manag)
	
```bash
az network vnet create \
--name uolhvnet"x"   \
--resource-group rgparticipante"x" \
--location eastus \
--address-prefix 192.168.0.0/16 \
--subnet-name manag \
--subnet-prefix 192.168.1.0/24
```

3.2 adicionar outras subnets via CLI

Front

```bash
az network vnet subnet create \
--address-prefix 192.168.2.0/24 \
--name front \
--resource-group rgparticipante"x" \
--vnet-name uolhvnet"x"
```

Back

```bash
az network vnet subnet create \
--address-prefix 192.168.3.0/24 \
--name back \
--resource-group rgparticipante"x" \
--vnet-name uolhvnet"x"
```

3.3 verificando sua nova vnet

```bash
az network vnet show \
-g rgparticipante"x" \
-n uolhvnet"x" \
--query '{Name:name,Where:location,Group:resourceGroup,Status:provisioningState,SubnetCount:subnets | length(@)}' \
-o table
```

3.4. verificando detalhes da subnet

```bash
az network vnet subnet list \
-g rgparticipante"x" \
--vnet-name uolhvnet"x" \
--query '[].{Name:name,CIDR:addressPrefix,Status:provisioningState}' \
-o table
```

##4. criar / configurar via CLI a NSG (Network Security Group)

4.1. criar a NSG

para a subnet manag

```bash
az network nsg create \
--resource-group rgparticipante"x" \
--name NSG-manag \
--location eastus
```

para a subnet front

```bash
az network nsg create \
--resource-group rgparticipante"x" \
--name NSG-front \
--location eastus
```

para a subnet back

```bash
az network nsg create \
--resource-group rgparticipante"x" \
--name NSG-back \
--location eastus
```

4.2 conferindo as regras default

```bash
az network nsg show \
-g rgparticipante"x" \
-n nsg-front \
--query 'defaultSecurityRules[].{Access:access,Desc:description,DestPortRange:destinationPortRange,Direction:direction,Priority:priority}' \
-o table
```

4.3 adicionando as regras necessárias para cada NSG

RDP para subnet Front

```bash
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
```

SSH para subnet Front

```bash
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
```

SSH para subnet Backend

```bash
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
```

RDP para a subnet Manag

```bash
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
```

SSH para subnet manag

```bash
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
```

HTTP para a subnet Front

```bash
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
```

4.4 BIND THE NSG TO SUBNET

NSG DA SUBNET MANAG
```bash
az network vnet subnet update \
--vnet-name uolhvnet"x" \
--name manag \
--resource-group rgparticipante"x" \
--network-security-group NSG-manag
```

NSG DA SUBNET FRONT
```bash
az network vnet subnet update \
--vnet-name uolhvnet"x" \
--name Front \
--resource-group rgparticipante"x" \
--network-security-group NSG-Front
```

NSG DA SUBNET BACK
```bash
az network vnet subnet update \
--vnet-name uolhvnet"x" \
--name Back \
--resource-group rgparticipante"x" \
--network-security-group NSG-Back
```

##5. criar via CLI as 2 vms (Win para Plesk e Linux para Cpanel) na subnet manag

(Esse step dependerá do UOL, caso esteja já montando um ambiente de teste com Jumpbox)

##6. criar via CLI a VM Linux (para CPanel) na subnet Front

6.1 listando as imagens disponiveis
```bash
az vm image list --offer CentOS --all --output table
```

6.2 listando os tamanhos de VMs disponiveis

```bash
az vm list-sizes --location eastus --output table
```

ex.: Standard_A1

6.3 criando a VM NA SUBNET FRONT

RELEMBRANDO: 
ATÉ AQUI CRIAMOS O RG, A VNET, A SUBNET, O NSG. 
ENTÃO ANTES DE CRIAR A VM PRECISAMOS CRIAR A NIC E O IP PUBLICO da VM.

```bash

   	 az network public-ip create --resource-group rgparticipante"x"   \
    	 --name part1ippubfront --allocation-method dynamic --idle-timeout 4

   	 az network nic create \
       	 -n part1nicnamefront \
       	 -g rgparticipante"x" \
        	--subnet front \
        	--public-ip-address part1ippubfront \
        	--vnet-name  uolhvnet"x"

az vm create \
  --resource-group rgparticipante"x" \
  --name part1vmfront \
  --image UbuntuLTS \
  --size Standard_DS1_v2 \
  --nics part1nicnamefront
  --data-disk-sizes-gb 128 \
  --admin-username participante1 \
  --admin-password Go010101you! \
  --authentication-type password
```

  6.4 ADICIONANDO O STACK LAMP

```bash
sudo apt update && sudo apt install lamp-server^
```

Faça o teste acessando o ip publico da VM
(a porta 80 para acesso via internet foi habilitada em step anterior durante criação das regras NSG da subnet)

##7. criando uma VM Linux na Subnet Back para o MySQL

RELEMBRANDO: 
ATÉ AQUI CRIAMOS O RG, A VNET, A SUBNET, O NSG. 
ENTÃO ANTES DE CRIAR A VM PRECISAMOS CRIAR A NIC E O IP PUBLICO A USAR.

```bash
    az network public-ip create --resource-group rgparticipante"x"   \
     --name part1ippubback --allocation-method dynamic --idle-timeout 4

    az network nic create \
        -n part1nicnameback \
        -g rgparticipante"x" \
        --subnet Back \
        --public-ip-address part1ippubback \
        --vnet-name  uolhvnet"x"

az vm create \
  --resource-group rgparticipante"x" \
  --name part1vmback \
  --image UbuntuLTS \
  --size Standard_DS1_v2 \
  --nics part1nicname
  --data-disk-sizes-gb 128 \ 
  --admin-username participante1 \
  --admin-password Go010101you! \
  --authentication-type password
```

OBSERVAÇÃO: O DISCO APENAS É ADICIONADO A VM, MAS NAO É ENTREGUE FORMATADO DENTRO DO LINUX.

  	7.1 ADICIONANDO O MYSQL (via stack LAMP)
  
```bash
sudo apt update && sudo apt install lamp-server^
```
 
7.2 TESTAR A CONEXÃO ENTRE AS VMs das Subnets Front e Back (via ip privado)

##8. criar uma Custom Image usando CLI

8.1 criando custom image no windows

(executaremos esse step se tivermos tempo)

8.2 criando custom image no linux

8.2.1 criando uma nova VM para criarmos posteriormente sua imagem “generalizada”

(há diferença entre Imagem Generalizada e Backup da Imagem)
```bash
    	az network public-ip create --resource-group rgparticipante"x"   \
     	--name part1destroyip --allocation-method dynamic --idle-timeout 4

    	az network nic create \
        	-n part1destroynic \
        	-g rgparticipante"x" \
        	--subnet front \
        	--public-ip-address part1destroyip \
       	--vnet-name  uolhvnet"x"

az vm create \
  	--resource-group rgparticipante"x" \
  	--name part1destroy \
  	--image UbuntuLTS \
  	--size Standard_DS1_v2 \
  	--nics part1destroynic
  	--admin-username participante"x" \
  	--admin-password Go010101you! \
  	--authentication-type password
  ```

8.2.2 preparando a VM para salvar a imagem

(aplicar os comandos abaixo na VM que terá a imagem salva)
```bash
sudo waagent -deprovision+user –force
```

(a partir daqui, usar os comandos na sua workstation de trabalho)
```bash
az vm deallocate --resource-group rgparticipante"x" --name part1destroy
az vm generalize --resource-group rgparticipante"x" --name part1destroy 
az image create \
--resource-group rgparticipante"x" \
--name imageteste \
--source part1destroy
```

8.2.3 listando as imagens disponiveis
```bash
az image list \
  --resource-group rgparticipante"x"
```
  
 8.2.4 criar uma vm a partir de uma imagem salva
  
(criaremos primeiro o ip público e a NIC como nas outras VMs)
```bash
 az network public-ip create --resource-group rgparticipante"x"   \
     --name vmfromimageip --allocation-method dynamic --idle-timeout 4

    az network nic create \
        -n vmfromimagenic \
        -g rgparticipante"x" \
        --subnet front \
        --public-ip-address vmfromimageip \
        --vnet-name  uolhvnet"x"

az vm create \
  --resource-group rgparticipante"x" \
  --name vmfromimage \
  --image <nomedaimagem> ou <enderecodaimagem> \
  --size Standard_DS1_v2 \
  --nics vmfromimagenic
  --admin-username participante"x" \
  --admin-password Go010101you! \
  --authentication-type password
```
  
 8.2.5 deletando uma imagem
```bash
az image delete \
    --name <nomedaimagem> \
    --resource-group rgparticipante"x"
```

##9. criar GW para VPN (via Portal ou CLI)

(faremos esse passo se houver tempo)

##10. criar o servico backup de 1 VM em operacao (VM linux) via CLI e Portal

10.1 Criando o Vault de Backup
https://docs.microsoft.com/en-us/azure/virtual-machines/linux/tutorial-backup-vms 

(criado na VM Linux de Front, demonstrado no Portal)

10.2 Criar um backup imediato (levará alguns minutos)

10.3 Fazer um teste de Restore de um arquivo

(Procedimento macro)
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

```bash
chmod +x nomedoarquivo.sh
./<nome_arquivo>.sh
```

10.6 acesse o arquivo que necessita recuperar e copie o

(navegue pelo caminho onde está seu arquivo e faça a cópia. Abaixo apenas um exemplo)

```bash
sudo cp ~/discomontado/Volume1/var/www/html/index.nginx-debian.html /<diretório_a_ser_copiado>
```

##11. Monitoramento básico da VM via Portal

(Explorar as opções de Métrica e Alertas)

##12. Segurança (Security Center)

(Explorar via Portal)

##13. Outras opções dentro da VM
o	auto-shutdown
o	disaster recovery (ver alternativa com ummanaged disk)
o	redeploy
	
13.1 novas features (roadmap):
	
o	Azure AD Managed Service Identity
	https://azure.microsoft.com/en-us/blog/keep-credentials-out-of-code-introducing-azure-ad-managed-service-identity/ 
	
o	Just-In-Time VM Access
	https://azure.microsoft.com/pt-br/blog/announcing-the-just-in-time-vm-access-public-preview/ 
	
##14. USANDO JENKINS

Referência:
https://docs.microsoft.com/en-us/azure/virtual-machines/linux/tutorial-jenkins-github-docker-cicd 
	
PASSOS MACROS:
		
1.	Create a Jenkins VM
2.	Install and configure Jenkins
3.	Create webhook integration between GitHub and Jenkins
4.	Create and trigger Jenkins build jobs from GitHub commits
5.	Create a Docker image for your app
6.	Verify GitHub commits build new Docker image and updates running app
		
14.1 Create a Jenkins VM

14.1.1 Criar o arquivo cloud-init-jenkins.txt

OBSERVAÇÃO:
o arquivo cloud-init-jenkins.txt devera estar no local em que será rodado o comando “az vm create ... “

CONTEÚDO DO ARQUIVO:

```bash

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
```
		
14.1.2 criando a VM para o Jenkins

```bash
	az network public-ip create --resource-group rgparticipante"x"   \
     --name part1jenkinsip --allocation-method dynamic --idle-timeout 4

    	az network nic create \
       -n part1jenkinsnic \
       -g rgparticipante"x" \
       --subnet manag \
       --public-ip-address part1jenkinsip \
       --vnet-name  uolhvnet"x"

az vm create \
  	--resource-group rgparticipante"x" \
  	--name part1jenkinsvm \
  	--image UbuntuLTS \
  	--size Standard_DS1_v2 \
  	--nics part1jenkinsnic \
  	--admin-username participante1 \
  	--admin-password Go010101you! \
  	--authentication-type password \
  	--custom-data cloud-init-jenkins.txt
```
  
14.1.3 liberando as portas para o Jenkins e Node.js
   
(8080 liberado na subnet manag via NSG)

```bash

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
```

(1337 liberado na subnet manag via NSG)

```bash

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
```

14.1.4. Adquirindo a senha inicial do Jenkins
```bash
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```

Continua em:
https://docs.microsoft.com/en-us/azure/virtual-machines/linux/tutorial-jenkins-github-docker-cicd 
 
#LAB EXTRA

##15. criar um cluster kubernetes como exemplo

Referência:
https://docs.microsoft.com/pt-br/azure/aks/kubernetes-walkthrough 

```bash
az provider register -n Microsoft.ContainerService
az group create --name myResourceGroup --location eastus
az aks create --resource-group myResourceGroup --name myK8sCluster --node-count 1 --generate-ssh-keys
az aks install-cli
az aks get-credentials --resource-group myResourceGroup --name myK8sCluster
kubectl get nodes
kubectl create -f azure-vote.yml
```

##16. criar uma arquitetura Altamente disponível com Availability Set e Load Balancer

Implementação de duas VMs CentOS (em um mesmo Availability Set) com servidor web Apache, usando load balancer externo com NAT para SSH (portas 50001, 50002).
https://github.com/matiasma/azureeverywhere
(material criado e cedido pelo Marcelo Matias)





