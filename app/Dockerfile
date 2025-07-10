# Use imagem base alpine com versão específica (não latest)
FROM nginx:1.25-alpine

# Criar usuário não-root
RUN addgroup -g 1001 -S appuser && \
    adduser -S -D -H -u 1001 -h /var/cache/nginx -s /sbin/nologin -G appuser -g appuser appuser

# Configurar nginx para rodar na porta 8080 (não-privilegiada)
RUN sed -i 's/listen       80;/listen       8080;/' /etc/nginx/conf.d/default.conf && \
    sed -i 's/listen  \[::\]:80;/listen  [::]:8080;/' /etc/nginx/conf.d/default.conf

# Ajustar permissões para usuário não-root
RUN chown -R appuser:appuser /var/cache/nginx && \
    chown -R appuser:appuser /var/log/nginx && \
    chown -R appuser:appuser /etc/nginx/conf.d && \
    touch /var/run/nginx.pid && \
    chown -R appuser:appuser /var/run/nginx.pid

# Copiar arquivo HTML
COPY --chown=appuser:appuser index.html /usr/share/nginx/html/index.html

# Mudar para usuário não-root
USER appuser

# Expor porta não-privilegiada
EXPOSE 8080

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD wget --no-verbose --tries=1 --spider http://localhost:8080/ || exit 1
