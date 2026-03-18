FROM haproxy:2.8

COPY haproxy.cfg /usr/local/etc/haproxy/haproxy.cfg

EXPOSE 80

CMD ["haproxy", "-f", "/usr/local/etc/haproxy/haproxy.cfg"]
