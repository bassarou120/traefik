✅ 2️⃣ Créer un Réseau Docker
Traefik doit gérer plusieurs services, donc créons un réseau dédié :

sh
Copy
Edit
docker network create traefik-net
✅ 3️⃣ Configurer docker-compose.yml pour Traefik
Crée un fichier docker-compose.yml pour Traefik :

yaml
Copy
Edit
version: '3.8'

services:
  traefik:
    image: traefik:v3.0
    container_name: traefik
    restart: always
    ports:
      - "80:80"    # HTTP
      - "443:443"  # HTTPS
      - "8080:8080" # Dashboard (désactiver en prod)
    networks:
      - traefik-net
    volumes:
      - "/var/run/docker.sock:/var/run/docker.sock:ro"
      - "./traefik.yml:/etc/traefik/traefik.yml"
      - "./acme.json:/acme.json"
    environment:
      - TZ=UTC
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.api.rule=Host(`traefik.yourdomain.com`)"
      - "traefik.http.routers.api.service=api@internal"
      - "traefik.http.routers.api.entrypoints=websecure"

networks:
  traefik-net:
    external: true
✅ 4️⃣ Configurer traefik.yml
Crée un fichier traefik.yml :

yaml
Copy
Edit
global:
  checkNewVersion: true
  sendAnonymousUsage: false

entryPoints:
  web:
    address: ":80"
  websecure:
    address: ":443"

api:
  dashboard: true
  insecure: true  # ⚠️ Désactiver en prod

providers:
  docker:
    exposedByDefault: false

certificatesResolvers:
  myresolver:
    acme:
      email: your-email@example.com
      storage: acme.json
      httpChallenge:
        entryPoint: web
Remplace your-email@example.com par ton adresse email.

✅ 5️⃣ Initialiser le fichier acme.json pour SSL
Ce fichier stocke les certificats Let’s Encrypt :

sh
Copy
Edit
touch acme.json
chmod 600 acme.json
✅ 6️⃣ Lancer Traefik
Démarre Traefik avec :

sh
Copy
Edit
docker compose up -d
✅ 7️⃣ Ajouter un Service avec Traefik (Exemple: Whoami)
Ajoute un service derrière Traefik dans un autre docker-compose.yml :

yaml
Copy
Edit
version: '3.8'

services:
  whoami:
    image: traefik/whoami
    container_name: whoami
    networks:
      - traefik-net
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.whoami.rule=Host(`whoami.yourdomain.com`)"
      - "traefik.http.routers.whoami.entrypoints=websecure"
      - "traefik.http.routers.whoami.tls.certresolver=myresolver"

networks:
  traefik-net:
    external: true
Puis lance le service :

sh
Copy
Edit
docker compose up -d
✅ 8️⃣ Tester l'installation
Ouvre http://traefik.yourdomain.com:8080 pour voir le dashboard de Traefik.

Remplace yourdomain.com par ton vrai domaine et configure tes DNS pour pointer vers ton serveur.

