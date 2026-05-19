# sou-lab-cni

Laboratorio di configurazione automatica di un ambiente di monitoring con Ansible e Podman.

## Cosa fa questo progetto

Tramite un singolo comando Ansible, vengono avviati automaticamente 3 container:

- **HaProxy** — reverse proxy, smista il traffico verso Prometheus e Grafana
- **Prometheus** — raccoglie metriche di sistema
- **Grafana** — visualizza le metriche con grafici e dashboard

## Architettura

```
http://localhost:8080/prometheus  →  HaProxy  →  Prometheus :9090
http://localhost:8080/grafana     →  HaProxy  →  Grafana    :3000
```

HaProxy è l'unico punto di accesso dall'esterno. Prometheus e Grafana non sono esposti direttamente.

## Struttura del progetto

```
sou-lab-cni/
├── playbook.yml                        # Entry point Ansible
├── README.md
└── roles/
    └── sou_podman/
        ├── defaults/main.yml           # Variabili (porte, immagini, cartelle)
        ├── tasks/main.yml              # Logica: crea cartelle, copia config, avvia container
        ├── handlers/main.yml           # Riavvio automatico dei container se la config cambia
        └── templates/
            ├── haproxy.cfg.j2          # Configurazione HaProxy
            ├── prometheus.yml.j2       # Configurazione Prometheus
            └── grafana.ini.j2         # Configurazione Grafana
```

## Requisiti

- [Homebrew](https://brew.sh)
- Ansible
- Podman
- Collezione Ansible per Podman

## Installazione dipendenze

```bash
brew install ansible
brew install podman
ansible-galaxy collection install containers.podman
```

Inizializza e avvia la VM di Podman:

```bash
podman machine init
podman machine start
```

## Come eseguire

```bash
git clone <repo-name>
cd sou-lab-cni
ansible-playbook playbook.yml --ask-become-pass
```

## Verifica

Controlla che i container siano in esecuzione:

```bash
podman ps
```

Apri il browser:

- Prometheus → http://localhost:8080/prometheus
- Grafana → http://localhost:8080/grafana (credenziali default: `admin` / `admin`)

## Variabili configurabili

Le variabili si trovano in `roles/sou_podman/defaults/main.yml` e possono essere modificate liberamente.

| Variabile | Default | Descrizione |
|-----------|---------|-------------|
| `haproxy_port` | `8080` | Porta esposta da HaProxy |
| `prometheus_port` | `9090` | Porta interna Prometheus |
| `grafana_port` | `3000` | Porta interna Grafana |
| `haproxy_image` | `haproxy:latest` | Immagine container HaProxy |
| `prometheus_image` | `prom/prometheus:latest` | Immagine container Prometheus |
| `grafana_image` | `grafana/grafana:latest` | Immagine container Grafana |

## Note

- I dati di Prometheus e Grafana sono persistenti nella cartella `~/sou-lab/`
- Se modifichi un file di configurazione e rilanciamo il playbook, il container corrispondente si riavvia automaticamente tramite gli handlers Ansible
- La porta 80 non è utilizzabile senza root su Linux, per questo HaProxy usa la porta 8080
