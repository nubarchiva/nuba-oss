# Propuesta: sección Installation para el README de nuba-oss

> Este archivo contiene el texto propuesto para añadir al README.md de la rama `develop`.
> Se insertaría entre la sección "Quick Start" y "Get Involved".

## Texto propuesto

```markdown
## 💾 Installation

Step-by-step installation guides for running nubarchiva on your own server:

- [Ubuntu 24.04 LTS](docs/install/Ubuntu24.04.md) — recommended for new installations
- [Ubuntu 22.04 LTS](docs/install/Ubuntu22.04.md)

### Requirements

- Linux server (Ubuntu 22.04+ or Debian 12+)
- 4 CPU / 8 GB RAM (minimum)
- PostgreSQL, Java 8 (for Apache Solr), Java 11+ (for Apache Tomcat 9)
```

## Notas

- Se referencia Ubuntu 24.04 como "recommended" porque es el LTS actual
- No se incluye Debian 12 porque no existe guía publicada en `develop`
- Cuando se añadan más plataformas, ampliar la lista
