---
class: ZSR
---
# Ansible
---
>**Ansible** to narzędzie wytworzone w 2012 roku i rozwijane współcześnie przez firmę Red Hat. Ansible używa języka **YAML** do opisu konfiguracji (nazywane Playbook-iem), działa w modelu bezagentowym (wszysto dzieje się z maszyny zarządzającej po SSH lub WinRM), do działania wymaga interpretera języka Python. **Ansible** jest udostępnione na licencji Open Source.

Ansible jest **deklaratywne**, a co za tym idzie **idempotentne**. Znaczy to jeśli stan jest taki jak zadany to nic się nie zmieni, a jeśli jest inny to zostanie podjęta akcja aby stał się taki jak zadany.

## Ansible vs Chef, Puppet
- **Ansible**: deklaratywny, *agentless*, YAML, stan zmienia się po zatwierdzeniu konfiguracji
- **Chef**: *agent-based*, Ruby
- **Puppet**: deklaratywny, *agent-based*, RubyDSL, stan jest aktualizowany do tego z konfiguracji regularnie co jakiś czas 

## Przykład konfiguracji
#TODO img z prez
- **Name** - nazwa playbooka
- **Hosts** - grupa hostów do zarządzania
- **Tasks** - lista zadań
	- **Name** - nazwa zadania
	- Nazwa modułu
		- Parametry modułu