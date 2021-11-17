FROM ghcr.io/linuxserver/baseimage-alpine-nginx:3.14

# environment variables
ENV HOME="/config"

# copy chia-dashboard-ui config
COPY config.ts /

# install chia-dashboard-core and chia-dashboard-ui
RUN \
 echo "**** install build packages ****" && \
 apk add --no-cache --virtual=build-dependencies \
  curl \
  g++ \
  make \
  yarn \
  npm && \
 echo "**** install runtime packages ****" && \
 apk add --no-cache \
	git \
	nodejs && \
 npm config set unsafe-perm true && \
 echo "**** install chia-dashboard-core ****" && \
 git -C /app clone https://github.com/felixbrucker/chia-dashboard-core.git && \
 echo "**** install chia-dashboard-ui ****" && \
 git -C /app clone https://github.com/felixbrucker/chia-dashboard-ui.git && \
 echo "**** install node modules ****" && \
 npm install --force --prefix /app/chia-dashboard-core && \
 npm install --force --prefix /app/chia-dashboard-ui && \
echo "**** build chia-dashboard-ui ****" && \
 mv -f /config.ts /app/chia-dashboard-ui/src/app/config.ts && \
 cd /app/chia-dashboard-ui && yarn run build && \
 echo "**** cleanup ****" && \
 apk del --purge \
	build-dependencies && \
 rm -rf \
	/root \
	/tmp/* && \
 mkdir -p \
	/root

# add local files
COPY root/ /

# ports
EXPOSE 80
