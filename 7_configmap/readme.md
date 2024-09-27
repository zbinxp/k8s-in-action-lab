# configmap volume lab
steps:
- create configmap
- create pod using `nginx-config.yaml`
- expose port using `kubectl port-forward fortune-configmap-volume 8080:80 &`
- check if gzip encoding enabled, `curl -H "Accept-Encoding: gzip" -I localhost:8080`
- check foler content: `k exec fortune-configmap-volume -c web-server -- ls /etc/nginx/conf.d`