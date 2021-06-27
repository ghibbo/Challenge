# Challenge
Realizzare un playbook Ansible che permetta di svolgere le seguenti attività:

1. Provisioning di VMs CentOS. Le VM possono essere locali o su un Cloud provider a scelta.
2. Configurare le VM:
	* Assicurarsi che la partizione utilizzata da Docker abbia almeno 10GB di spazio disponibile
3. Setup di Docker sulle VM
4. Configurare Docker:
	* Esporre le API REST del Docker Daemon in modo sicuro
	* Assicurarsi che il Docker Daemon sia configurato come un servizio che parta automaticamente all'avvio del sistema
5. Configurare un Docker Swarm sulle VM, che sia accessibile in modo sicuro.
	* Assicurarsi di riuscire ad interagire e deployare servizi sullo Swarm dalla macchina locale.
6. Opzionalmente:
	* Testare un task a scelta dei precedenti utilizzando Molecule

Rappresentare ogni attività con opportuni ruoli Ansible e relativi task. 

Versioning del Codice:

1. Versionare il codice su un repository pubblico su Github.com

Continuous Integration:
1. Configurare una pipeline di Continuous Integration su un tool a scelta (Travis)
2. La pipeline deve:
  * Eseguire il linting del codice e fallire in caso di errori, che vanno opportunamente corretti



## Descrizione

Questo playbook fornisce ***n*** VM Centos, le VM sono locali.
Le VM sono partizionate per garantire che Docker disponga di almeno 10 GB di spazio disponibile.
Docker espone in modo sicuro le API REST di Docker Daemon.
Docker Daemon è configurato come un servizio che si avvia automaticamente all'avvio del sistema.
Inizializza e configura Docker Swarm sul nodo manager e aggiunge i nodi worker allo Swarm.
Testa I' interazione e distribuzione di servizi sui nodi del cluster Swarm dalla macchina locale. 



![](challenge.png)



## Pre-Requisiti Lab Locale

Con l'aiuto di due piccoli strumenti, possiamo rendere il test locale degli script di provisioning più veloce e più piacevole. 

Utilizzeremo Vagrant per avviare una VM locale, fornire quella VM con Ansible.

Vagrant è uno degli strumenti preferiti dagli sviluppatori. Vagrant è uno strumento progettato per consentire agli utenti di creare e configurare ambienti di sviluppo leggeri, riproducibili e portatili con gli ambienti di virtualizzazione. 

E' possibile utilizzare Ansible con Vagrant per automatizzare il provisioning della macchine secondo requisiti e settaggi voluti. 

> vm.provision "ansible", playbook: "centos.yml"

Per verificare che l'ambiente di sviluppo sia correttamente configurato dobbiamo utilizzare una macchina Linux o MacOs.

Pacchetti necessari:

* Vagrant
* Ansible
* Ambiente di virtualizzazione (**VirtualBox**, KVM, VMware etc..)

Nella cartella 'startup_test' sono presenti 2 file:

* Vagrantfile
* cestos.yml

Il file **Vagrantfile** crea 2 macchine CentOS 8 con indirizzi di rete privati. 

Il file **centos.yml** contiene una configurazione minimale di una macchina virtuale.

Verifica iniziale dell'ambiente di Test:

```v
vagrant up
```



## Provisioning
Vagrantfile del progetto:

```v
Vagrant.configure("2") do |config|
  
  number_of_machines = 2
  
  box_name = "centos/8"

  base_ip = 100
  base_ip_addresses = "192.168.10"

  ip_addresses = (1..number_of_machines).map{ |i| "#{base_ip_addresses}.#{base_ip + i}" }

  config.vm.provider "virtualbox" do |v|
    v.memory = 2048
  end

  (1..number_of_machines).each do |i|
    config.vm.define "centos_vm#{i}" do |box|
      box.vm.box = box_name
      box.vm.network "private_network", ip: "#{ip_addresses[i-1]}"
      box.vm.hostname = "centos-vm#{i}"
      box.vm.provision "ansible", playbook: "main.yml"
    end
  end

end

```



Inventory File
------------

Il file inventario Ansible descrive i dettagli degli host nel cluster, nonché la configurazione. Una volta che l'inventario è stato definito, utilizziamo il modello per selezionare gli host o i gruppi per la configurazione di docker e swarm.

>  Esempio:

centos-vm1 ansible_ssh_host=192.168.10.101

centos-vm2 ansible_ssh_host=192.168.10.102

centos-vm3 ansible_ssh_host=192.168.10.103



[docker-nodes]

centos-vm1

centos-vm2

centos-vm3



[swarm-managers]

centos-vm1



[swarm-workers]

centos-vm2

centos-vm3





[docker:vars]

ansible_python_interpreter=/usr/bin/python3Role Variables



## Ruoli

Un ruolo contiene le attività del playbook Ansible, oltre a tutti i file, le variabili, i modelli e i gestori di supporto necessari per eseguire le attività. Un ruolo è un'unità completa di automazione che può essere riutilizzata e condivisa. 

Utilizzando lo strumento da riga di comando ansible-galaxy, si puoi creare un ruolo con il comando init. 

```Ansible
ansible-galaxy init role
```

I ruoli principali sono:

1. Inizializzazione
2. Docker 
3. Swarm



## Configurazione Iniziale

Questo ruolo si occupa di creare i settaggi e le configurazioni base di ogni singola VM.

Settaggi e Configurazioni:

* Installa pacchetti base:
  * bash-completion
  * vim
  * nano
  * tmux
* Crea utente e password:
  * user: vmuser
  * password: Password1
* Aggiunge utente nel gruppo sudoers
* Accesso SSH

 

## Docker

Questo ruolo si occupa di installare, configurare, securizzare e testare docker su ogni VM.

Struttura:

* Check vecchie versioni non installate
* Creazione di un docker repository
* Installazione:
  * Docker
  * Pip
  * Docker SDK
* Aggiunta dell'utente 'vmuser' al gruppo docker
* Update del servizio docker
  * Docker Engine API
* Securizzazione del servizio
* Test 

Docker fornisce un'API per l'interazione con il demone Docker (chiamato API Docker Engine), nonché SDK per Go e Python. Gli SDK consentono di creare e ridimensionare app e soluzioni Docker in modo rapido e semplice. Nel progetto si utilizza direttamente l'API di Docker Engine. 

Per utilizzare l'API Docker Engine, è necessario abilitare un socket TCP all'avvio del daemon. Per impostazione predefinita, un socket di unix viene creato in /var/run/docker.sock. Tuttavia, possiamo configurare il demone utilizzando l'opzione -H (Systemd socket).

Con l'opzione -H tcp://0.0.0.0:7777, il demone aprirà un socket TCP in ascolto sulla porta TCP 7777 per tutte le interfacce di sistema. 

> ​	Example:
>
> Possiamo recuperare l'elenco dei contenitori utilizzando qualsiasi client http: 
>
> ```
> curl -X GET http://192.168.10.101:7777/containers/json?all=1
> ```

L'API è pubblica e disponibile per chiunque. Questa non è una buona idea, soprattutto se vogliamo esporre questa API su Internet. Fortunatamente, possiamo configurare una connessione TLS protetta utilizzando certificati autofirmati.

> ​	generate-ssl-certs.sh
>
> ```bash
> mkdir server-cert
> openssl genrsa -out ./server-cert/docker-daemon-server.key 1024
> openssl req -new -key ./server-cert/docker-daemon-server.key -x509 -days 180 -out ./server-cert/docker-daemon-server.crt
> cat ./server-cert/docker-daemon-server.key ./server-cert/docker-daemon-server.crt >./server-cert/docker-daemon-server.pem
> chmod 600 ./server-cert/docker-daemon-server.key ./server-cert/docker-daemon-server.pem
> 
> mkdir client-certopenssl 
> genrsa -out ./client-cert/docker-daemon-client.key 1024
> openssl req -new -key ./client-cert/docker-daemon-client.key -x509 -days 180 -out ./client-cert/docker-daemon-client.crt
> cat ./client-cert/docker-daemon-client.key ./client-cert/docker-daemon-client.crt >./client-cert/docker-daemon-client.pem
> chmod 600 ./client-cert/docker-daemon-client.key ./client-cert/docker-daemon-client.pem
> 
> ```
>
> 

A questo punto dobbiamo abilitare il daemon ad utilizzare questi certificati TLS.

> ​	-H tcp://0.0.0.0:7777 --tlsverify --tlscacert=/......



## Swarm

Questo ruolo si occupa di Impostare e configurare un Docker Swarm:

* Determina lo stato di Swarm di ciascun nodo
* Ogni nodo manager e classificalo come funzionale o non inizializzato
* Esegue lo stesso controllo sui nodi worker e li classifica come collegati o scollegati al nodo manager 
* Inizializza il nodo manager:   docker swarm init
* Recupera i token manager e worker di unione dal nodo manager attivo
* Unisce i nodi worker al cluster tramite token



## Test







