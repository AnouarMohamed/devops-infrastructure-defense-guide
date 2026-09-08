# Architecture globale

## Les trois phases

```mermaid
flowchart TB
    subgraph P1[Phase 1 — découverte dynamique]
        T1[Internet : 80 et 443]
        T2[Traefik]
        T3[Services Swarm décrits par labels]
        T1 --> T2 --> T3
    end

    subgraph P2[Phase 2 — edge explicite]
        N1[Internet : 80 et 443]
        N2[Nginx + Certbot]
        N3[Catalogue central de routes]
        N1 --> N2 --> N3
    end

    subgraph P3[Phase 3 — laboratoire Kubernetes]
        K1[ingress-nginx + cert-manager]
        K2[Namespaces apps, data et ops]
        K3[Policies, backups et télémétrie]
        K1 --> K2 --> K3
    end

    P1 -. décision .-> P2
    P2 -. trajectoire .-> P3
```

Lecture :

- Traefik optimise la découverte de changements fréquents.
- Nginx optimise l'explicitation d'un catalogue stable.
- K3s prépare la standardisation et l'écosystème Kubernetes.

Il ne faut pas présenter ces phases comme « mauvais outil → bon outil → meilleur outil ». Chaque phase répond à une contrainte différente.

## Chemin d'une requête dans la cible Swarm

```mermaid
sequenceDiagram
    autonumber
    participant U as Navigateur
    participant D as DNS
    participant E as Nginx edge
    participant R as DNS Docker
    participant A as Service applicatif
    participant B as Base privée

    U->>D: Résoudre app.example.com
    D-->>U: Adresse du VPS
    U->>E: TLS avec SNI + requête HTTP Host
    E->>E: Sélectionner virtual host et politiques
    E->>R: Résoudre stack_service
    R-->>E: VIP ou adresse de tâche
    E->>A: Transmettre sur le réseau overlay
    A->>B: Requête sur le réseau interne
    B-->>A: Résultat
    A-->>E: Réponse applicative
    E-->>U: Réponse HTTPS
```

À expliquer :

1. Le DNS public mène au VPS.
2. Le SNI sert au choix du certificat ; le Host sert au routage HTTP.
3. Nginx ne contacte pas un port public de l'application.
4. Docker résout le nom interne du service.
5. La base reste derrière une deuxième frontière réseau.

## Frontières de confiance

```mermaid
flowchart TB
    U[Zone non fiable : Internet]

    subgraph EDGE[Frontière edge]
        E[80/443, TLS, Host allowlist, auth et limites]
    end

    subgraph APPS[Zone applications]
        A[WordPress, GLPI, Jenkins, Superset, Passbolt]
    end

    subgraph DATA[Zone données]
        B[MySQL, MariaDB, MongoDB, volumes]
    end

    subgraph OPS[Zone exploitation]
        O[Logs, métriques, alertes et sauvegardes]
    end

    U --> E
    E --> A
    A --> B
    A --> O
    B --> O
```

Formulation orale :

> « Je ne raisonne pas uniquement par conteneur. Je raisonne par frontière de confiance : exposition publique, edge, applications, données et exploitation. »

## Pourquoi le diagramme reste simple

Un diagramme d'architecture ne doit pas afficher toutes les connexions possibles. Il doit répondre à une question : ici, **dans quel ordre une requête traverse-t-elle les frontières ?** Les ports et exceptions détaillés restent dans une matrice séparée.

