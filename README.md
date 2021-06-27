# Challenge
Realizzare un playbook Ansible che permetta di svolgere le seguenti attivita':

1. Provisioning di VMs CentOS. Le VM possono essere locali o su un Cloud provider a scelta.
2. Configurare le VM:
	a. Assicurarsi che la partizione utilizzata da Docker abbia almeno 10GB di spazio disponibile
3. Setup di Docker sulle VM
4. Configurare Docker:
	a. Esporre le API REST del Docker Daemon in modo sicuro
	b. Assicurarsi che il Docker Daemon sia configurato come un servizio che parta automaticamente all'avvio del sistema
5. Configurare un Docker Swarm sulle VM, che sia accessibile in modo sicuro.
Assicurarsi di riuscire ad interagire e deployare servizi sullo Swarm dalla macchina locale.
Opzionalmente:
6. Testare un task a scelta dei precedenti utilizzando Molecule
Rappresentare ogni attivita' con opportuni ruoli Ansible e relativi task. E' altamente
consigliato il riuso di ruoli e playbook Ansible resi disponibili nella community open-
source (alcuni dei quali gia' linkati nel testo a mo' di esempio e in basso tra i
riferimenti utili). In caso di riutilizzo di codice e' importante motivare I criteri di
selezione del ruolo, conoscerne le feature e I contenuti, e saper descrivere eventuali
personalizzazioni effettuate ai fini di svolgere le attivita' di cui sopra.
Versioning del Codice:
1. Versionare il codice su un repository pubblico su Github.com
Continuous Integration:
1. Configurare una pipeline di Continuous Integration su un tool a scelta (Travis)
2. La pipeline deve:
	a. Eseguire il linting del codice e fallire in caso di errori, che vanno opportunamente corretti

## Descrizione
Questo playbook fornisce 'n' VM Centos, le VM sono locali.
Le VM sono partizionate per garantire che Docker disponga di almeno 40 GB di spazio disponibile.
Docker  espone in modo sicuro le API REST di Docker Daemon.
Docker Daemon è configurato come un servizio che si avvia automaticamente all'avvio del sistema.
Inizializza Docker Swarm sul nodo manager .
Interazione e distribuzione di servizi su Swarm dalla macchina locale. 
![](challenge.png)

## Pre-Requisiti Lab Locale
......

## Provisioning
Vagrant supporta i sistemi di provisioning con Ansible.
Questa sfida utilizza il provider locale di Ansible.
Vagrant gestirà il download e l'installazione di Ansible sul guest virtuale,
e quindi eseguire un playbook specificato localmente su quel guest virtuale. 

Inventory File
------------

If you want to use this role ......


Role Variables
--------------

## Configurazione Iniziale



## Docker



## Swarm

