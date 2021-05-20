FROM ghcr.io/linuxserver/baseimage-alpine-nginx:3.13

COPY config.ts /

RUN \
 echo "**** install build packages ****" && \
 apk add --no-cache --virtual=build-dependencies \
	curl \
	g++ \
	make \
	yarn \
	nodejs-npm && \
 echo "**** install runtime packages ****" && \
 apk add --no-cache \
	git \
	nodejs && \
 npm config set unsafe-perm true && \
 echo "**** install chia-dashboard-ui ****" && \
 git -C /app clone https://github.com/felixbrucker/chia-dashboard-ui.git && \
 echo "**** install node modules ****" && \
 rm -f /app/chia-dashboard-ui/src/app/config.ts && \
 cp /config.ts /app/chia-dashboard-ui/src/app/config.ts && \
 npm install --prefix /app/chia-dashboard-ui && \
 echo "**** build chia-dashboard-ui with config.ts ****" && \
 cd /app/chia-dashboard-ui && yarn run build && \
 echo "**** cleanup ****" && \
 apk del --purge \
	build-dependencies && \
 rm -rf \
	/root \
	/tmp/* \
	/config.ts && \
 mkdir -p \
	/root

# add local files
COPY root/ /

# ports
EXPOSE 80
