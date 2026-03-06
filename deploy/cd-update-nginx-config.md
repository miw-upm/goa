# NO VERIFICADO!!!!!!!!!

name: CD. Deploy native Nginx config to AWS Lightsail
on:
push:
branches:
- nginx

jobs:
cd:
name: Deploy Nginx config (native)
runs-on: ubuntu-22.04

    steps:
      - name: ⬇️ Checkout repository
        uses: actions/checkout@v4

      - name: 📦 Upload new default config
        uses: appleboy/scp-action@v0.1.7
        with:
          host: ${{ secrets.AWS_LIGHTSAIL_IP }}
          username: ${{ secrets.AWS_LIGHTSAIL_USER }}
          key: ${{ secrets.AWS_LIGHTSAIL_SSH_KEY }}
          source: "docs/default"
          target: "/tmp/nginx/"

      - name: 🚀 Install config + test + reload
        uses: appleboy/ssh-action@v1.2.1
        with:
          host: ${{ secrets.AWS_LIGHTSAIL_IP }}
          username: ${{ secrets.AWS_LIGHTSAIL_USER }}
          key: ${{ secrets.AWS_LIGHTSAIL_SSH_KEY }}
          script: |
            set -euo pipefail

            TARGET="/etc/nginx/sites-available/default"
            SRC="/tmp/nginx/docs/default"

            if [ ! -f "$SRC" ]; then
              echo "No se encontró el fichero subido"
              exit 1
            fi

            TS=$(date +%Y%m%d-%H%M%S)

            # Backup del actual
            sudo cp -a "$TARGET" "${TARGET}.bak.${TS}"

            # Instalar nuevo config
            sudo install -m 644 "$SRC" "$TARGET"

            # Validar config
            if ! sudo nginx -t; then
              echo "Config inválida ❌. Restaurando backup..."
              sudo mv -f "${TARGET}.bak.${TS}" "$TARGET"
              sudo nginx -t
              exit 1
            fi

            # Recargar nginx (sin cortar conexiones activas)
            sudo systemctl reload nginx

            echo "Despliegue completado ✅"