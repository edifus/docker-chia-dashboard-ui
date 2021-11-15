FROM ghcr.io/linuxserver/baseimage-alpine-nginx:3.14

# environment variables
ENV HOME="/config"

# copy chia-dashboard-ui config
COPY config.ts /

# install chia-dashboard-core and chia-dashboard-ui
RUN \
 echo "**** install build packages ****" && \
 apk add --no-cache --virtual=build-dependencies \
  sed \
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
 echo "**** install chia-dashboard-core ****" && \
 git -C /app clone https://github.com/felixbrucker/chia-dashboard-core.git && \
 echo "**** install chia-dashboard-ui ****" && \
 git -C /app clone https://github.com/felixbrucker/chia-dashboard-ui.git && \
 echo "**** install node modules ****" && \
 npm install --prefix /app/chia-dashboard-core && \
 npm install --prefix /app/chia-dashboard-ui && \
 echo "**** update config.ts to use env.process ****" && \
 sed -i \
  -e '/^export const discordClientId/ s,^.*$,export const discordClientId = proccess.env.DISCORD_CLIENT_ID || null;,' \
  -e '/^export const githubClientId/ s,^.*$,export const githubClientId = proccess.env.GITHUB_CLIENT_ID || null;,' \
  -e '/^export const googleClientId/ s,^.*$,export const googleClientId = proccess.env.GOOGLE_CLIENT_ID || null;,' \
  -e '/^export const apiBaseUrl/ s,^.*$,export const apiBaseUrl = proccess.env.API_BASE_URL || 'http://localhost:5000/api';,' \
  -e '/^export const enablePeriodicUpdates/ s,^.*$,export const enablePeriodicUpdates = proccess.env.PERIODIC_UPDATES || true;,' \
  -e '/^export const requestDiscordGuildPermission/ s,^.*$,export const requestDiscordGuildPermission = proccess.env.DISCORD_PERM_REQUIRED || true;,' \
  /app/chia-dashboard-ui/src/app/config.ts && \
echo "**** build chia-dashboard-ui ****" && \
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
