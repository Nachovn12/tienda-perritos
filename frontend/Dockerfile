# Etapa 1: preparación de archivos estáticos
FROM alpine:3.20 AS builder

WORKDIR /app

COPY index.html app.js ./

# Etapa 2: servidor Nginx seguro
FROM nginx:1.27-alpine AS runner

# Limpia contenido por defecto
RUN rm -rf /usr/share/nginx/html/*

# Copia archivos desde la etapa builder
COPY --from=builder /app/index.html /usr/share/nginx/html/index.html
COPY --from=builder /app/app.js /usr/share/nginx/html/app.js

# Copia configuración global de Nginx
COPY nginx.conf /etc/nginx/nginx.conf

# Crear usuario no root y ajustar permisos necesarios
RUN addgroup -S appgroup && adduser -S appuser -G appgroup \
  && chown -R appuser:appgroup /usr/share/nginx/html \
  && chown -R appuser:appgroup /var/cache/nginx \
  && chown -R appuser:appgroup /var/log/nginx \
  && touch /tmp/nginx.pid \
  && chown appuser:appgroup /tmp/nginx.pid

USER appuser

EXPOSE 8080

CMD ["nginx", "-g", "daemon off;"]
