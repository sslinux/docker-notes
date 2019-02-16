FROM alpine:3.4

COPY docker-entrypoint.sh /docker-entrypoint.sh
RUN ["chmod","+x","/docker-entrypoint.sh"]
ENTRYPOINT ["/docker-entrypoint.sh"]

EXPOSE 6379
CMD [ "redis-server" ]
