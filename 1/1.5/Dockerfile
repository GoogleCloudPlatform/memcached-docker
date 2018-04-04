FROM gcr.io/google-appengine/debian9

# add our user and group first to make sure their IDs get assigned consistently, regardless of whatever dependencies get added
RUN groupadd -r memcache && useradd -r -g memcache memcache
ENV MEMCACHED_VERSION 1.5.7
ENV MEMCACHED_SHA1 31d6f6b80668025e4616aa2ad5c7a45f24ed9665

RUN set -x \
	&& buildDeps=' \
		ca-certificates \
		dpkg-dev \
		gcc \
		libc6-dev \
		libevent-dev \
		libsasl2-dev \
		make \
		perl \
		wget \
	' \
	&& apt-get update && apt-get install -y $buildDeps --no-install-recommends \
	&& rm -rf /var/lib/apt/lists/* \
	&& mkdir -p /usr/src/memcached_build \
	&& mkdir -p /usr/src/memcached \
	&& cd /usr/src/memcached \
	&& wget -O memcached.tar.gz "https://memcached.org/files/memcached-$MEMCACHED_VERSION.tar.gz" \
	&& echo "$MEMCACHED_SHA1  memcached.tar.gz" | sha1sum -c - \
	&& tar -xzf memcached.tar.gz -C /usr/src/memcached_build --strip-components=1 \
	&& cd /usr/src/memcached_build \
	&& ./configure \
		--build="$(dpkg-architecture --query DEB_BUILD_GNU_TYPE)" \
		--enable-sasl \
	&& make -j "$(nproc)" \
	&& make install \
	&& cd / && rm -rf /usr/src/memcached_build \
	&& apt-mark manual \
		libevent-2.0-5 \
		libsasl2-2 \
	&& apt-get purge -y --auto-remove $buildDeps \
	&& memcached -V

COPY docker-entrypoint.sh /usr/local/bin/
RUN ln -s usr/local/bin/docker-entrypoint.sh /entrypoint.sh # backwards compat
ENTRYPOINT ["docker-entrypoint.sh"]

USER memcache
EXPOSE 11211
CMD ["memcached"]
