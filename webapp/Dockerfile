# ──────────────────────────────────────────────────────────────
# Stage: Use a lightweight Nginx web server on Alpine Linux
# Alpine is a tiny Linux distro (~5MB), perfect for containers
# ──────────────────────────────────────────────────────────────
FROM nginx:alpine

# Copy our custom web page into the Nginx default serving folder
# Nginx will automatically serve index.html when someone visits the app
COPY index.html /usr/share/nginx/html/index.html

# Tell Docker this container listens on port 80 (standard HTTP)
# This is documentation — actual port mapping happens in ECS Task Definition
EXPOSE 80
