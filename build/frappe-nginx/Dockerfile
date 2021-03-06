FROM bitnami/node:12-prod

WORKDIR /home/frappe/frappe-bench
RUN mkdir -p /home/frappe/frappe-bench/sites \
    && echo "frappe" > /home/frappe/frappe-bench/sites/apps.txt

RUN install_packages git

RUN mkdir -p apps sites/assets  \
    && cd apps \
    && git clone --depth 1 https://github.com/frappe/frappe

RUN cd /home/frappe/frappe-bench/apps/frappe \
    && yarn \
    && yarn run production \
    && rm -fr node_modules \
    && yarn install --production=true \
    && node --version \
    && npm --version \
    && yarn --version

RUN git clone --depth 1 https://github.com/frappe/bench /tmp/bench \
    && mkdir -p /var/www/error_pages \
    && cp -r /tmp/bench/bench/config/templates /var/www/error_pages

RUN cp -R /home/frappe/frappe-bench/apps/frappe/frappe/public/* /home/frappe/frappe-bench/sites/assets/frappe \
    && cp -R /home/frappe/frappe-bench/apps/frappe/node_modules /home/frappe/frappe-bench/sites/assets/frappe/

FROM nginx:latest
COPY --from=0 /home/frappe/frappe-bench/sites /var/www/html/
COPY --from=0 /var/www/error_pages /var/www/
COPY build/common/nginx-default.conf.template /etc/nginx/conf.d/default.conf.template
COPY build/frappe-nginx/docker-entrypoint.sh /

RUN apt-get update && apt-get install -y rsync && apt-get clean

VOLUME [ "/assets" ]

ENTRYPOINT ["/docker-entrypoint.sh"]
CMD ["nginx", "-g", "daemon off;"]
